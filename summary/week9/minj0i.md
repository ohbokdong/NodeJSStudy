# 6장 익스프레스 웹 서버 만들기(2)

## 6.4 req, res 객체 살펴보기

### req

- `req.app`: req 객체를 통해 app 객체에 접근. `req.app.get('port')`와 같은 식으로 사용
- `req.body`: body-parser 미들웨어가 만드는 요청의 본문을 해석한 객체
- `req.cookies`: cookie-parser 미들웨어가 만드는 요청의 쿠키를 해석한 객체.
- `req.ip`: 요청의 ip 주소
- `req.params`: 라우트 매개변수에 대한 정보가 담긴 객체
- `req.query`: 쿼리스트링에 대한 정보가 담긴 객체
- `req.signedCookies`: 서명된 쿠키들은 req.cookies 대신 여기에 담겨 있음
- `req.get(헤더 이름)`: 헤더의 값을 가져오고 싶을 때 사용하는 메서드

### res

- `res.app`: req.app처럼 res 객체를 통해 app 객체에 접근할 수 있다
- `res.cookie(키, 값, 옵션)`: 쿠키를 설정하는 메서드
- `res.clearCookie(키, 값, 옵션)`: 쿠키를 제거하는 메서드
- `res.end()`: 데이터 없이 응답을 보냄
- `res.json(JSON)`: JSON 형식의 응답을 보냄
- `res.redirect(주소)`: 리다이렉트할 주소와 함께 응답을 보냄
- `res.render(뷰, 데이터)`: 다음 절에서 다룰 템플릿 엔진을 렌더링해서 응답할 때 사용하는 메서드
- `res.send(데이터)`: 데이터와 함께 응답을 보냄. 데이터는 문자열일 수도 있고 HTML일 수도 있으며, 버퍼일 수도 있고 객체나 배열일 수도 있다
- `res.sendFile(경로)`: 경로에 위치한 파일을 응답
- `res.set(헤더, 값)`: 응답의 헤더를 설정
- `res.status(코드)`: 응답 시의 HTTP 상태 코드를 지정

> 메서드 체이닝

코드양 줄이기

```
res
  .status(201)
  .cookie('test', 'test')
  .redirect('/admin');
```

## 6.5 템플릿 엔진 사용하기

템플릿 엔진은 자바스크립트를 사용해서 HTML을 렌더링  
대표적인 템플릿 엔진인 퍼그(Pug)와 넌적스(Nunjucks)

### 6.5.1. 퍼그(제이드)

흠 개인적으로 쓰는 거 첨 봄

```JS
// app.js

...
app.set('port', process.env.PORT || 3000);
app.set('views', path.join(__dirname, 'views'));
app.set('view engine', 'pug');

app.use(morgan('dev'));
...
```

> HTML 엔티티와 이스케이프

- `<`: &lt;
- `>`: &gt;
- `&`: &amp;
- `띄어쓰기`: &nbsp;
- `"`: &quot;
- `'`: &apos;

### 6.5.2. 넌적스
