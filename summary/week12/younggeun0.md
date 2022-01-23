# 노드 JS 스터디 9장 익스프레스로 SNS 서비스 만들기

> 9장은 배운 것을 바탕으로 웹 서비스를 제작하는 튜토리얼

## 9장 핵심 정리

* 서버는 요청에 응답하는 것이 핵심 임무이므로 요청을 수락하든 거절하든 상관없이 반드시 응답해야 함. 이 때 한 번만 응답해야 에러가 발생하지 않음
* 개발 시 서버를 매번 수동으로 재시작하지 않으려면 `nodemon`을 사용하는 것이 좋음
* `dotenv`패키지와 .env 파일로 유출되면 안 되는 비밀 키를 관리해야 함
* 라우터는 `routes 폴더`에, 데이터베이스는 `models 폴더`에, html 파일은 `views 폴더`에 구분하여 저장하면 프로젝트 규모가 커져도 관리하기 쉬움
* 데이터베이스를 구성하기 전에 데이터 간 1:1, 1:N, N:M 관계를 잘 파악해야 함
* routes/middlewares.js처럼 라우터 내에 미들웨어를 사용할 수 있다는 것을 기억할 것
* Passport 인증 과정과 `serializeUser`, `deserializeUser`가 언제 호출되는지 파악하고 기억해둘 것
* 프런트엔드 form 태그의 인코딩 방식이 multipart일 때는 multer 같은 multipart 처리용 패키지를 사용하는 것이 좋음


## 9.1 프로젝트 구조 갖추기

* 첨부된 프로젝트 파일 내용을 확인

1. `npm init`으로 package.json 설정
2. 필요한 모듈 설치

```bash
npm i express cookie-parser express-session morgan mlulter dotenv nunjucks sequelize mysql2 sequelize-cli
```

3. config, migrations, models, seeders 디렉토리 생성

```bash
npx sequelize init
```

4. app.js 에 기본 설정, router 설정
5. view 만들기(nunjucks 사용)

## 9.2 DB 세팅하기

* MySQL과 시퀄라이즈로 DB 구축, 필요한 테이블 정의
  * 로그인 기능 - 사용자 테이블
  * 게시글 - 게시글 테이블
  * 해시테그 - 해시태그 테이블

1. models 폴더 안에 DB와 연결할 user.js, post.js, hashtag.js 생성
2. user.js에 유저테이블과 연결할 관계 정의

    * provider는 로그인 방식(local/kakao)
    * timestamps, paranoid를 true로 설정해 createdAt, updatedAt, deletedAt 컬럼도 자동생성되도록 함
3. post.js엔 게시글테이블과 연결할 관계 정의, 게시글 등록자의 아이디를 담은 컬럼은 관계 설정 시 시퀄라이즈가 알아서 생성해줌
4. hashtag.js에 해시태그테이블과 연결할 관계 정의, 해시태그 모델을 따로 두는 것은 태그를 검색하기 위함
5. models/index.js 에 테이블과 연결된 각각의 모델들을 시퀄라이즈 객체에 연결
6. 각 모델 간 관계를 associate 함수 안에 정의

    * 유저와 게시글 관계 User 1 : Post N
    * 팔로잉-팔로워 유저 관계 User N : User M
      * 동일 테이블 간 N:M 관계는 as 옵션을 사용해야 모델 구분 가능
      * 단, as는 foreignKey와 반대되는 모델을 가리킨다는 점 주의)
    * 게시글과 해시태그 관계 Post N : HashTag M
      * PostHashtag란 중간 모델이 생김

```js
// 관계 설정 후 아래와 같이 모델을 참조 가능
db.sequelize.models.PostHashtag
db.sequelize.models.Follow
```

7. config.json 후 커맨드 입력해서 DB 생성

```bash
npx sequelize db:create
```

8. 서버와 모델을 연결

```js
// app.js
...
const { sequelize } = require('./models');
...
sequelize.sync({ force: false })
  .then(() => {
    console.log('데이터베이스 연결 성공');
  })
  .catch((err) => {
    console.error(err);
  });
```

## 9.3 Passport 모듈로 로그인 구현하기

* 세션과 쿠키 처리 등 복잡한 작업이 많으므로 검증된 모듈을 사용하는 것을 권장(`Passport`)

1. Passport 관련 패키지 설치

```bash
npm i passport passport-local passport-kakao bcrypt
```

2. Passport 모듈을 미리 app.js와 연결

```js
// app.js
...
const passport = require('passport'); // index.js는 require 시 이름 생략 가능
const passportConfig = require('./passport');
...
passportConfig(); // 패스포트 설정
...
app.use(session({
  resave: false,
  saveUninitialized: false,
  secret: process.env.COOKIE_SECRET,
  cookie: {
    httpOnly: true,
    secure: false,
  },
}));
app.use(passport.initialize());
app.use(passport.session());

app.use('/', pageRouter);
,,,
```

3. passport 폴더 내부에 index.js를 만들고 관련 코드 작성
    * `passport.initialize` 미들웨어는 요청(req) 객체에 passport 설정을 심고 `passport.session` 미들웨어는 req.session 객체에 passport 정보를 저장
    * req.session 객체는 express-session에서 생성하므로 passport 미들웨어는 express-session 미들웨어보다 뒤에 연결해야 함

4. passport 폴더 내 index.js 파일 만들고 Passport 관련 코드 작성
    * `serializeUser`는 로그인 시에만 실행됨
      * req.session(세션) 객체에 어떤 데이터를 저장할지 정하는 메서드
      * done 함수 첫 번째 인자는 에러 발생 시 사용, 두 번째 인수는 저장하고 싶은 데이터를 넣음
    * `deserializeUser`는 매 요청 시 실행
      * passport.session 미들웨어가 이 메서드를 호출, serializeUser의 done의 두 번째 인수로 넣었던 데이터가 deserializeUser의 매개변수가 됨(여기선 사용자 id)
      * id로 DB에서 사용자 정보를 조회함
    * **`serializeUser`는 사용자 객체를 세션에 아이디로 저장, `deserializeUser`는 세션에 저장한 아이디를 통해 사용자 정보 객체를 불러오는 것**
      * 세션에 불필요한 데이터를 담아두지 않기 위한 과정
    
```bash
# 전체 로그인 처리 과정
# 로그인
1. 라우터를 통해 로그인 요청이 들어옴
2. 라우터에서 passport.authenticate 메서드 호출
3. 로그인 전략 수행 ('local' or 'kakao')
4. 로그인 성공 시 사용자 정보 객체와 함께 req.login 호출
5. req.login 메서드가 passport.serializeUser 호출
6. req.session에 사용자 아이디만 저장
7. 로그인 완료

# 로그인 이후
1. 요청이 들어옴
2. 라우터에 요청이 도달하기 전에 passport.session 미들웨어가 passport.deserializeUser 메서드 호출
3. req.session에 저장된 아이디로 데이터베이스에서 사용자 조회
4. 조회된 사용자 정보를 req.user에 저장
5. 라우터에서 req.user 객체 사용 가능
```

> Passport 는 로그인 시 동작을 전략(Strategy)란 용어로 표현함

* 로컬 로그인이란 자체적으로 회원가입 후 로그인 하는 것을 의미
  * `passport-local` 모듈이 필요함
  * 로컬 로그인 전략을 세워야 함, 로그인에만 해당하는 전략이므로 회원가입은 따로 만들어야 함

5. 로그인 라우터 접근 권한을 제어하는 미들웨어 추가(middlewares.js)
    * 이미 로그인한 경우 회원가입, 로그인 라우터에 접근을 막아야 함
    * 로그인하지 않은 경우, 로그아웃 라우터에 접근을 막아야 함
    * Passport는 req 객체에 isAuthenticated 메서드를 추가함 (로그인 여부 판단 가능)

```js
// routes/middlewares.js
exports.isLoggedIn = (req, res, next) => {
  if (req.isAuthenticated()) {
    next();
  } else {
    res.status(403).send('로그인 필요');
  }
};

exports.isNotLoggedIn = (req, res, next) => {
  if (!req.isAuthenticated()) {
    next();
  } else {
    const message = encodeURIComponent('로그인한 상태입니다.');
    res.redirect(`/?error=${message}`);
  }
};
```

```js
// routes/page.js
const { isLoggedIn, isNotLoggedIn } = require('./middlewares');
...
router.get('/profile', isLoggedIn, (req, res) => {
  res.render('profile', { title: '내 정보 - NodeBird' });
});

router.get('/join', isNotLoggedIn, (req, res) => {
  res.render('join', { title: '회원가입 - NodeBird' });
});
...
```

6. 회원가입, 로그인, 로그아웃 라우터 작성([routes/auto.js](https://github.com/ohbokdong/NodeJSStudy/blob/main/summary/week12/nodebird/routes/auth.js))
7. 로컬 로그인 전략 작성([passport/localStrategy.js](https://github.com/ohbokdong/NodeJSStudy/blob/main/summary/week12/nodebird/passport/localStrategy.js))

```js
// routes/auth.js
...
router.post('/login', isNotLoggedIn, (req, res, next) => {
  // 로컬 로그인 전략 설정
  passport.authenticate('local', (authError, user, info) => {
    if (authError) {
      console.error(authError);
      return next(authError);
    }
    if (!user) {
      return res.redirect(`/?loginError=${info.message}`);
    }
    return req.login(user, (loginError) => {
      if (loginError) {
        console.error(loginError);
        return next(loginError);
      }
      return res.redirect('/');
    });
  })(req, res, next); // 미들웨어 내의 미들웨어에는 (req, res, next)를 붙입니다.
});
...
```

8. 카카오 로그인 전략 작성([passport/kakaoStrategy.js](https://github.com/ohbokdong/NodeJSStudy/blob/main/summary/week12/nodebird/passport/kakaoStrategy.js))
    * 로컬 전략과 다른 점은 passport.authenticate 메서드에 콜백 함수를 제공하지 않는다는 점
    * REST API 키를 [kakao developers](https://developers.kakao.com/)에서 앱 등록 후 발급받아 .env파일에 추가
    * 앱 설정/플랫폼/Web 사이트 도메인 추가
    * 제품 설정/카카오 로그인에서 카카오 로그인 활성화 후 RedirectURI 수정
    * 이메일 등록을 위해 제품 설정/카카오 로그인/동의 항목에서 카카오 계정으로 정보 수집 후 제공 체크

```js
// routes/auth.js
...
// 카카오 로그인 전략 설정
router.get('/kakao', passport.authenticate('kakao'));

router.get('/kakao/callback', passport.authenticate('kakao', {
  failureRedirect: '/',
}), (req, res) => {
  res.redirect('/');
});
...
```

9. auth 라우터를 app.js에 연결

```js
// app.js
...
const authRouter = require('./routes/auth');
...

app.use('/', pageRouter);
app.use('/auth', authRouter);
...
```

### 9.4 multer 패키지로 이미지 업로드 구현하기

* 이미지를 어떻게 저장할 것인지는 서비슷 특성에 따라 달라짐
  * 예제에선 input 태그를 통해 이미지를 선택할 때 바로 업로드를 진행하고, 업로드된 사진 주소를 클라이언트에 알림
  * 게시글을 저장할 때는 데이터베이스에 직접 이미지 데이터를 넣는 대신 이미지 경로만 저장함

1. post 라우터 설정([routes/post.js](https://github.com/ohbokdong/NodeJSStudy/blob/main/summary/week12/nodebird/routes/post.js))
    * 예제에선 서버 디스크에 이미지를 저장하지만, 실제 서버 운영 시 서버에 문제 발생 시 이미지가 제공되지 않거나 손실될 수 있으므로 AWS S3나 클라우드 스토리지(Cloud Storage) 같은 정적 파일 제공 서비스를 사용하여 이미지를 따로 저장하고 제공하는 것이 좋음

2. 게시글 작성 기능이 추가돼 메인 페이지 로딩 시 메인 페이지와 게시글을 함께 로딩하도록 수정([routes/page.js](https://github.com/ohbokdong/NodeJSStudy/blob/main/summary/week12/nodebird/routes/page.js))

### 9.5 프로젝트 마무리하기

1. 다른 사용자를 팔로우하는 기능 추가([routes/user.js](https://github.com/ohbokdong/NodeJSStudy/blob/main/summary/week12/nodebird/routes/user.js))
2. 사용자 정보를 불러올 때 팔로워와 팔로잉 목록도 같이 불러오기 위해 deserializeUser를 조작([passport/index.js](https://github.com/ohbokdong/NodeJSStudy/blob/main/summary/week12/nodebird/passport/index.js))
    * include에서 계속 attributes를 지정하는 이유는 실수로 비밀번호 조회하는걸 막기 위함
    * deserializeUser 캐싱하기
      * 라우터가 실행되기 전에 deserializeUser가 먼저 실행됨, 따라서 모든 요청이 들어올 때마다 매번 사용자 정보를 조회하게 돼 DB에 부담이 될 수 있음
      * 따라서 사용자 정보가 빈번하게 바뀌는 것이 아니면 캐싱해두는 것이 좋음
        * 캐싱하는 동안 팔로워와 팔로잉 정보는 갱신되지 않음
        * 캐싱 시간은 서비스 정책에 따라 조절해야 함
      * 실제 서비스에서는 메모리에 캐싱하기보다는 레디스 같은 데이터베이스에 사용자 정보를 캐싱함
3. 팔로잉/팔로워 숫자와 팔로우 버튼을 표시([routes/page.js](https://github.com/ohbokdong/NodeJSStudy/blob/main/summary/week12/nodebird/routes/page.js))

4. 해시태그로 조회하는 /hashtag 라우터 추가
5. routes/post.js, routes/user.js를 app.js에 연결, 업로드한 이미지를 제공할 라우터도 express.static 미들웨어로 uploads 폴더와 연결해 사진들은 uploads 폴더 내 사진들이 /img 주소로 제공됨

```js
app.use('/img', express.static(path.join(__dirname, 'uploads')));
```

### 9.5.1 스스로 해보기(skip)

## 함께 보면 좋은 자료

* [Passport 공식 문서](http://www.passportjs.org/)
* [passport-local 공식 문서](https://www.npmjs.com/package/passport-local)
* [passport-kakao 공식 문서](https://www.npmjs.com/package/passport-kakao)
* [bcrypt 공식 문서](https://www.npmjs.com/package/bcrypt)
* [카카오 로그인](https://developers.kakao.com/docs/latest/ko/kakaologin/common)