# 4장 http 모듈로 서버 만들기

## 4.1. 요청과 응답 이해하기

```node
const http = require("http");

const server = http.createServer((req, res) => {
    res.writeHead(200, {"Content-Type": "text/html; charset=utf-8"});
    res.write("<h1>Hello Node1</h1>");
    res.end("<p>Hello Server</p>");
}).listen(8080);

server.on("listening", () => {
    console.log("8080번 포트에서 대기 중입니다.");
});

server.on("error", (error) => {
    console.err(error);
});
```
- `createServer()` 메서드 뒤에 `listen()` 메서드를 붙여 클라이언트에 개방할 포트 번호 입력
- `on("listening"...)` 이벤트는 서버가 8080포트를 열었을 때 실행됨(오류 발생 시 `on("error", ...)`)
- `res.write()` 또는 `res.end()` 메서드에 html 문서를 담은 `버퍼`를 인자로 넣을 수도 있음

> #### 응답은 무조건 보내야 함
> - `try catch` 문 또는 `error 이벤트 리스너`의 끝에는 `res.end()`를 통해 응답을 보내야 함
>   - 안보내면 `Timeout` 처리됨

#### 4.1.1 HTTP 상태 코드

<table>
    <caption>HTTP 상태 코드</caption>
    <thead>
        <tr>
            <th>코드</th>
            <th>요약</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>2XX</td>
            <td>200(성공), 201(작성)...</td>
        </tr>
        <tr>
            <td>3XX</td>
            <td>리다이렉션(다른 페이지로 이동). 301(영구 이동), 302(임시 이동), 304(요청 응답으로 캐시 사용)...</td>
        </tr>
        <tr>
            <td>4XX</td>
            <td>요청 오류. 400(잘못된 요청), 401(권한 없음), 404(찾을 수 없음)...</td>
        </tr>
        <tr>
            <td>5XX</td>
            <td>요청은 왔지만 서버의 오류. 500(내부 서버 오류), 502(불량 게이트웨이), 503(서비스 사용 불가)...</td>
        </tr>
    </tbody>
</table>

## 4.2. REST와 라우팅 사용하기

- `REST` : `Representational State Transfer`
    - `서버의 자원 정의` 및 `자원에 대한 주소 지정` 방법(일종의 약속)
        - 여기서 `자원` 이란 꼭 파일이어야 하는 것은 아니고 서버가 행할 수 있는 것들을 통틀어 의미함

### 4.2.1. HTTP 요청 메서드 종류

- **GET** : 서버 자원 가져오기. 요청 본문에 데이터 넣으면 안되는데 데이터를 서버로 보내야 한다면 쿼리 스트링 사용
- **POST** : 서버 자원 신규 등록. 요청 본문에 새로 등록할 데이터를 넣어 보냄
- **PUT** : 서버 자원을 요청에 들어 있는 자원으로 치환. 요청 본문에 치환할 데이터 넣어 보냄
- **DELETE** : 서버 자원 삭제. 요청 본문에 데이터를 넣지 않음
- **OPTION** : 요청을 하기 전에 통신 옵션을 설명하기 위해 사용

주소 하나가 요청 메서드를 여러개 가질 수 있다.  GET 메서드의 /user 주소로 요청을 보내면 사용자 정보를 가져오는 요청이라는 것을, POST 메서드의 /user 주소는 새로운 사용자를 등록하려 한다는 것을 알 수 있다. 만약 위의 메서드로 표현하기 애매한 로그인 같은 동작이 있다면 그냥 POST를 사용하면 된다.

- REST 장점
    - 주소와 메서드만 보고 요청의 내용을 알아볼 수 있음
    - 또한 GET 메서드 같은 경우에는 브라우저에서 캐싱(기억)할 수도 있으므로 성능이 좋아짐
    - 그리고 HTTP 통신을 사용하면 클라이언트가 누구든 같은 방식으로 서버와 소통할 수 있음(서버와 클라이언트가 분리되어 있음)
    - REST를 따르는 서버를 RESTful 하다고 표현

## 4.3 쿠키와 세션 이해하기

```node
const http = require('http');

http.createServer((req, res) => {
  console.log(req.url, req.headers.cookie);
  res.writeHead(200, { 'Set-Cookie': 'mycookie=test' });
  res.end('Hello Cookie');
})
  .listen(8083, () => {
    console.log('8083번 포트에서 서버 대기 중입니다!');
  });
```
- `req.headers.cookie` 속성을 통해 클라이언트측 브라우저에 기록된 쿠키 정보에 접근 가능
  - 클라이언트 측에서 따로 쿠키에 대한 처리를 하지않아도 클라이언트측 브라우저가 쿠키 정보를 요청 헤더에 포함시켜 보냄
- `res.writeHead()` 메서드의 첫 번째 인자로 `HTTP 상태 코드`를, 두 번째 인자로 `헤더 옵션` 입력
  - 예제에서는 `Set-Cookie` 옵션을 통해 쿠키를 세팅함

### 4.3.1. Set-Cookie 의 값으로 쓰일 수 있는 것들

<table>
    <caption>Set-Cookie 값</caption>
    <thead>
        <tr>
            <th>값</th>
            <th>설명</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>쿠키명=쿠키값</td>
            <td>기본적인 쿠키의 값</td>
        </tr>
        <tr>
            <td>Expires=날짜</td>
            <td>만료 기한. 이 기한이 지나면 쿠키는 제거됨</td>
        </tr>
        <tr>
            <td>Max-age=초</td>
            <td>해당 초가 지나면 쿠키는 제거됨</td>
        </tr>
        <tr>
            <td>Domain=도메인명</td>
            <td>쿠키가 전송될 도메인을 특정함. 기본값은 현재 도메인</td>
        </tr>
        <tr>
            <td>Path=URL</td>
            <td>쿠키가 전송될 URL을 특정함. 기본값은 '/'이고 기본값일 경우 모든 URL에서 쿠키 전송가능</td>
        </tr>
        <tr>
            <td>Secure</td>
            <td>HTTPS일 경우에만 쿠키 전송</td>
        </tr>
        <tr>
            <td>HttpOnly</td>
            <td>설정 시 자바스크립트에서 쿠키에 접근 불가능. 쿠키 조작 방지</td>
        </tr>
        <tr>
            <td>HttpOnly</td>
            <td>설정 시 자바스크립트에서 쿠키에 접근 불가능. 쿠키 조작 방지</td>
        </tr>
        <tr>
            <td>session</td>
            <td>세션 생성</td>
        </tr>
    </tbody>
</table>

> #### 쿠키에 들어가면 안되는 글자
> - 한글, 줄바꿈 등이 있음
>   - 한글의 경우 `encodeURIComponent`로 감싸서 넣음 

## 4.4 http와 http2

- https 모듈은 웹 서버에 SSL 암호화를 추가해줌
  - GET, POST 요청 시 오가는 데이터를 암호화하여 중간에 다른 사람이 요청을 가로채도 내용을 확인할 수 없게 만듬
- https 모듈을 사용하기 위해서는 암호화 적용을 인증해줄 수 있는 기관으로부터 인증서를 구입해야 함
  - 인증서 발급 과정은 복잡하고 도메인도 필요함

```node
const https = require('https');
const fs = require('fs');

https.createServer({
  cert: fs.readFileSync('도메인 인증서 경로'),
  key: fs.readFileSync('도메인 비밀키 경로'),
  ca: [
    fs.readFileSync('상위 인증서 경로'),
    fs.readFileSync('상위 인증서 경로'),
  ],
}, (req, res) => {
  res.writeHead(200, { 'Content-Type': 'text/html; charset=utf-8' });
  res.write('<h1>Hello Node!</h1>');
  res.end('<p>Hello Server!</p>');
})
  .listen(443, () => {
    console.log('443번 포트에서 서버 대기 중입니다!');
  });
```
- 기존 `http` 모듈과 다르게 `https` 모듈의 `createServer()` 메서드는 인자를 두 개 받음
  - 첫 번째 인수는 인증서에 관련된 옵션 객체
    - 인증서를 구입하면 pem이나 crt, 또는 key 확장자를 가진 파일들을 제공하는데 해당 파일들을 `fs.readFileSync()` 메서드로 읽어서 `cer`, `key`, `ca` 옵션에 알맞게 넣으면 됨
- 노드의 `https2` 모듈은 암호화 뿐만 아니라 최신 HTTP 프로토콜인 `http/2`를 사용할 수 있게함
  - 해당 프로토콜은 요청, 응답 방식이 `http/1.1`보다 개선되어 훨씬 효율적으로 정보를 보냄. 따라서 http/2를 쓰면 웹의 속도도 많이 개선됨(200p)

## 4.5 cluster

- 서버의 코어 개수가 1개 이상이어도 노드는 보통 코어를 하나만 활용함
- `cluster` 모듈을 설정하면 코어 하나당 노드 프로세스 하나가 돌아가게 할 수 있음
- 메모리를 공유하지 못하고, 세션을 메모리에 저장하는 경우 문제가 될 수 도 있다는 단점...
  - ...은 레디스 등의 서버를 도입하여 해결할 수 있음

```node
const cluster = require('cluster');
const http = require('http');
const numCPUs = require('os').cpus().length;

if (cluster.isMaster) {
  console.log(`마스터 프로세스 아이디: ${process.pid}`);
  // CPU 개수만큼 워커를 생산
  for (let i = 0; i < numCPUs; i += 1) {
    cluster.fork();
  }
  // 워커가 종료되었을 때
  cluster.on('exit', (worker, code, signal) => {
    console.log(`${worker.process.pid}번 워커가 종료되었습니다.`);
    console.log('code', code, 'signal', signal);
    cluster.fork();
  });
} else {
  // 워커들이 포트에서 대기
  http.createServer((req, res) => {
    res.writeHead(200, { 'Content-Type': 'text/html; charset=utf-8' });
    res.write('<h1>Hello Node!</h1>');
    res.end('<p>Hello Cluster!</p>');
    setTimeout(() => { // 워커 존재를 확인하기 위해 1초마다 강제 종료
      process.exit(1);
    }, 1000);
  }).listen(8086);

  console.log(`${process.pid}번 워커 실행`);
}
```
- 클러스터에는 `마스터 프로세스`와 `워커 프로세스`가 있음

### 순서
1. 초기에 서버를 실행하면 `마스터`라고 하는 프로세스가 코드를 실행함. `마스터 프로세스`는 서버 CPU 개수만큼 워커 프로세스를 생성(`cluster.fork()` 메서드. 시스템 기반 API)함
2. 마스터 프로세스가 생성한 `워커 프로세스` 또한 생성이 되면 코드를 첫 번째 줄부터 다시 실행하고 워커는 `else`문의 `http.createServer()` 메서드를 실행함

> #### 각각의 워커끼리는 각자의 상태를 공유 하지 않는다
> - 즉 메모리를 공유하지 않는다는 뜻
> - 세션 정보를 `워커1`과 `워커2`가 다르게 들고 있다는 얘기가 됨
>  - 이렇게 되면 로그인을 할때마다 세션이 있을때도 없을때도 있게됨


#### 워커 프로세스가 종료 되었을 때 새로 하나 생성하기
- 권장되는 방식은 아니지만 서버가 종료되는 현상을 방지할 수 있음
```node
...
// 서버야 버텨줘
cluster.on("exit", (worker, code, signal) => {
   cluster.fork();
});
```