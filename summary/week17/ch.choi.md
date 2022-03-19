# 14장 CLI 프로그램 만들기
[공식 홈페이지](https://nodejs.org/ko/)  

본 내용에 사용된 책 이미지는 [Node.js 교과서](https://book.naver.com/bookdb/book_detail.nhn?bid=16418778) 이미지를 참고하였습니다.


명령줄 인터페이스(Command Line Interface) 기반으로 동작하는 노드 프로그램을 만듬

CLI는 콘솔 창을 통해 프로그램을 수행하는 환경을 뜻함

반대 개념은 그래픽 사용자 인터페이스 (Graphic Uer Interface) 가 있음


## 14.1 간단한 콘솔 명령어 만들기

node 와 npm 명령어는 노드를 설치해야만 사용할 수 있지만, 신기하게 nodemon, rimraf와 같은 명령어는 해당 패키지를 npm을 통해 전역 설치 하면 콘솔에서 명령어로 사용할 수 있음 (뭐가 신기한거지?)

저런 명령어를 만드는 것이 이 장의 목표임

명령어로 만들고 싶은 패키지가 다른 사람의 소유라도 패키지명과 콘솔 명령어를 다르게 만들 수 있음 걱정 ㄴㄴ

예시
sequelize-cli 는 sequelize 명령어를 씀

```javascript
//index.js
#!/usr/bin/env node 
//윗 줄은 윈도우에서는 주석, 리눅스, 유닉스 환경은의미가 있음 
// /usr/bin/env에 등록된 node 명령어로 이 파일을 실행하라는 뜻
condole.log('Hello CLI');
```

위 예제는 단순히 Hello CLI 출력하는 파일

CLI 프로그램을 만들기 위해서는 package.json을 수정해야함

```json

{
  // ...
  "license": "ISC",
  "bin":{
    "cli":"./index.js"
  }
  //bin 속성을 추가 해야함
}
```

bin 속성을 통해 실행될 파일을 설정함

콘솔로 cli 치면 index.js 실행

테스트 하려면 현재 패키지를 전역 설치 해야함  
현재 패키지를 전역 설치 하려면 패키지명 없이 치면 됨

```cmd
npm i -g
```

패키지 설정을 했다면 콘솔에 cli 를 치게되면
Hello CLI 출력을 확인 할 수 있음

전역 패키지이므로 npx cli로 실행이 됨

노드에서 argv를 받는 방법은 process.argv 사용하면 됨

```javascript
#!/usr/bin/env node 
condole.log('Hello CLI', process.argv);
```

코드가 변경 되었으니 다시 전역 설치를 해야하는가? 놉

코드가 업데이트될 때마다 다시 설치할 필요는 없음

bin 속성으로 index.js가 설정 되었기 때문에 수정된 index.js가 실행 됨 (javascript의 장점)

process.argv에 배열로 입력값이 들어옴

간단하게 입력 받을 수 있는 readline 이라는 모듈을 써볼거임


```javascript
//index.js
#!/usr/bin/env node
const readline = require('readline');

const rl =readline.createInterface(
  {
    input: process.stdin,
    output: process.stdout,
  }
);
rl.question('fdljsdfljksdf',(answer)=>{
  if(answer === 'y')
  {

  }
  rl.close();
});

```

readline 모듈 가지고 html 만드는거 작업 하는게 있는데 코드 보면 바로 알 수 있으니 패쓰 별 다른게 없음

설치한 CLI 프로그램 삭제 방법

CLI 프로그램 삭제 방법은 간단
npm 전역 제거 명령어 호출 하면 가능

```cmd
npm rm -g [패키지명]
```

## 14.2 commander, inquirer 사용하기

CLI 프로그램을 만들기 위한 여러 라이브러리가 있음
대표적을 yargs, commander, meow 등등이 있음

commander를 이용해 예제 프로그램 제작 할 예정

```cmd
npm i commander@5 imquirer chalk
```

commander 패키지에서 program 객체를 불러 오고 다양한 메서드를 사용하면 됨

- version : 프로그램 버전을 설정
- usage : 메서드 사용법을 설정 
- name : 명령어의 이름
- command : 명렁어 설정
- description : 명령어에 대한 설명
- alias : 명령어 별칭 설정
- option : 명령어의 부가적 옵션 설정
- requiredOption  : 필수 옵션 지정 
- action : 명령어의 실제 동작 정의 메서드
- help : 설명서를 보여줌
- parse : program 객체 마지막에 붙이는 메서드, process.argv를 인수로 받아 파싱함


## 14.3 프로젝트 마무리 하기

### 14.3.1 스스로 해보기
해보세요~

### 14.3.2 핵심 정리
- 노드는 단순히 서버가 아니라 자바스크립트를 실행 하는 것을 기억
- npm에는 서버를 위한 패지키 뿐만 아니라 다양한 패키지도 준비가 되어 있음
- 명령어에 대한 설명을 자세히 적을 것
- 프로그래밍 할 때 반복되는 작업을 최소화 하는 프로그램을 제작 하자 (자동화가 짱이지)


