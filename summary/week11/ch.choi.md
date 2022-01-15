# 8장 mongoDB

[공식 홈페이지](https://nodejs.org/ko/)  

본 내용에 사용된 책 이미지는 [Node.js 교과서](https://book.naver.com/bookdb/book_detail.nhn?bid=16418778) 이미지를 참고하였습니다.

## 8.1 NoSQL vs SQL

- MySQL: 대표적 SQL 데이터 베이스  
- mongoDB: 대표적 NoSQL 데이터 베이스

|SQL(MySQL)|NoSQL(mongoDB)|
|------|------|
|규칙에 맞는 데이터 입력|자유로운 데이터 입력|
|테이블 간 JOIN 지원| 컬렉션 간 JOIN 미지원|
|안정성, 일관성| 확장성, 가용성|

*위 NoSQL은 몽고 디비의 특징임으로 다른 데이터 베이스와 차이가 있을 수 있음

NoSQL은 고정된 데이블이 없음  
대신 테이블에 상응하는 컬렉션이라는 개념이 있지만 컬럼을 따로 정의 하자 읺음

MySQL은 users 테이블을 만들 때 name, age, married 등의 컬럼과 자료형, 옵션을 정의 하지만
mongoDB는 그냥 users 컬랙션을 맏들고 끝. 컬렉션에는 어떤 데이터든 들어 갈 수 있음

mongoDB는 MySQL과 달리 JOIN기능이 없음   
 JOIN을 흉내 낼 수는 있지만 하나의 쿼리로 여러 테이블을 합치는 작업이 항상 가능하지는 않음

[MySQL JOIN의 이해](https://yoo-hyeok.tistory.com/98)

동시에 쿼리를 수행하는 경우 쿼리가 섞여 예상치 못한 결과를 낼 가능성이 있는 것이 단점

단점에도 불과하고 사용하는 이유는 확장성과 가용성 때문

데이터의 일관성을 보장해주는 기능이 약하지만 데이터를 빠르게 넣을 수 있고 쉽게 여러 서버에 데이터를 분산할 수 있음

용어 
|MySQL|mongoDB|
|------|------|
|테이블|컬렉션|
|로우| 다큐먼트|
|컬럼| 필드|

애플리케이션을 만들 때 꼭 한가지 데이터 베이스만 사용해야 하는 것은 아님  
다른 특징을 가지고 알맞는 곳에 사용하면 됨

## 8.2 몽고 디비 설치하기
- 설치들 하셨죠?
## 8.3 컴퍼스 설치하기
- 당연히 하셨죠?

 __[Windows Service](https://ko.wikipedia.org/wiki/%EC%9C%88%EB%8F%84%EC%9A%B0_%EC%84%9C%EB%B9%84%EC%8A%A4)__  
  오랜 시간 동안 실행되며 특정한 기능을 수행하는 실행 파일이며, 사용자 간섭을 요구하도록 설계되지 않았다. 윈도우 서비스는 보통 마이크로소프트 윈도우 운영 체제가 시동될 때 실행되며 윈도우가 실행되고 있는 한 백그라운드 모드에서 실행된다.
  
  다른 운영체제도 비슷한 것들이 있음

## 8.4 데이터 베이스 및 컬렉션 생성하기

여기서는 이전에 MySQL로 만들었던 테이블에 상응하는 컬렉션을 만들어 보자

|명령어|행동| 비고|
|------|------|------|
|use __[데이터베이스 명]__|데이터 베이스 만드는 명령||
|show dbs|데이터베이스 목록을 확인|데이터가 있어요 db에 보임|
|db|현재 사용 중인 데이터 베이스를 확인||

[이미지]


컬렉션 만들기 
|명령어|행동| 비고|
|------|------|------|
|db.createCollection(__'컬렉션 명'__)|컬렉션 생성|다큐먼트를 넣으면 자동 생성됨|

컬렉션 확인
|명령어|행동| 비고|
|------|------|------|
|show collection|생성된 컬렉션 확인||

## 8.5 CRUD 작업 하기

몽고디비에서 CRUD 작업을 해보자

## 8.5.1 Create(생성)
컬렉션에 컬럼을 정의 하지 않아도 되므로 컬렉션에는 아무 데이터나 넣을 수 있음  
이러한 자유로움이 몽고디비의 장점, 단점은 무엇이 들어올지 모름

몽고디비는 자바스크립트 문법을 사용하므로 자바스크립크의 자료형을 사용  
추가로 몇 가지 자료형이 더 있음
  
### __몽고디비의 자료형__
____
    자바스크립트의 자료형
    Date
    정규표현식 같은 자바스크립트의 객체
    Binary Data (자주 씀)
    ObjectID (자주 씀)
    int
    long
    Decimal
    Timestamp (자주 씀)
    JavaScript

주의 : Undefined 와 Symbol은 몽고디비에서 자료형으로 사용하지 않음

ObjectID: 고유한 값을 사용해 다큐먼트를 조회 할 때 사용할 수 있음 (MySQL에서 기본 키와 비슷)

다큐먼트 생성하기

|명령어|행동| 비고|
|------|------|------|
|db.__컬렉션명__.save(__다큐먼트__)|해당 컬렉션에 다큐먼트를 추가||

예시 
```java
mongo
use nodejs //nodejs 사용
db.users.save({
    name: 'zero',
    age: 24,
    married:false,
    comment:'안녕하세요. 몽고디비 사용중',
    createdAt: new Date()
});
```
명령이 성공적이면 writeResult({"nInserted":1}) 이라는 응답이 출력됨  
다큐먼트가 한 개 생성되었다는 뜻 

## 8.5.2 Read(조회)
다큐먼트 조회

|명령어|행동| 비고|
|------|------|------|
|db.__컬렉션명__.find({__조건__},{__특정 필드__})|조건에 해당하는 다큐먼트 출력|find({} 모두 출력|

조회 예시 
```java
db.users.find(
    {}, //컬렉션 내 모든 다큐먼트에서
    {
        _id: 0, //id 필드 가져오기 x
        name: 1, //name 필드 가져오기
        married: 1 // married 필드 가져오기
    }
);
```

조회 시 조건 주기 예시
```java
db.users.find(
    {
        age :{$gt:30}, //30 초과이고
        married: true // 결혼을 했다면
    },
    {
        _id: 0, //id 필드 가져오기 x
        name: 1, //name 필드 가져오기
        age: 1 // married 필드 가져오기
    }
);
```
몽고디비는 자바사크립트 객체를 사용해서 명령어 쿼리를 생성해야 하므로 아래와 같은 특수한 연산자가 사용됨

__[조건에 자주 쓰이는 연산자](https://docs.mongodb.com/manual/reference/operator/query/)__ 
|연산자|행동|
|------|------|
|$gt|초과|
|$gte|이상|
|$lt|미만|
|$lte|이하|
|$ne|같지 않음|
|$or|또는|
|$in|배열 요소 중 하나|

$or은 조건을 배열에 넣어서 사용, 배열안의 조건에 해당한다면 true

$or 사용 예시
```java
db.users.find(
    {
        $or:[{age: {$gt:30}}, {married: false}] // 나이가 30 초과 거나 결혼을 안했거나, 조건이 배열로 감싸져 있음
    },
    {
        _id: 0, //id 필드 가져오기 x
        name: 1, //name 필드 가져오기
        age: 1 // married 필드 가져오기
    }
);
```

### [메서드 사용하기](https://docs.mongodb.com/manual/reference/method/db.collection.find/#modify-the-cursor-behavior) 
쿼리를 통해 나온 cursor에 여러 메서드를 사용할 수 있음. 

|메서드|행동|비고|
|------|------|------|
|sort({__조건__})|정렬|-1 내림차, 1 오름차|
|limit(__갯수__)|조회할 다큐먼트 개수를 설정|skip을 통해 건너뛰기 가능
|skip(__갯수__)|몇개를 건너 뛸기 설정|

## 8.5.3 Update(수정)

기존 데이터를 수정해보자


|명령어|행동| 비고|
|------|------|------|
|db.__컬렉션명__.update({__조건__},{__수정할 객체__})|수정할 다큐먼트의 객체를 수정함||

update 예시
```java
db.users.update(
    {
        name: 'nero'
    },
    {
        $set: {comment: '안녕하세요. 왜 변경하세요'}
    }
);
```

`$set` 연산자는 어떤 필드를 수정할지 정하는 연산자, 만약 이 연산자를 사용하지 않고 일반 객체를 넣으면 다큐먼트 통째로 두 번째 인수로 주어진 객체로 수정됨 주의 요망!!

일부를 필드만 수정하고 싶을때는 반드시 $set 연산자를 지정해야함

## 8.5.4 Delete(삭제)
데이터 삭제를 해보자

|명령어|행동| 비고|
|------|------|------|
|db.__컬렉션명__.remove({__조건__})|조건에 해당하는 다큐먼트 삭제||


지금까지 CRUD 였습니다.

노드와 연동하여 서버에서 데이터베이스를 조작할 수 있게 해야험

노드와 몽고디비를 연동해주고 쿼리까지 만들어주는 라이브러리가 있음

다음절 ㄱㄱ

## 8.6 몽구스 사용하기

MySQL에 시퀄라이즈가 있다면 mongoDB는 몽구스가 있음  
몽구스는 ORM(Object Document Mapping)

몽고디비는 자바스크립트인데 왜 자바스크립트 객체와 매핑하는 이유는?  
- 몽고디비에 없어서 불편한 기능을 몽구스가 보완해주기 때문

스키마라는 것이 있음

몽구스는 몽고디비에 데이터를 넣기전에 노드 서버 단에서 데이터를 한번 필더링 하는 역할을 함   
따라서 데이터를 넣을때 하는 실수를 최소화 할 수 있음

몽구스의 장점
- populate 메서드를 통해 관계가 있는 데이터를 쉽게 가져옴 (JOIN 대체)
- ES2015 프로미스 문법과 가독성이 높은 쿼리 빌더를 지원


populate를 통해 쿼리 한 번에 데이터를 합쳐서 가져오는 것은 아니지만 작업을 직접하지 않아도 됨

몽구스 패키지 설치 하기
```cmd
npm i mongoose
```

## 8.6.1 몽고디비 연결하기

노드와 몽고디비를 몽구스를 통해 연결해보자

몽고디비는 주소를 사용해 연결함

주소 형식:mongodb://[username:password@]host[:port][/[database][?option]]  
__[]부분은 없어도 됨__

```javascript
//schemas/index.js
const mongoose = require('mongoose');

const connect = () => {
    if(process.env.NODE_EVN != 'production')
    {
        mongoose.set('debug', true); //개발 환경일때는 디버그 
    }
    //몽고디비 URL로 연결 시도
    mongoose.connect('mongodb://이름:비밀번호@localhost:27017/admin',{
    dbname: 'nodejs', //어떤 디비 쓸건지
    useNewUrlParser: true, //
    useCreateIndes: true,
    },(error) =>{
        //에러 발생 시 인지하기 위한 로그 추가
        if(error) 
        {
            console.log('몽고디비 연결 에러',error);
        }else{
            console.log('몽고디비 연결 성공');
        }
    });
};

mongoose.connection.on('error', (error) => {
    console.error('몽고디비 연결 에러', error);
});

mongoose.connection.on('disconnected', () => {
    console.error('몽고디비 연결이 끊겼습니다. 연결 재시도 ㄱㄱ');
    connect();
});
module.exports = connect;
```

```javascript
//app.js
const express = require('express');
const path = require('path');
const morgan = require('morgan');

const nunjucks = require('nunjucks');
const connect = require('./schemas');

const app = express();
app.set('port', process,env.PORT|| 3002);
app.set('view engine', 'html');
nunjucks.configure('views',{
    express: app,
    watch: true,
});

connect(); //mongoDB connect 

app.use(express.static(path.join(__dirname,'public')));
app.use(express.json());
app.use(express.urlencoded({extended:false}));

app.use((req, res, next)=>{
    const error = new Error(`${req.method} ${req.url} 라우터가 없음`);
    error.status = 404;
    next(error);
});
app.use((err, req, res, next)=>{
    res.locals.message = err.message;
    res.locals.error = process.env.NODE_ENV !== 'production' ? err: {};
    res.status(err.status || 500);
    res.render('error');
});

app.listen(app.get('port'), ()=>{
    console.log(app.get('port'), '대기중...');
});

```

## 8.6.2 스키마 정의하기
몽구스 스키마 만들기

schemas 폴더에 user.js와 comment.js를 만들어 보자

```javascript
//schemas/user.js
const mongoose = require('mongoose');

const {schema} = mongoose;

//DB 입력 처럼 스키마를 만듦
const userSchema = new Schema({
    name: {
        type: String,
        required: true, // 필수 입력값
        unique = true, //고유값이여야함
    },
    age:{
        type:Number,
        required: true,
    },
    married:{
        type:Boolean,
        required: true,
    },
    comment: String, //옵션 없이 간단히 선언도 가능
    createdAt:{
        type: Date,
        default:Date.now, //기본 입력값- 데이터 생성시 시간 입력이 됨 
    },
});



module.exports = mongoose.module('User', userSchema); //model 메서드를 통해 스키마와 몽고디비의 컬렉션을 연결
```

몽구스는 알아서 _id를 기본키로 생성하기 때문에 적어불 필요는 없음

나머지 필드의 스펙만 입력 ㄱㄱ

몽구스 스키마의 특이점
- String 사용
- Number
- Buffer
- Boolean
- Mixed
- ObjectID
- Array  

위 값을 가질 수 있음


```javascript
//schemas/comment.js
const mongoose = require('mongoose');

const {schema} = mongoose;
const { Types: {ObjectID} } = Schema;

//DB 입력 처럼 스키마를 만듦
const commentsSchema = new Schema({
    commenter: {
        type: ObjectID,
        required: true, // 필수 입력값
        ref = 'User', // User 스키마의 사용자 ObjectID가 들어간다는 뜻, JOIN과 비슷한 기능을 할때 사용됨
    },
    comment:{
        type:String,
        required: true,
    },
    createdAt:{
        type: Date,
        default:Date.now, //기본 입력값- 데이터 생성시 시간 입력이 됨 
    },
});

module.exports = mongoose.module('Comment', commentsSchema); //model 메서드를 통해 스키마와 몽고디비의 컬렉션을 연결
```

## 8.6.3 쿼리 수행하기

[코드 많은건 생략]

```javascript
//routes/index.js
const express = require('express');

const User = require('../schemas/user');

const router = express.Router();

router.get('/', async(req, res, next)=>{
    try{
        const users = await User.find({}); //몽고디비에서 모든 사용자 검색
        res.render('mongoose', {users});

    }
    catch (err)
    {
        next(err);
    }
});
```

몽구스도 기본적으로 프로미스를 지원하므로 async/await, try/catch문을 사용 가능

```javascript
//routes/users.js
const express = require('express');
const User = require('../schemas/user');

const Comment = require('../schemas/comment');

const router = express.Router();

router.route('/')
    .get(async(req, res,next) => {
        try{
            const users = await User.find({});
            res.json(users);
        }
        catch(err)
        {
            next(err);
        }
    })
    .post(async(req, res, next)=> {
        try
        { //사용자 등록
            const user = await User.create({
                name: req.body.name,
                age: req.body.age,
                married: req.body.married,
            });
            console.log(user);
            res.status(201).json(user);
        }
        catch(err)
        {
            next(err);
        }
    });
//댓글을 조회하는 라우터
router.get('/:id/comments',async(req, res, next)=>{
    try{
        const comments = await Comment.find({
            commenter:req.params.id
        }).populate('commenter');
        res.json(comments);
    }
    catch(err)
    {
        next(err);
    }
});
module.exports = router;
```

사용자 등록 시 모델.create 메서드로 저장함  
정의한 스키마에 부합하지 않는 데이터를 넣었을 때는 몽구스가 에러를 발생 시킴 _id는 자동으로 생성됨

댓글 조회 라우터에 find 메서드에는 옵션이 추가됨. 먼저 댓글을 쓴 사용자의 아이디로 댓글을 조회 하고 populate메서드로 관련 있는 컬렉션의 다큐먼트를 불러 올 수 있음

Comment 스키마 commenter필드의 `ref`가 `User`로 되어 있으므로 알아서 users컬렉션의 사용자 다큐먼트를 찾아 합침


```javascript
//routes/comments.js

const express = require('express');
const Comment = require('../schemas/comment');

const router = express.Router();

router.post('/', async(req, res, next)=>{
    try
    {
        //댓글을 저장
        const comment = await Comment.create({
            commenter:req.body.id,
            comment: req.body.comment,
        });
        console.log(comment);
        //path로 어떤 것과 합칠 지 정하고 합친 결과를 반환
        const result = await Comment.populate(comment,{path:'commenter'});
        res.status(201).json(result);
    }
    catch(err)
    {
        next(err);
    }
});

router.route('/:id')
    .patch(async(req, res, next) =>{
        try
        {
            const result = await Comment.update({
                _id: req.params.id, //id를 통해 수정할 댓글 찾고
            },
            {
                comment: req.body.comment, // 해당 필드를 수정
                // 몽구스 기본이 $set 임으로 통째로 변경 할 위험이 없음
            });
            res.json(result);
        }
        catch(err)
        {
            next(err);
        }
    })
    .delete(async(req, res, next)=>{
        try
        {
            const result = await Comment.remove({
                _id:req.params.id //해당 id의 다큐먼트를 삭제
                });
                res.json(result);
        }
        catch(err)
        {
            next(err);
        }
    });
module.exports = router;
```

애플리케이션을 만들기 위한 준비는 끝남

로그인 구현, 이미지 업로드, JWT 토큰 인증, 실시간 데이터 전송, 외부 API 연동 등은 예제로 진행 할 예정

## 8.7 함께 보면 좋은 자료
- [몽고디비 문서](https://docs.mongodb.com/)
- [몽고디비 자료형](https://docs.mongodb.com/)
- [컴퍼스 메뉴얼]()
- [몽구스 문서]()