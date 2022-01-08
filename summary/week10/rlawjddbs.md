# 7장 MySQL

[**건너뛰기**](#7.6. 시퀄라이즈 사용하기)

## 7.1. 데이터베이스란? 

- 관련성있고 중복없는 데이터의 집합
  - 이러한 데이터베이스를 관리하는 시스템 = `DBMS(DataBase Management System)`
    - 교재는 DBMS 중에서 `관계형 DBMS(Relational DBMS)`[^1]를 사용

## 7.2. MySQL 설치하기

- 생략

## 7.3. 워크벤치 설치하기

- 생략

## 7.4. 데이터베이스 및 테이블 생성하기

### 7.4.1. 데이터베이스 및 테이블 생성하기

```shell
mysql> CREATE SCHEMA `nodejs` DEFAULT CHARACTER SET UTF8;
Query OK, 1 row affected, 1 warning (0.01 sec)

mysql> USE nodejs;
Database changed
```

### 7.4.2. 테이블 생성하기

<details>
    <summary>생성 코드</summary>

```shell
mysql> CREATE TABLE nodejs.users (
    ->     id INT NOT NULL AUTO_INCREMENT, 
    ->     name VARCHAR(20) NOT NULL,
    ->     age INT UNSIGNED NOT NULL,
    ->     married TINYINT NOT NULL,
    ->     comment TEXT NULL,
    ->     created_at DATETIME NOT NULL DEFAULT now(),
    ->     PRIMARY KEY (id),
    ->     UNIQUE INDEX name_UNIQUE (name ASC)
    -> )
    -> COMMENT '사용자 정보' 
    -> DEFAULT CHARSET=UTF8
    -> ENGINE=InnoDB;
Query OK, 0 rows affected, 1 warning (0.03 sec)

mysql> CREATE TABLE nodejs.comments (
    ->     id INT NOT NULL AUTO_INCREMENT,
    ->     commenter INT NOT NULL,
    ->     comment VARCHAR(100) NOT NULL,
    ->     created_at DATETIME NOT NULL DEFAULT now(),
    ->     PRIMARY KEY(id),
    ->     INDEX commenter_idx (commenter ASC),
    ->     CONSTRAINT commenter
    ->     FOREIGN KEY (commenter)
    ->     REFERENCES nodejs.users (id)
    ->     ON DELETE CASCADE
    ->     ON UPDATE CASCADE
    -> )
    -> COMMENT '댓글'
    -> DEFAULT CHARSET = UTF8
    -> ENGINE = InnoDB;
Query OK, 0 rows affected, 1 warning (0.02 sec)
```

</details>

- DEFAULT CHARSET을 UTF8로 설정하지 않으면 한글이 입력되지 않으니 반드시 설정
- 테이블 정보 확인하는 명령어는 `DESC 테이블명`

```shell
mysql> DESC users;
+------------+--------------+------+-----+-------------------+-------------------+
| Field      | Type         | Null | Key | Default           | Extra             |
+------------+--------------+------+-----+-------------------+-------------------+
| id         | int          | NO   | PRI | NULL              | auto_increment    |
| name       | varchar(20)  | NO   | UNI | NULL              |                   |
| age        | int unsigned | NO   |     | NULL              |                   |
| married    | tinyint      | NO   |     | NULL              |                   |
| comment    | text         | YES  |     | NULL              |                   |
| created_at | datetime     | NO   |     | CURRENT_TIMESTAMP | DEFAULT_GENERATED |
+------------+--------------+------+-----+-------------------+-------------------+
```

- 현재 Database에 생성된 테이블 목록은 `SHOW TABLES` 명령어로 확인할 수 있음

```shell
mysql> SHOW TABLES;
+------------------+
| Tables_in_nodejs |
+------------------+
| comments         |
| users            |
+------------------+
```

## 7.5. CRUD 작업하기

- 아래의 코드만 입력해봅시다. 😅

<details>
    <summary>Insert Code</summary>

```mysql
-- ---------
-- zero
-- ---------
INSERT INTO nodejs.users (
    name,
    age,
    married,
    comment
) VALUES (
    'zero',
    24,
    0,
    '자기소개 1'
);

-- ---------
-- nero
-- ---------
INSERT INTO nodejs.users (
    name,
    age,
    married,
    comment
) VALUES (
    'nero',
    32,
    1,
    '자기소개 2'
);
```
</details>


## 7.6. 시퀄라이즈 사용하기

- `시퀄라이즈(Sequelize)`는 `ORM(Object-relational Mapping)`[^2] 라이브러리

### 시퀄라이즈 설치

<details>
    <summary>package.json</summary>

```json
{
    "name": "learn-sequelize",
    "version": "0.0.1",
    "description": "시퀄라이즈를 배우자",
    "main": "app.js",
    "scripts": {
        "start": "nodemon app"
    },
    "author": "ZeroCho",
    "license": "MIT",
    "dependencies": {
        "express": "4.17.2",
        "morgan": "1.10.0",
        "mysql2": "2.3.3",
        "nunjucks": "3.2.3",
        "sequelize": "6.12.5",
        "sequelize-cli": "6.3.0"
    },
    "devDependencies": {
        "nodemon": "^2.0.4"
    }
}
```
</details>

```shell
% npm i express morgan nunjucks sequelize sequelize-cli mysql2
``` 

- `sequelize-cli` - 시퀄라이즈 명령어를 실행하기 위한 패키지
- `mysql2` - MySQL과 시퀄라이즈를 이어주는 드라이버(JDBC 같은 건가?)

### 시퀄라이즈 초기화

```shell
npx sequelize init

Sequelize CLI [Node: 14.9.0, CLI: 6.3.0, ORM: 6.12.5]

Created "config/config.json"
Successfully created models folder at "생...략".
Successfully created migrations folder at "생...략".
Successfully created seeders folder at "생...략".
```

1. config, models, migrations, seeders 폴더가 생성됨
2. models 폴더 안에 `index.js`가 생성되었는지 확인
3. sequelize-cli가 자동으로 생성해주는 코드는 그대로 사용하지 않고 다음과 같이 수정

```js
'use strict';

const Sequelize = require('sequelize');
const env = process.env.NODE_ENV || 'development'; // default - development
const config = require(__dirname + '/../config/config.json')[env];
const db = {};

const sequelize = new Sequelize(config.database, config.username, config.password, config);

db.sequelize = sequelize;

module.exports = db;
```

- `Sequelize`는 시퀄라이즈 패키지이자 생성자
- `config/config.json`에서 데이터 베이스 설정을 불러온 후 `new Sequelize`를 통해
  `MySQL 연결 객체` 생성
- `db` 모듈을 내보냄으로서 본 모듈 내부에 할당된 `MySQL 연결 객체` 재사용 가능

### 7.6.1. MySQL 연결하기

- `시퀄라이즈`를 통해 `익스프레스 앱`과 `MySQL`을 연결해야 함

<details>
    <summary>겁나 긴 코드보기</summary>

```js
const express = require('express');
const path = require('path');
const morgan = require('morgan');
const nunjucks = require('nunjucks');

const { sequelize } = require('./models');

const app = express();
app.set('port', process.env.PORT || 3001);
app.set('view engine', 'html');
nunjucks.configure('views', {
    express: app,
    watch: true,
});
sequelize.sync({ force: false }) // true 시 서버 실행 시마다 테이블을 재생성함
    .then(() => {
        console.log('데이터베이스 연결 성공');
    })
    .catch((err) => {
        console.error(err);
    });

app.use(morgan('dev'));
app.use(express.static(path.join(__dirname, 'public')));
app.use(express.json());
app.use(express.urlencoded({ extended: false }));

app.use((req, res, next) => {
    const error =  new Error(`${req.method} ${req.url} 라우터가 없습니다.`);
    error.status = 404;
    next(error);
});

app.use((err, req, res, next) => {
    res.locals.message = err.message;
    res.locals.error = process.env.NODE_ENV !== 'production' ? err : {};
    res.status(err.status || 500);
    res.render('error');
});

app.listen(app.get('port'), () => {
    console.log(app.get('port'), '번 포트에서 대기 중');
});
```
</details>

- `require('./models')`는 `require('./models/index.js')`와 같음
  - 폴더 내의 `index.js` 파일은 require 시 이름을 생략할 수 있음
- `db.sequelize`를 불러와서 sync 메서드를 사용해 서버 실행 시 MySQL과 연동되도록 함
- `sequelize.sync` 메서드 내부에 `force: false` 옵션은 서버 실행 시마다 테이블을 재생성하는
  옵션을 off 함

#### 7.6.1.1. config.json 정보

- development 쪽 정보만 현재 MySQL 커넥션과 일치하게 수정
- 본 설정 파일은 `process.env.NODE_ENV`가 `development`일 때 적용됨(기본적으로 development임)

### 7.6.2. 모델 정의하기

- MySQL에서 정의한 테이블을 시퀄라이즈에서도 정의해야 함

<details>
    <summary>models/user.js</summary>

```js
const Sequelize = require('sequelize');

module.exports = class User extends Sequelize.Model {
    static init(sequelize) {
        return super.init({
            name: {
                type: Sequelize.STRING(20),
                allowNull: false,
                unique: true,
            },
            age: {
                type: Sequelize.INTEGER.UNSIGNED,
                allowNull: false,
            },
            married: {
                type: Sequelize.BOOLEAN,
                allowNull: false,
            },
            comment: {
                type: Sequelize.TEXT,
                allowNull: true,
            },
            created_at: {
                type: Sequelize.DATE,
                allowNull: false,
                defaultValue: Sequelize.NOW,
            },
        }, {
            sequelize,
            timestamps: false,
            underscored: false,
            modelName: 'User',
            tableName: 'users',
            paranoid: false,
            charset: 'utf8',
            collate: 'utf8_general_ci',
        });
    }

    static associate(db) {}
};
```

- `super.init` 메서드의 첫 번째 인자는 테이블 컬럼에 대한 설정, 두 번째 인자는
  테이블 자체에 대한 설정
- 시퀄라이즈는 자동으로 id를 기본 키로 연결하므로 id 컬럼은 적어줄 필요 X, 나머지 컬럼 스펙만 입력
- MySQL 테이블과 컬럼 내용이 일치해야 정확하게 대응됨

</details>

<details>
    <summary>models/comment.js</summary>

```js
const Sequelize = require('sequelize');

module.exports = class Comment extends Sequelize.Model {
    static init(sequelize) {
        return super.init({
            comment: {
                type: Sequelize.STRING(100),
                allowNull: false,
            },
            created_at: {
                type: Sequelize.DATE,
                allowNull: true,
                defaultValue: Sequelize.NOW,
            },
        }, {
            sequelize,
            timestamps: false,
            modelName: 'Comment',
            tableName: 'comments',
            paranoid: false,
            charset: 'utf8mb4',
            collate: 'utf8mb4_general_ci',
        });
    }
  
    static associate(db) {}
};
```

- `users.commenter` 컬럼에 대한 설정 정보가 빠진 것처럼 보이는데 시퀄라이즈 자체에서
  관계를 따로 정의할 수 있어서 생략된 것임

</details>

#### 7.6.2.1. MySQL과 시퀄라이즈의 자료형 비교

- 시퀄라이즈의 자료형은 MySQL과 다르므로 모델 정의할 때 주의

|MySQL|Sequelize|
|---|---|
|VARCHAR(100)|STRING(100)|
|INT|INTEGER|
|TINYINT|BOOLEAN|
|DATETIME|DATE|
|INT UNSIGNED|INTEGER.UNSIGNED|
|NOT NULL|allowNull: false|
|UNIQUE|unique: true|
|DEFAULT now()|defaultValue: Sequelize.NOW|

#### 7.6.2.2. super.init 메서드의 두 번째 인자 옵션

|속성|설명|
|---|---|
|**sequelize**|`satic init` 메서드의 매개변수와 연결되는 옵션으로 db.sequelize 객체를 넣어야 함|
|**timestamp**|`true` 설정 시 시퀄라이즈는 `createdAt`과 `updatedAt` 컬럼을 추가하며 `row`의 추가 및 수정 시각이 자동으로 입력됨 |
|**underscored**|시퀄라이즈는 기본적으로 테이블명, 컬럼명을 `camel_case(낙타체)`로 바꾸는데, 본 속성을 true로 설정하면 이를 `snake_case(underscore_case)`로 바꿈|
|**modelName**|모델 이름을 설정할 수 있음. 노드 프로젝트에서 사용함|
|**tableName**|실제 데이터베이스의 테이블 이름 설정|
|**paranoid**|`true` 설정 시 `deletedAt` 컬럼을 추가하며 `row` 삭제 시 완전히 지워지지 않고 deletedAt에 지운 시각이 기록됨. row 조회 시 deletedAt의 값이 `null`인 row를 조회함|
|**charset, collate**|각각 `utf8`과 `utf8_general_ci`로 설정해야 한글이 입력됨. `이모티콘`까지 입력할 수 있게 하고 싶으면 `utf8mb4`, `utf8mb4_general_ci` 입력|

### 7.6.3. 관계 정의하기

#### 7.6.3.1. 1:N 관계

- `models/user.js`와 `models/comment.js`의 `associate` 메서드를 통해 `1:N(일대다)`
  관계 설정 가능

```js
// user.js
...
    static associate(db) {
        db.User.hasMany(db.User, {foreignKey: 'commenter', sourceKey: 'id'});
    }
};

// comment.js
...
    static associate(db) {
        db.Comment.belongsTo(db.User, {foreignKey: 'commenter', targetKey: 'id'});
    }
};
```

- 다른 모델의 정보가 들어가는 테이블에 `belongsTo` 사용
- 시퀄라이즈는 위에서 정의한 대로 모델 간 관계를 파악하여 `Comment` 모델에
  foreignKey(외래키)인 commenter 컬럼을 추가함
- `sourceKey`의 id와 `targetKey`의 id 모두 `User` 모델의 id를 참조함

<details>
    <summary>models/index.js 수정</summary>

```js
'use strict';

const Sequelize = require('sequelize');
const User = require('./user');
const Comment = require('./comment');

const env = process.env.NODE_ENV || 'development';
const config = require(__dirname + '/../config/config.json')[env];
const db = {};

const sequelize = new Sequelize(config.database, config.username, config.password, config);

db.sequelize = sequelize;
db.Sequelize = sequelize;

db.User = User;
db.Comment = Comment;

User.init(sequelize);
Comment.init(sequelize);

User.associate(db);
Comment.associate(db);

module.exports = db;
```

- npm start 시 

</details>

#### 7.6.3.2. 1:1 관계

- `hasMany` 대신 `hasOne` 메서드 사용

```js
db.User.hasOne(db.Info, {foreignKey: 'UserId', sourceKey: 'id'});
db.User.belongsTo(db.User, {foreignKey: 'UserId', targetKey: 'id'});
```

#### 7.6.3.3. N:M 관계

```js
db.Post.belongsToMany(db.Hashtag, {through: 'PostHashtag'});
db.hashtag.belongsToMany(db.Post, {through: 'PostHashtag'});
```

- 양쪽 모델 모두 `belongsToMany` 메서드 사용
- N:M 관계의 특성상 새로운 모델이 생성됨(through 속성에 새로운 모델의 이름을 적으면 됨)
  - 328p
- 자동으로 만들어진 `PostHashtag` 모델은 `db.sequelize.models.PostHashtag`로 접근 가능

> N:M 에서는 데이터를 조회할 때 여러단계를 거침
> - `#노드` 해시태그를 사용한 게시물을 조회하는 경우
>   1. `#노드`를 `Hashtag` 모델에서 조회
>   2. Hashtag 모델에서 가져온 \#node의 id를 바탕으로 PostHashtag 모델에서
>   Hashtag id가 1인 postId들을 찾아 PostHashtag 모델에서 hashtagId가 1인 postId
>   들을 찾아 Post 모델에서 정보를 가져옴

### 7.6.4 쿼리 알아보기

- 시퀄라이즈로 CRUD 작업을 하려면 먼저 시퀄라이즈 쿼리를 알아야 함
- 각 쿼리는 `프로미스`를 반환하므로 `then`을 붙여 `결과값`을 받을 수 있음

<details>
    <summary>JPA같은 시퀄라이즈</summary>

```js
// INSERT INTO nodejs.users (name, age, married, comment) VALUES ('zero', 24, 0, '자기소개1')
const { User } = require('../models');
User.create({
    name: 'zero',
    age: 24,
    married: false,
    comment: '자기소개1'
});

// SELECT * FROM nodejs.users;
User.findAll({});

// SELECT * FROM nodejs.users LIMIT 1;
User.findOne({});

// SELECT name, married FROM nodejs.users;
User.findAll({
    attributes: ['name', 'married']
});

// SELECT name, age FROM nodejs.users WHERE married = 1 AND age > 30;
const { Op } = require('sequelize'); // 시퀄라이즈에서 제공하는 연산자 모듈
const { User } = require('../models');
User.findAll({
    attributes: ['name', 'age'],
    where: {
        married: true,
        age: { [Op.gt]: 30 } // greater than
    }
});

// SELECT id, name FROM users WHERE married = 0 OR age > 30;
const { Op } = require('sequelize');
const { User } = require('../models');
User.findAll({
    attributes: ['id', 'name'],
    where: {
        [Op.or]: [{ married: false }, { age: { [Op.gt]: 30 } }]  
    }
});

// SELECT id, name FROM users ORDER BY age DESC;
User.findAll({
    attribute: ['id', 'name'],
    order: [['age', 'DESC']] 
});

// SELECT id, name FROM users ORDER BY age DESC LIMIT 1;
User.findAll({
    attributes: ['id', 'name'],
    order: [['age', 'DESC']],
    limit: 1
});

// SELECT id, name FROM users ORDER BY age DESC LIMIT 1 OFFSET 1;
User.findAll({
    attributes: ['id', 'name'],
    order: ['age', 'DESC'],
    limit: 1,
    offset: 1
});

// UPDATE nodejs.users SET comment = '바꿀 내용' WHERE id = 2;
User.update({
    comment: '바꿀 내용'
}, {
    where: { id: 2 }
});

// DELETE FROM nodejs.users WHEREid = 2;
User.destroy({
    where: { id: 2 }
})
```

> ### 주의
> - MySQL은 undefined 자료형을 지원하지 않으므로 where 옵션에는 undefined가 들어가면 안 됨

</details>

#### 7.6.4.1. 관계 쿼리

- `findOne`이나 `findAll` 메서드를 호출할 때 프로미스의 결과로 모델을 반환함(findAll은 배열로 반환)
- 관계 쿼리란 `JOIN` 기능을 말함
- User 모델은 Comment 모델과 hasMany-belongTo 관계가 맺어져 있으며
  특정 사용자를 가져오면서 그 사람의 댓글까지 모두 가져오고 싶다면 include 속성을 사용하면 됨

```js
const user = await User.findOne({
    include: [{
        model: Comment
    }]
});
console.log(user.Comments); // 헐

// 또 다른 방법
const user = await User.findOne({});
const comments = await user.getComments();
console.log(comments);
```

- 댓글은 여러 개일 수 있으므로(`hasMany`) `user.Comments`로 접근 가능
- 관계를 설정했다면 `setComments(수정)`, `addComment(하나 생성)`, `addComments(여러 개 생성)`,
  `removeComments(삭제)` 메서드를 지원함. 동사 뒤에 모델명이 붙는 형식
> - 동사 뒤에 모델 이름을 바꾸고 싶다면 관계 설정 시 as 옵션 사용 가능
> ```js
> // 관계를 설정할 때 as로 등록
> db.User.hasMany(db.Comment, { 
>     foreignKey: 'commenter', sourceKey: 'id', as: 'Answers' 
> });
> ```
> - as를 설정하면 include 시 추가되는 댓글 객체도 `user.Answers`로 바뀜

> - include, 관계 쿼리 메서드에도 `where`, `attributes` 같은 옵션 사용 가능
> ```js
> const user = await User.findOne({
>     include: [
>         model: Comment,
>         where: {
>             id: 1
>         },
>         attributes: ['id']
>     ]
> });
> // 또는
> const comments = await user.getComments({
>     where: {
>         id: 1
>     },
>     attributes: ['id']
> })
> ```
> 
> - 관계 쿼리 시 조회는 위와 같이 하지만 수정, 생성, 삭제는 다른점이 있음
> ```js
> const user = await User.findOne({});
> const comment = await Comment.create();
> await user.addComment(comment);
> // 또는
> await user.addComment(comment.id);
> await user.addComments([...]); // 여러 개를 추가할 때는 배열로 추가할 수 있음
> ```
> - 관계 쿼리 메서드의 인수로 추가할 댓글 모델을 넣거나 댓글의 아이디를 넣으면 됨

#### 7.5.4.2. SQL 쿼리하기

```js
const [result, metadata] = await sequelize.query('SELECT * FROM comments');
console.log(result);
```

------

[^1]: 키와 값들의 간단한 관계를 테이블화 시킨 매우 간단한 원칙의 전산정보 데이터 베이스  
[^2]: 자바스크립트 객체와 데이터베이스의 릴레이션을 매핑해주는 도구