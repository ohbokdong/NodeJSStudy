# Chap15. AWS와 GCP로 배포하기

## 15.1 서비스 운영을 위한 패키지
### 15.1.1 morgan과 express-session
- process.env.NODE_ENV 는 배포 환경인지 개발 환경인지 판단할 수 있는 환경변수
- morgan을 combined 모드로 사용하면 더 많은 사용자 정보를 로그로 남겨서 추후 버그 해결시 유용하게 사용할 수 있다.
- 배포 환경일 때는 proxy와 cookie.secure를 true로 바꾼다
    - https 적용을 위해 노드 서버 앞에 다른 서버를 두었을 때

### 15.1.2 시퀄라이즈
js파일의 경우 dotenv모듈을 사용할 수 있으므로 .env 패스워드를 바꿔준다.

### 15.1.3 cross-env
corss-env 패키지를 사용하면 동적으로 process.env(환경 변수)를 바꿔줄 수 있다.
* crossenv 라는 패키지는 악성 패키지이므로 항상 패키지설치 시 조심하자!

### 15.1.4 sanitize-html, csurf
XSS(Cross Site Scripting), CSRF(CROSS SITE REQUEST FORGERY) 공격을 막기 위한 패키지   
- XSS는 악의적인 사용자가 사이트에 스크립트를 삽입하는 공격, 게시글이나 댓글 등 업로드할 때 자바스크립트가 포함된 태그를 올리면 그 스크립트 실행되어 오류 내는 것
    - 사용자가 업로드한 HTML을 sanitize-html 함수로 감싸면 허용하지 않는 태그나 스크립트 제거됨
- CSRF는 사용자가 의도치 않게 공격자가 의도한 행동을 하게 만드는 공격. ex) 특정 페이지 방문 시 자동 로그아웃, 게시글이 써지는 현상 유도
    - 내가 한 행동이 내가 한게 맞다 인증 -> CSRF토큰 사용, csurf 패키지가 토큰 쉽게 발급 및 검증하도록 도와줌

### 15.1.5 pm2
서버 운영을 위한 패키지로   
가장 큰 기능은 서버가 에러로 인해 꺼졌을 때 `서버를 다시 켜주는 것`   
`멀티 프로세싱`지원으로 노드 프로세스 개수를 한 개 이상으로 늘릴 수 있다 - 하나의 프로세스가 받는 부하가 적어지므로 서비스를 더 원활하게 운영할 수 있다.   
단점: 서버의 메모리 같은 자원을 공유하는 것은 아님 => 세션을 공유하게 하는 멤캐시드나 레디스 같은 서비스를 사용하여 단점 극복

### 15.1.6 winston
console.log, console.error 등으로 확인하는 걸 서버 종료되면서 에러가 날라가는 걸 방지하기 위해 쓰는 패키지
- createLogger / level, format, transports 등의 설정이 있음
    - level: 로그의 심각도(error, warn, info, veribose, debug, silly)
    - format: 로그의 형식(json, label, timestamp, printf, simple, combine). 시간은 timestamp 추천
    - transports: 로그 저장 방식
        - new transports.File은 파일로 저장
        - new transports.Console은 배포 환경이 아닌 경우 파일뿐만 아니라 콘솔에도 출력

### 15.1.7 helmet, hpp
서버의 각종 취약점을 보완해주는 패키지로 각각의 공식 문서 참조

### 15.1.8 connect-redis
멀티 프로세스 간 세션 공유를 위해 레디스와 익스프레스를 연결해주는 패키지   
기존에는 로그인할 때 express-session의 세션 아이디와 실제 사용자 정보가 메모리에 저장됨   
서버종료로 메모리 날아가면 로그인 풀려버림 => 방지를 위해 세션 아이디와 실제 사용자 정보를 데이터베이스에 저장하는데, 이때 사용하는 데이터 베이스가 레디스   
레디스 데이터베이스도 설치해줘야 함(redislabs)   
Redis의 Endpoint => Host   
Redis PORT, PASSWORD 등도 복사해서 저장

### 15.1.9 nvm, n
노드 버전을 업데이트하기 위한 패키지로 윈도우에서는 nvm-installer를 사용, 리눅스나 맥에서는 n

## 15.2 깃과 깃허브
모두 아는내용^_^