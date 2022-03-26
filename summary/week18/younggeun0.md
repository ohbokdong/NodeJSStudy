# Node.js 교과서 정리 15, AWS와 GCP로 배포하기

> AWS(Amazon Web Service), GCP(Google Cloud Platform)

## 15.1 서비스 운영을 위한 패키지
- 서비스 개발 도중엔 서버에 문제가 생겨 꺼져도 큰 문제가 되지 않으나 출시 후엔 서비스 자체에 심각한 타격을 입음
- 또 취약점을 노린 공격이 들어올 수도 있음 - 이러한 문제를 막기위한 최소한의 조치가 필요함

### 15.1.1 morgan과 express-session
- 익스프레스 미들웨어 일부가 개발용으로 설정돼 있음, 배포용으로 설정변경이 필요함
- process.env.NODE_ENV는 배포 환경인지 개발 환경인지 판단할 수 있는 환경 변수
	- 주로 배포 환경일 때 morgan은 combined 모드로 사용, 개발환경일 때 dev로 사용
		- combined모드일 때 더 많은 사용자 정보를 로그로 남김

```js
if (process.env.NODE_ENV === 'production') {
  app.use(morgan('combined'));
} else {
  app.use(morgan('dev'));
```

- process.env.NODE_ENV는 .env에 넣을 수 없음 (개발/ 배포 환경에 따라 값이 바뀌어야 하는데 .env 파일은 정적 파일이기 떄문)
- express-session을 배포용으로 설정
	- 사용자에 따라 설정할 필요가 없을 수도 있음
		- https 적용을 위해 노드 서버 앞에 다른 서버를 둔 배포환경일 때는 proxy를 true로 설정
		- https 적용할 때만  cookie.secure 를 true로 설정

```js
app.use(cookieParser(process.env.COOKIE_SECRET));
const sessionOption = {
  resave: false,
  saveUninitialized: false,
  secret: process.env.COOKIE_SECRET,
  cookie: {
    httpOnly: true,
    secure: false,
  },
};
if (process.env.NODE_ENV === 'production') {
  sessionOption.proxy = true;
  // sessionOption.cookie.secure = true;
}
app.use(session(sessionOption));
app.use(passport.initialize());
```


### 15.1.2 시퀄라이즈
- 데이터베이스도 배포 환경으로 설정, 시퀄라이즈의 경우 수정이 필요
- 시퀄라이즈에서 가장 큰 문제는 비밀번호가 하드코딩돼 있는 것, JSON 파일이므로 변수를 쓸 수 없음
	- 다행히 시퀄라이즈에선 json 대신 js파일을 설정으로 쓸 수 있게 지원함
	- js 파일이므로 dotenv 모듈을 사용 가능, 보안 규칙에 따라 나머지 정보도 process.env로 바꿔도 됨
		- username, host 속성은 각각 아이디와 DB 서버 주소 역할을 하므로 숨기는게 좋음
	- process.env의 값에 따라 아래 object 내용이 적용됨 (development, production)
		- 쿼리 수행 내용이 노출되지 않기 위해서 production일 때 logging 값을 false로 설정해서 숨김

```js
// config/config.js
require('dotenv').config();

module.exports = {
  development: {
    username: 'root',
    password: process.env.SEQUELIZE_PASSWORD,
    database: 'nodebird',
    host: '127.0.0.1',
    dialect: 'mysql',
  },
  test: {
    username: "root",
    password: process.env.SEQUELIZE_PASSWORD,
    database: "nodebird_test",
    host: "127.0.0.1",
    dialect: "mysql"
  },
  production: {
    username: 'root',
    password: process.env.SEQUELIZE_PASSWORD,
    database: 'nodebird',
    host: '127.0.0.1',
    dialect: 'mysql',
    logging: false,
  },
};
```

### 15.1.3 cross-env
- cross-env 패키지를 사용하면 동적으로 process.env(환경변수)를 변경가능, 모든 운영체제에서 동일한 방법으로 환경 변수를 변경 가능
- 스크립트를 실행할 때 process.env를 동적으로 설정하는 방법
	- 아래 명령어에서 처럼 입력 시 NODE_ENV 와 PORT 는 process.env.NODE_ENV process.env.PORT로 사용 가능
	- 단, 리눅스와 맥은 되지만 윈도우에서는 이렇게 설정할 수 없음

```json
// package.json
  "scripts": {
    "start": "NODE_ENV=production PORT=80 start server",
    "dev": "nodemon server",
    "test": "jest"
  },
```

- 위 방식을 윈도우도 지원하기 위해 cross-env를 사용

```bash
npm i cross-env
```

```json
  "scripts": {
    "start": "cross-env NODE_ENV=production PORT=80 start server",
    "dev": "nodemon server",
    "test": "jest"
  },
```

### 15.1.4 sanitize-html, csurf
- sanitize-html과 csurf 패키지는 각각 XSS(Cross Site Scripting), CSRF(Cross Site Request Forgery) 공격을 막기 위한 패키지
	- XSS는 악의적인 사용자가 사이트에 스크립트를 삽입하는 공격, 악성 사용자가 게시글이나 댓글 등을 업로드할 때 자바스크립트가 포함된 태그를 올리면, 나중에 다른 사용자가 그 게시글이나 댓글을 볼 때 그 스크립트가 실행되어 예기치 못한 동작을 하게 됨
		- 따라서 서버에선 사용자가 게시글을 업로드할 때 스크립트가 포함돼 있는지 검사해서 존재하면 제거해야 함, 다만, 공격성 스크립트의 유형이 많으므로 라이브러리의 도움을 받는 것이 좋음
	- CSRF는 사용자가 의도치 않게 공격자가 의도한 행동을 하게 만드는 공격, 예를 들면 특정 페이지를 방문할 때 저절로 로그아웃이 되거나 게시글이 써지는 현상을 유도할 수 있음. 은행과 같은 사이트에선 다른 사람에게 송금하는 행동을 넣는 등 상황에 따라 크게 악용될 수 있는 공격
		- 이 공격을 막으려면 내가 한 행동이 내가 한 것이 맞다는 점을 인증해야 함, 이 때 CSRF 토큰이 사용디고, csurf 패키지는 이 토큰을 쉽게 발급하거나 검증할 수 있도록 도움

```bash
npm i sanitize-html csurf
```

- 사용자가 업로드한 html을 sanitize-html 함수로 감싸면 허용하지 않는 태그나 스크립트는 제거됨
	- 두 번째 인수로 허용할 부분에 대한 옵션을 넣을 수 있는데, 옵션 목록은 공식 문서를 참고하면 됨

```js
const sanitizeHtml = require('sanitize-html');

const html = "<script>location.href = 'https://gilbut.co.kr'</script";
console.log(sanitizeHtml(html)); // ''
```

- GET/form 라우터는 form을 렌더링하는 라우터고 POST/form 라우터는 form에서 보낸 데이터를 처리하는 라우터
	- 익스프레스 미들웨어 형식으로 동작, form 같은 것을 렌더링할 때 CSRF 토큰을 같이 제공

```js
const csrf = require('csurf');
const csrfProtection = csrf({ cookie: true });

app.get('/form', csrfProtection, (req, res) => {
    res.render('csrf', { csrfToken: req.csrfToken() });
});

app.post('/form', csrfProtection, (req, res) => {
    res.send('ok');
});
```


### 15.1.5 pm2
- 원활한 서버 운영을 위한 패키지, 가장 큰 기능은 서버가 에러로 꺼졌을 때 서버를 다시 켜주는 것
- 또 하나의 중요한 기능은 멀티 프로세싱, 멀티 스레딩은 아니지만 멀티 프로세싱을 지원하여 노드프로세스 개수를 한 개 이상으로 늘릴 수 있음
	- 기본적으로는 CPU 코어를 하나만 사용하는데, pm2를 사용해서 프로세스를 여러 개 만들면 다른 코어들까지 사용할 수 있음
	- 클라이언트로부터 요청이 올 때 알아서 요청을 여러 노드 프로세스에 고르게 분배함 => 하나의 프로세스가 받는 부하가 적어지므로 서비스를 더 원활하게 운영할 수 있음
- 단점은 멀티 스레딩이 아니므로 서버의 메모리 같은 자원을 공유하지는 못함
	- 세션을 메모리에 저장하는 경우, 메모리를 공유하지 못해 프로세스 간 세션이 공유되지 않음
	- 이 문제를 극복하기 위해 레디스로 세션을 공유하는 방법을 사용

```bash
npm i pm2
```

- pm2는 nodemon처럼 콘솔에 입력하는 명령어
	- 리눅스나 맥에선 1024 이하 포트를 사용하려면 관리자 권한이 필요(sudo)
	- -i 뒤에 생성하길 원하는 프로세스 개수를 기입, 0은 현재 CPU 코어 개수만큼 프로세스를 생성한다는 뜻
		- -1 은 CPU 코어 개수보다 한 개 덜 생성하겠다는 뜻

```json
  "scripts": {
    "start": "cross-env NODE_ENV=production PORT=80 pm2 start server.js -i 0",
    "dev": "nodemon server",
    "test": "jest"
  },
```

- pm2 종료 시 명령어

```bash
npx pm2 kill
```

- pm2 프로세스 모니터링

```bash
npx pm2 monit
```

- 실제 서버 운영 시, 서비스 규모가 커질수록 비용이 발생할 가능성이 커지므로 놀고 있는 코어까지 클러스터링으로 작동하게 하는 것이 비용을 절약하는 것
	- 하지만 프로세스간 메모리 공유가 안되기 때문에 최대한 프로세스 간 공유하는 것이 없도록 설계해야 함
	- 공유하는 데이터가 있다면 DB를 사용해야 함

### 15.1.6 winston
- 실제 서버를 운영할 때 console.log와 console.error를 대체하기 위한 모듈
	- console 객체의 메서드들은 언제 호출되었는지 파악하기 힘들고, 서버가 종료되면 로그도 사라짐
	- 로그를 파일이나 다른 DB에 저장해야 할 때 winston을 사용함
  
```bash
npm i winston
```

- winston 패키지의 createLogger 메서드로 logger를 만들고 다른 파일에서 사용
	- level - 로그의 심각도
	- format - 로그의 형식
	- transport - 로그 저장 방식

```js
// logger.js
const { createLogger, format, transports } = require('winston');

const logger = createLogger({
  level: 'info',
  format: format.json(),
  transports: [
    new transports.File({ filename: 'combined.log' }),
    new transports.File({ filename: 'error.log', level: 'error' }),
  ],
});

if (process.env.NODE_ENV !== 'production') {
  logger.add(new transports.Console({ format: format.simple() }));
}

module.exports = logger;
```

- winston-daily-rotate-file이란 패키지도 좋다더라(로그를 날짜별로 관리할 수 있게 해주는 패키지)


### 15.1.7 helmet, hpp
- 서버 각종 취약점을 보완해주는 패키지, 익스프레스 미들웨어로 사용가능
- 모든 취약점을 방어해주진 않으므로, 서버를 운영 시 주기적으로 취약점을 점검해줘야 함

```bash
npm i helmet hpp
```

- helmet과 hpp가 방어해주는 취약점 목록은 각 공식문서에 잘 나와있다고 함, 기본적으로 배포 전에 넣어 주는게 좋다고 함
	- 때로 너무 보안 규칙을 엄격하게 적용해서 방해되는 경우도 있으니 공식 문서를 보면서 필요 없는 옵션은 해제해야 함

```js
...
const passport = require('passport');
const helmet = require('helmet');
const hpp = require('hpp');
...
if (process.env.NODE_ENV === 'production') {
  app.enable('trust proxy');
  app.use(morgan('combined'));
  app.use(helmet({ contentSecurityPolicy: false }));
  app.use(hpp());
} else {
  app.use(morgan('dev'));
}
...
```


### 15.1.8 connect-redis
- 멀티 프로세스 간 세션 공유를 위해 레디스와 익스프레스를 연결해주는 패키지, 기존 로그인 시 express-session의 세션 아이디와 실제 사용자 정보가 메모리에 저장됨
	- 때문에 서버가 종료돼 메모리가 날아가면 접속자들의 로그인이 모두 풀리게 됨
	- 이를 해결하기 위해 세션 아이디와 실제 사용자 정보를 DB에 저장, 이 때 사용하는 데이터베이스가 레디스
	- 메모리 기반의 데이터베이스라 성능이 우수해 많이 사용됨

```bash
npm i redis connect-redis
```

- 레디스를 쓰려면 connect-redis 패키지 뿐만 아니라 레디스 데이터베이스를 설치해야 함, 서버에 직접 설치 가능하나 redislabs와 같이 호스팅해주는 서비스를 쓰는것이 편리함
	- 책에서는 redislabs에서 무료 호스팅 서버를 생성해서 사용

```bash
# .env
REDIS_HOST=redis-18954.c92.us-east-1-3.ec2.cloud.redislabs.com
REDIS_PORT=18954
REDIS_PASSWORD=JwTwGgKM4P0OFGStgQDgy2AcXvZjX4dc
```

- connect-redis 패키지로부터 세션을 넣어 RedisStore객체를 require (connect-redis는 express-session에 의존성이 있음)
- redis 패키지 createClient 메서드로 redisClient 객체를 생성
	- url, password 속성에 접속 정보를 입력
	- express-session 미들웨어에 store 옵션을 추가, 기본적으로 메모리에 세션을 저장하지만 이제 RedisStore에 저장함
		- RedisStore의 옵션으로 client 속성에 redisClient 객체를 연결하면 됨
		- 이제 세션 정보가 메모리 대신 레디스에 저장됨(로그인 후 서버를 껐다 켜도 로그인이 유지됨)
			- 실제 서비스에서 서버 업데이트 시 로그인이 풀리는 현상을 막을 수 있음

```js
...
const hpp = require('hpp');
const redis = require('redis');
const RedisStore = require('connect-redis')(session);

dotenv.config();
const redisClient = redis.createClient({
  url: `redis://${process.env.REDIS_HOST}:${process.env.REDIS_PORT}`,
  password: process.env.REDIS_PASSWORD,
});
...
app.use(cookieParser(process.env.COOKIE_SECRET));
const sessionOption = {
  resave: false,
  saveUninitialized: false,
  secret: process.env.COOKIE_SECRET,
  cookie: {
    httpOnly: true,
    secure: false,
  },
  store: new RedisStore({ client: redisClient }),
};
...
```

### 15.1.9 nvm, n
- 노드 버전을 업데이트하기 위한 패키지
	- 윈도에선 nvm-installer를 사용, 리눅스나 맥에선 n 패키지를 사용하면 편함
		- 윈도 
			- nvm list 노드 버전 확인
			- nvm install 버전명 특정 버전 설치
				- latest 입력 시 최신 버전 설치됨
			- nvm use 버전명 설치된 버전 사용
		- 맥, 리눅스
			- n 설치된 버전 확인
			- n 버전명 새 버전을 설치

```bash
sudo npm i -g n
```

- 노드 버전 변경 후 npm 충돌 시 npm rebuild 로 해결 가능
	- 이걸로도 해결 안되면 node_modules 폴더를 제거 후 npm i 로 재설치



## 15.2 깃과 깃허브 사용하기
- 깃 - 분산형 버전 관리 시스템
- 깃허브 - 깃으로부터 업로드한 소스 코드를 서버에 저장할 수 있는 원격 저장소


## 15.3-4 AWS 사용하기, 배포하기
- AWS 계정생성( 왜 안돼?? )
- AWS Management Console - Lightsail 선택, 인스턴스 생성(3.5달러/달, 첫 달 무료)
	- 무료 배포는 heroku.com나 openshift.com 를 이용
- 인스턴스 생성 후 ssh 연결하면 인스턴스와 연결된 콘솔창이 표시됨
	- 깃허브에 올린 프로젝트를 클론해서 내려받고 기존 아파치 서버 종료

```bash
#git clone <프로젝트 url>
git clone https://github.com/아이디/node-deploy

cd /opt/bitnami
sudo ./ctlscript.sh stop apache

cd ~/node-deploy
npm i
npx sequelize db:create --env production
sudo npm start
```

- 지금은 .env가 깃허브에 올라가 있어서 git clone으로 내려받았지만, 실무에선 깃허브에 올리면 안됨
	- .env파일은 서버 내에서 생성해야 함
- 교재에선 웹 서버와 DB 서버를 하나의 인스턴스에서 실행하고 있지만, 나중엔 별도로 분리하는 것이 좋음
	- 서버 하나에 문제가 생겼을 때 다른 서버에 영향을 미치는 것을 막기 위함
	- AWS에선 EC2라는 컴퓨팅 서비스와 MySQL 전용 서비스인 RDS를 따로 운영하므로 이 서비스를 이용하면 됨
- 인스턴스에 표시된 퍼블릭 IP를 입력해서 접속하면 됨
	- https 프로토콜로 접속하기 위해선 도메인 구입 및 인증서 발급 등의 별도 설정이 필요
- 퍼블릭 IP 대신 도메인을 사용하고 싶다면 도메인 판매처에서 도메인을 구입한 후 AWS의 Route 53 서비스에서 연결하면 됨


## 15.5-6 GCP 시작하기, 배포하기
- 구글 클라우드 플랫폼(GCP)은 구글 계정으로 사용 가능
	- GCP 서비스 약관 동의
- 프로젝트(vM 인스턴스) 생성
	- 리전은 us-east1, us-central1, us-west1만 무료(그것도 1세대 f1-micro 인스턴스인 경우에만 해당)
	- 월 합산 730시간 정도만 무료이고 f1-micro인스턴스더라도 나머지 지역을 고른 경우 요금이 청구됨
	- 한국은 asia-northeast3지역, 유료지만 가입 시 받는 크레딧이 있어 당분간 비용을 지불하지 않아도 됨
		- 크레딧 소진 전에 인스턴스를 제거하면 됨
- 무료로 사용하고 싶으면 n1시리즈의 f1-micro인스턴스를 설정할 것(리전도 무료 리전으로 설정)
	- 교재 
		- 머신 구성은 E2 e2-micro로 설정함
		- 운영체제는 Ubuntu 18.04 LTS
		- ID/ API엑세스는 모든 클라우드 api에 대한 전체 엑세스 허용을 선택
		- 마지막으로 방화벽은 http와 https 트래픽을 둘 다 허용
- 인스턴스 생성 후 SSH 연결을 누르면 인스턴스 콘솔로 접근 가능
	- 루트 계정으로 변경
	- AWS에서 배포했던 동일  과정 순서대로 진행
```
sudo su
```

- GCP도 웹서버/ DB 서버를 분리하는게 좋음
	- GCP는 컴퓨트 엔진이라는 컴퓨팅 서비스와 MySQL 전용 서비스인 클라우드 SQL(Cloud SQL)을 따로 운영하므로 이 서비스를 이용하면 됨

- 혹시나 서버가 실행되지 않으면  아래 명령어로 진단 가능

```bash
sudo npx pm2 logs --error
```

- 에러 해결 후 아래 명령어로 재시작

```bash
sudo npx pm2 reload all
```

- GCP에서 배포 후 도메인 사용을 원할 시 GCP 클라우드 DNS(Cloud DNS) 서비스에서 연결하면 됨


## 15.7 함께 보면 좋은자료 
- 생략
