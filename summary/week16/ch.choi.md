# 13장 실시간 경매 시스템 만들기
[공식 홈페이지](https://nodejs.org/ko/)  

본 내용에 사용된 책 이미지는 [Node.js 교과서](https://book.naver.com/bookdb/book_detail.nhn?bid=16418778) 이미지를 참고하였습니다.

이번 장은 배운것을 모두 사용해 웹서비스를 만듬  
서버와 클라이언트, 데이터 베이스가 주고 받는 요청과 응답, 세선, 데이터 흐름등을 주목

## 13.1 프로젝트 구조 갖추기

### BD 모델 구성

프로젝트에서 사용되는 모델은 3개

사용자, 제품, 경매 모델로 구성

![DB 관계도](https://github.com/ohbokdong/NodeJSStudy/blob/main/summary/week6/db.PNG)


### 사용자 인증

로컬 전략을 사용함.


### 프로그램 흐름

경매 시스템은 회원가입, 로그인, 경매 상품 등록, 방 참여, 경매 진행 으로 이뤄짐

라우터는 GET /, GET /join, GET /good, Post /good 으로 이뤄짐

GET /은 메인 화면 렌더링 시 경매 진행중인 상품 목록을 가져옴. soldid가 없는 경우 낙찰자가 없음으로 경매가 진행중인걸로 판단 함


## 13.2 서버센트 이벤트 사용 하기

경매는 시간이 생명 하지만
클라이언트를 믿을 수 없음. 그래서 서버에서 시간을 내려 줄 것임

단방향 통신만 필요하기 때문에 서버센트 이벤트를 사용해봄

웹 소켓은 경매 진행 시 다른 사람 정보를 받고 해야하기 때문에 사용 할 예정 이며 웹 소켓과 서버센트를 같이 사용할 수 있음

``` java
//app.js
const passportConfig = require('./passport');
const sse = require('./sse');
const webSocket = require('./socket');

const server = app.listen(app.get('port'), () => {
  console.log(app.get('port'), '번 포트에서 대기중');
});

webSocket(server, app);
sse(server);
```

```java
//sse.js
const SSE = require('sse');

module.exports = (server) => {
  const sse = new SSE(server);
  sse.on('connection', (client) => { // 서버센트이벤트 연결
  //연결 되었을 때 할 동작 정의
    setInterval(() => {
      client.send(Date.now().toString());
    }, 1000);
  });
};
```

웹소켓 구현 ㄱㄱ

```java
//socket.js
const SocketIO = require('socket.io');

module.exports = (server, app) => {
  const io = SocketIO(server, { path: '/socket.io' });
  app.set('io', io);
  io.on('connection', (socket) => { // 웹 소켓 연결 시
    const req = socket.request;
    const { headers: { referer } } = req;
    const roomId = referer.split('/')[referer.split('/').length - 1];
    socket.join(roomId);
    socket.on('disconnect', () => {
      socket.leave(roomId);
    });
  });
};
```

사용자 정의 네임 스페이스를 안쓰고 기본 네임스페이스(/)로 연결

경매 화면에서 실시간으로 입찰 정보를 올리기 위해 웹 소켓을 사용 

클라 연결 시 주소로 부터 경매방 아이디를 받아와서 socket.join으로 해당 방에 입장

연결이 끊겼다면 socket.leave로 해당방을 나감

서버센트 이벤트의 단점은 IE나 엣지 에서 사용할 수 없음. 그래서 IE 나 엣지 브라우저를 위해 클라 코드에 EventSource 폴리필 을 넣었음

아래 클라 코드가 서버센트를 받아서 실행 하는 부분

```html
<script>
    const es = new EventSource('/sse');
    es.onmessage = function (e) {
      document.querySelectorAll('.time').forEach((td) => {
        const end = new Date(td.dataset.start); // 경매 시작 시간
        const server = new Date(parseInt(e.data, 10));
        end.setDate(end.getDate() + 1); // 경매 종료 시간
        if (server >= end) { // 경매가 종료되었으면
          return td.textContent = '00:00:00';
        } else {
          const t = end - server; // 경매 종료까지 남은 시간
 //...
        }
      });
    };
  </script>
```

서버로 부터 받은 데이터는 e.data에 들어 있음

개발자 도구 -> network 에서  
GET /sse를 클릭해서 EventStream을 보게 되면 데이터를 받는 모습을 볼 수 있음 (해보진 않음.그림 13-4 참고)


라우터에 GET /good/:id 와 Post /goood/:id/bid를 추가 

```java
//routes/index.js

router.get('/good/:id', isLoggedIn, async (req, res, next) => {
  try {
    const [good, auction] = await Promise.all([
      Good.findOne({
        where: { id: req.params.id },
        include: {
          model: User,
          as: 'Owner', // 이부분 주의 할것
        },
      }),
```

상품 모델에 사용자 모델을 include 할때 as 속성을 사용한 것에 주의 해야함. 두번 연결되어 있으므로 어떤 관계를 include 할지 as 속성으로 지정해야함

POST /good/:id/bid 는 클라이언트로 부터 입찰 정보를 저장 함

브라우저 2개 띄워서 확인 ㄱㄱ

## 13.3 스케줄링 구현하기

경매 종료 후 낙찰자가 정해지는 시스템을 구현 해야함

node-schedule 모듈을 이용하여 해결

```java
 schedule.scheduleJob(end, async () => {
      const success = await Auction.findOne({
        where: { GoodId: good.id },
        order: [['bid', 'DESC']],
      });
      await Good.update({ SoldId: success.UserId }, { where: { id: good.id } });
      await User.update({
        money: sequelize.literal(`money - ${success.bid}`),
      }, {
        where: { id: success.UserId },
      });
    });
```
위 코드와 같이 scheduleJob 메서드를 사용 하여
시간과 콜백을 넣으면 됨

node-schedule 패키지의 단점은 스케줄링이 노드 기반으로 작동 하기 때문에 노드가 종료되면 스케줄 예약도 같이 종료됨

노드를 계속 켜두면 되지만 에러로 종료되면 ㅂㅂ

이를 보완하기 위한 방법이 필요함 

낙찰자가 없으면 생성된지 24시간이 지난 경매를  찾아 낙찰자를 정하는 기능을 추가함

checkAuction.js 참고

checkAuction은 서버를 재시작하면 낙찰자를 지정하는 작업을 수행하게끔 app.js 에 추가 함

## 13.4 프로젝트 마무리 하기

마지막으로 낙찰자가 내역을 볼 수 있도록 구성

```java

router.get('/list', isLoggedIn, async (req, res, next) => {
  try {
    const goods = await Good.findAll({
      where: { SoldId: req.user.id },
      include: { model: Auction },
      order: [[{ model: Auction }, 'bid', 'DESC']],
    });
    res.render('list', { title: '낙찰 목록 - NodeAuction', goods });
  } catch (error) {
    console.error(error);
    next(error);
  }
});
```
낙찰된 상품과 입찰 내역을 조회한 후 렌더링 함

입찰 내역은 내림차순으로 정렬하여 낙찰자의 내역이 가장 위에 오도록 함

Note = 운영체제의 스케줄러

node-schedule 패키지로 등록한 스케줄은 노드 서버가 종료될때 같이 종료되지만 단점이 있음
이를 극복하려면 운영체제의 스케줄러를 사용하는 것이 좋음 윈도우에서는 schtasks, 맥과 리눅스는 cron이 대표적임 노드에서 child_process를 통해 호출 할 수 있음

### 13.4.2 핵심 정리

- 서버에서 클라이언트로 보내는 일방향 통신은 웹 소켓 대신 서버센트 이벤트를 사용해도 된다.
- 기존 입찰 내역은 데이터베이스에서 불러오고, 방 참여 후에 추가되는 내역은 웹 소켓에서 불러 옵니다. 이 둘은 매끄럽게 연결하는 방법을 기억해 둡시다.
- 코드가 길어질 것 같으면 app.js 로부터 분리 합시다.
- 사용자의 입력값은 프런트엔트 와 백엔드 모두에서 체크하는 게 좋습니다.
- 스케줄링을 통해 주기적으로 일어나는 작업을 처리할 수 있지만 노드 서버가 계속 켜져 있어야만 하므로 노드 서버가 꺼졌을 때 대처 할 방법을 마련해야 합니다.

### 13.4.3 함께 보면 좋은 자료
(책 ㄱㄱ)

