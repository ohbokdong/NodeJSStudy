# 11. 노드 서비스 테스트하기

## 11.1 테스트 준비하기
* 사용할 패키지: jest
* 설치 Command: $ npm i -D jest
* 설정: packages.json
```json
"scripts": {
    "start": "nodemon app",
    "test": "jest"
}
```
* 테스트용 파일 확장자: .test 또는 .spec
* middlewares.test.js
```js
test('1 + 1 은 2입니다.', () => {
    expect(1+1).toEqual(2);
});
```
* 실행: npm start
![](https://images.velog.io/images/developerelen/post/5f23c1b5-3071-441c-bb44-65a1cb82975d/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA%202022-02-11%20%E1%84%8B%E1%85%A9%E1%84%8C%E1%85%A5%E1%86%AB%2012.12.06.png)

## 11.2 유닛 테스트 (jest)
* 작은 단위의 함수나 모듈이 의도된 대로 정확히 작동하는지 확인하는 테스트 
* 함수가 수정되었을 때 고장나게 되면 어떤 부분이 고장나는지를 테스트를 통해 알 수 있음
* 테스트 코드도 기존 코드가 변경된 것에 맞춰서 수정해야 함

* 테스트 할 때, 함수에 들어가는 인수는 가짜 객체와 함수를 넣으면 됨.
  * 테스트의 역할은 코드나 함수가 제대로 실행되는지를 검사하고 값이 일치하는 지를 검사하는 것이기 때문
```js
/// routes/middlewares.test.js
const {isLoggedIn, isNotLoggedIn} = require('./middlewares');

describe('isLoggedIn', () => {
    const res = {
        status: jest.fn(() => res),
        send: jest.fn(),
    };
    const next = jest.fn();
    
    test('로그인 되어 있으면 isLoggedIn이 next를 호출해야 함', () => {
        const req = {
            isAuthenticated: jest.fn(() => true),
        };
        isLoggedIn(req, res, next);
        expect(next).toBeCalledTimes(1);
    });
    test('로그인 되어 있지 않으면 isLoggedIn이 에러를 응답해야 함', () => {
        const req = {
            isAuthenticated: jest.fn(() => false),
        };
        isLoggedIn(req, res, next);
        expect(res.status).toBeCalledWith(403);
        expect(res.send).toBeCalledWith('로그인 필요');
    });
});
```

### 데이터 베이스 모델 mocking
* 데이터베이스와 연결되어있는 모델은 테스트 환경에서 mocking을 해주어야 함
    * jest.mock 메서드 사용
    * jest.mock('모킹할 모듈의 경로').findOne.mockReturnValue();
* 이렇게 mocking을 할 경우, 실제 데이터베이스에 있는 데이터가 아니기 때문에 실제로 동작을 잘 하는지 알 수 없음.

    -> 이럴 경우 '통합 테스트'나 '시스템 테스트'를 해야함 

## 11.3 테스트 커버리지
: 전체 코드 중에서 어떤 코드가 테스트 되고 테스트되지 않는지 위치를 알려주는 jest의 기능
: 명시적으로 테스트하고 require한 코드만 커버리지 분석이 됨

* 설정: packages.json
  * --coverage를 입력하면 jest가 테스트 커버리지를 분석함
```json
"scripts": {
    "start": "nodemon app",
    "test": "jest", 
    "coverage": "jest --coverage"
}
```
* 실행: $ npm run coverage
![](https://images.velog.io/images/developerelen/post/13777058-5611-44b5-b8c3-dc5504325525/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA%202022-02-19%20%E1%84%8B%E1%85%A9%E1%84%8C%E1%85%A5%E1%86%AB%203.49.45.png)
* File (파일과 폴더 이름)
* Stmts (구문 비율)
* Branch (if문 등의 분기점 비율)
* Funcs (함수 비율)
* Lines (코드 줄 수 비율)
* Uncoverage Line #s (커버되지 않은 줄 위치)

## 11.4 통합 테스트 (supertest)
* 하나의 라우터를 통째로 (여러 개의 미들웨어 + 라이브러리)가 유기적으로 작동하는지 확인하는 테스트
* 설치: $npm i -D supertest
* supertest를 사용하기 위해서는 app 객체를 모듈로 만들어 분리해야 함. 
* 통합 테스트에서는 데이터베이스 코드를 모킹하지 않으므로 데이터베이스에 실제로 테스트용 데이터가 저장됨.
    * 실제로 서비스 중인 데이터베이스에 테스트용 데이터가 들어가면 안 되므로, 테스트용 데이터베이스를 따로 만드는 것이 좋음
```js
const request = require('supertest');
const {sequelize} = require('../models');
const app = require('../app');
const { describe } = require('../models/user');

// <1>
beforeAll(async () => {
    await sequelize.sync();
});

// <2>
describe('POST /join', () => {
    test('로그인 안 했으면 가입', (done) => {
        request(app)
        .post('/auth/join')
        .send({
            email: 'sgmsgood@gmail.com',
            nick: 'elen',
            password: 'nodejsbook'
        })
        .expect('Location','/')
        .expect(302, done);
    });
});

// <3>
describe('POST /join', () => {
    // <3-1>
    const agent = request.agent(app);
    // <3-2>
    beforeEach((done) => {
        agent
        .post('/auth/login')
        .send({
            email: 'sgmsgood@gmail.com',
            password: 'nodejsbook'
        })
        .end(done);
    });
});

// <4>
test('이미 로그인 했으면 redirect/', (done) => {
    const message = encodeURIComponent('로그인한 상태입니다.');
    agent
        .post('/auth/join')
        .send({
            email: 'sgmsgood@gmail.com',
            nick: 'elen',
            password: 'nodejsbook'
        })
        .expect('Location',`/?error=${message}`)
        .expect(302, done);
    });

// <5>
describe('POST /login', () => {
    test('로그인 수행', async (done) => {
        request(app)
        .post('/auth/login')
        .send({
            email: 'sgmsgood@gmail.com',
            password: 'nodejsbook'
        })
        .expect('Location','/')
        .expect(302, done);
    });
});

// <6>
afterAll(async () => {
    await sequelize.sync( { force: true});
});
```
<1>
* beforeAll(): 현재 테스트를 실행하기 전에 수행되는 코드
* sequalize.sync()를 넣어 데이터베이스에 테이블 생성
* afterApp: 모든 테스트가 끝난 후
* beforeEach: 각각의 테스트 수행 전
* afterEach: 각각의 테스트 수행 후

<2>
* 회원가입 테스트

<3>
* 로그인 한 상태에서 회원가입을 시도하는 경우 테스트 
<3-1>
* 로그인 한 이후 회원가입을 하는 순서가 중요하므로 agent를 만들고 하나 이상의 요청에서 재활용 가능
<3-2>
* beforeEach는 각각의 실행에 앞서 먼저 실행되는 부분. 회원가입 테스트를 위해 아까 생성한 agent 객체로 로그인을 먼저 수행.
* end(done)으로 beforeEach 함수가 마무리 되었음을 알림

<4>
* 로그인 된 agent로 회원가입 테스트를 진행

<5>
* supertest 패키지로부터 request 함수를 불러와서 app 객체를 인수로 넣음.
* 여기에 get, post, put, patch, delete 등의 메서드로 원하는 라우터에 요청을 보낼 수 있음. 
* 데이터는 send에 담아서 보내고, 그 후 예상되는 응답의 결과를 expect 메서드의 인수로 제공하면 그 값이 일치하는지 테스트
* supertest를 사용하면 app.listen을 수행하지 않고도 서버 라우터를 실행할 수 있음.

<6>
* 테스트 종료 시 데이터를 정리하는 코드

## 11.5 부하 테스트 (artillery)
: 서버가 얼마만큼의 요청을 견딜 수 있는지 테스트 하는 방법

: 서버는 접속자들의 정보를 저장하기 위해 각각의 접속자마다 일정한 메모리를 할당하는데, 이렇게 사용하는 메모리의 양이 증가하다가 서버의 메모리 용량을 넘게 되면 OOM(Out of Memory)와 같은 문제가 발생함.

: 임의적으로 부하가 될 수 있는 요청을 실행하고, 얼마나 견딜 수 있는지 테스트
* --count: 가상의 사용자 수
* -n: 요청 횟수 
```
$ npx artillery quick --count 100 -n 50 http://localhost:8001
```
```
// median과 p95의 값 차이가 적을 수록 좋음
Request latency: //응답 지연 속도
    min: 9.1 // 최소
    max: 303.1 // 최대
    median: 176.2 // 중간
    p95: 252.2 // 하위 95%
    p99: 276.7 // 하위 99%
```

### 11.5.1 사용자 행동 모방 시나리오 작성 테스트
```json
{
    "config": {
        "target" : "http://localhost:8001",
        "phases" : [
            {
                "duration": 60,
                "arrivalRate": 30
            }
        ]
    }, 
    "scenarios": [{
        "flow": [
            {
                "get": {
                    "url": "/"
                }
            }, {
                "post": {
                    "url": "/auth/login",
                    "json": {
                        "email": "sgmsgood@gmail.com",
                        "password": "nodejsbook"
                    }
                }
            }, {
                "get": {
                    "url": "/hashtag?hashtag=nodebird"
                }
            }
        ]
    }]
}
```
* config: target을 현재 서버로 잡고, 60초 동안 매초 30명의 사용자를 생성
* senarios: 가상 사용자들이 어떤 행동을 할 지 시나리오 작성
     * get 접속 -> 로그인 (Post /auth/login) -> 해시태그 검색(GET /hashtag?hashtag=nodebird)


* 테스트를 진행할 수록 요청을 처리하는 속도가 점점 느려짐 -> 서버가 부하 테스트를 하는 정도의 요청을 감당하지 못함.
    * 서버 사양 업그레이드 or 서버를 여러개 둠 or 클러스터링 기법을 통해 서버를 여러개 실행
    * 요청-응답 시 데이터베이스에 접근할 때 시간이 많이 소요되므로, 최대한 데이터베이스에 접근하는 요청을 줄이는게 좋음

## 11.6 프로젝트 마무리하기
* 테스트하기 어려운 패키지는 모킹하고, 테스트할 수 있는 패키지는 그대로 테스트 하기. (모킹할 때, 실제 상황에서는 에러가 발생할 수 있음을 염두에 두어야 함)

* 다양한 테스트를 주기적으로 수행해 서비스를 안정적으로 유지, 점검하는 것이 중요함
