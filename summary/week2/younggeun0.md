# 2장 알아두어야 할 자바스크립트 (1/2)

---

## 2.1 ES2015+

* 2015년에 등장한 JS문법 == ES2015 == ES6

### const, let
  * var는 함수 스코프, 호이스팅이 됐음
  * const, let은 블록 스코프
  * const로 선언한 변수는 상수, let은 재할당 가능
  * 기본적으로 const로 변수 선언 후 값을 할당해야 하는 경우 let으로 바꿔쓰는걸 추천

```js
if (true) {
    var x = 3; // var는 함수 스코프
}

console.log(x); // 3

if (true) {
    const y = 3; // const, let은 블럭 스코프
}

console.log(y); // Uncaught ReferenceError: y is not defined

const a = 0; // const는 재할당 불가
a = 1; // Uncaught TypeError: Assignment to constant variable.

let b = 0; // let은 재할당 가능
b = 1; // 1

const c; //Uncaught SyntaxError: Missing Initializer in const declaration
```

### 템플릿 문자열

* 기존 문자열은 큰따옴표(")나 작은 따옴표(')로 감싸야 했고 변수를 문자열에 넣을 때 '+'연산을 해야 했지만 백틱(\`)으로 감싸고 문자열 안에 `${변수}`를 넣어 간단히 문자열로 만들 수 있는 템플릿 문자열이 추가됨
  * 이스케이프 처리도 필요없음

```js
var n1 = 1;
var n2 = 2;
var ret = 3;

var str = n1 + ' 더하기 ' + n2 + '는 \"' + result + '\"';

// 아래처럼 간단히 변수를 문자열에 넣어 사용가능
var str2 = `${n1} 더하기 ${n2} 는 "${result}"`;
```

### 객체 리터럴

* 객체 리터럴에 편리한 기능이 추가됨
* 속성명과 변수명이 동일한 경우, 한 번만 써도 됨

```js
{ name : name, age : age } // ES5
{ name, age } // ES2015
```

* 객체 속성명을 동적으로 생성 가능
  * ES5에선 객체 리터럴 바깥에서 속성명을 동적으로 설정해야만 했음

```js
var logNode = function() {
    console.log('Node');
};
var oldObj = {
    logJS : function() {
        console.log('JS');
    },
    logNode : logNode
};
var es = 'ES';
oldObj[es + 6] = 'Fantastic';

var newObj = {
    logJS : function() {
        console.log('JS');
    },
    logNode,
    [es + 6] : 'Fantastic'
};

// 결과 동일
oldObj.logNode();
oldObj.logJS();
console.log(oldObj.ES6);

newObj.logNode();
newObj.logJS();
console.log(newObj.ES6);
```

### 화살표 함수

* 화살표 함수에선 function 선언 대신 `=>` 기호로 함수를 선언

```js
function add1(x, y) {
    return x + y;
}

const add2 = (x, y) => {
    return x + y;
}

const add3 = (x, y) => x + y;
const add4 = (x, y) => (x + y);
```

* 매개변수가 한 개면 소괄호로 묶어주지 않아도 됨

```js
const not2 = x => !x;
```

* **기존 function과 다른 점은 this 바인드 방식**
  * logFriends란 함수 내 function 선언문/ 화살표 함수 사용 시 this가 의미하는(참조하는) 객체가 다름
  * **function 선언문 this : window**
      * 바깥 상위 스코프를 받기 위해선 클로저를 활용, 별도의 변수를 선언하고 참조해 사용해야 함
  * **화살표 함수 this : rel2**
      * 바깥 상위 스코프의 this를 그대로 물려 받음 

```js
var rel1 = {
    name: 'young',
    friends: ['f1', 'f2', 'f3'],
    logFriends: function() {
        var that = this; // rel1을 가리키는 this를 that에 저장
        this.friends.forEach(function (friend) {
            debugger; // this == window
            console.log(that.name, friend);
        });
    },
};

rel1.logFriends();

var rel2 = {
    name: 'geun',
    friends: ['f1', 'f2', 'f3'],
    logFriends: function() {
        this.friends.forEach(friend => {
            debugger; // this == rel2
            console.log(this.name, friend);
        });
    },
};

rel2.logFriends();
```

* 기본적으로 화살표 함수를 쓰되, this를 사용해야 하는 경우 화살표 함수와 함수 선언문(function) 중 하나를 고름 됨

### 구조분해 할당

* 구조분해 할당을 사용하면 객체와 배열로부터 속성이나 요소를 쉽게 꺼낼 수 있음
* 코드 줄 수를 상당히 줄여주므로 유용

```js
var candyMachine = {
    status : {
        name: 'node',
        count: 5,
    },
    getCandy: function() {
        this.status.count--;
        return this.status.count;
    },
};

var getCandy = candyMachine.getCandy;
var count = candyMachine.status.count;

// == 
const { getCandy, status: { count }} = candyMachine;
// candyMachine 객체 안의 속성을 찾아 변수와 매칭함
// count처럼 여러 단계 안의 속성도 찾을 수 있음
// 단, 구조분해 할당을 사용하면 함수의 this가 달라질 수 있음, 달라진 this를 원래대로 바꿔주려면 bind 함수를 따로 사용해야 함

// 배열의 구조 분해 할당
var arr = ['node', {}, 10, true};
var node = arr[0];
var obj = arr[1];
var bool = arr[3];

const arr2 = ['node', {}, 10, true];
const [node, obj, , bool] = arr2;
```

### 클래스

* 다른 언어처럼 클래스 기반으로 동작하는 것은 아니고 프로토타입 기반으로 동작하지만 보기 좋게 클래스로 바꾼 것

```js
// 상속 예제
var Human = function(type) { // Human 생성자 함수
    this.type = type || 'human';
};

Human.isHuman = function(human) {
    return human instanceof Human;
};

Human.prototype.breathe = function() {
    alert("haaaam");
};

var Young = function(type, firstName, lastName) {
    Human.apply(this, arguments); // 상속하는 부분
    this.firstName = firstName;
    this.lastName = lastName;
};

Young.prototype = Object.create(Human.prototype); // Human의 prototype을 Young이 상속함
Young.prototype.constructor = Young;
Young.prototype.sayName = function() {
    alert(this.firstName + " " + this.lastName);
};

var oldYoung = new Young("human", "young", "oh");
Human.isHuman(oldYoung); // true
```


* 위 코드를 클래스 기반 코드로 바꾼 결과
  * 생성자 함수는 constructor 안으로 들어감
  * isHuman같은 클래스 함수는 static 키워드로 전환됨
  * 프로토타입 함수들도 모두 class 블록 안에 포함돼 어떤 함수가 어떤 클래스 소속인지 보기 쉬워짐
  * 상속도 extends 키워드로 쉽게 가능
  * **단, 클래스 문법으로 바뀌었더라도 자바스크립트는 프로토타입 기반으로 동작!**

```js
class Human {
    constructor(type = 'human') {
        this.type = type;
    }

    static isHuman(human) {
        return human istanceof Human;
    }

    breathe() {
        alert("Haaaam");
    }
}

class Young extends Human {
    constructor(type, firstName, lastName) {
        super(type);
        this.firstName = firstName;
        this.lastName = lastName;
    }

    sayName() {
        super.breathe();
        alert(`${this.firstName} ${this.lastName}`);
    }
}

const newYoung = new Young('human', 'young', 'oh');
Human.isHuman(newYoung); // true
```

### 프로미스


* ES2015부터 JS와 노드의 API들이 콜백 대신 프로미스(Promise) 기반으로 재구성돼 콜백 지옥(Callback hell) 현상을 극복했다는 평가를 받음
* **프로미스는 쉽게 설명하면 실행은 바로하되, 결과값은 나중에 받는 객체**
  * 결과값은 실행이 완료된 후 then이나 catch 메서드를 통해 받음
  * new Promise는 바로 실행되지만, 결과값은 then을 붙였을 때 받게 됨
* 프로미스 규칙
  * 먼저 프로미스 객체를 생성, 내부에 resolve와 reject를 매개변수로 갖는 콜백함수를 넣음
  * promise 변수에 then과 catch 메서드를 붙일 수 있음
  * 프로미스 내부에서 resolve가 호출되면 then이 실행되고, reject가 호출되면 catch가 실행됨
    * resolve와 reject에 넣어준 인수는 각각 then과 catch의 매개변수로 받을 수 있음
  * finally 부분은 성공/실패 상관없이 실행됨

```js
const condition = true; //true면 resolve, false면 reject
const promise = new Promise((resolve, reject) => {
    if (condition) {
        resolve('성공');
    } else {
        reject('실패');
    }
});

// 다른 코드가 들어갈 수 있음 (new Promise와 promise.then 사이에 다른 코드가 들어갈 수도 있음)
promise
    .then((message) => {
        console.log(message); // 성공(resolve)한 경우 실행
    })
    .catch((error) => {
        console.error(error); // 실패(reject)한 경우 실행
    })
    .finally(() => {
        console.log('무조건');
    });
```


* then이나 catch에서 다시 다른 then이나 catch를 붙일 수 있음
  * 이전 then의 return 값을 다음 then의 매개변수로 넘김
    * 단, then에서 new Promise를 return해야 다음 then에서 받을 수 있음
  * 프로미스를 return 한 경우, 프로미스가 수행된 후 다음 then이나 catch가 호출됨

```js
promise
    .then((message) => {
        return new Promise((resolve, reject) => {
            resolve(message);
        });
    })
    .then((message2) => {
        console.log(message2);
        return new Promise((resolve, reject) => {
            resolve(message2);
        });
    })
    .then((message3) => {
        console.log(message3);
    })
    .catch((error) => {
        console.error(error);
    });
```


* 콜백을 쓰는 패턴 예제1
  * 콜백 함수가 세 번 중첩된 상태, 콜백 함수가 나올 때마다 코드의 깊이가 깊어지고 각 콜백 함수마다 에러도 따로 처리를 해줘야 함

```js
function findAndSaveUser(Users) {
    Users.findOne({}, (err, user) => { // 첫 번째 콜백
        if (err) {
            return console.error(err);
        }

        user.name = 'young';
        user.save((err) => { // 두 번째 콜백
            if (err) {
                return console.error(err);
            }

            User.findOne({ gender: 'm' }, (err, user) => { // 세 번째 콜백
                // 생략...
            });
        });
    });
}
```

* 코드의 깊이가 세 단계 이상 깊어지지 않음
  * then 메서드들은 순차적으로 실행됨
  * 콜백에서 매번 따로 처리했던 에러도 마지막 catch에서 한 번에 처리가능
  * 메서드가 프로미스 방식을 지원할 때만 아래와 같이 바꿀 수 있음
    * findOne과 save메서드 내부적으로 프로미스 객체를 가지고 있다고 가정했기 때문에 가능(new Promise)

```js
function findAndSaveUser(Users) {
    Users.findOne({})
        .then(user => {
            user.name = 'young';
            return user.save();
        })
        .then(user => {
            return Users.findOne({ gender: 'm' });
        })
        .then(user => {
            // 생략...
        })
        .catch(err => {
            console.error(err);
        });
}
```


* 프로미스 여러 개를 한 번에 실행할 수 있는 방법도 있음
  * 기존 콜백 패턴이었다면 콜백을 여러 번 중첩해서 사용해야 함, 하지만 Promise.all을 이용하면 간단히 가능

```js
// Promise.resolve는 즉시 resolve하는 프로미스를 만드는 방법, 즉시 reject하는 Promise.reject도 있음
const promise1 = Promise.resolve("성공1");
const promise2 = Promise.resolve("성공2");

// 모든 프로미스가 resolve 될 때까지 기다린 후 then으로 넘어감
Promise.all([promise1, promise2])
    .then(result => {
        // result 매개변수에 각각 프로미스 결과값이 배열로 들어옴
        console.log(result); // ['성공1', '성공2']
    })
    .catch(error => {
        // 하나라도 reject되면 catch로 넘어감
        console.error(error);
    });
```

### async/ await

* 노드 7.6버전부터 지원되는 기능, ES2017에서 추가됨, 노드처럼 비동기 위주 프로그래밍 시 많이 도움됨
* 프로미스가 콜백지옥을 해결했지만 then과 catch가 반복되기 때문에 여전히 코드가 장황함
* async/ await 문법은 프로미스를 사용한 코드를 한 번 더 깔끔하게 줄임
  * 일반 함수 대신 async function으로 교체
  * 프로미스 앞에 await을 붙임
    * 함수가 해당 프로미스를 resolve될 때까지 기다린 뒤 다음 로직으로 넘어감
  * 에러는 부분은 try/catch 문 catch에서 처리

```js
async function findAndSaveUser(Users) {
    try {
        let user = await Users.findOne({});
        user.name = 'young';
        user = await user.save();
        user = await Users.findOne({ gender: 'm' });
        // 생략...
    } catch (error) {
        console.log(error);
    }
}

// 화살표 함수도 async와 같이 사용 가능
const findAndSaveUser = async (Users) => {
    try {
        let user = await Users.findOne({});
        user.name = 'young';
        user = await user.save();
        user = await Users.findOne({ gender : 'm' });
        // 생략
    } catch (error) {
        console.error(error);
    }
};
```


* for문과 async/ await 함수를 같이 써서 프로미스를 순차적으로 실행 가능
  * for문을 함께 쓰는 것도 노드 10버전부터 지원하는 ES2018 문법
  * 아래는 for await of 문을 사용해서 프로미스 배열을 순회하는 모습
  * async 함수의 반환값은 항상 Promise로 감싸짐, 따라서 실행 후 then을 붙이거나 또다른 async 함수 안에서 awiat을 붙여 처리 가능

```js
const promise1 = Promise.resolve("성공1");
const promise2 = Promise.resolve("성공2");

(async () => {
    for await (promise of [promise1, promise2]) {
        console.log(promise);
    }
})();
```

* 앞으로 중첩되는 콜백함수가 있다면 프로미스를 거쳐 async/await 문법으로 바꾸는 연습을 추천(코드가 간결해짐)

```js
async function findAndSaveUser(Users) {
    // 생략
}
findAndSaveUser().then(() => { /* 생략 */ });
// 또는
async function other() {
    const result = await findAndSaveUser();
}
```

## 2.2 프론트엔드 자바스크립트

* 프론트엔드에 사용되는 기능들을 설명
* 예제 코드를 위해 몇가지 소개

### AJAX

* 비동기적 웹 서비스를 개발할 때 사용하는 기법
* **페이지 이동 없이 서버에 요청을 보내고 응답을 받는 기술**
* 보통 AJAX 요청은 jQuery나 axios 같은 라이브러리를 이용해서 보냄
  * 브라우저에서 XMLHttpRequest 객체를 제공하긴 하지만 사용방법이 복잡하고 서버에서는 사용할 수 없으므로 책에서는 axios를 사용
* 프론트앤드에서 사용하려면 HTML 파일을 만들고 script 태그를 추가해야 함

```html
<!-- front.html -->
<script src="https://cdn.jsdelivr.net/npm/axios/dist/axios.min.js"></script>
<script>
    // js code
</script>
```


* axios.get 함수의 인수로 요청을 보낼 주소를 넣음
  * axios.get도 내부에 new Promise가 들어있으므로 then과 catch를 사용할 수 있음
  * result.data에는 서버로부터 보낸 데이터가 들어있음

```js
axios.get('https://www.zerocho.com/api/get')
    .then(result => {
        console.log(result);
        console.log(result.data); // {}
    })
    .catch(error => {
        console.error(error);
    });
```

![img1](https://github.com/ohbokdong/NodeJSStudy/blob/main/summary/week2/img/img1.png?raw=true)


* 프로미스이므로 async/ await 방식으로 변경 가능
  * 익명함수라서 즉시 실행을 위해 코드를 소괄호로 감싸서 호출한 예제

```js
(async () => {
    try {
        const result = await axios.get('https://www.zerocho.com/api/get');
        console.log(result);
        console.log(result.data);
    } catch (error) {
        console.error(error);
    }
})();
```

* POST 방식의 요청 예시
  * POST 요청에서는 데이터를 서버로 보낼 수 있음, 두 번째 인수로 데이터를 넣어 보내는 것이 다름
  * 아래 코드는 CORS로 응답이 안돼 확인불가

```js
(async () => {
    try {
        const result = await axios.post('https://www.zerocho.com/api/post/json', {
            name: 'zerocho',
            birth: '1994',
        });
        console.log(result);
        console.log(result.data);
    } catch (error) {
        console.error(error);
    }
})();
```

### FormData

* HTML form 태그의 데이터를 동적으로 제어할 수 있는 기능, AJAX와 주로 함께 사용됨
* 먼저 FormData 생성자로 formData 객체를 만듦
  * 생성된 객체의 append 메서드로 키-값 형식의 데이터를 저장 가능
  * append 메서드를 여러번 사용해서 키 하나에 여러 개의 값을 추가 가능
  * has 메서드는 주어진 키에 해당하는 값이 있는 여부를 반환
  * get 메서드는 주어진 키에 해당하는 값 하나를 가져옴
  * getAll 메서드는 해당하는 모든 값을 가져옴
  * delete 메서드는 현재 키를 제거하는 메서드
  * set은 현재 키를 수정하는 메서드

```js
const formData = new FormData();

formData.append('name', 'young');
formData.append('item', 'orange');
formData.append('item', 'melon');

formData.has('item'); // true
formData.has('money'); // false

formData.get('item'); // orange
formData.getAll('item'); // ['orange', 'melon']

formData.append('test', ['hi', 'zero']);
formData.get('test'); // hi, zero

formData.delete('test');
formData.get('test'); // null
formData.set('item', 'apple');
formData.getAll('item'); // ['apple']
```

* formData를 axios로 서버에 보냄

```js
(async () => {
    try {
        const formData = new FormData();
        formData.append('name', 'zerocho');
        formData.append('birth', 1994);

        const result = await axios.post('https://www.zerocho.com/api/post/formdata', formData);
        console.log(result);
        console.log(result.data);
        
    } catch (error) {
        console.error(error);
    }
})();
```

### encodeURIComponent, decodeURIComponent

* AJAX 요청을 보낼 때 URL에 한글이 들어가는 경우가 있는데 서버 종류에 따라 서버가 한글을 이해하지 못하는 경우가 존재
    * 이럴 땐 window 객체의 [encodeURIComponent](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/encodeURIComponent) 메서드를 사용
        * encodeURIComponent 메서드는 URI의 특정한 문자를 UTF-8로 인코딩해 URI 구성요소로 인코딩한 새로운 문자열을 반환함

```js
(async () => {
    try {
        const result = await axios.get(`https://www.zerocho.com/api/search/${encodeURIComponent('노드')}`);


        console.log(result);
        console.log(result.data);
    
    } catch (error) {
        console.error(error);
    }
})();
```

* 받는 쪽에서는 decodeURIComponent를 사용함

![img2](https://github.com/ohbokdong/NodeJSStudy/blob/main/summary/week2/img/img2.png?raw=true)

### 데이터 속성과 dataset

* 노드를 웹 서버로 사용하는 경우, 클라이언트(프론트앤드)와 빈번하게 데이터를 주고받게 됨
  * 이 때 서버에서 보내준 데이터를 프론트앤드 어디에 넣어야 할지 고민하게 됨
* 프론트앤드에 데이터를 내려보낼 때 첫 번째로 고려할 점은 보안
  * 클라이언트(프론트앤드)에 민감한 데이터를 내려보내는 것은 위험!
  * 보안과 무관한 데이터들은 자유롭게 보내도 됨
* JS 변수에 저장해도 되지만 HTML5에도 HTML과 관련된 데이터를 저장하는 공식적인 방법이 있음
* **데이터 속성(Data Attribute)**
  * HTML 태그 속성으로 data-로 시작하는 것들을 넣는 것
  * 데이터 속성의 장점은 JS로 쉽게 접근 가능한 점
    * `data-` 접두어는 사라지고 -뒤에 위치한 글자는 대문자가 됨
  * 반대로 dataset에 데이터를 넣어도 HTML 태그에 반영됨
    * dataset.monthSalary = 10000;을 넣으면 data-month-salary="10000"이라는 속성이 생김

```html
<ul>
    <li data-id="1" data-user-job="programmer">Zero</li>
    <li data-id="2" data-user-job="coder">Young</li>
    <li data-id="3" data-user-job="anotherCoder">Chung</li>
</ul>
<script>
    console.log(document.querySelector('li').dataset);
    // { id; '1', userJob: 'programmer' }
    document.querySelector('li').dataset.monthSalary = 10000;
</script>
```

![img3](https://github.com/ohbokdong/NodeJSStudy/blob/main/summary/week2/img/img3.png?raw=true)

## 2.3 함께 보면 좋은 자료

* 책 참고...
