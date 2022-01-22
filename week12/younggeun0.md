# 노드 JS 스터디 9장 익스프레스로 SNS 서비스 만들기

## 9.5.2 핵심 정리

* 서버는 요청에 응답하는 것이 핵심 임무이므로 요청을 수락하든 거절하든 상관없이 반드시 응답해야 함. 이 때 한 번만 응답해야 에러가 발생하지 않음
* 개발 시 서버를 매번 수동으로 재시작하지 않으려면 `nodemon`을 사용하는 것이 좋음
* `dotenv`패키지와 .env 파일로 유출되면 안 되는 비밀 키를 관리해야 함
* 라우터는 `routes 폴더`에, 데이터베이스는 `models 폴더`에, html 파일은 `views 폴더`에 구분하여 저장하면 프로젝트 규모가 커져도 관리하기 쉬움
* 데이터베이스를 구성하기 전에 데이터 간 1:1, 1:N, N:M 관계를 잘 파악해야 함
* routes/middlewares.js처럼 라우터 내에 미들웨어를 사용할 수 있다는 것을 기억할 것
* Passport 인증 과정과 serializeUser, deserializeUser가 언제 호출되는지 파악하고 기억해둘 것
* 프런트엔드 form 태그의 인코딩 방식이 multipart일 때는 multer 같은 multipart 처리용 패키지를 사용하는 것이 좋음

## 함께 보면 좋은 자료

* [Passport 공식 문서](http://www.passportjs.org/)
* [passport-local 공식 문서](https://www.npmjs.com/package/passport-local)
* [passport-kakao 공식 문서](https://www.npmjs.com/package/passport-kakao)
* [bcrypt 공식 문서](https://www.npmjs.com/package/bcrypt)
* [카카오 로그인](https://developers.kakao.com/docs/latest/ko/kakaologin/common)