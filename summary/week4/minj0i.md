# 3장 노드 기능 알아보기

## 3.1 REPL 사용하기

읽고(Read), 해석하고(Eval), 결과물을 반환하고(Print), 종료할 때까지 반복한다(Loop)

```
    $ node
```

## 3.2 JS파일 실행하기

```
    $ node [자바스크립트 파일 경로]
```

## 3.3 모듈로 만들기

`모듈`이란 특정한 기능을 하는 함수나 변수들의 집합  
모듈로 만들면 재사용이 가능하다.  
보통 파일 하나가 모듈 하나가 됨  
모듈 하나가 여러 개의 모듈을 사용할 수 있으며, 모듈 하나가 여러 개의 모듈에 사용되기도 함  
(단점) 모듈 간 관계가 얽히면 구조 파악이 힘들어짐  
require 함수나 module 객체 등은 내장 객체

## 3.4 노드 내장 객체 알아보기

브라우저의 window 객체와 비슷

### 3.4.1 global

브라우저의 window와 같은 전역 객체이며, 전역 객체이므로 모든 파일에서 접근할 수 있으며 생략도 가능  
require 함수도 global.require에서 global이 생략된 것  
노드 콘솔에 로그를 기록하는 console 객체도 원래는 global.console

global 객체 내부에는 매우 많은 속성이 들어 있으며 내부를 보려면 REPL을 이용해야 함

**노드의 window, document 객체**

노드에 DOM이나 BOM이 없으므로 window와 document 객체는 노드에서 사용할 수 없다.  
노드에서 window 또는 document를 사용하면 에러가 발생합니다.

```JS
$ node
> global
{
  global: [Circular *1],
  clearInterval: [Function: clearInterval],
  clearTimeout: [Function: clearTimeout],
  ...
}
> global.console
{
  log: [Function: bound consoleCall],
  warn: [Function: bound consoleCall],
  dir: [Function: bound consoleCall],
  ...
}
```

전역 객체라는 점을 이용하여 파일 간에 간단한 데이터를 공유할 때 사용하기도 한다, 하지만 남용하면 나중에 유지보수 어려움

### 3.4.2 console
