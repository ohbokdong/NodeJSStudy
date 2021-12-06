# 5장 패키지 매니저
[공식 홈페이지](https://nodejs.org/ko/)  

본 내용에 사용된 책 이미지는 [Node.js 교과서](https://book.naver.com/bookdb/book_detail.nhn?bid=16418778) 이미지를 참고하였습니다.

## 5.1 npm 알아보기
이름 그래도 노드 패키지 매니저  
대부분의 자바 스크립트 프로그램은 패키지라는 이름으로 npm에 등록되어 있음. 필요한 패키지가 있다면 npm에서 찾아 설치 하면 됨  
대부분 오픈 소스고 웹을 개발할 때 많은 도움이 됨.

패키지가 다른 패키지를 사용 할 수도 있음 -> 의존 관계가 있음

------

### NOTE
npm 대체자로 yarn이 있음. 페이스 북이 만든 패키지 매니저  
리액트나 리액트 네이티브 같은 페이스북 진영 프레임 워크를 사용할 때 종종 쓰임. 

------

## 5.2 package.json으로 패키지 관리하기

서비스에 필요한 패키지를 추가하다 보면 양이 많아짐. 사용하는 패키지 마다 고유한 버전이 있으며 이를 어디서 관리 해야함.  
패키지를 설치 할 때 버전이 달라 문제가 발생 하기도 함.  
이때 버전을 관리 하기 위한 파일로 package.json을 사용

노드 프로젝트 시작 할때 package.json을 만들고 시작 하자!  
npm은 package.json을 만드는 명령어를 제공함

![커맨드 샘플](https://github.com/ohbokdong/NodeJSStudy/blob/main/summary/week7/sample.PNG)

```json
//npm init 해서 작성된 결과파일
//package.json
{
  "name": "testproject", //package name
  "version": "1.0.0", //package version
  "description": "node test project",
  "main": "cluster.js", //entry point 
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1", //test command
    "start": "node server.js"
  },
  "author": "chchoe",
  "license": "ISC"
}
```

| 이름  | 내용  |
|---|---|
| package name | 패키지의 이름 |
|version|패키지의 버전 (npm의 버전은 다소 엄격하게 관리됨 5.3장 ㄱㄱ)|
|entry point |자바스크립트 실행 파일 진입점. 보통 module.exports를 하는 파일을 지정|
|test command | 코드를 테스트 할 때 입력할 명령어|
|git repository | 코드를 저정해둔 깃 저장소 주소를 의미|
|keywords | 키워드는 npm 공식 홈페이지에서 패키지를 쉽게 찾을 수 있게 해줌|
|license|해당 패키지의 라이센스|
------
------
### NOTE
오픈 소스라고 해서 모든 패키지가 아무런 제약이 업는것은 아님.  
라이센스별 제한 사항이 있으므로 설치 전 확인 할 것  
ISC, MIT,BSD 라이센스는 사용 패키지와 라이센스를 밝히면 사용 가능  

GPL은 조심해야함 배포할 때 자신의 패키지도 공개 해야함

------

## 명령어 

## npm run test 
package.json에 test command를 실행 시킴  
(npm start로 scripts의 start 를 실행 할 수도 있음)

## npm install <package name>
설치 하고 싶은 패키지를 설치. package.json이 있는 위치에서 해야 package.json에 추가 됨  

```json
{
  "name": "testproject",
  "version": "1.0.0",
  "description": "node test project",
  "main": "cluster.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "start": "node server.js"
  },
  "author": "chchoe",
  "license": "ISC",
  "dependencies": {
    "express": "^4.17.1" //새로 추가된 모습
  }
}
```

dependencies 라는 속성이 생겼고 설치된 패키지 이름과 버전이 저장됨  
버전 앞에 ^표시가 붙는데 특별한 의미가 있다고 함  

------
### NOTE
--save 안붙혀도 됨. ㅅㄱ
------

추가로 node_mobules라는 폴더도 생성 됨.  
그 안에 설치한 패키지들이 들어 있음. 분명 express 하나만 설치 했지만 여러가지 패키지가 들어 있음.  
이는 express가 의존 하는 패키지들임. 이런 의존 관계가 복잡하기 때문에 package.json으로 관리가 필요

package-lock.json이라는 파일도 생성 되는데 express외에도 node_modules에 들어 있는 패키지들의 정확한 버전과 의존 관계가 담겨 있음.

개발용 패키지도 설치 할 수 있다고 함. 하지만 잘 안쓸거 같아서 안씀 ㅂㅂ

"devDependencies"라는 키값으로 저장되는 것만 기억하자

npm에는 전역 설치라는 옵션도 있음. 패키지를 현재 폴더의 node_modules에 설치하는 것이 아니라 npm이 설치 되어 있는 
폴더에 설치함.  
전역 설치 했다고 패키지를 모든 곳에서 사용한다는 뜻은 아님. 대부분 명령어로 사용하기 위해 전역 설치를 함.  

## npm install --global <package name>
리눅스나 맥에서는 관리자 권한이 필요해 sudo를 붙여야함.  

설치한 패키지들을 지워도 package.json 만 있다면 언제든지 npm install로 설치 할 수 있음.  중요한 파일은 package.json 임


------
### NOTE npx
전역 설치한  패키지는 package.json에 기록되지 않아 다시 설치 할때 어려움이 있음  

```cmd
npm install --save-dev rimraf
npx rimraf node_modules
```

이러한 경우 package.json의 devDependencies 속성에 기록 후 npx 명령어를 붙여 실행하면 전역 설치 한것과 같은 효과를 얻음

------

------
### NOTE npm에 등록 되지 않은 패키지
모든 패키지가 npm에 등록되어 있는 것은 아님.  
일부 패키지는 오픈소스거나 개발중이므로 깃허브나 nexus 등 저장소에 보관되어 있을 수 있음
npm install <저장소 주소>로 설치 할 수 있음
------

## 5.3 패키지 버전 이해하기

노드 패키지들의 버전은 세 자리로 이루어져 있음.  
SemVer방식의 넘버링을 따르기 때문  
(Semmantic Versioning) 

패키지 버전을 업그레이드 했는데 그것을 사용하는 다른 패키지에서 에러가 발생한다면 문제가 됨.  
많은 패키지들이 얽히다 보면 문제가 더 심각해짐.  
따라서 버전 번호를 어떻게 정하고 올려야 하는지를 명시하는 규칙이 등장 그게 SemVer


버전의 처음은 major  

버전의 두번째 minor

버전의 세번째 patch

![SemVer](https://github.com/ohbokdong/NodeJSStudy/blob/main/summary/week7/5-5.PNG)

새버전을 배포한 후에는 그 버전의 내용을 수정하면 안됨.  
만약 수정이 생긴다면 major, minor, patch 버전 중 하나를 의미에 맞게 올려서 새로운 버전으로 배포해야함.  
이렇게 하면 배포된 버전 내용이 바뀌지 않아서 패키지 간 의존 관계에 큰 도움이 됨.

major 버전이 업데이트 되면 기존 코드와 호환되지 않을 확률이 큼.  
minor와 patch는 비교적 안심하고 버전을 올릴수 있음 버그 수정 같은 불편이 해결됨  

package.json에는 SemVer 식 세자리 표현 외 ^이나 ~, >,< 같은 문자가 붙어 있음.  
이 문자는 버전에는 포함되지 않지만 설치 하거나 업데이트 할때 어떤 버전을 설치해야하는지 알려줌.  

가장 많이 보이는 기호는 ^이며 minor버전까지만 설치하거나 업데이트 함.  
~기호를 사용하면 patch버전까지만 설치, 업데이트 함  ~보다 ^이 많은 이유는 minor 버전까지는 하위 호환이 보장되기 때문

## 5.4 기타 npm 명령어
npm으로 설치한 패키지를 사용하다 새로운 버전이 나올 때가 있음.  npm outdated 명령어로 업데이트할 수 있는 패키지가 있는지 확인 하면 됨

![npm outdated](https://github.com/ohbokdong/NodeJSStudy/blob/main/summary/week7/5-6.PNG)

Current와 Wanted가 다르면 업데이트가 필요한 경우 임.  

npm update <package name> 을 통해 업데이트 가능. lastest는 가장 최신 버전이지만 package.json의 버전 범위와 다르면 설치를 안함

npm uninstall <package name>을 통해서 패키지 제거

npm search <keyword> 로 npm의 패키지를 검색

npm info <package name> 을 통해 세부 정보를 파악 할 수 있음.

다른 명령어는 npm -help ㄱㄱ


## 5.5 패키지 배포하기
는 패스...
배포 하실분이 찾아보는게 맞을 듯



