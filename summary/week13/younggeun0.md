# 노드 JS 스터디 10장 웹 API 서버 만들기

## 10장 핵심 정리
* API는 다른 애플리케이션의 기능을 사용할 수 있게 해주는 창구
* 모바일 서버를 구성할 때 서버를 REST API 방식으로 구현하면 됨
* API 사용자가 API를 쉽게 사용할 수 있도록 사용 방법, 요청 형식, 응답 내용에 관한 문서를 준비할 것
* JWT 토큰의 내용은 공개되며 변조될 수 있다는 것을 기억할 것, 단, 시그니처를 확인하면 변조여부를 체크가능
* 토큰을 사용하여 API의 오남용을 막음, 요청 헤더에 토큰이 있는지 항상 확인하는 것이 좋음
* app.use 외에도 router.use를 활용하여 라우터 간에 공통되는 로직을 처리 가능
* cors나 passport.authenticate처럼 미들웨어 내에서 미들웨어를 실행 가능, 미들웨어를 선택적으로 적용하거나 커스터마이징할 때 이 기법을 사용
* 브라우저와 서버의 도메인이 다르면 요청이 거절된다는 특성(CORS)을 이해할 것, 서버와 서버 간 요청에서는 CORS 문제가 발생하지 않음

## 10.1 API 서버 이해하기
- **API**
	- Application Programming Interface의 두문자어
	- 다른 애플리케이션에서 현재 프로그램의 기능을 사용할 수 있게 허용하는 접점을 의미
- **웹 API**
	- 다른 웹 서비스의 기능을 사용하거나 자원을 가져올 수 있는 창구
	- 흔히 API를 열었다, 만들었다 라고 표현
		- 다른 프로그램에서 현재 기능을 사용할 수 있게 허용했음을 뜻함
	- 다른 사람에게 정보를 제공하고 싶은 부분만 API를 열어놓고, 제공하고 싶지 않은 부분은 API를 만들지 않는 것
	- 또한 API를 열어 놓았다 하더라도 모든 사람이 정보를 가져갈 수 있는 것이 아니라 인증된 사람만 일정 횟수 내에서 가져가게 제한을 둘 수도 있음
- **웹 API 서버**
	- 서버에 API를 올려서 URL을 통해 접근할 수 있게 만든 것
- **크롤링(Crawling)**
	- 웹 사이트가 자체적으로 제공하는 API가 없거나 API 이용에 제한이 있을 때 사용하는 방법
	- 표면적으로 보이는 웹 사이트의 정보를 일정 주기로 수집해 자체적으로 가공하는 기술
	- 웹 사이트에서 직접 제공하는 API가 아니므로, 원하는 정보를 얻지 못할 가능성이 있음
		- 웹 사이트에서 제공하길 원치 않는 정보를 수집한다면 법적인 문제가 발생할 수도 있음
	- 서비스 제공자 입장에서도 주기적으로 크롤링을 당하면 웹 서버의 트래픽이 증가하여 서버에 무리가 가므로, 웹 서비스를 만들 때 공개해도 되는 정보들은 API를 통해 가져가게 하는 것이 좋음


## 10.2 프로젝트 구조 갖추기
- 10장 예제는 9장 예제 nodebird 서비스와 db를 공유, 다른 서비스가 nodebird의 데이터나 서비스를 이용할 수 있도록 창구를 만드는 것
	- nodebird api를 만들어 토큰 인증을 한 유저는 json 데이터를 요청하고 응답받을 수 있는 웹 api 서버를 만들 것
- 도메인 모델 생성
	- 도메인은 인터넷 주소를 뜻함
	- 인터넷 주소(host), 도메인 종류(type), 클라이언트 비밀 키(clientSecret)이 들어감
		- 비밀 키는 다른 개발자들이 api를 사용할 때 필요한 키, 유출되면 다른 사람을 사칭해서 요청을 보낼 수 있으므로 주의해야 함
			- 한 가지 안전 장치로 요청을 보낸 도메인까지 일치해야 요청을 보낼 수 있게 제한하도록 할 것
- 도메인 등록을 하는 이유는 등록한 도메인에서만 API를 사용할 수 있게 하기 위함
	- 요청을 보낼 때 응답하는 곳과 도메인이 다르면 CORS(Cross-Origin Resource Sharing) 에러가 발생할 수 있음
	- CORS 문제를 해결하려면 API 서버 쪽에서 미리 허용할 도메인을 등록해야 함 

```js
// nodebird-api/models/domain.js
const Sequelize = require('sequelize');

module.exports = class Domain extends Sequelize.Model {
  static init(sequelize) {
    return super.init({
      host: {
        type: Sequelize.STRING(80),
        allowNull: false,
      },
      type: {
        type: Sequelize.ENUM('free', 'premium'), // 값을 제한하는 데이터 형식, 어겼을 때 에러 발생 
        allowNull: false,
      },
      clientSecret: {
        type: Sequelize.STRING(36),
        allowNull: false,
      },
    }, {
      sequelize,
      timestamps: true,
      paranoid: true,
      modelName: 'Domain',
      tableName: 'domains',
    });
  }

  static associate(db) {
    db.Domain.belongsTo(db.User);
  }
};
```

* 구조 분해 할당(ES6)

```js
var o = {p: 42, q: true};
var {p: foo, q: bar} = o;

console.log(foo); // 42
console.log(bar); // true
```

```js
// nodebird-api/routes/index.js
const express = require('express');
const { v4: uuidv4 } = require('uuid'); // 구조분해 할당,  uuid.v4를 uuid4에 할당함
const { User, Domain } = require('../models');
const { isLoggedIn } = require('./middlewares');

const router = express.Router();

router.get('/', async (req, res, next) => {
  try {
    const user = await User.findOne({
      where: { id: req.user && req.user.id || null },
      include: { model: Domain },
    });
    res.render('login', {
      user,
      domains: user && user.Domains,
    });
  } catch (err) {
    console.error(err);
    next(err);
  }
});

router.post('/domain', isLoggedIn, async (req, res, next) => {
  try {
    await Domain.create({
      UserId: req.user.id,
      host: req.body.host,
      type: req.body.type,
      clientSecret: uuidv4(),
    });
    res.redirect('/');
  } catch (err) {
    console.error(err);
    next(err);
  }
});

module.exports = router;
```

## 10.3 JWT 토큰으로 인증하기

- API 서비스를 제공하는 입장에선 nodebird앱이 아닌 다른 클라이언트가 데이터를 가져갈 수 있게 해야하는 만큼 별도의 인증 과정이 필요함
- 책 예제에선 JWT 토근으로 인증하는 방법을 사용
- **JWT(JSON Web Token)**
	- JSON 형식의 데이터를 저장하는 토큰, 아래 세 부분으로 구성됨
	- **헤더(HEADER)**
		- 토큰 종류와 해시 알고리즘 정보가 들어있음
	- **페이로드(PAYLOAD)**
		- 토큰의 내용물이 인코딩된 부분
	- **시그니처(SIGNATURE)**
		- 일련의 문자열이며, 시그니처를 통해 토큰이 변조되었는지 여부를 확인할 수 있음
		-  시그니처는 JWT 비밀 키로 만들어짐
	- JWT 는 내용을 볼 수 있어 민감한 정보를 넣으면 안됨
	- JWT 를 쓰는 이유는 내용이 있기 때문, 내용이 없는 랜덤한 토큰을 받으면 db에서 토큰의 주인이 누군지 권한은 무엇인지 매 요청마다 체크해야 함
		- JWT 는 비밀키만 알지 않는 이상 변조가 불가능, 변조된 토큰은 시그니처 비밀 키를 통해 검사할 때 들통남
		- 때문에 외부에 노출되도 되는 정보에 한해 사용자 이름, 권한 같은 것을 넣어두고 안심하고 사용 가능
	- 단점은 용량이 큼
		- 내용물이 들어 있으므로 랜덤한 토큰을 사용할 때 비교해서 용량이 클 수 밖에 없음
		- 매 요청 시 토큰이 오가서 데이터 양이 증가함
	- 랜덤 스트링을 사용해서 매번 사용자 정보를 조회하는 작업의 비용이 더 큰지, 내용물이 들어 있는 JWT 토큰을 사용해서 발생하는 데이터 비용이 더 큰지 비교해서 사용여부를 결정하면 됨

```bash
# JWT 모듈 설치
npm i jsonwebtoken
```

```bash
# nodebird-api/.env
JWT_SECRET=jwtSecret
```

- 사용자가 쿠키처럼 헤더에 토큰을 넣어 보냄
	- 요청 헤더에 저장된 토큰 (req.headers.authorization) 을 사용, jwt.verify 메서드로 토큰을 검증
	- jwt.verify(<검증할 토큰>, <토큰의 비밀키>)
- 비밀키가 일치하지 않으면 인증받을 수 없고 catch 문으로 이동
	- 토큰의 내용은 조금 전 넣은 사용자 아이디와 닉네임, 발급자, 유효 기간 등
- req.decoded를 통해 다음 미들웨어에서 토큰의 내용물을 사용가능

```js
// nodebird-api/routes/middlewares.js
const jwt = require('jsonwebtoken');
...
exports.verifyToken = (req, res, next) => {
  try {
    req.decoded = jwt.verify(req.headers.authorization, process.env.JWT_SECRET);
    return next();
  } catch (error) {
    if (error.name === 'TokenExpiredError') { // 유효기간 초과
      return res.status(419).json({
        code: 419,
        message: '토큰이 만료되었습니다',
      });
    }
    return res.status(401).json({
      code: 401,
      message: '유효하지 않은 토큰입니다',
    });
  }
};
```

- 토큰을 발급하는 라우터(POST /v1/token), 사용자가 토큰을 테스트해볼 수 있는 라우터(GET /v1/test)
	- POST /v1/token 라우터
		- 전달받은 클라이언트 비밀 키로 도메인이 등록된 것인지 확인, 등록되지 않은 경우 에러 메시지로 응답, 등록된 도메인이면 토큰을 발급해 응답함
		- 토큰은 jwt.sign 메서드로 발급 가능
			-  jwt.sign(<토큰 내용>, <토큰 비밀키>, <토큰 설정>)
	- GET /v1/test
		- 사용자가 발급받은 토큰을 테스트해볼 수 있는 라우터
		- 토큰을 검증하는 미들웨어를 거친 후, 검증이 성공했다면 토큰의 내용물을 응답으로 보냄
- 라우터의 이름 v1은 버전을 뜻함, SemVer형태로 1.0.0과 같이 지어도 됨
- 단, 버전은 한 번 정해지고 함부로 수정하면 안됨, 사용하는 곳이 생기면 버전 명을 변경함으로 API 사용자들에게 영향이 생길 수 있기 때문

```js
// nodebird-api/routes/v1.js
const express = require('express');
const jwt = require('jsonwebtoken');

const { verifyToken } = require('./middlewares');
const { Domain, User } = require('../models');

const router = express.Router();

router.post('/token', async (req, res) => {
  const { clientSecret } = req.body;
  try {
    const domain = await Domain.findOne({
      where: { clientSecret },
      include: {
        model: User,
        attribute: ['nick', 'id'],
      },
    });
    if (!domain) {
      return res.status(401).json({
        code: 401,
        message: '등록되지 않은 도메인입니다. 먼저 도메인을 등록하세요',
      });
    }
    const token = jwt.sign({
      id: domain.User.id,
      nick: domain.User.nick,
    }, process.env.JWT_SECRET, {
      expiresIn: '1m', // 1분
      issuer: 'nodebird',
    });
    return res.json({
      code: 200,
      message: '토큰이 발급되었습니다',
      token,
    });
  } catch (error) {
    console.error(error);
    return res.status(500).json({
      code: 500,
      message: '서버 에러',
    });
  }
});

router.get('/test', verifyToken, (req, res) => {
  res.json(req.decoded);
});

module.exports = router;
```

- 만든 라우터를 서버에 연결

```js
// nodebird-api/app.js
const v1 = require('./routes/v1');
...
app.use('/v1', v1);
app.use('/auth', authRouter);
```

**JWT 토큰으로 로그인하려면**
- 세션을 사용하지 않고 로그인할 수 있기 때문에 최근엔 JWT 토큰을 사용해서 로그인하는 방법이 많이 사용되고 있음
- 로그인 완료 시 세션에 데이터를 저장하고 세션 쿠키를 발급하는 대신 JWT 토큰을 쿠키로 발급하면 됨
- 아래 같이 authenticate 메서드의 두 번째 인수로 옵션을 주면 세션을 사용하지 않을 수 있음
	- 세션에 데이터를 저장하지 않으므로 serializeUser, deserializeUser는 사용하지 않음
	- 그 후 모두 라우터에 verifyToken 미들웨어를 넣어 클라이언트에서 보낸 쿠키를 검사한 후 토큰이 유효하면 라우터로 넘어가고, 그렇지 않으면 401이나 419에러를 응답하면 됨

```js
...
router.post("/login", isNotLoggedIn, (req, res, next) => {
    passport.authenticate("local", { session: false }, (authError, user, info) => {
        if (authError) {
...
```

- JWT 토큰안에 사용자 권한을 담아 사용하면 사용자 권한 확인을 위해 db를 사용하지 않을 수 있어 서비스 규모가 클수록 데이터베이스의 부담을 줄일 수 있음

**클라이언트에서 JWT 를 사용하고 싶다면**
- 클라이언트 환경에서는 process.env.JWT_SECRET(비밀키) 가 노출되면 안됨
- 그럼에도 verify나 sign같은 메서드를 사용해야 한다면 RSA 같은 양방향 비대칭 암호화 알고리즘을 사용해야 함
- 서버 환경에서는 비밀 키를 사용하고 클라이언트 환경에서는 공개 키를 사용하는 방식으로 클라이언트에서 비밀 키가 노출되는 것을 막을 수 있음
- https://www.npmjs.com/package/jsonwebtoken 에서 PEM 키를 사용하는 부분 참고


## 10.4 다른 서비스에서 호출하기
- API 제공 서버를 만들었으니 API를 사용하는 서비스를 만들 것 (nodecat)
- localhost:4000/test 요청 시 8002 포트 API 서버에 post 요청을 보냄(localhost:8002/v1/token)
	- .env에 설정한 클라이언트 비밀키를 갖고 요청해서 토큰이 조회된 경우 응답받음

```js
// /nodecat/app.js - 4000 포트로 구동, 기본 설정(생략)
// /nodecat/routes/index.js
const express = require('express');
const axios = require('axios');

const router = express.Router();

router.get('/test', async (req, res, next) => { // 토큰 테스트 라우터
  try {
    if (!req.session.jwt) { // 세션에 토큰이 없으면 토큰 발급 시도
      const tokenResult = await axios.post('http://localhost:8002/v1/token', {
        clientSecret: process.env.CLIENT_SECRET,
      });
      if (tokenResult.data && tokenResult.data.code === 200) { // 토큰 발급 성공
        req.session.jwt = tokenResult.data.token; // 세션에 토큰 저장
      } else { // 토큰 발급 실패
        return res.json(tokenResult.data); // 발급 실패 사유 응답
      }
    }
    // 발급받은 토큰 테스트
    const result = await axios.get('http://localhost:8002/v1/test', {
      headers: { authorization: req.session.jwt },
    });
    return res.json(result.data);
  } catch (error) {
    console.error(error);
    if (error.response.status === 419) { // 토큰 만료 시
      return res.json(error.response.data);
    }
    return next(error);
  }
});

module.exports = router;
```

## 10.5 SNS API 서버 만들기
- 내가 올린 포스트와 해시태그 검색 결과를 가져오는  API를 추가

```js
// nodebird-api/routes/v1.js
...
const { Domain, User, Post, Hashtag } = require('../models');
...

router.get('/posts/my', verifyToken, (req, res) => {
  Post.findAll({ where: { userId: req.decoded.id } })
    .then((posts) => {
      console.log(posts);
      res.json({
        code: 200,
        payload: posts,
      });
    })
    .catch((error) => {
      console.error(error);
      return res.status(500).json({
        code: 500,
        message: '서버 에러',
      });
    });
});

router.get('/posts/hashtag/:title', verifyToken, async (req, res) => {
  try {
    const hashtag = await Hashtag.findOne({ where: { title: req.params.title } });
    if (!hashtag) {
      return res.status(404).json({
        code: 404,
        message: '검색 결과가 없습니다',
      });
    }
    const posts = await hashtag.getPosts();
    return res.json({
      code: 200,
      payload: posts,
    });
  } catch (error) {
    console.error(error);
    return res.status(500).json({
      code: 500,
      message: '서버 에러',
    });
  }
});
```

- nodecat 앱에서 내가 올린 포스트 정보를 요청하는 라우터와 해시태그 검색결과를 요청하는 라우터 추가
	- request함수를 통해 토큰이 만료된 경우, 419 메시지를 응답받고 재요청해서 토큰을 갱신하고 요청하도록 함

```js
// nodecat/routes/index.js
const express = require('express');
const axios = require('axios');

const router = express.Router();
const URL = 'http://localhost:8002/v1';

axios.defaults.headers.origin = 'http://localhost:4000'; // origin 헤더 추가
const request = async (req, api) => {
  try {
    if (!req.session.jwt) { // 세션에 토큰이 없으면
      const tokenResult = await axios.post(`${URL}/token`, {
        clientSecret: process.env.CLIENT_SECRET,
      });
      req.session.jwt = tokenResult.data.token; // 세션에 토큰 저장
    }
    return await axios.get(`${URL}${api}`, {
      headers: { authorization: req.session.jwt },
    }); // API 요청
  } catch (error) {
    if (error.response.status === 419) { // 토큰 만료시 토큰 재발급 받기
      delete req.session.jwt;
      return request(req, api);
    } // 419 외의 다른 에러면
    return error.response;
  }
};

router.get('/mypost', async (req, res, next) => {
  try {
    const result = await request(req, '/posts/my');
    res.json(result.data);
  } catch (error) {
    console.error(error);
    next(error);
  }
});

router.get('/search/:hashtag', async (req, res, next) => {
  try {
    const result = await request(
      req, `/posts/hashtag/${encodeURIComponent(req.params.hashtag)}`,
    );
    res.json(result.data);
  } catch (error) {
    if (error.code) {
      console.error(error);
      next(error);
    }
  }
});
```

## 10.6 사용량 제한 구현하기
- 인증된 사용자더라도 과도하게 API를 사용하면 API 서버에 무리가 감, 따라서 일정 기간 내 API를 사용할 수 있는 횟수를 제한하여 서버 트래픽을 줄이는 것이 좋음
	- 유료 서비스라면 과금 체계별로 횟수에 차이를 둘 수 있음, 아래는 예시
		- 무료는 1시간에 몇 번 허용
		- 유료는 1시간에 100번 허용
	- 이러한 기능 또한 npm 패키지로 있음
		- https://www.npmjs.com/package/express-rate-limit

```bash
# nodebird-api에 설치
npm i express-rate-limit
```

- apiLimiter 미들웨어를 라우터에 넣으면 라우터에 사용량 제한이 걸림
	- RateLimit 미들웨어 옵션으로 windowMs(기준 시간), max(허용 횟수), handler(제한 초과 시 콜백 함수) 등이 있음
- deprecated 미들웨어는 사용하면 안되는 라우터에 붙여줌

```js
// nodebird-api/routes/middlewares.js
const jwt = require('jsonwebtoken');
const RateLimit = require('express-rate-limit');

...
exports.apiLimiter = new RateLimit({
  windowMs: 60 * 1000, // 1분
  max: 10,
  delayMs: 0,
  handler(req, res) {
    res.status(this.statusCode).json({
      code: this.statusCode, // 기본값 429
      message: '1분에 한 번만 요청할 수 있습니다.',
    });
  },
});

exports.deprecated = (req, res) => {
  res.status(410).json({
    code: 410,
    message: '새로운 버전이 나왔습니다. 새로운 버전을 사용하세요.',
  });
};
```

- 클라이언트로 보내는 응답 코드는 정리해두면 클라이언트가 프로그래밍 할 때 많은 도움이돼 좋음 
	- 예제에서 만든 응답 코드
		- 200 - JSON 데이터입니다
		- 401 - 유효하지 않은 토큰입니다
		- 410 - 새로운 버전이 나왔습니다. 새로운 버전을 사용하세요
		- 419 - 토큰이 만료됐습니다
		- 429 - 1분에 한 번만 요청할 수 있습니다
		- 500 ~ 기타 서버 에러

- 사용량 제한이 추가됐으므로 기존 API 버전과 호환되지 않음, 새로운 v2 라우터를 만듦

```js
// nodebird-api/routes/v2.js
const express = require('express');
const jwt = require('jsonwebtoken');

const { verifyToken, apiLimiter } = require('./middlewares');
const { Domain, User, Post, Hashtag } = require('../models');

const router = express.Router();

router.post('/token', apiLimiter, async (req, res) => {
  const { clientSecret } = req.body;
  try {
    const domain = await Domain.findOne({
      where: { clientSecret },
      include: {
        model: User,
        attribute: ['nick', 'id'],
      },
    });
    if (!domain) {
      return res.status(401).json({
        code: 401,
        message: '등록되지 않은 도메인입니다. 먼저 도메인을 등록하세요',
      });
    }
    const token = jwt.sign({
      id: domain.User.id,
      nick: domain.User.nick,
    }, process.env.JWT_SECRET, {
      expiresIn: '30m', // 30분
      issuer: 'nodebird',
    });
    return res.json({
      code: 200,
      message: '토큰이 발급되었습니다',
      token,
    });
  } catch (error) {
    console.error(error);
    return res.status(500).json({
      code: 500,
      message: '서버 에러',
    });
  }
});

router.get('/test', verifyToken, apiLimiter, (req, res) => {
  res.json(req.decoded);
});

router.get('/posts/my', apiLimiter, verifyToken, (req, res) => {
  Post.findAll({ where: { userId: req.decoded.id } })
    .then((posts) => {
      console.log(posts);
      res.json({
        code: 200,
        payload: posts,
      });
    })
    .catch((error) => {
      console.error(error);
      return res.status(500).json({
        code: 500,
        message: '서버 에러',
      });
    });
});

router.get('/posts/hashtag/:title', verifyToken, apiLimiter, async (req, res) => {
  try {
    const hashtag = await Hashtag.findOne({ where: { title: req.params.title } });
    if (!hashtag) {
      return res.status(404).json({
        code: 404,
        message: '검색 결과가 없습니다',
      });
    }
    const posts = await hashtag.getPosts();
    return res.json({
      code: 200,
      payload: posts,
    });
  } catch (error) {
    console.error(error);
    return res.status(500).json({
      code: 500,
      message: '서버 에러',
    });
  }
});

module.exports = router;
```

- 기존 v1 라우터를 사용할 때는 deprecated 미들웨어를 사용해 경고 메시지를 띄움

```js
// nodebird-api/routes/v1.js
...
const router = express.Router();

router.use(deprecated);

router.post('/token', async (req, res) => {
...
```

- 실제 서비스 운영 시 새로운 버전의 API가 나왔다고 이전 버전을 닫기보다 일정 기간을 두고 옮겨가는 것이 좋음
	- 사용자가 변경된 부분을 자신의 코드에 반영할 시간이 필요하기 때문 (노드 LTS 방식 참고)
- 새로 만든 라우터를 서버와 연결

```js
// nodebird-api/app.js
...
app.use(passport.session());

app.use('/v1', v1);
app.use('/v2', v2);
app.use('/auth', authRouter);
...
```

- nodebird-api는 서버가 재시작되면 사용량이 초기화되므로 **실제 서비스에서 사용량을 저장할 데이터베이스를 따로 마련하는 것이 좋음**
	- 보통 레디스가 많이 사용됨
	- 단 express-rate-limit 패키지는 데이터베이스와 연결하는 것을 지원하지 않으므로 npm에 새로운 패키지를 찾아보거나 직접 구현해야 함


## 10.7 CORS 이해하기

- nodecat 프런트(localhost:4000/)에서 nodebird-api(localhost:8002/v2/token)를 호출하면?
	- 지금은 서버 비밀키를 프론트에 전달해서 다시 요청해서 같은 키를 사용하지만, 실제론 서버, 프런트 비밀키를 따로 두는게 좋음(노출되므로)

```js
// nodecat/routes/index.js
...
router.get('/', (req, res) => {
  res.render('main', { key: process.env.CLIENT_SECRET });
});
...
```

```html
<!-- nodecat/views/main.html -->
<!DOCTYPE html>
<html>
  <head>
    <title>프론트 API 요청</title>
  </head>
  <body>
  <div id="result"></div>
  <script src="https://unpkg.com/axios/dist/axios.min.js"></script>
  <script>
    axios.post('http://localhost:8002/v2/token', {
      clientSecret: '{{key}}',
    })
      .then((res) => {
        document.querySelector('#result').textContent = JSON.stringify(res.data);
      })
      .catch((err) => {
        console.error(err);
      });
  </script>
  </body>
</html>
```

- localhost:4000에 접속하면 CORS 에러가 발생함(No 'Access-Control-Allow-Origin' header is present on the requested resource.)
	- 브라우저와 서버의 도메인이 일치하지 않으면 기본적으로 요청이 차단됨
	- 이 현상은 브라우저에서 서버로 요청을 보낼 떄만 발생하고, 서버에서 서버로 요청을 보낼 떄는 발생하지 않음
	- **현재 요청을 보내는 클라이언트(localhost:4000)와 요청을 받는 서버(localhost:8002)의 도메인이 다름, 이 문제를 CORS(Cross-Origin-Resource Sharing) 문제라고 부름**
- 네트워크 탭에서 Method가 POST 대신 OPTIONS로 표시됨
	- OPTIONS 메서드는 실제 요청을 보내기 전에 서버가 이 도메인을 허용하는지 체크하는 역할을 함
- **CORS 문제를 해결하기 위해선 응답 헤더에 Access-Control-Allow-Origin 헤더를 넣어야 함**
	- 이 헤더는 클라이언트 도메인의 요청을 허락하겠다는 뜻을 가짐
- nodebird-api에 cors 패키지 설치

```bash
npm i cors
```

- CORS 문제를 해결하면 요청을 보내는 주체가 클라이언트라서 비밀 키(process.env.CLIENT_SECRET)가 모두에게 노출됨
	- 이 문제를 막기 위해 비밀 키 발급 시 허용한 도메인을 적게 했던 것
	- **호스트와 비밀 키 모두 일치할 때만 CORS를 허용하게 수정하면 됨**

```js
// nodebird-api/routes/v2.js
...
const cors = require('cors');
...
router.use(async (req, res, next) => {
  const domain = await Domain.findOne({
    where: { host: url.parse(req.get('origin')).host }, // url 모듈 parser 메서드로 프로토콜 떼어냄
  });
  if (domain) {
    cors({
      origin: req.get('origin'),    // 허용할 도메인, 여러 개의 도메인을 허용하고 싶으면 배열로 주면 됨
      credentials: true,    // 도메인 간 쿠키 공유 옵션
    })(req, res, next);
  } else {
    next();
  }
});
```

- 특정 도메인만 허용하기 때문에 Access-Control-Allow-Origin값이 *가 아닌 localhost:4000으로 표시됨
- 미들웨어 안에서 미들웨어에 인수를 직접 줘 호출 가능, 아래 두 코드는 같은 역할

```js
// 1.
router.use(cors());

// 2.
router.use((req, res, next) => 
    cors()(req, res, next);
});
```

- 클라이언트와 서버가 같은 비밀키를 쓰는 예제였지만 일반적으로 여러 비밀 키를 구분해서 발급해줌(카카오는 네이티브 앱, REST API, JavaScript, Admin 키가 개별로 발급됨)


**프록시 서버**
- **CORS 문제를 해결하는 또 다른 방법은 프록시(대리인) 서버를 사용하는 것**
- 서버에서 서버로 요청을 보낼 때는 CORS 문제가 발생하지 않는다는 것을 이용한 방법
	- 브라우저와 도메인이 같은 서버를 만든 후, 브라우저에서는 API 서버 대신 프록시 서버에 요청을 보냄
	- 그 후 프록시 서버에서 요청을 받아 다시 API 서버로 요청을 보냄
	- 서버-서버 간의 요청이므로 CORS 문제가 발생하지 않음
- 프록시 서버는 직접 구현해도 되지만 http-proxy-middleware 같은 npm 패키지를 사용하면 쉽게 익스프레스와 연동 가능