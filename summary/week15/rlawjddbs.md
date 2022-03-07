# 12장 웹 소켓으로 실시간 데이터 전송하기

## 12.1. 웹 소켓 이해하기

### 웹 소켓

- HTML 5에 새로 추가된 스펙
- 실시간 양방향 데이터 전송을 위한 기술
- HTTP와 다르게 WS 프로토콜 사용
  - 브라우저와 서버가 WS 프로토콜 지원 시 사용 가능


### 웹 소켓 특징

- 최초 웹 소켓 연결이 이뤄지면 그 후 계속 연결 상태 유지 가능
- 업데이트할 내용을 서버에서 바로 클라이언트에 알림
- HTTP 프로토콜과 포트 공유 - 다른 포트에 연결할 필요가 없음


### 웹 소켓 외 데이터 전송 방식

![폴링 vs. SEE vs. 웹 소켓](https://github.com/ohbokdong/NodeJSStudy/blob/main/summary/week15/type.png)  
  
1. 폴링(polling)
   - 클라이언트에서 서버로 주기적으로 새로운 업데이트가 있는지 확인하는 요청을 보냄

2. 서버센트 이벤트(Server Sent Events)
   - `EventSource` 라는 객체 사용
   - 최초 연결 후 서버가 클라이언트에 지속적으로 데이터를 보냄
   - 단방향 통신 - 클라이언트가 서버로 데이터를 보낼 수 없음
   - 주식 차트 업데이트, SNS 등에서 사용


## 12.2. ws 모듈로 웹 소켓 사용하기

<details>
  <summary>1. package.json</summary>

```json
{
  "name": "gif-chat",
  "version": "0.0.1",
  "description": "GIF 웹소켓 채팅방",
  "main": "app.js",
  "scirpts": {
    "start": "nodemon app"
  },
  "autor": "dbs",
  "license": "ISC",
  "dependencies": {
    "cookie-parser": "^1.4.5",
    "dotenv": "^8.2.0",
    "express": "^4.17.1",
    "express-session": "^1.17.1",
    "morgan": "^1.10.0",
    "nunjucks": "^3.2.1",
    "ws": "8.5.0"
  },
  "devDependencies": {
    "nodemon": "^2.0.3"
  }
}
```
</details>

<details>
  <summary>2. 콘솔</summary>

```shell
$ npm i
```
</details>

<details>
  <summary>3. .env</summary>

```properties
COOKIE_SECRET=gifchat
```
</details>

<details>
  <summary>4. app.js</summary>

```js
const express = require('express');
const path = require('path');
const morgan = require('morgan');
const cookieParser = require('cookie-parser');
const session = require('express-session');
const nunjucks = require('nunjucks');
const dotenv = require('dotenv');

dotenv.config();
const webSocket = require('./socket');
const indexRouter = require('./routes');

const app = express();
app.set('port', process.env.PORT || 8005);
app.set('view engine', 'html');

nunjucks.configure('views', {
    express: app,
    watch: true
});

app.use(morgan('dev'));
app.use(express.static(path.join(__dirname, 'public')));
app.use(express.json());
app.use(express.urlencoded({extended: false}));
app.use(cookieParser(process.env.COOKIE_SECRET));
app.use(session({
    resave: false,
    saveUninitialized: false,
    secret: process.env.COOKIE_SECRET,
    cookie: {
        httpOnly: true,
        secure: false
    }
}));

app.use('/', indexRouter);

app.use((req, res, next) => {
    const error = new Error(`${req.method} ${req.url} 라우터가 없습니다.`);
    error.status = 404;
    next(error);
});

app.use((err, req, res, next) => {
    res.locals.message = err.message;
    res.locals.error = process.env.NODE_ENV !== 'production' ? err : {};
    res.status(err.status || 500);
    res.render('error');
});

const server = app.listen(app.get('port'), () => {
    console.log(app.get('port'), '번 포트에서 대기 중');
});

webSocket(server);
```
</details>

<details>
  <summary>5. routes/index.js</summary>

```js
const express = require('express');

const router = express.Router();

router.get('/', (req, res) => {
    res.render('index');
});

module.exports = router;
```
</details>

<details>
   <summary>6. socket.js - 서버 로직</summary>

```js
const WebSocket = require('ws');

module.exports = (server) => {

   const wss = new WebSocket.Server({ server });

   wss.on('connection', (ws, req) => {

      const ip = req.headers['x-forwarded-for'] || req.connection.remoteAddress;

      console.log('새로운 클라이언트 접속', ip);

      ws.on('message', (message) => {
         console.log(message);
      });

      ws.on('error', (error) => {
         console.log(error);
      });

      ws.on('close', () => {
         console.log('클라이언트 접속 해제', ip);
         clearInterval(ws.interval);
      });

      ws.interval = setInterval(() => { // 3초마다 클라이언트로 메시지 전송
         if (ws.readyState === ws.OPEN) {
            ws.send('서버에서 클라이언트로 메시지를 보냅니다.');
         }
      }, 3000);

   });

};
```
</details>

<details>
   <summary>7. views/index.html - 클라이언트 로직</summary>
   
```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <meta name="viewport"
          content="width=device-width, user-scalable=no, initial-scale=1.0, maximum-scale=1.0, minimum-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>GIF 채팅방</title>
</head>
<body>
    <div>F12를 눌러 console 탭과 network 탭을 확인하세요.</div>
    <script>
       
        const webSocket = new WebSocket('ws://localhost:8005');
        
        // 서버와 연결이 맺어지는 경우
        webSocket.onopen = function() {
            console.log('서버와 웹소켓 연결 성공');
        };
        
        // 서버로부터 메시지가 오는 경우
        webSocket.onmessage = function (event) {
            console.log(event.data);
            webSocket.send('클라이언트에서 서버로 답장을 보냅니다.');
        }
       
    </script>
</body>
</html>
```
</details>

## 12.3. Socket.IO 사용하기

- 이전 절의 ws 패키지는 간단하게 웹 소켓을 사용하고자 할 때 좋음
- 구현할 서비스가 복잡해지면 `Socket.IO` 사용 권장
  - 편의 기능이 많이 추가되어 있음

<details>
   <summary>1. 콘솔</summary>

```shell
$ npm i socket.io@2
```
</details>

<details>
    <summary>2. socket.js</summary>

```js
const SocketIO = require('socket.io');

module.exports = (server) => {

    const io = SocketIO(server, { path: '/socket.io' });

    io.on('connection', (socket) => { // 웹 소켓 연결 시

        const req = socket.request;
        const ip = req.headers['x-forwarded-for'] || req.connection.remoteAddress;
        console.log('새로운 클라이언트 접속!', ip, socket.id, req.ip);

        // ---------------------
        // init event
        // ---------------------
        // 소켓 연결 해제 시 event
        socket.on('disconnect', () => {
            console.log('클라이언트 접속 해제', ip, socket.id);
            clearInterval(socket.interval);
        });

        // 오류 발생 시 event
        socket.on('error', (error) => {
            console.error(error);
        });

        // 클라이언트 메시지 수신 event
        socket.on('reply', (data) => {
            console.log(data);
        });

        // 3초마다 클라이언트로 메시지 전송
        socket.interval = setInterval(() => {
            // .emit('이벤트명', 전송할 데이터);
            socket.emit('news', 'Hello Socket.IO');
        }, 3000);

    });

};
```

- `socket.io` 패키지를 불러와 익스프레스 서버와 연결
- `SocketIO` 객체의 두 번째 인수로 옵션 객체를 넣어 서버에 관한 여러 가지 설정을 할 수 있음
- 여기서는 클라이언트가 접속할 경로인 `path` 옵션만 사용
  - 클라이언트에서도 동일한 path를 사용해야 함
- `connection` 이벤트는 클라이언트가 접속했을 때 발생, 콜백으로 소켓 객체(`socket`) 반환
- `socket.request` - 요청 객체에 접근 가능한 속성
- `socket.request.res` - 응답 객체에 접근 가능한 속성
- `socket.id` - 소켓 고유의 아이디로 소켓의 주인이 누구인지 특정할 수 있음
- `reply`는 클라이언트 측에서 직접 만든 이벤트 (사용자가 발생시킨 이벤트)

> 이런식으로 이벤트명을 사용하는 것이 ws 모듈과는 다른 부분

</details>

<details>
    <summary>3. views/index.html</summary>

```html
<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport"
          content="width=device-width, user-scalable=no, initial-scale=1.0, maximum-scale=1.0, minimum-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>GIF 채팅방</title>
</head>
<body>
    <div>F12를 눌러 console 탭과 network 탭을 확인하세요.</div>
    <script src="/socket.io/socket.io.js"></script>
    <script>
        const socket = io.connect('http://localhost:8005', {
            path: '/socket.io',
            // transports: ['websocket']
        });

        socket.on('news', function(data) {
           console.log(data);
           socket.emit('reply', 'Hello Node.JS');
        });
    </script>
</body>
</html>
```

- `/socket.io/socket.io.js`는 `Socket.io`에서 클라이언트로 제공하는 스크립트(실제 파일 X)
- 이 스크립트를 통해 서버와 유사한 API로 웹 소켓 통신 가능
- Socket.IO는 먼저 폴링 방식으로 서버와 연결하고 웹 소켓을 사용할 수 있다면 웹 소켓으로 업그레이드 함
- 웹 소켓을 지원하지 않는 브라우저는 폴링 방식으로, 웹 소켓을 지원하는 브라우저는 웹 소켓 방식으로 사용 가능
- 처음부터 웹 소켓만 사용하고 싶다면, 위 소스 주석 처리된 부분을 주석 해제 하면 됨
</details>

## 12.4. ~ 12.7. 실시간 GIF 채팅방 만들기

- `몽고디비`, 몽고디비 ODM `몽구스` 사용
- [소스 너무 길다](https://github.com/ohbokdong/NodeJSStudy/tree/main/summary/week15/socket.io)

<del>
<details>
    <summary>1. 콘솔</summary>

```shell
$ npm i mongoose multer axios color-hash cookie-signature
```

- color-hash - 랜덤 색상 구현 모듈
- cookie-signature - 쿠키 암호화 패키지
</details>
</del>

<del>
<details>
    <summary>2. schemas/room.js - 채팅방 스키마</summary>

```js
const mongoose = require('mongoose');

const { Schema } = mongoose;

const roomSchema = new Schema({
    title: {
        type: String,
        require: true
    },
    max: {
        type: Number,
        required: true,
        default: 10,
        min: 2
    },
    owner: {
        type: String,
        required: true
    },
    password: String,
    createdAt: {
        type: Date,
        default: Date.now
    }
});

module.exports = mongoose.model('Room', roomSchema);
```

- 방 제목(title), 최대 수용 인원(max), 방장(owner), 비밀번호(password), 생성 시간(createdAt)
- 수용 인원은 기본적으로 10명, 최소 인원은 2명 이상으로 설정
</details>
</del>

<del>
<details>
    <summary>3. schemas/chat.js - 채팅 스키마</summary>

```js
const mongoose = require('mongoose');

const { Schema } = mongoose;
const { Type: { ObjectId} } = Schema;
const chatSchema = new Schema({
    room: {
        type: ObjectId,
        required: true,
        ref:'Room'
    },
    user: {
        type: String,
        required: true
    },
    chat: String,
    gif: String,
    createdAt: {
        type: Date,
        default: Date.now
    }
});

module.exports = mongoose.model('Chat', chatSchema);
```

- 채팅방 아이디(room), 채팅 한 사람(user), 채팅 내역(chat), GIF 이미지 주소(img), 채팅 시간(createdAt)
- room 필드는 Room 스키마와 연결, Room 컬렉션의 ObjectId가 들어가게 됨
</details>
</del>

<del>
<details>
    <summary>4. schemas/index.js - 몽고디비와 연결</summary>

```js
const mongoose = require('mongoose');

const { MONGO_ID, MONGO_PASSWORD, NODE_ENV } = process.env;
const MONGO_URL = `mongodb://${MONGO_ID}:${MONGO_PASSWORD}@localhost:27017/admin`;

const connect = () => {
    if (NODE_ENV !== 'production') {
        mongoose.set('debug', true);
    }
    mongoose.connect(MONGO_URL, {
        dbName: 'gifchat',
        useNewUrlParser: true,
        useCreateIndex: true
    }, (error) => {
        if (error) {
            console.log('몽고디비 연결 에러', error);
        } else {
            console.log('몽고디비 연결 성공');
        }
    });
};

mongoose.connection.on('error', (error) => {
    console.error('몽고디비 연결 에러', error);
});

mongoose.connection.on('disconnected', () => {
    console.error('몽고디비 연결이 끊겼습니다. 연결을 재시도 합니다.');
    connect();
});

module.export = connect;
```
</details>
</del>

<del>
<details>
    <summary>5. .env</summary>

```properties
COOKIE_SECRET=gifchat
MONGO_ID=root
MONGO_PASSWORD=nodejsbook
```
</details>
</del>

<del>
<details>
    <summary>6. app.js - 서버 실행 시 몽고디비에 바로 접속</summary>

```js
const express = require('express');
const path = require('path');
const morgan = require('morgan');
const cookieParser = require('cookie-parser');
const session = require('express-session');
const nunjucks = require('nunjucks');
const dotenv = require('dotenv');

dotenv.config();
const webSocket = require('./socket');
const indexRouter = require('./routes');
const connect = require('./schemas');

const app = express();
app.set('port', process.env.PORT || 8005);
app.set('view engine', 'html');

nunjucks.configure('views', {
    express: app,
    watch: true
});
connect();

app.use(morgan('dev'));
app.use(express.static(path.join(__dirname, 'public')));
app.use(express.json());
app.use(express.urlencoded({extended: false}));
app.use(cookieParser(process.env.COOKIE_SECRET));
app.use(session({
    resave: false,
    saveUninitialized: false,
    secret: process.env.COOKIE_SECRET,
    cookie: {
        httpOnly: true,
        secure: false
    }
}));

app.use(session({
    resave: false,
    saveUninitialized: false,
    secret: process.env.COOKIE_SECRET,
    cookie: {
        httpOnly: true,
        secure: false
    }
}));

app.use((req, res, next) => {
    if (!req.session.color) {
        const colorHash = new ColorHash();
        req.session.color = ColorHash.hex(req.sessionID);
    }
    next();
});

app.use('/', indexRouter);

app.use((req, res, next) => {
    const error = new Error(`${req.method} ${req.url} 라우터가 없습니다.`);
    error.status = 404;
    next(error);
});

app.use((err, req, res, next) => {
    res.locals.message = err.message;
    res.locals.error = process.env.NODE_ENV !== 'production' ? err : {};
    res.status(err.status || 500);
    res.render('error');
});

const server = app.listen(app.get('port'), () => {
    console.log(app.get('port'), '번 포트에서 대기 중');
});

webSocket(server, app);
```
</details>
</del>

<del>
<details>
    <summary>7. views/layout.html</summary>

```html
<!doctype html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport"
          content="width=device-width, user-scalable=no, initial-scale=1.0, maximum-scale=1.0, minimum-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>{{title}}</title>
    <link rel="stylesheet" href="/main.css">
</head>
<body>
    {% block content %}
    {% endblock %}
    {% block script %}
    {% endblock %}
</body>
</html>
```
</details>
</del>

<del>
<details>
    <summary>8. public/main.css</summary>

```css
* { box-sizing: border-box; }
.mine { text-align: right; }
.system { text-align: center; }
.mine img, .other img {
    max-width: 300px;
    display: inline-block;
    border: 1px solid silver;
    border-radius: 5px;
    padding: 2px 5px;
}
.mine div:first-child, .other div:first-child { font-size: 12px; }
.mine div:last-child, .other div:last-child {
    display: inline-block;
    border: 1px solid silver;
    border-radius: 5px;
    padding: 2px 5px;
    max-width: 300px;
}
#exit-btn { position: absolute; top: 20px; right: 20px; }
#chat-list { height: 500px; overflow: auto; padding: 5px; }
#chat-form { text-align: right; }
label[for='gif'], #chat, #chat-form [type='submit'] {
    display: inline-block;
    height: 30px;
    vertical-align: top;
}
label[for='gif'] { cursor: pointer; padding: 5px; }
#gif { display: none; }
table, table th, table td {
    text-align: center;
    border: 1px solid silver;
    border-collapse: collapse;
}
```
</details>
</del>

<del>
<details>
    <summary>9. views/main.html</summary>

```html
{% extends 'layout.html' %}

{% block content %}
<h1>GIF 채팅방</h1>
<fieldset>
  <legend>채팅방 목록</legend>
  <table>
    <thead>
    <tr>
      <th>방 제목</th>
      <th>종류</th>
      <th>허용 인원</th>
      <th>방장</th>
    </tr>
    </thead>
    <tbody>
    {% for room in rooms %}
      <tr data-id="{{room._id}}">
        <td>{{room.title}}</td>
        <td>{{'비밀방' if room.password else '공개방'}}</td>
        <td>{{room.max}}</td>
        <td style="color: {{room.owner}}">{{room.owner}}</td>
        <td>
          <button
            data-password="{{'true' if room.password else 'false'}}"
            data-id="{{room._id}}"
            class="join-btn"
          >입장
          </button>
        </td>
      </tr>
    {% endfor %}
    </tbody>
  </table>
  <div class="error-message">{{error}}</div>
  <a href="/room">채팅방 생성</a>
</fieldset>
<script src="/socket.io/socket.io.js"></script>
<script>
  const socket = io.connect('http://localhost:8005/room', { // 네임스페이스
    path: '/socket.io',
  });

  socket.on('newRoom', function (data) { // 새 방 이벤트 시 새 방 생성
    const tr = document.createElement('tr');
    let td = document.createElement('td');
    td.textContent = data.title;
    tr.appendChild(td);
    td = document.createElement('td');
    td.textContent = data.password ? '비밀방' : '공개방';
    tr.appendChild(td);
    td = document.createElement('td');
    td.textContent = data.max;
    tr.appendChild(td);
    td = document.createElement('td');
    td.style.color = data.owner;
    td.textContent = data.owner;
    tr.appendChild(td);
    td = document.createElement('td');
    const button = document.createElement('button');
    button.textContent = '입장';
    button.dataset.password = data.password ? 'true' : 'false';
    button.dataset.id = data._id;
    button.addEventListener('click', addBtnEvent);
    td.appendChild(button);
    tr.appendChild(td);
    tr.dataset.id = data._id;
    document.querySelector('table tbody').appendChild(tr); // 화면에 추가
  });

  socket.on('removeRoom', function (data) { // 방 제거 이벤트 시 id가 일치하는 방 제거
    document.querySelectorAll('tbody tr').forEach(function (tr) {
      if (tr.dataset.id === data) {
        tr.parentNode.removeChild(tr);
      }
    });
  });

  function addBtnEvent(e) { // 방 입장 클릭 시
    if (e.target.dataset.password === 'true') {
      const password = prompt('비밀번호를 입력하세요');
      location.href = '/room/' + e.target.dataset.id + '?password=' + password;
    } else {
      location.href = '/room/' + e.target.dataset.id;
    }
  }

  document.querySelectorAll('.join-btn').forEach(function (btn) {
    btn.addEventListener('click', addBtnEvent);
  });
</script>
{% endblock %}

{% block script %}
<script>
  window.onload = () => {
    if (new URL(location.href).searchParams.get('error')) {
      alert(new URL(location.href).searchParams.get('error'));
    }
  };
</script>
{% endblock %}
```

- `io.connect` 메서드의 주소 뒤에 `/room` 을 붙임
  - 네임스페이스라고 부르며, 서버에서 /room 네임스페이스를 통해 보낸 데이터만 받을 수 있음
- `socket`에는 미리 `newRoom`과 `removeRoom` 이벤트를 달아 둠
  - 서버에서 웹 소켓으로 해당 이벤트를 발생시키면 이벤트 리스너의 콜백 함수가 실행
</details>

<details>
    <summary>10. views/room.html</summary>

```html
<!doctype html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport"
          content="width=device-width, user-scalable=no, initial-scale=1.0, maximum-scale=1.0, minimum-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>{{title}}</title>
    <link rel="stylesheet" href="/main.css">
</head>
<body>
    {% block content %}
    {% endblock %}
    {% block script %}
    {% endblock %}
</body>
</html>
```
</details>
</del>

<del>
<details>
    <summary>11. views/chat.html</summary>

```html
{% extends 'layout.html' %}

{% block content %}
<h1>{{title}}</h1>
<a href="/" id="exit-btn">방 나가기</a>
<fieldset>
    <legend>채팅 내용</legend>
    <div id="chat-list">
        {% for chat in chats %}
        {% if chat.user === user %}
        <div class="mine" style="color: {{chat.user}}">
            <div>{{chat.user}}</div>
            {% if chat.gif %}}
            <img src="/gif/{{chat.gif}}">
            {% else %}
            <div>{{chat.chat}}</div>
            {% endif %}
        </div>
        {% elif chat.user === 'system' %}
        <div class="system">
            <div>{{chat.chat}}</div>
        </div>
        {% else %}
        <div class="other" style="color: {{chat.user}}">
            <div>{{chat.user}}</div>
            {% if chat.gif %}
            <img src="/gif/{{chat.gif}}">
            {% else %}
            <div>{{chat.chat}}</div>
            {% endif %}
        </div>
        {% endif %}
        {% endfor %}
    </div>
</fieldset>
<form action="/chat" id="chat-form" method="post" enctype="multipart/form-data">
    <label for="gif">GIF 올리기</label>
    <input type="file" id="gif" name="gif" accept="image/gif">
    <input type="text" id="chat" name="chat">
    <button type="submit">전송</button>
</form>
<script src="/socket.io/socket.io.js"></script>
<script>
    const socket = io.connect('http://localhost:8005/chat', {
        path: '/socket.io',
    });
    socket.on('join', function (data) {
        const div = document.createElement('div');
        div.classList.add('system');
        const chat = document.createElement('div');
        div.textContent = data.chat;
        div.appendChild(chat);
        document.querySelector('#chat-list').appendChild(div);
    });
    socket.on('exit', function (data) {
        const div = document.createElement('div');
        div.classList.add('system');
        const chat = document.createElement('div');
        div.textContent = data.chat;
        div.appendChild(chat);
        document.querySelector('#chat-list').appendChild(div);
    });
</script>
{% endblock %}
```

- mine - 내 메시지, system - 시스템 메시지, other - 남의 메시지
- `/chat` 네임스페이스로 보낸 데이터만 받을 수 있음
- `socket`에 `join`, `exit` 이벤트 리스너를 연결하여 사용자의 입장과 퇴장에 관한 데이터가 웹 소켓으로 전송될 대 호출되도록 함

</details>
</del>

<del>
<details>
    <summary>12. socket.js</summary>

```js
const SocketIO = require('socket.io');
const axios = require('axios');
const cookieParser = require('cookie-parser');
const cookie = require('cookie-signature');

module.exports = (server, app, sessionMiddleware) => {

    const io = SocketIO(server, {path: '/socket.io'});
    app.set('io', io);
    const room = io.of('/room');
    const chat = io.of('/chat');

    io.use((socket, next) => {
        cookieParser(process.env.COOKIE_SECRET)(socket.request, socket.request.res, next);
        sessionMiddleware(socket.request, socket.request.res, next);
    });

    // ---------------------
    // init event
    // ---------------------
    // 채팅방 목록 연결
    room.on('connection', (socket) => {
        console.log('room 네임스페이스에 접속');
        socket.on('disconnect', () => {
            console.log('room 네임스페이스 접속 해제');
        });
    });

    // 채팅방 연결
    chat.on('connection', (socket) => {
        console.log('chat 네임스페이스에 접속');
        const req = socket.request;
        const { headers: { referer } } = req;
        const roomId = referer
            .split('/')[referer.split('/').length - 1]
            .replace(/\?.+/, '');
        socket.join(roomId);

        // ---------------------
        // join message
        // ---------------------
        socket.to(roomId).emit('join', {
            user: 'system',
            chat: `${req.session.color}님이 입장하셨습니다.`
        });

        socket.on('disconnect', () => {
            console.log('chat 네임스페이스 접속 해제');
            socket.leave(roomId);

            // ---------------------
            // exit message
            // ---------------------
            const currentRoom = socket.adapter.rooms[roomId];
            const userCount = currentRoom ? currentRoom.length : 0;

            if (userCount === 0) { // 접속자 0명일 경우 방 삭제
                const signedCookie = req.signedCookies['connect.sid'];
                const connectSID = cookie.sign(signedCookie, process.env.COOKIE_SECRET);
                axios.delete(`http://localhost:8005/room/${roomId}`, {
                    headers: {
                        Cookie: `connect.sid=s%3A${connectSID}`
                    }
                })
                    .then(() => {
                        console.log('방 제거 요청 성공');
                    })
                    .catch((error) => {
                        console.error(error);
                    });
            } else {
                socket.to(roomId).emit('exit', {
                    user:'system',
                    chat: `${req.session.color}님이 퇴장하셨습니다.`
                });
            }

            socket.to(roomId).emit('join', {
                user: 'system',
                chat: `${req.session.color}님이 입장하셨습니다.`
            });
        });
    });

    // ---------------------
    // init event
    // ---------------------
    // 웹 소켓 연결 시 event
    io.on('connection', (socket) => {

        const req = socket.request;
        const ip = req.headers['x-forwarded-for'] || req.connection.remoteAddress;
        console.log('새로운 클라이언트 접속!', ip, socket.id, req.ip);

        // 소켓 연결 해제 시 event
        socket.on('disconnect', () => {
            console.log('클라이언트 접속 해제', ip, socket.id);
            clearInterval(socket.interval);
        });

        // 오류 발생 시 event
        socket.on('error', (error) => {
            console.error(error);
        });

        // 클라이언트 메시지 수신 event
        socket.on('reply', (data) => {
            console.log(data);
        });

        // 3초마다 클라이언트로 메시지 전송
        socket.interval = setInterval(() => {
            socket.emit('news', 'Hello Socket.IO');
        }, 3000);

    });

};
```

1. `app.set('io', io)`로 라우터에서 io 객체를 쓸 수 있게 저장
    - `req.app.get('io')`로 접근 가능
2. `of` 메서드는 `Socket.IO`에 네임스페이스를 부여하는 메서드
    - 기본적으로 Socket.IO는 `/` 네임스페이스에 접속하지만 of 메서드를 사용하면 다른 네임스페이스를 만들어 접속 가능
      - 같은 네임스페이스끼리만 데이터 전달
3. 네임스페이스마다 각각 이벤트 리스너를 붙일 수 있음
4. `socket.join`, `socket.leave` 는 각각 `방`에 들어가고 방에서 나가는 메서드

> ### Socket.IO의 방(room) 개념
> ![네임스페이스의 방](https://github.com/ohbokdong/NodeJSStudy/summary/week15/room.png)   
> Socket.IO의 네임스페이스보다 세부적인 개념으로 같은 네임스페이스 안에서도 같은 방에 들어 있는 소켓끼리만 데이터를 주고받을 수 있다.
> `socket.request.headers.referer`를 통해 현재 웹 페이지의 URL을 가져올 수 있는데, `split`, `replace` 메서드를 사용해 URL에서 방 아이디 부분을 추출했다.

5. `io.use` 메서드에 미들웨어를 장착할 수 있으며 이 부분은 모든 웹 소켓 연결 시마다 실행 됨
6. `socket.to(방아이디)` 메서드로 특정 방에 데이터를 보낼 수 있음

</details>
</del>
