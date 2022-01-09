> 데이터 베이스는 관련성을 가지며 중복이 없는 데이터들의 집합이다. 
이러한 데이터베이스를 관리하는 시스템을 DBMS라고 부르며 계속 데이터가 보존되므로 서버 종료 여부와 상관없이 데이터를 지속적으로 사용할 수 있다.

## 1. 데이터베이스 생성하기
* 예약어는 대문자로 쓰는 것 추천 (nodejs와 같은 사용자가 직접 만든 이름과 구분하기 위해서)
* 예약어 끝에는 세미콜론(;)을 붙혀야 실행됨

> CREATE SCHEMA [데이터베이스 명];

```
//DEFAULT CHARCTER SET utf8: 한글 사용
mysql> CREATE SCHEMA `nodejs` DEFAULT CHARCTER SET utf8;
mysql> use nodejs;
```
## 2. 테이블
* 테이블이란 데이터가 들어갈 수 있는 틀을 의미하며, 테이블에 맞는 데이터만 들어갈 수 있음.![](https://images.velog.io/images/developerelen/post/d2912721-ab95-49a6-9d16-888347a8d6eb/db_row_column.jpeg)

> * 테이블 생성 
CREATE TABLE [데이터베이스 명, 테이블 명];

> * 테이블 확인
DESC [테이블 명];

> * 테이블 제거
DROP TABLE [테이블명]

> * 테이블 보기
SHOW TABLES;

```query
mysql> CREATEJ TABLE nodejs.users(
    -> id INT NOT NULL AUTO_INCREMENT,
    -> name VARCHAR(20) NOT NULL,
    -> age INT UNSIGNED NOT NULL,
    -> married TINYINT NOT NULL,
    -> comment TEXT NULL,
    -> create_at DATETIME NOT NULL DEFAULT now(),
    -> PRIMARY KEY(id),
    -> UNIQUE INDEX name_UNIQUE (name ASC))
    -> COMMENT = '사용자 정보'
    -> DEFAULT CHARACTER SET = utf8    
    -> ENGINE = InnoDB;    
```
### 2-1. 컬럼 자료형
* **INT**: 정수 (소수까지 저장하고 싶으면 FLOAT 또는 DOUBLE 자료형을 사용)
* **VARCHAR(자릿수)**: 가변 길이 
  * VARCHAR(10): 0~10인 문자열 가능
* **CHAR(자릿수)**: 고정 길이
  * CHAR(10): 반드시 길이가 10인 문자열
* **TEXT**: 긴 글을 쓸 때 사용
  * VARCHAR: 수 백자 이내의 문자열
  * TEXT: 그 이상
* **TINYINT**: -128 ~ 127까지의 정수를 저장할 때 사용. 1 또는 0만 저장한다면 Boolean과 같은 역할 가능
* **DATETIME**: 날짜와 시간에 대한 정보만 담는 TIME 자료형도 있음.

### 2-2. 컬럼 옵션
* **NULL** or **NOT NULL**: 빈 칸을 허용할지 여부
* **AUTO_INCREMENT**: 숫자를 저절로 올림. id 컬럼에 설정됨
* **UNSINGED**: 숫자 자료형에 적용되는 옵션. 음수 무시
  * INT 숫자 자료형은 기본적으로 음수 범위 지원
  * FLOAT / DOUBLE에는 UN 적용 불가능
* **ZEROFILL**: 숫자의 자리가 고정되어 있을 때 사용가능. 비어있는 숫자의 자리에 0 입력
  * INT(4) -> 0001로 비어있는 자릿수에 0으로 채워줌
* **DEFAULT now()**: 데이터베이스 저장 시 해당 컬럼에 값이 없으면 MySQL이 기본값을 대신 넣음. now()는 현재 시간 저장
  * now() 대신 CURRENT_TIMESTAMP 가능
* 해당 컬럼이 기본 키인 경우 PRIMARY_KEY 옵션 설정
  * PRIMARY_KEY란 로우를 대표하는 고유한 값 (id)
* **UNIQUE INDEX**: 해당 값이 고유해야 하는지에 대한 옵션

### 2-3. 테이블 자체 설정
* **COMMENT**: 테이블에 대한 보충 설명. 이 테이블이 무슨 역할을 하는지 적어두며 필수 아님
* **DEFAULT CHARACTER SET**: utf8로 설정하지 않으면 한글이 입력되지 않으므로 필수
* **ENGINE**: 엔진을 무엇으로 사용할 것인가

### 2-4. 외래키 (foreign key)
* 다른 테이블의 기본키를 저장하는 컬럼

> CONSTARINT [제약조건명] FOREIGN KEY [컬럼명] REFERENCES [참고하는 컬럼명]

### 2-5. CASCADE
> ON DELETE CASCADE
  ON UPDATE CASCADE
  사용자 정보가 수정되거나 삭제되면 그것과 연결된 댓글 정보도 같이 수정하거나 삭제한다는 뜻
  
# CRUD

## 1. CREATE (생성)
* 데이터를 생성해서 데이터베이스에 넣는 작업
> INSERT INTO [테이블명] ([컬럼1], [컬럼2], [컬럼3], ...) VALUES ([값1], [값2], [값3], ...);

## 2. READ (조회)
* 데이터 베이스에 있는 데이터를 조회
> SELECT * FROM [테이블명];

* 특정 컬럼 조회
> SELECT [컬럼명] FROM [테이블명];

* 특정 조건의 데이터 조회
> SELECT [컬럼명] FROM [테이블명] WHERE [조건문];
>> SELECT name, age FROM nodejs.users WHERE married = 1 AND age > 30;

* 정렬 (ASC: 오름차순 / DESC: 내림차순)
> ORDER BY [컬럼명] [ASC|DESC];

* 건너뛰어 조회
> SELECT [컬럼명] FROM [테이블명] OFFSET [건너뛸 숫자];
>> OFFSET 20 : 처음 20개를 건너뛰고 다음 21~40의 20개 게시물을 조회

## 3. UPDATE (수정)
> UPDATE [테이블명] SET [컬럼명=바꿀 값] WHERE [조건];

## 4. DELETE (삭제)
> DELETE [테이블명] WHRER [조건];
  
# 시퀄라이즈
> ORM(Object-relational Mapping)으로 분류되는 시퀄라이즈는, 자바스크립트 객체와 데이터의 릴레이션을 매핑해주는 도구입니다. 즉, 자바스크립트의 구문을 알아서 SQL로 바꿔줍니다.

## 1. sequalize 설치하기
* sequilize-cli : 시퀄라이즈 명령어를 실행하기 위한 패키지
* mysql2는 MySQL과 시퀄라이즈를 이어주는 드라이버

> $ npm i express morgan numjucks sequelize sequelize-cli mysqul2
$ npm i -D nodemon
* 실행
$ npx sequelize init

* MySQL과 연동할 때에는 config 폴더 안의 config.json 정보가 사용되는데, 이 설정은 process.env.NODE_ENV가 development일 때 적용됨

## 2. 모델 정의하기
* MySQL에 정의한 테이블을 시퀄라이즈에서 정의
* MySQL의 테이블은 시퀄라이즈의 모델과 대응됨
* 시퀄라이즈는 기본적으로 모델 이름은 단수형, 테이블 이름은 복수형으로 사용

* init 메서드
  * 테이블에 대한 설정
  * 첫번째 인수: 테이블 컬럼에 대한 설정
    * 시퀄라이즈는 알아서 id를 기본 키로 연결하므로 id 컬럼은 적어줄 필요가 없음
  * 두번째 인수: 테이블 자체에 대한 설정
* associate 메서드
  * 다른 모델과의 관계를 적음

```js
const Sequelize = require('sequelize');

module.export = class User extends Sequelize.Model {
    static init(sequelize) {
        return super.init({}, {});
    }
    static associate(db) { }
}
```

### 2-1. MySQL vs 시퀄라이즈 자료형

|MySQL|시퀄라이즈|
|:------:|:------:|
|VARCHAR(100)|STRING(100)|
|INT|INTEGER|
|TINYINT|BOOLEAN|
|DATETIME|DATE|
|INT UNSIGNED|INTEGER.UNSIGNED|
|NOT NULL|allowNull: false|
|UNIQUE|unique: true|
|DEFAULT now()| defaultValue: Sequelize.NOW|

### 2-2. 테이블 옵션
* **sequelize**: static init 메서드의 매개변수와 연결되는 옵션으로 db.sequelize 객체를 넣어야 함.
* **timestamps**: true이면 시퀄라이즈는 createdAt과 updatedAt 컬럼을 추가합니다. 각각 로우가 생성될 때와 수정될 때의 시간이 자동으로 입력됨. 
* **underscored**: 시퀄라이즈는 기본적으로 테이블명과 컬럼명을 캐멀 케이스(camel case)로 만듦. 이를 스네이크 케이스로 바꾸는 옵션
* **modelName**: 모델 이름을 설정할 수 있음. 노드 프로젝트에서 사용
* **tableName**: 실제 데이터베이스의 테이블 이름이 됨. 기본적으로는 모델 이름을 소문자 및 복수형으로 만듦. 모델 이름이 User라면 테이블 이름은 users가 됨
* **paranoid**: true로 설정하면 deletedAt이라는 컬럼이 생김. 로우를 삭제할 때 완전히 지워지지 않고 deletedAt의 값이 null인 로우(삭제되지 않았다는 뜻)를 조회. (나중에 로우를 복원하기 위함)
* **charset 과 collate**: 각각 utf8과 utf8_general_ci로 설정해야 한글이 입력됨. 이모티콘도 입력하고 싶을 경우, utf8mb4와 utf8mb4_general_ci를 입력

## 1. 관계 정의하기
### 1-1. 1:N (사용자 1명 당, 댓글 여러개일 경우)
* 연결된 테이블의 로우들을 같이 불러옴
* hasMany() / belogsTo() 메서드 사용
* foreign Key를 설정하지 않으면 ' 모델명 + 기본키'인 컬럼이 모델에 생성

```js
static associate(db) {
  db.User.hasMany(db.Comment, { foreignKey: 'commenter', souceKey: 'id' });
  db.Comment.belongsTo(db.User, { foreignKey: 'commenter', targetKey: 'id' });
}
```

### 1-2. 1:1 (사용자와 사용자 정보)
* hasOne 메서드 사용
* 1:1 관계일 때에도 hasOne과 belogsTo가 반대로 되어서는 안됨
```js
db.User.hasOne(db.Info, { foreignKey: 'UserId', sourceKey: 'id'});
db.Info.hasOne(db.User, { foreignKey: 'UserId', targetKey: 'id'});
```

### 1-3. N:M (게시글과 해시태그 모델)
* belogsToMany 메서드 사용
* N:M 관계의 특성상 새로운 모델이 생성됨. through 속성 그 이름 적으면 됨
```js
db.Post.belogsToMany(db.Hashtag, { through: 'PostHashtag' });
db.HashTag.belogsToMany(db.Post, { through: 'PostHashtag' });
```

## 2. 시퀄라이즈 쿼리
* 쿼리는 프로미스를 반환하므로 then을 붙여 결괏값을 받을 수 있음

### 2-1. 로우 생성
* 데이터를 넣을 때 MySQL의 자료형이 아니라, 시퀄라이즈 모델에 정의한 자료형대로 넣어야 함.
  * MySQL에서 TINYINT로 0을 설정한 경우, 시퀄라이즈 모델에 맞춰 false로 표기해야 함.
```js
INSERT INTO nodejs.users(name, age, married, comment) VALUES ('zero', 24, 0, '자기소개1);

const { User } = require('../models');
User.create({
  name: 'zero',
  age: 24,
  married: false,
  comment: '자기소개1',
});                                                              
```

### 2-2. 조회
#### 2-2-1. findAll(): 모든 데이터 조회
```js
SELECT * FROM nodejs.users;
User.findAll({});
```

#### 2-2-2. findOne(): 하나의 데이터만 가져옴
```js
SELECT * FROM nodejs.users LIMIT 1;
User.findOne({});
```

#### 2-2-3. attributes: 원하는 컬럼의 값만 조회
```js
SELECT name, married FROM nodejs.users;
User.findAll({
	attributes: ['name', 'married'];
});
```

* MySQL에서는 Undefined라는 자료형을 지원하지 않으므로 where 옵션에는 undefined가 들어가면 안됨.
대신 null 사용.
```js
SELECT name, married FROM nodejs.users WHERE married = 1 AND age > 30;

const {Op} = require('sequelize');
const {User} = require('../models');

User.findAll({
	attributes: ['name', 'age'],
  	where: {
      married: true,
      age: { [Op.gt]: 30 },
    }
});
```

```js
SELECT name, married FROM nodejs.users WHERE married = 1 AND age > 30;

const {Op} = require('sequelize');
const {User} = require('../models');

User.findAll({
	attributes: ['name', 'age'],
  	where: {[Op.or]: [
        {married: false}, 
        {age: { [Op.gt]: 30 } }
      ]}
});
```

#### 2-2-4. ORDER BY
```js
SELECT id, name FROM users ORDER BY age DESC LIMIT 1 OFFSET 1;

User.findAll({
	attributes: ['id', 'name'],
    order: [['age', 'DESC']],
    limit: 1,
  	offset: 1,
});
```

### 2-3. 수정
```js
UPDATE nodejs.users SET comment = '바꿀 내용' WHERE id = 2;

User.update({
  comment: '바꿀 내용',
},{
  where: { id: 2 },
});
```

### 2-4. 삭제
```js
DELETE FROM nodejs.users WHERE id = 2;

User.destroy({
  where: { id: 2 },
});
```

## 3. 관계 쿼리
* findOne이나 findAll 메서드를 호출할 때 프로미스의 결과로 모델을 반환
* MySQL의 JOIN 기능

```js
// 관계 설정
const user = await User.findOne({
  include: [{
  	model: Comment,
  }]
});

//조회
const comments = await user.getComments();

//수정
const comments = await user.setComments();

//하나 생성
const comments = await user.addComment();

//여러 개 생성
const comments = await user.addComments();

//삭제
const comments = await user.removeComments();
```

* 모델 명 이름을 바꾸고 싶을 경우, as 사용
```js
// 관계 설정
db.User.hasMany(db.Comment, { foreignKey: 'commenter', soruceKey: 'id', as: 'Answers'});

const user = await User.findOne({});

//조회
const comments = await user.getAnswers();
```

## SQL 쿼리하기
* 만일 시퀄라이즈의 쿼리를 사용하기 싫거나 어떻게 해야할 지 모르겠다면 직접 SQL문을 통해 쿼리할 수도 있음.
```js
const [result, metadata] = await sequelize.query('SELECT * from commets');
```













