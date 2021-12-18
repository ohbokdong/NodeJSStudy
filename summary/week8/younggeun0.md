# 노드 JS 스터디 6장 익스프레스 웹 서버 만들기

## 6.1 익스프레스 프로젝트 시작하기

* 웹 서버 제작 편의성을 높인 프레임워크 == express
* 익스프레스는 http 모듈의 요청과 응답 객체에 추가 기능들을 부여함
* 익스프레스 프로젝트 만들기
	* npm init이나 직접 package.json을 만듦
	* express 인스톨

```bash
# express 모듈 인스톨
npm i express

# 개발을 위해 nodemon(변경사항 시 자동으로 서버 리프레시해주는 앱) 설치
npm i -D nodemon
```

```js
const express = require('express');
const path = require('path');

// express 모듈을 실행해 app 변수에 할당, 내부에 hjttp 모듈이 내장돼 있어 서며 역할을 할 수 있음
const app = express();

// 앱에 전역 객체를 저장할 수 있음
app.set('port', process.env.PORT || 3000);

app.get('/', (req, res) => {

  // express에선 res.write나 res.end 대신 res.send를 사용하면 됨
  // res.send('Hello, Express');

  // sendFile로 파일 응답 가능, 단, 파일 경로는 path 모듈을 사용해서 지정
  res.sendFile(path.join(__dirname, '/index.html'));
});


// listen 부분은 http 웹서버와 동일
app.listen(app.get('port'), () => {
  console.log(app.get('port'), '번 포트에서 대기 중');
});
```


### 6.2 자주 사용하는 미들웨어
* 미들웨어는 익스프레스의 핵심, 요청과 응답의 중간에 위치하여 미들웨어라 부름
	* 뒤에 나오는 라우터와 에러 핸들러 또한 미들웨어의 일종
	* 미들웨어가 익스프레스의 전부
* 미들웨어는 app.use와 함께 사용
	* app.use에 매개변수가 req, res, next인 함수를 넣으면 됨
	* 주소를 첫 번째 인수로 넣어주지 않으면 미들웨어는 모든 요청에서 실행됨
	* 주소를 넣으면 해당 요청에서만 실행됨

```js
// 미들웨어는 위에서부터 아래로 순서대로 실행되며서 요청과 응답 사이에 특별한 기능 추가 가능
app.use((req, res, next) => {
    console.log("모든 요청에 다 실행됨");
    next(); // 다음 미들웨어 호출
});

app.use("/abc", 미들웨어); // /abc 요청에서만 미들웨어 실행됨
app.get("/", (req, res, next) => {
    console.log("GET / 요청에서만 실행됨");
    next();
}, (req, res) => { // 여러개의 미들웨어를 연결 가능, next()를 호출해야 다음 미들웨어로 넘어갈 수 있음
    throw new Error("에러는 에러 처리 미들웨어로 감");
});

// 에러 처리 미들웨어는 매개변수가 4개
// 에러 처리 미들웨어를 연결해주지 않아도 기본적으로 익스프레스가 에러를 처리하긴 하지만 실무에선 직접 연결해주는게 좋음
app.use((err, req, res, next) => {
    console.log(err);
    res.status(500).send(err.message);
});
```

...

* 실무에서 자주 사용되는 패키지들
	* 설치한 패키지들을 app.use에 연결해 사용, req, res, next와 같은 매개변수들이 안보이는 이유는 미들웨어 메서드 내부에 있기 때문


```bash
# .env
COOKIE_SECRET=cookiesecret
```

```js
// app.js
const express = require('express');
const morgan = require('morgan');
const cookieParser = require('cookie-parser');
const session = require('express-session');
const dotenv = require('dotenv');
const path = require('path');

// dotenv 패키지는 .env 파일을 읽어 process.env로 만듦
// process.env를 별도 파일로 관리하는 이유는 보안과 설정 편의성 때문
dotenv.config();

const app = express();
app.set('port', process.env.PORT || 3000);

app.use(morgan('dev'));
app.use('/', express.static(path.join(__dirname, 'public')));
app.use(express.json());
app.use(express.urlencoded({ extended: false }));
app.use(cookieParser(process.env.COOKIE_SECRET));
app.use(session({
  resave: false,
  saveUninitialized: false,
  secret: process.env.COOKIE_SECRET,
  cookie: {
    httpOnly: true,
    secure: false,
  },
  name: 'session-cookie',
}));

const multer = require('multer');
const fs = require('fs');

try {
  fs.readdirSync('uploads');
} catch (error) {
  console.error('uploads 폴더가 없어 uploads 폴더를 생성합니다.');
  fs.mkdirSync('uploads');
}
const upload = multer({
  storage: multer.diskStorage({
    destination(req, file, done) {
      done(null, 'uploads/');
    },
    filename(req, file, done) {
      const ext = path.extname(file.originalname);
      done(null, path.basename(file.originalname, ext) + Date.now() + ext);
    },
  }),
  limits: { fileSize: 5* 1024* 1024 },});
app.get('/upload', (req, res) => {
  res.sendFile(path.join(__dirname, 'multipart.html'));
});
app.post('/upload', upload.single('image'), (req, res) => {
  console.log(req.file);
  res.send('ok');
});

app.get('/', (req, res, next) => {
  console.log('GET / 요청에서만 실행됩니다.');
  next();
}, (req, res) => {
  throw new Error('에러는 에러 처리 미들웨어로 갑니다.')
});
app.use((err, req, res, next) => {
  console.error(err);
  res.status(500).send(err.message);
});

app.listen(app.get('port'), () => {
  console.log(app.get('port'), '번 포트에서 대기 중');
});
```

### 6.2.1 morgan

* morgan 미들웨어는 요청과 응답에 대한 정보를 콘솔에 기록함
	* 인수로 dev 외에 combined, common, short, tiny 등을 넣을 수 있음
	* 개발 환경에선 dev를, 배포 환경에선 combined를 주로 씀

```js
app.use(morgan('dev'));

// dev 모드 기준으로 [HTTP 메서드] [주소] [HTTP 상태코드] [응답 속도] - [응답 바이트]를 의미
```


### 6.2.2 static

* 기본으로 제공돼 따로 설치할 필요 없이 express 객체 안에서 꺼내 사용
* static 미들웨어는 정적인 파일들을 제공하는 라우터 역할을 함
	* 요청 경로와 정적파일의 위치를 맵핑하는 기능
	* 외부인이 서버 구조를 쉽게 파악할 수 없게 하기 위함
* 정적 파일을 알아서 제공하므로 fs.readFile로 파일을 직접 읽어 전송할 필요 없음
* 요청 경로에 파일이 없으면 내부적으로 next()를 호출, 파일이 발견된 경우 다음 미들웨어는 실행되지 않음

```js
// app.use("요청 경로", express.static("실제 경로"));

app.use('/', express.static(path.join(__dirname, 'public')));
```


### 6.2.3 body-parser

* 요청 본문에 있는 데이터를 해석해서 req.body 객체로 만들어주는 미들웨어
	* 보통 폼 데이터나 AJAX 요청 데이터를 처리
	* 단, 멀티파트(이미지, 동영상, 파일) 데이터는 처리하지 못 함, 멀티파트 데이터는 multer 모듈을 사용
* express에 4.16+부터 포함돼 설치 불필요
	* 단, JSON, URL-encoded 형식 데이터 외에도 Raw, Text 형식의 데이터를 추가로 해석하기 위해선 body-parse를 직접 설치해야 함
	* Raw는 요청의 본문이 버퍼 데이터일 때, Text는 텍스트 데이터일 때 해석하는 미들웨어

```bash
npm i body-parser
```

```js
const bodyParser = require("body-parser");

app.use(bodyParser.raw());
app.use(bodyParser.text());
```

* 요청 데이터의 종류
	* JSON
		* JSON 형태의 데이터 전달 방식
	* URL-encoded
		* 주소 형식으로 데이터를 보내는 방식
		* 폼 전송은 URL-encoded 방식을 주로 사용

```js
app.use(express.json());

// urlencoded extended 옵션이 false면 querystring 모듈을 사용하여 쿼리스트링을 해석
// true면 qs 모듈을 사용해서 쿼리스트링을 해석, qs 모듈은 내장 모듈이 아닌 npm 패키지, querystring 모듈이 확장된 모듈
app.use(express.urlencoded({ extended: false }));
```

* bodyparser를 이용하면 req.on("data"), req.on("end") 스트림 처리가 필요 없음
	* 패키지 내부에서 스트림 처리해 req.body에 추가하기 때문


### 6.2.4 cookie-parser

* cookie-parser는 요청에 동봉된 쿠키를 해석해 req.cookies 객체로 만듦
	* 유효기간 지난 쿠키는 알아서 걸러냄
* 서명된 쿠키가 있는 경우, 제공한 비밀 키를 통해 해당 쿠키가 내 서버가 만든 쿠키임을 검증 가능
	* 쿠키 위조를 막기 위해 비밀 키를 통해 만들어낸 서명을 쿠키 값 뒤에 붙임
	* 서명이 붙은 쿠키는 name=young.sign 같은 모양이 됨
	* 서명된 쿠키는 req.cookies 대신 req.signedCookies 객체에 들어 있음


```js
// app.use(cookieParser(비밀키))

app.use(cookieParser(process.env.COOKIE_SECRET));
```

* cookie-parser는 쿠키를 생성할 때 쓰이는 것은 아님(요청에 담긴 쿠키 파싱용)
* 쿠키를 생성/ 제거하기 위해 res.cookie, res.clearCookie 메서드를 사용해야 함
	* 옵션은 쿠키 모듈 옵션과 동일함(domain, expires, httpOnly, maxAge, path, secure 등)

```js
// res.cookie(키, 값, 옵션) 형식으로 사용

res.cookie("name", "young", {
    expire: new Date(Date.now() + 900000),
    httpOnly: true,
    secure: true,
});

// 쿠키 삭제 시엔 키, 값, 옵션 모두 일치해야 지워짐
// 단, expires, maxAge 옵션은 일치할 필요 없음
// signed란 옵션이 true면 쿠키 뒤에 서명이 붙음 (내 서버가 쿠키를 만들었다는 것을 검증 가능), 켜두는 것이 좋음
res.clearCookie("name", "young", { httpOnly: true, secure: true });
```

### 6.2.5 express-session

* 세션 관리용 미들웨어, 로그인 등 이유로 세션을 구현하거나 특정 사용자를 위한 데이터를 임시적으로 저장해둘 때 매우 유용
* 세션은 사용자별로 req.session 객체 안에 유지됨
* express-session 1.5 버전 이전에는 내부적으로 cookie-parser를 사용해 cookie-parser 미들웨어보다 뒤에 위치해야 했지만 1.5 버전 이후부터는 사용하지 않게 돼 순서가 상관 없어짐
	* 그래도 cookie-parser 미들웨어 뒤에 놓는 것이 안전


```js
// express-session은 인수로 세션에 대한 설정을 받음

app.use(session({
  resave: false,              // 요청이 올 때 세션에 수정 사항이 생기지 않더라도 세션을 다시 저장할지 설정하는 것
  saveUninitialized: false,   // 세션에 저장할 내역이 없더라도 처음부터 세션을 생성할지 설정하는 것

  // express-session은 세션 관리 시 클라이언트에 쿠키를 보냄(세션 쿠키), 쿠키 서명용 값
  // cookie-parser의 secret 값과 같에 설정하는게 좋음
  secret: process.env.COOKIE_SECRET,    
  
  // 세션 쿠키에 대한 설정, 일반적인 쿠키 옵션 모두 제공됨
  cookie: {
    httpOnly: true,    // 클라이언트가 쿠키 확인하지 못하도록 함
    secure: false,     // https가 아닌 환경에서도 사용하도록 함(배포 시 true 권장)

    // 데이터베이스를 연결하면 store란 옵션으로 세션을 저장하는게 좋음(보통 레디스가 쓰임)
  },
  name: 'session-cookie',
}));
```

* express-session으로 만들어진 req.session 객체에 값을 대입하거나 삭제해서 세션을 변경가능

```js
// 세션 등록
req.session.name = "young";

// 세션 아이디 확인
req.sessionID;

// 세션 모두 제거
req.session.destroy();

// 세션을 강제로 저장하기 위한 req.session.save 메서드가 존재하지만
// 일반적인 요청이 끝날 때 자동으로 호출되므로 직접 save 메서드를 호출할 일은 거의 없음
```

* 세션 쿠키는 express-session에서 서명해 쿠키 값 앞에 s:가 붙음
	* encodeURIComponent 함수가 실행되어 s%3A가 됨
	* 쿠키값이 저걸로 시작되면 express-session 미들웨어에 의해 암호화된 것이라고 생각하면 됨


### 6.2.6 미들웨어의 특성 활용하기

* 여러개의 미들웨어를 장착가능
	* 아래 미들웨어들은 내부적으로 next()를 호출하여 연달아 사용 가능
	* next를 호출하지 않는 미들웨어는 res.send나 res.sendFile 등의 메서드로 응답을 보내야 함
		* 때문에 정적 파일을 제공하는 경우 static 밑에 미들웨어는 실행되지 않음

```js
app.use(
    morgan("dev"),
    express.static("/", path.join(__dirname, "public")),
    express.json(),
    express.urlencoded({ extended: false }),
    cookieParser(process.env.COOKIE_SECRET),
);
```

* 미들웨어 장착 순서에 따라 어떤 미들웨어는 실행되지 않을 수 있다는걸 기억
* 만약 next도 호출하지 않고 응답도 보내지 않으면 클라이언트는 응답을 받지 못해 하염없이 기다림
* next함수에 인수(문자열)를 넣으면 넣으면 다음 라우터의 미들웨어로 바로 이동하고 그 외의 인수를 넣으면 에러 처리 미들웨어로 이동함
	* 라우터에서 에러가 발생할 때 에러는 next(err)를 통해 에러 처리 미들웨어로 넘김
* 미들웨어 간 데이터를 전달하는 방법도 있음
	* 세션을 사용한다면 req.session 객체에 데이터를 넣어도 되지만 세션이 유지되는 동안 데이터가 계속 유지되는 단점이 있음
	* 요청이 끝날 때까지만 유지하고 싶으면 req 객체에 데이터를 넣어두면 됨
		* 현재 요청이 처리되는 동안 req.data를 통해 미들웨어 간 데이터를 공유 가능
		* 새로운 요청이 오면 req.data는 초기화 됨
		* 속성명은 data일 필욘 없지만 다른 미들웨어와 겹치지 않게 조심해야 함 (예로 req.body 같은 건 겹침)

```js
app.use((req, res, next) => {
    req.data = "데이터 넣기";
    next();
}, (req, res, next) => {
    console.log(req.data); // 데이터 받기
    next();
});
```

* app.set은 익스프레스에서 데이터를 저장할 수 있는 메서드
	* app.set은 익스프레스에서 전역적으로 사용됨, 앱 전체의 설정을 공유할 때 사용
	* 개인정보 등 값을 저장하기 부적절하므로 req 객체를 통해 개인 데이터를 전달하는게 좋음
* 미들웨어 안에 미들웨어를 넣는 패턴을 사용하면 분기처리를 할 수 있음

```js
app.use((req, res, next) => {
    if (process.env.NODE_ENV === "production") {
        morgan("combined")(req, res, next);
    } else {
        morgan("dev")(req, res, next);
    }
});
```

### 6.2.7 multer

* 이미지, 동영상 등 여러가지 파일들을 멀티파트 형식으로 업로드할 때 사용하는 미들웨어
	* 멀티파트 형식이란 enctype이 multipart/form-data인 폼을 통해 업로드하는 데이터 형식을 의미

```html
<!-- multipart.html -->
<form id="form" action="/upload" method="post" enctype="multipart/form-data">
  <input type="file" name="image" />
  <button type="submit">업로드</button>
</form>
```

* 이런 폼을 통해 업로드하는 파일은 body-parser로 처리할 수 없고 직접 파싱(해석)하기도 어려워 multer란 미들웨어를 따로 사용

```js
const multer = require('multer');
const fs = require('fs');

app.get('/upload', (req, res) => {
  res.sendFile(path.join(__dirname, 'multipart.html'));
});

// uploads 폴더를 미리 만듦
try {
  fs.readdirSync('uploads');
} catch (error) {
  console.error('uploads 폴더가 없어 uploads 폴더를 생성합니다.');
  fs.mkdirSync('uploads');
}

// 기본 설정, 함수의 인수로 설정 객체를 넣음
const upload = multer({
  // storage - 어디에 어떤 이름으로 저장할지 설정하는 옵션
  storage: multer.diskStorage({
    // destination과 filename함수 req 객체엔 요청에 대한 정보가
    // file 객체에는 업로드한 파일에 대한 정보가 있음
    // done 매개 변수는 함수
    destination(req, file, done) {

      // done 첫 번째 인수에는 에러가 있다면 에러를 넣음
      // 두 번째 인수는 실제 경로나 파일 이름을 넣음, req나 file의 데이터를 가공해서 done으로 넘기는 형식
      // 현재는 uploads란 폴더에 [파일명+현재시간.확장자] 파일명으로 업로드하는 방식, uploads폴더가 미리 꼭 존재해야 함
      done(null, 'uploads/');
    },
    filename(req, file, done) {
      const ext = path.extname(file.originalname);
      done(null, path.basename(file.originalname, ext) + Date.now() + ext);
    },
  }),
  // 업로드에 대한 제한사항을 설정하는 옵션, 파일사이즈는 바이트 단위
  limits: { fileSize: 5* 1024* 1024 }, // 5MB 제한});

// 하나만 업로드하는 경우 single 미들웨어를 사용
app.post('/upload', upload.single('image'), (req, res) => {
  // signle 미들웨어를 라우터 미들웨어 앞에 넣어두면, multer 설정에 따라 파일 업려드 후 req.file 객체가 생성됨
  console.log(req.file);
  res.send('ok');
});
```

* req.file 객체의 모습

```js
{
  fieldname: 'image',
  originalname: 'KakaoTalk_20211217_230810270.jpg',
  encoding: '7bit',
  mimetype: 'image/jpeg',
  destination: 'uploads/',
  filename: 'KakaoTalk_20211217_2308102701639786389912.jpg',
  path: 'uploads\\KakaoTalk_20211217_2308102701639786389912.jpg',
  size: 161480
}
```

* 여러 파일을 업로드 하는 경우 input 태그에 multiple을 쓰고 array 메서드를 사용

```html
<!-- multipart.html -->
<form id="form" action="/upload" method="post" enctype="multipart/form-data">
  <input type="file" name="many" multiple />
  <button type="submit">업로드</button>
</form>
```

```js
app.post('/upload', upload.array('image'), (req, res) => {
  // 파일 결과도 req.file 대신 req.files에 배열로 들어옴
  console.log(req.files);
  res.send('ok');
});
```

* 파일을 여러개 업로드 하지만 input 태그나 폼 데이터의 키가 다른 경우 field 미들웨어를 사용

```html
<!-- multipart.html -->
<form id="form" action="/upload" method="post" enctype="multipart/form-data">
  <input type="file" name="image1" />
  <input type="file" name="image2" />
  <input type="text" name="title" />
  <button type="submit">업로드</button>
</form>
```

```js
app.post('/upload', upload.fields([{ name: 'image1' }, { name: 'image2' }]), 
  (req, res) => {
  console.log(req.files, req.body);
  res.send('ok');
});
* 로깅 결과[Object: null prototype] {
  image1: [
    {
      fieldname: 'image1',
      originalname: 'KakaoTalk_20211217_230810270.jpg',
      encoding: '7bit',
      mimetype: 'image/jpeg',
      destination: 'uploads/',
      filename: 'KakaoTalk_20211217_2308102701639787184681.jpg',
      path: 'uploads\\KakaoTalk_20211217_2308102701639787184681.jpg',
      size: 161480
    }
  ],
  image2: [
    {
      fieldname: 'image2',
      originalname: 'KakaoTalk_20211217_224125327_03.jpg',
      encoding: '7bit',
      mimetype: 'image/jpeg',
      destination: 'uploads/',
      filename: 'KakaoTalk_20211217_224125327_031639787184693.jpg',
      path: 'uploads\\KakaoTalk_20211217_224125327_031639787184693.jpg',
      size: 675179
    }
  ]
} [Object: null prototype] { title: 'hello world' }*/
```

* 파일을 업로드하지 않고도 멀티파트 형식으로 업로드하는 특수한 경우는 none 미들웨어를 사용

```html
<!-- multipart.html -->
<form id="form" action="/upload" method="post" enctype="multipart/form-data">
  <input type="text" name="title" />
  <button type="submit">업로드</button>
</form>
```

```js
app.post('/upload', upload.none(), (req, res) => {
  // 파일을 업로드하지 않았으므로 req.body만 존재
  console.log(req.body);
  res.send('ok');
});
```

## 6.3 Router 객체로 라우팅 분리하기

* 익스프레스를 사용하는 이유 중 하나는 라우팅을 깔끔하게 관리할 수 있기 때문
* app.js에 라우터를 많이 연결하면 app.js 코드가 매우 길어지므로 익스프레스에서 라우터를 분리할 수 있는 방법을 제공
	* routes 폴더를 만들고 그 안에 라우터 별로 js를 작성

```js
// routes/index.js
const express = require('express');

const router = express.Router();

// GET / 라우터
router.get('/', (req, res) => {
  res.send('Hello, Express');
});

module.exports = router;
```

```js
// routes/user.js
const express = require('express');

const router = express.Router();

// GET /user 라우터
router.get('/', (req, res) => {
  res.send('Hello, User');
});

module.exports = router;
```

```js
// app.js
const indexRouter = require('./routes');     // index.js는 생략가능("./routes/index.js") == ("./routes")
const userRouter = require('./routes/user');

...

// 만들었던 index.js와 user.js를 app.use를 통해 app.js에 연결
app.use('/', indexRouter);
app.use('/user', userRouter);

// 에러 처리 미들웨어 위에 404 상태 코드를 응답하는 미들웨어를 추가
// 에러 처리 미들웨어 위에 넣어둔 미들웨어는 일치하는 라우터가 없을 때 404 상태 코드를 응답하는 역할
app.use((req, res, next) => {
  res.status(404).send('Not Found');
});

// 에러 처리 미들웨어
app.use((err, req, res, next) => {
  console.error(err);
  res.status(500).send(err.message);
});
```
...

* index.js와 user.js는 모양이 거의 비슷하지만 다른 주소의 라우터 역할을 함
	* app.use로 연결할 때의 차이 때문, app.use로 연결할 때 주소가 합쳐짐
* next("route")는 라우터에 연결된 나머지 미들웨어를 건너뛰고 싶을 때 사용

```js
// 같은 주소의 라우터를 여러 개 만들어도, 라우터가 몇 개든 간 next()를 호출하면 다음 미들웨어가 호출됨
router.get("/", (req, res, next) => {
    next("route"); // 미들웨어가 체이닝 됐을 때 건너 뜀
}, function(req, res, next) {
    console.log("실행되지 않음1");
    next();
}, function(req, res, next) {
    console.log("실행되지 않음2");
    next();
});

router.get("/", (req, res) => {
    console.log("실행 됨");
    res.send("Hello, Express");
});
```

* 라우터 주소에 정규표현식을 비롯한 특수 패턴을 사용가능
	* 주의할 점은 일반 라우터보다 뒤에 위치해야 다른 라우터를 방해하지 않음
	* /user/like 같은 라우터는 /user/:id 같은 라우트 매개변수를 쓰는 라우터보다 위에 위치해야 함

```js
// 라우트 매개변수 패턴
// /users/1 이나 /users/123 등 요청도 이 라우터가 처리, id는 req.params 객체 안에 들어 있음(req.params.id)
// :type이면 req.params.type 으로 조회 가능
// 주소에 쿼리 스트링도 쓸 수 있으며 쿼리스트링 키-값 정보는 req.query 객체 안에 들어 있음
router.get("/user/:id", (req, res) => {
    console.log(req.params, req.query);
});

// /users/123?limit=5&skip=10 요청 시 req.params와 req.query 객체는 다음과 같음
// { id: '123' } { limit: '5', skip: '10' }
```

* 주소는 같지만 메서드가 다른 코드가 있을 땐 하나로 줄일 수 있음

```js
router.get("/abc", (req, res) => {
    res.send("GET /abc");
});

router.post("/abc", (req, res) => {
    res.send("POST /abc");
});

-------------------------------
// 하나로 합침
router.route("/abc")
    .get((req, res) => {
        res.send("GET /abc");
    })
    .post((req, res) => {
        res.send("POST /abc");
    });
```
