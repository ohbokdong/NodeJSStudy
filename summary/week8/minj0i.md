# 6장 익스프레스 웹 서버 만들기
웹 서버 프레임워크 - 대표: 익스프레스   
익스프레스는 http 모듈의 요청과 응답 객체에 추가 기능들을 부여하고 편리한 메서드들을 추가하여 기능 보완함   

## 6.1 익스프레스 프로젝트 시작하기
`npm init`으로 `package.json` 파일 생성
```JSON
{
  "name": "learn-express",
  "version": "0.0.1",
  "description": "익스프레스를 배우자",
  "main": "app.js",
  "scripts": {
    // app.js를 nodemon으로 실행한다는 뜻 -> 서버 코드 수정 시 nodemon모듈로 서버를 자동 재시작
    "start": "nodemon app"
  },
  "author": "ZeroCho",
  "license": "MIT"
}
```
```
npm i express
npm i -D nodemon
``` 
```JAVASCRIPT
// app.js
const express = requiref'express');

// express변수를 실행해 app 변수에 할당, 익스프레스 내부에 http모듈이 내장되어있으므로 서버 역할 가능 
const app = express();
// 포트번호 설정, app.get('키') 하면 값 가져오기 가능
app.set('port', process.env.PORT || 3000);

// res.write / res.end 대신 res.send로 사용 가능
// app.post, app.put, app.patch, app.delete, app.options 메서드 존재
app.get('/', (req, res) => {
  res.send('Hello, Express');
});

app.listen(app.get('port'), () => {
  console.log(app.get('port'), '번 포트에서 대기 중');
});
```
```
$ npm start
> learn-express@0.0.1 start C:\Users\zerocho\learn-express
> nodemon app

[nodemon] 2.0.3
[nodemon] to restart at any time, enter `rs`
[nodemon] watching dir(s): *.*
[nodemon] watching extensions: js,mjs,json
[nodemon] starting `node app.js`
3000 번 포트에서 대기 중
```

문자열 대신 HTML로 응답하고 싶다면 `res.sendFile` 메서드 사용
```JS
// app.js
...
app.get('/', (req, res) => {
  // res.send('Hello, Express');
  res.sendFile(path.join(__dirname, '/index.html'));
});
...
```

## 6.2 자주 사용하는 미들웨어
요청과 응답의 중간에 위치하여 미들웨어라고 부르는 것. `app.use`와 함께 사용함   
```JS
// app.js
...
app.set('port', process.env.PORT || 3000);

app.use((req, res, next) => {
  console.log('모든 요청에 다 실행됩니다.');
  next();
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
...
```
- app.use(req, res, next)인 함수를 사용
- next를 실행하지 않으면 다음 미들웨어가 실행되지 않음
- app.get, app.post는 GET요청, POST요청에서 미들웨어를 실행한다는 것

에러처리 미들웨어는 매개변수가 err, req, res, next로 네 개 => 6.5.3절에서 만나요

### 실무에서 자주 사용하는 패키지들
```
$ npm i morgan cookie-parser express-session dotenv
```
```JS
// app.js
const express = require('express');
const morgan = require('morgan');
const cookieParser = require('cookie-parser');
const session = require('express-session');
const dotenv = require('dotenv');
const path = require('path');

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

app.use((req, res, next) => {
  console.log('모든 요청에 다 실행됩니다.');
  next();
});
...
```
```
# .env
COOKIE_SECRET=cookiesecret
```
dotenv파일은 .env파일을 읽어서 process.env로 만들고 process.env.COOKIE_SECRET에 cookiesecret값이 할당되게 함   
> env파일을 쓰는 이유
비밀 키 들을 소스 코드에 적어두면 소스 유출 시 보안 위협

### **6.2.1 morgan**
```
# 콘솔
GET / 500 7.409 ms - 50
```
요청과 응답에 대한 정보를 콘솔에 기록함   
[HTTP 메서드] [주소] [HTTP 상태 코드] [응답 속도] - [응답 바이트]를 의미
```JS
// 위의 app.js 코드 중
app.use(morgan('dev'))
```
dev외 - combined, common, short, tiny 넣기 가능   

### **6.2.2 static**
정적인 파일들을 제공하는 라우터 역할이며 express 내장
```JS
// 위의 app.js 코드 중
// app.use('요청 경로', express.static('실제 경로'));
app.use('/', express.static(path.join(__dirname, 'public')));
```
실제 경로에는 public이 들어 있지 않음   
서버의 폴더 경로와 요청 경로가 달라 외부인이 서버 구조 쉽게 파악하기 힘듦 => 보안성   
정적 파일 알아서 제공하기 때문에 fs.readFile 필요 없음

### **6.2.3 body-parser**
요청 본문에 있는 데이터 해석해서 req.body 객체로 만들어주는 미들웨어. 멀티파트(이미지, 동영상, 파일) 데이터는 multer 모듈을 사용해야 함
```JS
app.use(express.json());
app.use(express.urlencoded({ extended: false}));
// extended-false: 노드의 querystring 모듈 사용해서 해석
// extended-true: qs모듈을 사용하여 쿼리스트링 해석

// raw, text 처리해야 하는 경우
app.use(bodyParser.raw());
app.use(bodyParser.text());
```

### **6.2.4 cookie-parser**
요청에 동봉된 쿠키를 해석해 req.cookies 객체로 만듬
```JS
// 위의 app.js 코드 중
app.use(cookieParser(비밀키));
```
해석된 쿠키는 req.cookies 객체에 저장   
ex) name=zerocho 쿠키 전송 => req.cookies = { name: 'zerocho' }   

서명된 쿠키가 있는 경우 name=zerocho.sign과 같은 형태가 되고 req.cookies 대신 req.signedCookies 객체에 들어 있게 됨   

쿠키 생성/제거: res.cookie, res.clearCookie 메서드를 사용 `res.cookie(키, 값, 옵션)`   
쿠키 지울 때 키와 값 외의 옵션도 정확히 일치해야 쿠키 지워짐

### **6.2.5 express-session**
세션 관리용 미들웨어. 로그인 등 세션 구현하거나 특정 사용자를 위한 데이터 임시 저장해둘 때 유용
```JS
// app.js
...
app.use(session({
  resave: false, // 요청이 올 때 세션에 수정 사항 안생겨도 다시 세션 저장할 거?
  saveUninitialized: false, // 세션 저장 내역 없어도 처음부터 세션 생성할 거?
  secret: process.env.COOKIE_SECRET,
  cookie: {
    httpOnly: true, // 클라이언트에서 쿠리 확인 하지 못함
    secure: false, // https가 아닌 환경에서도 사용할 수 있게 함
  },
  name: 'session-cookie', // secret과 같게 설정하는 것이 좋음. 기본 이름 connect.sid
  // store 옵션 - 데이터베이스를 연결하여 세션을 유지하도록 하는 것, 보통 레디스가 사용됨
}));
...
```
1.5버전 이전에는 내부적 cookie-parser를 사용하지 않기 때문에 안전을 위해 그 뒤에 놓는것이 안전

```JS
res.session.name = 'zerocho'; // 세션 등록
req.sessionID; // 세션 아이디 확인
req.session.destory(); // 세션 모두 제거
```
서명한 쿠키는 앞에 s:이 붙고, 실제로는 encodeURIComponent 함수가 실행되어 s%3A가 붙게 됨

### **6.2.6 미들웨어의 특성 활용하기**
next()에 route라는 문자열을 넣으면 다음 라우터의 미들웨어로 바로 이동하고, 그 외 인수 넣으면 에러 처리 미들웨어로 이동하게 됨. 이 때 인수는 err 매개변수가 됨


![next의동작](https://thebook.io/img/080229/243_1.jpg)

```JS
next(err)
(err, req, res, next) => ( )
```
세션에 데이터를 넣으면 세션 유지 기간 내내 데이터도 유지되므로    
요청 끝날 때 까지만 데이터를 넣고 싶으면 res.data = '데이터'로 지정 가능   
단, 새로운 요청이 오면 req.data 초기화 됨   
**다른 미들웨어와 겹치지 않도록 조심**


> app.set <-> req.data 

app.set은 전역에서 사용할 때만 사용

**미들웨어 유용한 패턴**
기존 미들웨어의 기능을 확장할 수 있음
```JS
app.use(morgan('dev'));
// 또는
app.use((req, res, next) => {
  morgan('dev')(req, res, next);
});
```

### **6.2.7 multer**
멀티파트 형식이란 enctype이 multipart/form-data인 폼을 통해 업로드하는 데이터의 형식
```html
<!-- multipart.html -->
<form action="/upload" method="post" enctype="multipart/form-data">
  <input type="file" name="image" />
  <input type="text" name="title" />
  <button type="submit">업로드</button>
</form>
```
```
npm i multer
```
```JS
const multer = require('multer');

const upload = multer({
  // storage
  storage: multer.diskStorage({
    // 어디에
    destination(req, file, done) {
      done(null, 'uploads/');
    },
    // 어떤 이름으로
    filename(req, file, done) {
      const ext = path.extname(file.originalname);
      done(null, path.basename(file.originalname, ext) + Date.now() + ext);
    },
  }),
  // 업로드에 대한 제한 사항 설정
  limits: { fileSize: 5 * 1024 * 1024 },
});
```
위 설정을 실제로 사용하려면 서버에 uploads폴더가 존재해야 함
- 파일 하나: upload.single(s)
```JS
const fs = require('fs');

try {
  fs.readdirSync('uploads');
} catch (error) {
  console.error('uploads 폴더가 없어 uploads 폴더를 생성합니다.');
  fs.mkdirSync('uploads');
}

app.post('/upload', upload.single('image'), (req, res) => { 
  console.log(req.file, req.body); 
  // req.file - 업로드 성공 시 결과
  // req.body - 파일이 아닌 데이터인 title이 들어 있음
  res.send('ok'); 
});
```
- 파일 여러개: upload.array()
  - 결과도 req.files 배열에 들어가 있음
  - 데이터의 키가 다른 경우는 아래 처럼
```JS
app.post('/upload',
  upload.fields([{ name: 'image1' }, { name: 'image2' }]),
  (req, res) => {
    console.log(req.files, req.body);
    res.send('ok');
  },
);
```
- 파일을 업로드하지 않고 멀티파트 형식으로 업로드하는 경우: upload.none()
  - 파일을 업로드하지 않았으므로 req.body만 존재

![multer미들웨어](https://thebook.io/img/080229/249.jpg)


실습코드는 다 해보셨죠?^_^

## 6.3 Router 객체로 라우팅 분리하기
익스프레스는 라우팅을 깔끔하게 관리할 수 있다   
routes 폴더를 만들고 그 안에 index.js, user.js를 작성
```JS
// routes/index.js
const express = require('express');

const router = express.Router();

// GET / 라우터
router.get('/', (req, res) => {
  res.send('Hello, Express');
});

module.exports = router;
```
```JS
// routes/user.js
const express = require('express');

const router = express.Router();

// GET /user 라우터
router.get('/', (req, res) => {
  res.send('Hello, User');
});

module.exports = router;
```
```JS
// app.js
...
const path = require('path');

dotenv.config();
const indexRouter = require('./routes'); // == './routes/index.js'
const userRouter = require('./routes/user');
...
  name: 'session-cookie',
}));

app.use('/', indexRouter);
app.use('/user', userRouter);

app.use((req, res, next) => {
  res.status(404).send('Not Found');
});

app.use((err, req, res, next) => {
...
```

아래 코드는 next('route')에서 해당 라우터를 찾아가서 그 미들웨어만 실행함
```JS
router.get('/', function(req, res, next) {
  next('route');
}, function(req, res, next) {
  console.log('실행되지 않습니다');
  next();
}, function(req, res, next) {
  console.log('실행되지 않습니다');
  next();
});
router.get('/', function(req, res) {
  console.log('실행됩니다');
  res.send('Hello, Express');
});
```

> 라우트 매개변수

router.get('/user/:id') 식으로 쓸 수 있음. 단 일반 라우터 뒤에 위치해야 함   
`/users/123?limit=5&skip=10`처럼 주소에 쿼리스트링을 쓰는 경우   
{id: '123'} {limit: '5', skip: '10'}   

미들웨어가 존재하지 않아도 익스프레스가 자체적으로 404에러를 처리해주기도 하지만 연결해주는 게 좋다
```JS
app.use((req, res, next) => {
  res.status(404).send('Not Found');
});
```

관련있는 코드는 묶는게 보기 좋음
```JS
router.route('/abc')
  .get((req, res) => {
    res.send('GET /abc');
  })
  .post((req, res) => {
    res.send('POST /abc');
  });
```