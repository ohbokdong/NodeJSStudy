# 10. 웹 API 서버 만들기

## 10.1 API 서버 이해하기

API(Application Programming Interface), 현재 프로그램의 기능을 사용할 수 있게 허용하는 접점을 의미한다.  
웹 API는 다른 웹 서비스의 기능을 사용하거나 자원을 가져올 수 있는 창구로 서버에 API를 올려서 URL을 통해 접근할 수 있게 만든 것을 웹 API서버라고 한다.

- 크롤링(Crawling): 웹 사이트가 자체적으로 제공하는 API가 없는 경우 표면적으로 보이는 웹 사이트의 정보를 일정 주기로 수집해 자체적으로 가공하는 기술, 크롤링 당하면 웹 서버 트래픽 증가하여 서버에 무리가 갈 수 있음

## 10.2 프로젝트 구조 갖추기

프로젝트 목표: 다른 서비스(추후에 만들 NodeDog 서버)에 NodeBird 서비스의 게시글, 해시태그, 사용자 정보를 JSON 형식으로 제공할 것이고, 인증을 받은 사용자에게만 일정한 할당량 안에서 API 호출하도록 허용할 것

- nodebird-api 폴더 만들고 package.json 파일 생성
- view/error.html 생성
- app.js 생성
- models/domain.js 생성
  - 도메일 모델에는 인터넷 주소(host), 도메인 종류(type), 클라이언트 비밀 키(clientSecret)/ API를 사용할 때 필요한 비밀 키가 들어감
  - ENUM이란 넣을 수 있는 값 제한하는 데이터 형식
  - UUID란 충돌 가능성이 매우 적은 랜덤한 문자열
  - 새로 생성한 도메인 모델을 시퀄라이즈와 연결 => 사용자 모델과 일대다 관계를 갖는다, 사용자 한 명이 여러 도메인을 소유할 수도 있기 때문
- models/index.js Domain 내용 추가
- models/user.js 내용 추가
- views/login.html
- routes/index.js
  - 도메인을 등록하는 화면. 로그인 안했으면 로그인 창/ 로그인 했다면 도메인 등록 화면
  - 서버 실행 후 localhost:8001로 접속
  - GET / 접속 시 로그인 화면 보여줌
  - 도메인 등록 라우터에서는 clientSecret값을 uuid 패키지 통해 생성
  - 도메인 등록 이유: 등록한 도메인에서만 API를 사용할 수 있게 하기 위해서
  - 무료와 프리미엄: 추후 사용량 제한을 구현
  - localhost:4000 도메인 등록 => NodeBird API를 사용할 도메인 주소

## 10.3 JWT 토큰으로 인증하기

API 서비스를 제공하는 입장(:8002)  
JWT는 JSON Web Token의 약어, JSON 형식의 데이터를 저장하는 토큰

- 헤더(HEADER): 토큰 종류와 해시 알고리즘 정보
- 페이로드(PAYLOAD): 토큰의 내용물이 인코딩된 부분
- 시그니처(SIGNATURE): 일련의 문자열, 시그니처를 통해 토큰이 변조되었는지 여부 확인 => JWT 비밀 키로 만들어야 함

https://jwt.io 에서 어떤 내용이 담겼는지 확인가능

내용이 노출되는 토큰을 사용하는 이유: 토큰 주인이 누군지, 권한이 무엇인지 데이터베이스를 조회하지않고 알 수 있기 때문에  
**_JWT 토큰은 JWT 비밀 키를 알지 않는 이상 변조가 불가능_**  
단점: 용량이 크다

### JWT 토큰 인증 과정 구현

```
npm i jsonwebtoken
```

- /.env 파일에 JWT_SECRET 생성
- routes/middlewares.js 에 exports.verifyToken 생성
  - 사용자가 쿠키처럼 헤더에 토큰을 넣어 보낼 것, jwt.verify 메서드로 토큰 검증
  - 메서드 첫 번째 인수: 토큰, 메서드 두 번째 인수: 토큰의 비밀 키
  - 유효기간 만료: 419 에러
  - 성공 시 토큰 내용 반환 후 req.decoded에 저장: 사용자 아이디, 닉네임, 발급자, 유효 기간
- routes/v1.js
  - 토큰 발급 라우터(POST /v1/token), 토큰 테스트 라우터(GET /v1/test)
  - 한 번 버전이 정해지면 함부로 수정하면 안됨, 사용자가 이미 존재하기 때문
  - POST /v1/token에서 클라이언트 비밀 키로 도메인 등록된 건지 아닌지 먼저 확인
  - GET /v1/test 는 검증 성공 시 토큰 내용물을 응답으로 보냄
  - JSON 형태에 code, message 속성이 존재하고, 토큰이 있는 경우 token 속성이 존재
- app.js
  - 라우터를 서버에 연결

**_JWT_**  
세션을 사용하지 않고 로그인 할 수 있기 때문에 최근 로그인에 많이 사용하고 있음  
로그인 완료 시 세션에 데이터를 저장하고 세션 쿠키를 발급하는 대신 JWT 토큰을 쿠키로 발급하면 되기 때문에.

```JS
// session을 사용하지 않을 수 있음
passport.authenticate('local', { session: false}, (authError, user, info) => {

})
```

클라이언트에서 JWT를 사용하고 싶다면 RSA 같은 양방향 비대칭 암호화 알고리즘을 사용하면 됨

## 10.4 다른 서비스에서 호출하기

API를 사용하는 서비스도 만들어 보자(Nodecat): nodebird-api를 통해 데이터를 가져오는 것

- /.env : 발급받은 clientSecret 키를 .env에 넣는다
- routes/index.js

  - get /test 라우터는 NodeCat 서비스가 토큰 인증 과정을 테스트해보는 라우터
  - 요청이 왔다 -> 세션에 발급받은 토큰이 저장되어있지 않다? -> POST :8002/v1/token 라우터로부터 토큰 발급 (HTTP 요청의 본문에 클라이언트 비밀 키를 실어 보냄) -> 발급 성공 시 GET :8002/v1/test 접근하여 토큰 유효한지 테스트 (JWT 토큰을 요청 대신 authorization 헤더에 넣는다)

- localohst:4000/test 접속 => NodeCat

## 10.5 SNS API 서버 만들기

- v1.js 포스트와 해시태그 검색 결과를 가져오는 라우터 업데이트
- routes/index.js
  - API에 요청을 보내는 함수 분리
  - GET /mypost 로 자신이 작성한 포스트 JSON형식으로 가져오는 라우터 생성
  - GET /search/:hashtag 라우터는 API를 사용해 해시태그를 검색하는 라우터

## 10.6 사용량 제한 구현하기

과도한 API사용은 서버에 무리를 가게 할 수 있음  
express-rate-limit

- routes/middleweares.js
  - verifyToken 미들웨어 아래에 apiLimiter 미들웨어와 deprecated 미들웨어를 추가
  - apiLimiter: 라우터에 사용량 제한이 걸림: windowMs(기준 시간), max(허용 횟수), delayMs(호출 간격), handler(제한 초과 시 콜백 함수)
  - deprecated: 사용하면 안 되는 라우터에 붙임
- v2.js 추가
  - 토큰 시간 30분으로 늘리고
  - 라우터에 사용량 제한 미들웨어 추가
- v1.js
  - deprecated 미들웨어 추가하여 v1으로 접근한 모든 요청에 deprecated 응답을 보내도록 함
- app.js
  - 새로 만든 라우터를 서버와 연결
- routes/index.js
  - 버전을 v1 -> v2로 바꿈
- 실제 서비스에서 사용량을 저장할 데이터베이스로 레디스가 많이 사용됨

## 10.7 CORS 이해하기

CORS, Cross-Origin Resource Sharing문제 - Access-Control-Allow-Origin에러 발생

브라우저와 서버의 도메인이 일치하지 않을때 기본적으로 요청이 차단 됨  
Access-Control-Allow-Origin 헤더를 넣으면 해결 됨  
cors 패키지를 설치해서 해결 가능

credentials: true -> 도메인 간 쿠키가 공유  
withCredentials: true -> 쿠키를 공유해야 하는 경우

- routes/v2.js
  - 호스트와 비밀 키가 모두 일치할 때만 CORS를 허용하게 수정

**프록시 서버**  
CORS 문제를 해결 할 수 있는 방법. 서버에서 서버로 요청 할 땐 CORS 문제가 발생하지 않는다는 것을 이용한 것.  
`http-proxy-middleware`패키지로 가능

## 10.8 프로젝트 마무리하기

### 핵심 정리

- API는 다른 애플리케이션의 기능을 사용할 수 있게 해주는 창구
- 모바일 서버를 구성할 때 서버를 REST API 방식으로 구현
- API 사용자가 쉽게 사용할 수 있는 문서 준비
- JWT토큰 내용은 공개되면 변조될 수 있다
- 토큰 사용하여 API의 오남용 막기 -> 서버 부하
- app.use외 router.use를 활용하여 라우터 간에 공통되는 로직을 처리할 수 있음
- cors나 passport.authenticate처럼 미들웨어 내에서 미들웨어를 실행할 수 있음
- 브라우저와 서버의 도메인이 다르면 요청이 거절됨(CORS)
