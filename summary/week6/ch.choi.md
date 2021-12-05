# 4장 HTTP 모듈로 서버 만들기

[공식 홈페이지](https://nodejs.org/ko/)  

본 내용에 사용된 책 이미지는 [Node.js 교과서](https://book.naver.com/bookdb/book_detail.nhn?bid=16418778) 이미지를 참고하였습니다.

## 4.1 요청과 응답 이해하기
서버는 클라이언트가 있기에 동작함.  
클라이언트에서 서버로 요청을 보내고, 서버에서는 요청의 내용을 처리한 뒤 클라이언트에 응답  
클라이언트로 부터 요청이 왔을 때 어떤 작업을 수행할지 이벤트리스터를 미리 등록 해야함.

```js
//createserver.js
const http = require('http')

http.createServer((req, res)=>{
    // 여기에 응답을 적음
});

```
요청이 들어올 때마다 매번 콜백 함수가 실행 됨. 

```js
//server1.js
const http = require('http')

http.createServer((req, res)=>{
    
    res.writeHead(200, {'Content-Type':'text/html; charset=utf-8'}); //헤더에 html 형식임을 알림
    res.write('<h1>Hello Node!</h1>'); //응답으로 보낼 데이터
    res.end('<p>Hello Server!</p>'); //end는 응답을 종료하는 메서드 (매개변수 가 있다면 일단 보냄)
}).listen(8080, ()=> {
    //listen 에 입력한 8080 포트로 대기 하겠다고 선언
    console.log('8080번 포트에서 서버 대기중 입니다.');
});
```
![흐름도](https://github.com/ohbokdong/NodeJSStudy/blob/main/summary/week6/flow.PNG)

```js
//cmd
node server1
8080번 포트에서 서버 대기중 입니다.
//웹브라우저에서 http://localhost:8080 또는 127.0.0.1:8080 으로 들어가면 res 의 값을 볼 수 있음
```


### 포트란?
포트는 서버 내 프로세스를 구분하는 번호  
서버는 프로세스의 포트를 다르게 할당 하여 들어오는 요청을 구분.  
유명한 포트 : 21(FTP), 80(http), 443(https), 3306(mysql)

한번에 여러 서버를 실행 할 수도 있음.  
createServer를 원하는 만큼 호출 하면 됨. 단 포트 번호는 틀려야 함.  

```js
const http = require('http')

http.createServer((req, res)=>{
    //...
}).listen(8080);
//localhost:8080로 접속 가능
http.createServer((req, res)=>{
    //...
}).listen(8081);
//localhost:8081로 접속 가능
```
res.write와 res.end 에 일일이 HTML을 적는것은 비효율
미리 HTML파일을 만들어서 전송하면 됨.
아래 코드는 미리 작성된 server2.html 파일을 데이터로 보냄
```js
const http = require('http);
const fs = require('fs').promises;

http.createServer(async(req, res)=> {
    try{
        const data= fs.readFile('./server2.html');
        res.writeHead(200, {'Content-Type':'text/html; charset=utf-8'});\
        res.end(data);
    } catch (err)
    {
        console.error(err);
        res.writeHead(500, {'Content-Type':'text/plain; charset=utf-8'});\
        res.end(err.message);
    }
}).listen(8081);
```

[HTTP 상태 코드](https://developer.mozilla.org/ko/docs/Web/HTTP/Status)  


## 4.2 REST와 라우팅 사용하기

서버에 요청을 보낼 때는 주소를 통해 요청의 내용을 표현  
주소가 /index.html 이면 서버의 index.html을 보내 달라는 뜻이고 about.html이면 about.html을 보내달라는 뜻  

이미지같은 파일을 요청 할 수도 있고 특정 동작을 행하는 것을 요청 할 수도 있음.  
요청의 내용이 주소를 통해 표현되므로 서버가 이해하기 쉬운 주소를 사용하는 것이 좋음. 여기서 REST가 등!장!

REST는 서버의 자원을 정의하고 자원에 대한 주소를 지정하는 방법을 가르킴  
자원이라고 해서 꼭 파일일 필요는 없고 서버가 행할 수 있는 것들을 통들어서 의미함  

주소는 의미를 명확히 전달하기 위해 명사로 구성  
단순히 명사만 있으면 무슨 동작을 행하라는 것인지 알기 어려움 REST에서는 주소 외에도 HTTP 요청 메서드를 사용

GET:서버 자원을 가져오고자 할 때 사용
POST: 서버에 자원을 새로 등록 하고자 할 때 사용
등등등 책 참고

주소 하나가 요청 메서드를 여러 개 가질 수 있음.  (표만들기 귀찮)
[예시]  
GET 메서드의 /user 주소는 사용자 정보를 가져오는 요청
POST 메서드의 /user 주소는 새로운 사용자를 등록 하는 요청

주소와 메서드만 보고 요청의 내용을 알아볼 수 있다는것이 장점

GET메서드 같은 경우에는 브라우저에서 캐싱할 수도 있음. 캐싱이 되면 성능이 좋아짐
(요청을 기다릴 필요가 없기 때문)

HTTP 통신을 사용하면 클라이언트가 누그든 상관없이 같은 방식으로 서버와 소통할 수 있음.  
여러 플랫폼에서 같은 서버에 소통 할 수 있음.  

REESTful 기억하세요.  

```js
//restServer.js

const http = require('http');
const fs - require('fs').promises;
http.creawteSerer(async(req, res)=>
 {
    try
    {
        if(req.method === 'GET') //메서드 타입을 구분 할 수 있음
        {
            if(req.url === '/') // 메서드 타입이 GET이고 주소가 /일때
            {
                const data = await fs.readFile('./restFront.html');
                res.writeHead(200, {'Content-Type':'text/html; charset=utf-8'});
                //헤더 관련해서는 아래에 링크를 참조하세요. 키벨류로 뭘 주는지 확인 가능합니다.
                return res.end(data); //전송 종료와 함께 데이터 보내기
            }
            else if (req.url === '/about')  // 메서드 타입이 GET이고 주소가 /about 일때
            {
                const data = await fs.readFile('./about.html');
                res.writeHead(200, {'Content-Type':'text/thml; charset=utf-8'});
                return res.end(data); //end 에 리던 하는 이유는?? 아래 나와요
            }
            try
            {
                const data = await fs.readFile('${res.rul}'); // /과/about외 다른 파일 요청인가??
                // 요청에 대한 파일이 있다면 전송스~
                return res.end(data);
            }
            catch (err)
            {
                //응 없어~
            }
        }
        //여기까지 온다면 에러임
        res.writeHead(404);
        return res.end('NOT Found');
    }
    catch (err)
    {
        res.writeHead(500,{'Content-Type':'text/plain; charset=utf-8'});
        res.end(err.message);
    }

}).listen(8082);

```
resServer.js가 핵심임.  
HTTP 요청 메서드를 구분하여 처리함, 존재하지 않는 파일이나 에러 상황에 따라 응답을 전송.

## NOTE
res.end 앞에 왜 return을??  
자바스크립트 문법은 return을 안하면 함수가 종료 안됨 (이거 실화임??)  
명시적으로 함수를 종료 시킴. res.end 같은 메서드가 여러번 실행 되면 에러가 발생함

하 너무 길다...
POST 보냈을 때 user면 메모리에 사용자 정보를 등록하고  
PUT 이면 메모리상에서 해당 user정보를 갱신하고  
DELETE를 하면 메모리상에서 해당 user정보를 삭제  

메모리 상에 정보를 저장한다면?? 서버가 맛탱이 가면 정보는 ㅂㅂ 그래서 DB를 이용하여 정보 저장 ㄱㄱ
DB에 접근 하는거 자체가 느림으로 꼭 DB에 넣어야 하는것인가 확인이 필요함.

크롬 개발자 모드에서 네트워크 요청을 실시간으로 볼수 있음.
개발자 모드 말고 여러 프로그램을 쓸 수 있음.

[HTTP 헤더란?](https://developer.mozilla.org/ko/docs/Web/HTTP/Headers)

## 4.3 쿠키와 세션 이해하기

클라이언트에서 보내는 요청에는 큰 단점이 있음. 누가 요청을 보내는지 모른다는 것!  
ip나 브라우저의 정보를 받아 올 수는 있지만 공용 ip, 공용 PC를 사용 할 경우 알 수 없음.

그렇다면?? 로그인을 구현해야 함.  
로그인을 하려면 쿠키와 세션을 알아야함. 웹 사이트에 방문해서 로그인을 할 때 내부적으로 쿠키와 세션을 사용

로그인 후 새로고침을 해도 로그아웃이 되지 않는 이유는 클라이언트가 서버에 여러분이 누군지 지속적으로 알려 주고 있기 때문

서버에 요청을 보낼때 쿠키를 같이 보냄. 쿠키는 유효기간이 있으며 단순한 키-값의 쌍임.

서버는 쿠키를 통해 사용자를 파악 할 수 있음.  
브라우저는 쿠키가 있다면 자동으로 보내주므로 따로 처리할 필요가 없음.
서버에서 브라우저로 쿠키를 보낼 때만 코드를 작성 해야함. 

쿠키를 통해 누구인지 추적이 가능, 개인정보 유출을 방지 하기 위해 쿠키를 지우라는 권고를 하는 이유가 이런 이유 때문임 (쿠키를 훔치려면??)

쿠키는 요청 헤더(Cookie))에 담겨 전송됨. 브라우저는 응답의 헤더(Set-Cookie))에 따라 쿠키를 저장함.


![그림 4-13](https://github.com/ohbokdong/NodeJSStudy/blob/main/summary/week6/4-13.PNG)

자 이제 실습 

```js
//cookie.js


server = http.createServer((req, res)=>{ 
    console.log(`url: ${req.url}, cookie: ${req.headers.cookie}`); // 쿠키가 있는지 확인
    res.writeHead(200, {'Set-Cookie': 'mycookie=test'}); //헤더에 mycookie=test 라는 쿠키를 전송
    res.end('Hellow Cookie');
});

server.listen(8083, ()=>{
    console.log('8083 대기중...');
});
```

```js
//cookie.js 결과
/ undefined // 처음엔 쿠키가 없음 -> 그래서 쿠키를 보냄
/favicon.icon {mycookie:'test'} // 쿠키를 보내서 값이 들어옴
// 총 두번 로그가 찍히는데 이유는 그림 4-13 참고 클라가 두번 요청을 하기 때문임
```

favicon.icon 은 뭐자??

웹사이트 탭에 보이는 이미지를 뜻함 (책 그림 4-14 참고)

첫번째 요청(/)에는 쿠키가 없어서 전송하였고  
두번째 요청(/favicon.icon)은 쿠키가 있어서 쿠키 내용이 있음.

```js


http.createServer(async(req, res) => {
    console.log(`cookie ${req.headers.cookie} , URL : ${req.url}`);
    //쿠키 파싱
    const cookies = parseCookies(req.headers.cookie);

    if(req.url.startsWith('/login')) //login 이라면
    {
        console.log(`req.head ${cookies}`);
        console.log('login');
        const { query } = url.parse(req.url);
        const { name } = qs.parse(query);
        const expires = new Date();
        expires.setMinutes(expires.getMinutes() + 5);
        res.writeHead(302, {
            Location: '/',
            'Set-Cookie': `name=${encodeURIComponent(name)}; Expires=${expires.toGMTString()} HttpOnly; Path=/`,
            //encodeURIComponent 쓰는 이유는 URL은 ASCII만 쓸수 있어서 인코딩을 해줌
            //Expires 을 설정함으로써 쿠키값의 유효 시간을 정함
            // 헤더 옵션은 위에 링크 ㄱㄱ
        });
        res.end();
    }
    else if(cookies.name) // 파싱한 쿠키 에서 name 이 있다면
    {
        res.writeHead(200, {'Content-Type': 'text/html; charset=utf-8'});
        res.end(`${cookies.name}님 안녕하세요.`);
    }
    else //쿠키가 없다면 로그인 페이지로 ㄱㄱ
    {
        try
        {
            console.log('cookies2'); 
            const data = await fs.readFile('./cookies2.html'); // 로그인 페이지 전송
            res.writeHead(200, {'Content-Type': 'text/html; charset=utf-8'});
            res.end(data);            
        }
        catch(err)
        {
            console.log('erro'); // 없다면 에러
            res.writeHead(500, {'Content-Type': 'text/plain; charset=utf-8'});
            res.end(err.message);            
        }
    }
}).listen(8083, ()=>{
    console.log('8083 대기중...');
});

```

쿠키가 있을 경우 브라우저를 새로고침을 해도 로그인이 유지됨. 하지만 위 방법은 상당히 위험함.

쿠키가 외부로 노출되기 때문에 위험함.  
브라우저 -> 개발자모드 -> applicaion 탭에서 쿠키를 볼 수 있음.  
따라서 위 방법으로 민감 정보를 넣어두게되면 유출 ㅂㅂ

![그림 4-15](https://github.com/ohbokdong/NodeJSStudy/blob/main/summary/week6/4-15.PNG)


세션 방법을 이용하여 보안을 강화 하자!

```js
const session = {}; //세션

http.createServer(async(req, res) => {
    console.log(`cookie ${req.headers.cookie} , URL : ${req.url}`);
    const cookies = parseCookies(req.headers.cookie);
    
    if(req.url.startsWith('/login'))
    {
        console.log(`req.head ${cookies}`);
        console.log('login');
        const { query } = url.parse(req.url);
        const { name } = qs.parse(query);
        const expires = new Date();
        expires.setMinutes(expires.getMinutes() + 5);

        const uniqueInt = Date.now(); //특정 사용자 인식을 위한 값
        //사용자에 대한 정보를 저장
        session[uniqueInt] = {
            name,
            expires,
        };
        res.writeHead(302, {
            Location: '/',
            //쿠키를 보내는게 아니라 서버에서 특정 사용자를 인식 할 수 있는 값을 전송
            'Set-Cookie': `session=${uniqueInt}; Expires=${expires.toGMTString()} HttpOnly; Path=/`,
        });
        res.end();
    }
    else if(cookies.session && session[cookies.uniqueInt].expires > new Date())
    {//세션 쿠키가 존재하고 만료 기간이 지나지 않았다면
        res.writeHead(200, {'Content-Type': 'text/html; charset=utf-8'});
        res.end(`${session[cookies.uniqueInt].name}님 안녕하세요.`);
    }
    else
    {
        try
        {
            console.log('cookies2');
            const data = await fs.readFile('./cookies2.html');
            res.writeHead(200, {'Content-Type': 'text/html; charset=utf-8'});
            res.end(data);            
        }
        catch(err)
        {
            console.log('erro');
            res.writeHead(500, {'Content-Type': 'text/plain; charset=utf-8'});
            res.end(err.message);            
        }
    }
}).listen(8083, ()=>{
    console.log('8083 대기중...');
});
```

브라우저 -> 개발자모드 -> applicaion 탭에서 쿠키가 아니라 특정 값이 노출 되는 것을 확인 할 수 있음.

![그림 4-17](https://github.com/ohbokdong/NodeJSStudy/blob/main/summary/week6/4-17.PNG)

세션 아이디는 꼭 쿠키를 사용할 필요는 없음. 많은 사이트가 쿠키를 사용함. 이유는 사용 방법이 제일 간단하기 때문에 사용.  
쿠키를 이용해 세션 아이디를 주고 받는 방식을 세션 쿠키라고 함.

실제 배포용 서버에서는 세션을 위와 같이 변수에 저장하지 않음. 서버가 멈추면 ㅂㅂ 됨

그래서 레디스나 멤캐시드 같은 데이버 베이스에 넣어둠


[레디스란?](https://ko.wikipedia.org/wiki/%EB%A0%88%EB%94%94%EC%8A%A4)
[멤캐시드?](https://ko.wikipedia.org/wiki/Memcached)

절대로 위의 코드를 실제 서비스에 사용해서는 안됨. 위 코드는 보안상 매우 취약함.

안전하게 사용하기 위해서는 다른 사람이 만든 검증된 코드를 사용하는 것이 좋습니다. 6장에서 세션을 처리하는 모듈을 사용해 제대로 된 세션 기능을 도입 할 예정임. 6장님 화이팅

## 4.4 Https와 Http2

Https 모듈은 웹 서버에 SSL 암호화를 추가

GET이나 POST 요청을 할 때 오가는 데이터를 암호화애서 다른 사람이 요청을 가로채더라도 내용을 확인할 수 없게됨.  
요새는 https 적용이 필수가 되는 추세
 
https는 아무나 사용 할 수 있는 것이 아님. 암호화를 적용하는 만큼 인증 해줄 기관이 필요함. 인증서는 기관에서 구입해야 하며 발급 과정이 복잡하고 도메인도 필요함. 인증서 발급은 여기선 안해요~

[SSL 인증서](https://www.digicert.com/kr/ssl-certificate/)

노드의 http2 모듈은 SSL암호화와 더불어 최신 HTTP 프로토콜인 http/2를 사용 할 수 있음.  
http/2는 요청 및 응답 방식이 기존 http/1.1보다 개선되어 훨씬 효율적으로 요청을 보냅니다.

http/2를 사용하면 웹의 속도도 많이 개선됨.
http/1.1도 파이프라인 기술이 적용되어 큰 차이는 나지 않지만 http/2가 훨씬 효율적임


![그림 4-19](https://github.com/ohbokdong/NodeJSStudy/blob/main/summary/week6/4-19.PNG)

[HTTP의 진화](https://developer.mozilla.org/ko/docs/Web/HTTP/Basics_of_HTTP/Evolution_of_HTTP)

```js
//server1-4.js
const http2 = require('http2');
const fs = require('fs');

http2.createSecureServer({
    cert : fs.readFileSync('인증서 경로'),
    key : fs.readFileSync('비밀키 경로'),
    ca: [
        fs.readFileSync('상위 인증서 경로'),
        fs.readFileSync('상위 인증서 경로'),
    ],
}, (req, res)=> {
    // 위 그대로 
}).listen(443); //https 는 443

```

## 4.5 cluster

cluster 모듈은 기본적으로 싱글 프로세스로 동작하는 노드가 CPU 코어를 모두 사용할 수 있게 해주는 모듈.

포트를 공유하는 노드 프로세서를 여러 개 둘 수 있으며 요청이 많이 들어왔을 때 병렬로 실행될 서버의 개수만큼요청이 분산되게 할 수 있음. 서버의 무리가 덜 가게 되는 셈

cluster 를 사용 하면 코어 하나당 노드 프로세스 하나가 돌아가게 할 수 있음. 성능 개선의 효과가 있음 하지만 장점만 있지는 않음. 메모리를 공유 하지 못하는 단점이 있음. 세션을 메모리에 저장하는 경우 문제가 되지만 레디스 등의 서버를 도입하여 해결 할 수 있음.


```js
const cluster = require('cluster');b //클러스터 모듈
const http = require('http');
const { off } = require('process');
const numCPUs = require('os').cpus().length;

if(cluster.isMaster) //메인 프로세스 라면
{
    console.log(`마스터 프로세서 ID ${process.pid}`);
    for(let i = 0; i < numCPUs; i+=1)
    {
        cluster.fork(); //cpu 코어만 큼 포크
    }
    // 각 코어 별로 마무리 동작 정의
    cluster.on('exit', (worker, code, signal)=> {
        console.log(`${worker.process.pid} 번 워커 종료`);
        console.log('code', code, 'signal', signal);
    });
}
else
{
    //워커 프로세스라면 서버 만들기 ㄱㄱ
    http.createServer((req, res)=> {
        res.writeHead(200, {'Context-Type':'text/html; charset=utf-8'});
        res.write('<h1>test</h1>');
        res.end('<p>testtest</p>');
    }).listen(8086); //하나의 포트임
    console.log(`${process.pid} 번 워커 실행`);
}
```


![그림 4-20](https://github.com/ohbokdong/NodeJSStudy/blob/main/summary/week6/4-20.PNG)

work_thread의 예제와 모양이 비슷 하지만 스레드가 아닌 프로세스

클러스터에는 마스터 프로세스와 워커 프로세스가 있음.  마스터 프로세스는 CPU 개수 만큼 프로세스를 생성  
8086포트 에서 대기 요청이 들어오면 만들어진 워커 프로세스에 요청을 분배

오류가 발생해도 서버를 정상 작동 시킬 수 있음. 종료된 워커를 다시 켜면 오류가 발생해도 계속 버틸 수  있음.  

```js
    cluster.on('exit', (worker, code, signal)=> {
        console.log(`${worker.process.pid} 번 워커 종료`);
        console.log('code', code, 'signal', signal);
        cluster.fork(); //오 안돼....
    });
```
위 방식은 좋지 않은 생각임. 근본적인 오류를 해결 하는 것이 좋음. 그래도 예기치 못한 에러로 인한 서버 종료는 막을 수 있음....

실무에서는 pm2등의 모듈로 cluster의 기능을 사용하곤 함 (15장님 화이팅)

REST와 라우팅을 하다보면 코드가 너무 길어짐. 관리하기가 어려워짐에 따라 편리하게 해줄 Express 모듈을 설치 해서 사용하면 됨 그건 다음 장에서 ㄱㄱ (5장님 화이팅)
