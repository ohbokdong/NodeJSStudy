# Chap15. AWS와 GCP로 배포하기

## 15.1 서비스 운영을 위한 패키지
### 15.1.1 morgan과 express-session
- process.env.NODE_ENV 는 배포 환경인지 개발 환경인지 판단할 수 있는 환경변수
- morgan을 combined 모드로 사용하면 더 많은 사용자 정보를 로그로 남겨서 추후 버그 해결시 유용하게 사용할 수 있다.
- 배포 환경일 때는 proxy와 cookie.secure를 true로 바꾼다
    - https 적용을 위해 노드 서버 앞에 다른 서버를 두었을 때

## 15.2 시퀄라이즈
