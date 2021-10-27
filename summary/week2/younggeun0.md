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

### 클래스

### 프로미스

### async/ await

## 2.2 프론트엔드 자바스크립트

### AJAX

### FormData

### encodeURIComponent, decodeURIComponent

### 데이터 속성과 dataset



## 2.3 함께 보면 좋은 자료

* 책 참고...