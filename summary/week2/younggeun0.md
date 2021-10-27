# 2장 알아두어야 할 자바스크립트 (1/2)

---

## 2.1 ES2015+

* 2015년에 등장한 JS문법 == ES2015 == ES6

### const, let
  * var는 함수 스코프로 호이스팅이 되었음
  * const, let은 블록 스코프로 변수를 선언하는 키워드
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

### 화살표 함수

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