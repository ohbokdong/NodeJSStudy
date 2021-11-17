# 1. REPL (Read, Eval, Print, Loop)
* Node에서 '읽고, 해석하고, (결과물을) 반환하고, 이것을 다시 반복'하는 기능을 제공하는 콘솔을 의미합니다.

![](https://images.velog.io/images/developerelen/post/a9bc0c97-2ebc-4c8f-9919-3bce1b43f0f0/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA%202021-11-15%20%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE%2011.32.02.png)

* 위의 REPL을 Visual Studio에서 작성하면 아래와 같습니다.
helloWorld.js
```js
function helloWorld() {
    console.log('Hello world');
    helloNode();
}

function helloNode() {
    console.log('Hello Node');
}

helloWorld();

// 실행
$ node
> helloWorld
```

# 2. 모듈로 만들기
> 노드는 코드를 모듈로 만들수 있다는 점에서 자바스크립트와 다릅니다. 

## 모듈
: 특정한 기능을 하는 함수나 변수들의 집합
: 하나의 프로그램이면서 다른 프로그램의 부품으로도 사용할 수 있음 (재활용성)
: 파일 하나 -> 모듈 하나
: **module.exports = 담고싶은 객체, 함수, 변수**
![](https://images.velog.io/images/developerelen/post/5555e4d6-934c-4129-90ce-020bef0d9155/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA%202021-11-17%20%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE%203.15.48.png)
<var.js>
```js
const odd = '홀수입니다.';
const even = '짝수입니다.';

module.exports = {
    odd, 
    even,
}
```
<func.js>
```js
// 1. 불러올 모듈의 경로를 require() 함수에 집어넣습니다.
const { odd, even } = require('./var');

function checkOddOrEven(num) {
    if (num % 2) {
        return odd;
    }
 
    return even;
}

module.exports = checkOddOrEven;
```
<index.js>
```js
const { odd, even } = require('./var');
const checkNumber = require('./func');

function checkStringOddOrEven(str) {
    if(str.length % 2) {
        return odd;
    }
    return even;
}

console.log(checkNumber(10));
console.log(checkStringOddOrEven('hello'));
```
![](https://images.velog.io/images/developerelen/post/957dbef8-f260-44f7-98e2-81a1417e689d/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA%202021-11-17%20%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE%203.24.24.png)
# 3. 노드 내장 객체
## 3-1). global (전역객체)
1) window와 같은 전역 객체 (모든 파일에서 접근 가능)
2) global.require -> require()로 사용 가능 (global 생략 가능)
(모듈을 만들 때 require() 함수만 사용했음 / console 역시 global.console 이었음.) 
3) global 객체 내부에는 많은 속성이 들어있으며, 확인을 위해서 REPL을 사용해야 함

![](https://images.velog.io/images/developerelen/post/cbdd51f2-0c0b-46d4-a34a-1edf8d808088/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA%202021-11-16%20%E1%84%8B%E1%85%A9%E1%84%8C%E1%85%A5%E1%86%AB%2012.27.04.png)


## 3-2). console (디버깅용)
### 3-2-1) console.time(레이블)
: console.timeEnd(레이블)과 대응되어 같은 레이블을 가진 time과 timeEnd사이의 시간을 측정합니다.
: time과 timeEnd의 레이블이 같아야 시간 측정이 가능합니다. (코드 상 레이블 = '시간 측정')
```js
console.time('시간 측정');
for(let i = 0; i < 100000; i++){

}
console.timeEnd('시간 측정');

// 결과값
// 시간 측정: 26.825ms
```
### 3-2-2) console.log(내용, 내용, ...)
: 평범함 로그를 표시합니다. 여러개의 내용을 나열해서 표시할 수 있습니다.
```js
const string = 'abc';
const number = 1;
const boolean = true;

console.log(string, number, boolean);

// 결과값
// abc 1 true
```

### 3-2-3) console.error(에러 내용)
: 에러를 콘솔에 표시합니다.
```js
console.error('에러 메시지는 console.error에 담아주세요');
```

### 3-2-4) console.table(배열)
: 배열의 요소로 객체 리터럴을 넣으면, 객체의 속성들이 테이블 형식으로 표시됨
```js
console.table([{name:'제로', birth: 1994}, {name:'hero', birth: 1988}]);

//결과값
┌─────────┬────────┬───────┐
│ (index) │  name  │ birth │
├─────────┼────────┼───────┤
│    0    │ '제로'  │ 1994  │
│    1    │ 'hero' │ 1988  │
└─────────┴────────┴───────┘
```

### 3-2-5) console.dir(객체, 옵션)
: 객체를 콘솔에 표시할 때 사용
: colors 옵션에 true를 넣으면 아래 이미지와 같이 콘솔에 색이 추가됩니다.
: depth는 객체 안의 객체를 몇 단계까지 보여줄지를 결정합니다.
```js
console.dir(obj, {colors:false, depth: 2});
console.dir(obj, {colors:true, depth:1});
```
![](https://images.velog.io/images/developerelen/post/520c4969-833f-4cbc-9a73-3cb470e31c31/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA%202021-11-16%20%E1%84%8B%E1%85%A9%E1%84%8C%E1%85%A5%E1%86%AB%2012.50.08.png)

### 3-2-6) console.trace(레이블)
: 에러가 어디서 발생했는지 추적할 수 있게 함
```js
function b() {
    console.trace('에러 위치 추적');
}
function a() {
    b();
}
a();
```

## 3-3. 타이머
: 타이머 기능을 설정할 수 있으며, 아이디를 사용하여 취소할 수도 있습니다.
### 3-3-1) setTimeout(콜백 함수, 밀리초) --(취소)--> clearTimeout(아이디)
: 주어진 밀리초(1/1,000) 이후, 콜백 함수 실행

### 3-3-2) setInterval(콜백 함수, 밀리초) --(취소)--> clearInterval(아이디)
: 주어진 밀리초마다 콜백 함수를 반복 실행

### 3-3-3) setImmediate(콜백 함수) --(취소)--> clearImmediate(아이디)
: 콜백 함수를 즉시 실행
( setImmediate(콜백)은 보통 setTimeout(콜백, 0)보다 먼저 동작하지만 예외의 상황이 있을 수 있으니, setTimeout(콜백, 0)은 사용하지 않는 것을 권장 )
```js
const timeout = setTimeout(() => {console.log('1.5초 후 실행');}, 1500);
const interval = setInterval(() => {console.log('1초마다 실행');}, 1000);
const timeout2 = setTimeout(() => {console.log('실행되지 않습니다.');}, 3000);

setTimeout(() => {
  clearTimeout(timeout2); 
  clearInterval(interval);
}, 2500);

const immediate = setImmediate(() => {console.log('즉시 실행');});
const immediate2 = setImmediate(() => {console.log('실행되지 않습니다.');});

clearImmediate(immediate2);

// (결과값)
// 즉시 실행
// 1초마다 실행
// 1.5초 후 실행
// 1초마다 실행
```

## 3-4. \__filename, __dirname
: 파일의 경로나 파일명을 제공해줍니다.
: 파일에 \__filename과 __dirname을 넣어두면 실행 시 현재 파일명과 현재 파일 경로로 바뀝니다.
```js
console.log(__filename);
console.log(__dirname);
```

## 3-5. module, exports, require
: 모듈을 만들 때 **module 객체**말고도 **exports 객체**로도 만들 수 있음
: module.exports와 exports는 같은 객체를 참조하기 때문에 가능

## 3-5-1) 주의 할 점
1. exports 객체를 사용할 경우 module.exports와의 참고가 깨지지 않도록 해야함
2. exports는 객체처럼 속성명과 속성값을 대입해야 함
3. exports를 사용할 때는 객체만 사용할 수 있으므로 함수를 대입한 경우 exports로 바꿀 수 없음
4. 같은 객체를 바라보는 exports와 module.exports를 동시에 사용하지 않는 것을 권장함

<module 객체로 모듈 생성>
```js
const odd = '홀수입니다.';
const even = '짝수입니다.';

module.exports = {
    odd, 
    even,
}
```

<exports 객체로 모듈 생성>
```js
exports.odd = '홀수입니다.';
exports.even = '짝수입니다.';
```

## 3-5-2) this
1. 최상위 스코프에 존재하는 this = module.exports 
2. 함수 선언 내부의 this = global

```js
console.log(this);// (결과값) {}
console.log(this === module.exports)// (결과값) true
console.log(this == exports)// (결과값) true

function whatIsThis() {
  // (결과값) function false true
  console.log('function', this === exports, this === global);
}

whatIsThis();
```

## 3-5-3) require
### require.cache / require.main
: require = 함수 = 객체
: require가 반드시 파일 최상단에 위치할 필요가 없고, module.exports도 최하단에 위치할 필요가 없음
: 한 번 require한 파일은 require.cache에 저장되므로 다음 번에 require할 때에는 새로 불러오지 않고 require.cache에 있는 것이 재사용됨
: require.main은 노드 실행 시 첫 모듈 (아래 코드에서는 '/Users/dev/node/session 3/require.js')

1. require.cache
```js
console.log('require가 가장 위에 오지 않아도 됩니다.');

module.exports = '저를 찾아보세요.';

require('./var');

console.log('require.cache입니다.');
console.log(require.cache);

console.log('require.main입니다.');
//require.js에서 실행할 경우 true
// var.js에서 실행할 경우 false
console.log(require.main === module);
//첫 모듈의 이름 확인 require.main.filename
console.log(require.main.filename);
```
<결과값>
```js
require가 가장 위에 오지 않아도 됩니다.
require.cache입니다.
[Object: null prototype] {
   // 속성명 1: require.js 파일 이름        // 속성값: 각 파일의 모듈 객체
  '/Users/dev/node/session 3/require.js': Module {
    id: '.',
    path: '/Users/dev/node/session 3',
    exports: '저를 찾아보세요.',

    filename: '/Users/dev/node/session 3/require.js',
    loaded: false,
    children: [ [Module] ],
    paths: [
      '/Users/dev/node/session 3/node_modules',
      '/Users/dev/node/node_modules',
      '/Users/dev/node_modules',
      '/Users/node_modules',
      '/Users/node_modules',
      '/node_modules'
    ]
  },
   // 속성명 2: var.js 파일 이름        // 속성값: 각 파일의 모듈 객체
  '/Users/dev/node/session 3/var.js': Module {
    id: '/Users/dev/node/session 3/var.js',
    path: '/Users/dev/node/session 3',
    exports: { odd: '홀수입니다.', even: '짝수입니다.' },
   
    filename: '/Users/dev/node/session 3/var.js',
    loaded: true,
    children: [],
    paths: [
      '/Users/dev/node/session 3/node_modules',
      '/Users/dev/node/node_modules',
      '/Users/dev/node_modules',
      '/Users/node_modules',
      '/Users/node_modules',
      '/node_modules'
    ]
  }
}
require.main입니다.
true
/Users/dev/node/session 3/require.js
```

### require로 순환 참조(circular dependency)될 경우
: 2개 이상의 파일에서 서로를 참조할 경우, 순환 참조 되는 대상을 빈 객체로 만듦
: 에러가 떨어지지 않고 빈 객체로 바꾸므로 순환 참조 되지 않도록 신경써야 함!

1. dep1.js
```js
const dep2 = require('./dep2');
console.log('require dep2', dep2);
module.exports = () => {
    console.log('dep2', dep2);
};
```

2. dep2.js
```js
const dep1 = require('./dep1');
console.log('require dep1', dep1);
module.exports = () => {
    console.log('dep1', dep1);
};
```

3. dep-run.js
```js
const dep1 = require('./dep1');
const dep2 = require('./dep2');

dep1();
dep2();
```
<결과값>
```js
require dep1 {}
require dep2 [Function (anonymous)]
dep2 [Function (anonymous)]
dep1 {}
```

## 3-6. process
: process 객체는 현재 실행되고 있는 노드 프로세스에 대한 정보를 담고 있음
### 3-6-1) process.version
: 설치된 노드의 버전 확인
```js
> process.version
'v16.13.0'
```
### 3-6-2) process.arch
: 프로세서 아키텍처 정보. arm, ia32 등의 값일 수도 있음
```js
> process.arch
'x64'
```
### 3-6-3) process.platform
: 운영체제 플랫폼 정보. linux나 darwin, freebsd등의 값일 수도 있음
```js
> process.platform
'darwin'
```
### 3-6-4) process.pid
: 현재 프로세스의 아이디. 프로세스를 여러 개 가질 때 구분할 수 있음
```js
> process.pid
20789
```
### 3-6-5) process.uptime()
: 프로세스가 시작된 후 흐른 시간
```js
> process.uptime()
49.625620815
```
### 3-6-6) process.execPath
: 노드의 경로
```js
> process.execPath
'/Users/.nvm/versions/node/v16.13.0/bin/node
```
### 3-6-7) process.cwd()
: 현재 프로세스가 실행되는 위치
```js
> process.cwd()
'/Users/elen'
```
### 3-6-8) process.cpuUsage()
: 현재 cpu 사용량
```js
> process.cpuUsage()
{ user: 361297, system: 146418 }
```
### 3-6-9) process.env
: 환경 변수 출력 및 임의 저장 가능
: 서비스의 중요한 키를 저장하는 공간으로도 사용됨 (서버, 데이터베이스의 비밀번호 및 각종 API키 등)
: 서비스 키를 넣는 방법은 운영체제마다 차이가 있으나, detenv를 사용할 경우 모든 운영체제에 동일하게 넣을 수 있는 방법이 있음
```js
const secretId = process.env.SECRET_ID;
const secretCode = process.env.SECRET_CODE;
```
### 3-6-10) process.nextTick(콜백)
: 이벤트 루프가 다른 콜백 함수들보다 nextTick의 콜백 함수를 우선으로 처리하도록 만듦
: process.nextTick과 Promise를 마이크로태스크 (microtask)라고 부름
```js
setImmediate(() => {
    console.log('immediate');
});
process.nextTick(() => {
    console.log('nextTick');
});
setTimeout(() => {
    console.log('timeout');
}, 0);
Promise.resolve().then(()=>console.log('promise'));
```
<결과값>
```js
nextTick
promise
timeout
immediate
```
### 3-6-11) process.exit(코드)
: 실행 중인 노드 프로세스를 종료
: 서버 환경에서 이 함수를 사용하면 서버가 멈추므로 특수한 경우를 제외하고는 서버에서 잘 작동하지 않음
: 서버 이외의 독립적인 프로그램에서는 사용
```js
let i = 1;
setInterval(() => {
    if(i === 5) {
        console.log('종료!');
      // 인수로 코드번호를 줄 수 있음 (0: 종료, 1: 비정상 종료)
        process.exit();
    }
    console.log(i);
    i += 1;
}, 1000);
```

# 4. 노드 내장 모듈 사용하기
## 4-1) OS
: 운영체제의 정보를 가지고 올 수 있음
```js
const os = require('os');

console.log('운영체제 정보--------------------------------------------------');
console.log('os.arch(): ', os.arch());
console.log('os.platform(): ', os.platform());
//운영체제의 종류를 보여줌
console.log('os.type(): ', os.type());
// 운영체제 부팅 이후 흐른 시간(초)을 보여줌 (!process.uptime()은 노드의 실행시간임)
console.log('os.uptime(): ', os.uptime());
// 컴퓨터의 이름
console.log('os.hostime(): ', os.hostname());
// 운영체제의 버전
console.log('os.release(): ', os.release());
```
```js
console.log('경로---------------------------------------------------------');
// 홈 디렉터리 경로
console.log('os.homedir(): ', os.homedir());
// 임시 파일 저장 경로
console.log('os.tmpdir(): ', os.tmpdir);
```
```js
console.log('CPU 정보-----------------------------------------------------');
// 컴퓨터의 코어 정보
console.log('os.cpus(): ', os.cpus());
console.log('os.cpus().length: ', os.cpus().length);
```
```js
console.log('메모리 정보-----------------------------------------------------');
// 사용 가능한 메모리(RAM)
console.log('os.freemem(): ', os.freemem());
// 전체 메모리 용량
console.log('os.totalmem(): ', os.totalmem());
```
## 4-2) path
: 폴더와 파일의 경로를 쉽게 조작하도록 도와주는 모듈
: 운영체제별로 경로 구분자가 다르기 때문에 필요함
* 윈도타입: C:\Users\elen
* POSIX 타입 (맥, 리눅스): /home/elen

: 파일 경로에서 파일명이나 확장자만 따로 떼어주는 기능을 구현해두어 직접 구현하지 않고도 편리하게 사용가능

### 4-2-1) path.sep
: 경로의 구분자 (윈도: / , POSIX: \)
### 4-2-2) path.delimiter
: 환경 변수의 구분자 (윈도: ;, POSIX: :)
### 4-2-3) path.dirname(경로)
: 파일이 위치한 폴더 경로
### 4-2-4) path.extname(경로)
: 파일의 확장자
### 4-2-5) path.basename(경로, 확장자)
: 파일의 이름(확장자 포함)
: 파일의 이름만 표시하고 싶을 경우, basename의 두 번째 인수로 파일의 확장자를 넣으면 편함
( 예: path.basename(path, path.extname(path)) )
### 4-2-6) path.parse(경로)
: 파일 경로를 root, dir, base, ext, name으로 분리
### 4-2-7) path.format(객체)
: path.parse()한 객체를 파일 경로로 합침
### 4-2-8) path.normalize(경로)
: /나 \를 실수로 여러 번 사용했거나 혼용했을 때 정상적인 경로로 변환
### 4-2-9) path.isAbsolute(경로)
: 파일의 경로가 절대 경로인지 상대 경로인지를 true / false로 알려줌
### 4-2-10) path.relative(기준경로, 비교경로)
: 경로를 두 개 넣으면 첫 번째 경로에서 두 번째 경로로 가는 방법을 알려줌
### 4-2-11) path.join(경로, ...)
: 여러 인수를 넣으면 하나의 경로로 합침
### 4-2-12) path.resolve(경로, ...)
: path.join()과 비슷하지만 차이가 있음. 
: /를 만날 시
* path.resolve: 절대 경로로 인식해서 앞의 경로 무시
* path.join: 상대 경로로 처리
```js
path.join('/a', '/b', 'c'); // (결과값) /a/b/c/
path.resolve('/a', '/b', 'c'); // (결과값) /b/c
```
## 4-3) url
: 인터넷 주소를 쉽게 조작하도록 도와주는 모듈

1) 노드 버전7에서 추가된 WHATWG 방식의 url 
* search 부분을 searchParamas라는 특수한 객체로 반환해줌
* search = ?'키=값'&'키=값'&... (예: ?sercate1=001001000#anchor)


2) 예전 방식의 url

<WHATWG와 노드의 주소 체계>
![](https://images.velog.io/images/developerelen/post/6d5b79aa-1ff0-4869-adbf-a3320de4bb69/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA%202021-11-17%20%E1%84%8B%E1%85%A9%E1%84%8C%E1%85%A5%E1%86%AB%209.42.07.png)

* url.parse(주소) : 주소를 분해함. WHATWG 방식과 비교하면 username과 password 대신 auth 속성이 있고, searchParams 대신 query가 있음
* url.format(객체) : WHATWG방식. url과 기존 노드의 url을 모두 사용할 수 있음. 분해되었던 url객체를 다시 원래 상태로 조립함.

* 노드의 url은 취향에 따라 사용하면 되지만, 노드 url의 형식을 꼭 사용해야 하는 경우가 있음.
  * host 부분 없이 pathname 부분만 오는 주소인 경우 WHATWG 방식이 처리할 수 없음


```js
const url = require('url');
const { URL } = url;
const myURL = new URL('http://www.gilbut.co.kr/book/bookList.aspx?sercate1=001001000#anchor');
console.log('new URL():', myURL);
console.log('url.format():', url.format(myURL));
console.log('------------------------------');
const parsedUrl = url.parse('http://www.gilbut.co.kr/book/bookList.aspx?sercate1=001001000#anchor');
console.log('url.parse():', parsedUrl);
console.log('url.format():', url.format(parsedUrl));
```

<결과>
```js
new URL(): URL {
  href: 'http://www.gilbut.co.kr/book/bookList.aspx?sercate1=001001000#anchor',
  origin: 'http://www.gilbut.co.kr',
  protocol: 'http:',
  username: '',
  password: '',
  host: 'www.gilbut.co.kr',
  hostname: 'www.gilbut.co.kr',
  port: '',
  pathname: '/book/bookList.aspx',
  search: '?sercate1=001001000',
  searchParams: URLSearchParams { 'sercate1' => '001001000' },
  hash: '#anchor'
}
url.format(): http://www.gilbut.co.kr/book/bookList.aspx?sercate1=001001000#anchor
------------------------------
url.parse(): Url {
  protocol: 'http:',
  slashes: true,
  auth: null,
  host: 'www.gilbut.co.kr',
  port: null,
  hostname: 'www.gilbut.co.kr',
  hash: '#anchor',
  search: '?sercate1=001001000',
  query: 'sercate1=001001000',
  pathname: '/book/bookList.aspx',
  path: '/book/bookList.aspx?sercate1=001001000',
  href: 'http://www.gilbut.co.kr/book/bookList.aspx?sercate1=001001000#anchor'
}
```

```js
const { URL } = require('url');

// URL 생성자로 주소 객체 생성 (주소에 search 객체 있음)
const myURL = new URL('http://www.gilbut.co.kr/?page=3&limit=10&category=nodejs&category=javascript');

// myURL 안에는 searchParams이라는 객체가 있고 이 객체는 search 부분을 조작하는 다양한 메서드를 지원함. 
console.log('searchParams: ', myURL.searchParams);

// 1. getAll(키): 키에 해당하는 모든 값들을 가져옴 
// category키에는 nodejs와 javascript라는 두가지 값이 있음
console.log('searchParams.getAll(): ', myURL.searchParams.getAll('category'));

// 2. get(키): 키에 해당하는 첫번째 값만 가져옴
console.log('searchParams.get():', myURL.searchParams.get('limit'));

// 3. has(키): 해당 키가 있는지 없는지를 검사
console.log('searchParams.has():', myURL.searchParams.has('page'));

// 4. keys(): searchParams의 모든 키를 반복키(iterator) 객체로 가져옴
console.log('searchParams.keys():', myURL.searchParams.keys());

// 5. values(): searchParams의 모든 값을 반복키 객체로 가져옴
console.log('searchParams.values():', myURL.searchParams.values());

// 6. append(키, 값): 해당 키를 추가함. 같은 키의 값이 있다면 유지하고 하나 더 추가함
myURL.searchParams.append('filter', 'es3');
myURL.searchParams.append('filter', 'es5');
console.log(myURL.searchParams.getAll('filter'));

// 7. set(키, 값): append와 비슷하지만, 같은 키의 값들을 모두 지우고 새로 추가함
myURL.searchParams.set('filter', 'es6');
console.log(myURL.searchParams.getAll('filter'));

// 8. delete(키): 해당 키를 제거
myURL.searchParams.delete('filter');
console.log(myURL.searchParams.getAll('filter'));

// 9. toString(): 조작한 searchParamas 객체를 다시 문자열로 만듦
console.log('searchParams.toString():', myURL.searchParams.toString());
myURL.search = myURL.searchParams.toString();
```

## 4-4) queryString
: WHATWG 방식의 url 대신 기존 노드의 url을 사용할 때, search 부분을 사용하기 쉽게 객체로 만드는 모듈

* querystring.parse(쿼리): url의 query부분을 자바스크립트 객체로 분해
* querystring.stringfy(객체): 분해된 query객체를 문자열로 다시 조립
```js
const url = require('url'); 
const querystring = require('querystring'); 

const parsedUrl = url.parse('http://www.gilbut.co.kr/?page=3&limit=10&category=nodejs&category=javascript'); 
const query = querystring.parse(parsedUrl.query);
// (결과값)
// querystring.parse(): [Object: null prototype] {
//   page: '3',
//   limit: '10',
//   category: [ 'nodejs', 'javascript' ]
// }
console.log('querystring.parse():', query);
console.log('querystring.stringify():', querystring.stringify(query));
// (결과값)
//querystring.stringify():page=3&limit=10&category=nodejs&category=javascript
```

출처: Node.js 교과서 - 길벗
