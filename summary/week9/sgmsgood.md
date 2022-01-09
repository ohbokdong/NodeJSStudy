# 6.4 req, res 객체 살펴보기
> 익스프레스의 req, res 객체는 http 모듈의 req, res 객체를 확정한 것

## req
* **req.app**: req 객체를 통해 app 객체에 접근 가능. req.app.get('app')과 같은 방식
* **req.body**: body-parser 미들웨어가 만드는 요청의 본문을 해석한 객체
* **req.cookies**: cookie-parser 미들웨어가 만드는 요청의 쿠키를 해석한 객체
* **req.ip**: 요청의 ip 주소가 담겨있음
* **req.params**: 라우트 매개변수에 대한 정보가 담긴 객체
* **req.query**: 쿼리스트링에 대한 정보가 담긴 객체
* **req.signedCookies**: 서명된 쿠키들은 req.cookies 대신 여기에 있음
* **req.get(헤더 이름)**: 헤더의 값을 가져오고 싶을 때 사용

## res
* **res.app**: req.app처럼 res 객체를 통해 app 객체에 접근 가능
* **res.cookie(키, 값, 옵션)**: 쿠키를 설정하는 메서드
* **res.clearCookie(키, 값, 옵션)**: 쿠키를 제거하는 메서드
* **res.end()**: 데이터 없이 응답을 보냄
* **res.json(JSON)**: JSON 형식의 응답을 보냄
* **res.redirect(주소)**: 리다이렉트할 주소와 함께 응답 보냄
* **res.render(뷰, 데이터)**: 다음 절에서 다룰 템플릿 엔진을 렌더링해서 응답할 때 사용하는 메서드
* **res.send(데이터)**: 데이터와 함께 응답 보냄. 데이터는 문자열, HTTP, 버퍼, 객체, 배열 모두 가능 
* **res.sendFile(경로)**: 경로에 위치한 파일 응답
* **res.set(헤더, 값)**: 응답의 헤더 설정
* **res.status(코드)**: 응답 시의 HTTP 상태 코드 지정

# 6.5 템플릿 엔진 사용하기
> 템플릿 엔진을 사용하면 자바스크립트를 사용하여 HTML을 렌더링 할 수 있게 됨. 
(엔진 사용하지 않을 경우, HTML은 데이터의 개수만큼 직접 일일이 코딩해야 함)

## 6.5.1 퍼그

### 1. 설치하기
> $ npm i pug

### 2. app.js 파일 수정
```js
app.set('views', path.join(__dirname, 'views'));
app.set('view engine', 'pug');                 
```

### 3. HTML 표현
* 화살 괄호 (<>) 없음 -> 탭 또는 스페이스로만 태그의 부모 자식 관계를 규명

**pug**
```js
doctype html
html
	head
    	title= title
		link(rel='stylesheet', href='/stylesheets/style.css')
```
**HTML**
```HTML
 <!DOCTYPE html>
<html>
  <head>
    <title>익스프레스</title>
    <link rel="stylesheet" href="/style.css"/>
  </head>
</html>
```

### 4. 변수
* html과 다르게 자바스크립트 변수를 템플릿에 렌더링할 수 있음. res.render 호출 시 보내는 변수를 퍼그가 처리함
* res.render(템플릿, 변수 객체)는 익스프레스가 res 객체에 추가한 템플릿 렌더링을 위한 메서드
  * index.pug를 HTML로 렌더링하면서 {title:'Express'}라는 객체를 변수로 집어넣음
  * layout.pug와 index.pug의 title 부분이 모두 Express로 치환됨
```js
router.get('/', function(req, res, next) {
  res.render('index', { title:'Express'});
});
```

* res.render 메서드에 두 번째 인수로 변수 객체를 넣는 대신, res.locals 객체를 사용해서 변수를 넣을 수도 있음.
```js
router.get('/', fuction(req, res, next) {
	res.locals.title = 'Express';
	res.render('index');
});
```

**1. 변수 사용 방법**
**퍼그**
```js
h1= title
p Welcome to ${title}
button(class=title, type='submit') 전송
input(placeholder=title + '연습')
```

```HTML
<h1>Express</h1>
<p>Welcome to express</p>
<button class="Express" type="submit">전송</button>
<input placeholder="Express 연습"/>
```
