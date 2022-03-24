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