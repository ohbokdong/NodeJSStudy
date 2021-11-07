# 2장 알아두어야 할 자바스크립트 (2/2)

## 2.2 프런트엔드 자바스크립트
### 2.2.1 AJAX(Asynchronous Javascript And XML)
비동기적 웹 서비스 개발 시 사용하는 기법으로 페이지 전환 없이 새로운 데이터를 불러오는 사이트 대부분이 사용하고 있다고 보면 됨.   
예전에는 XML형식을 사용했지만 현재는 JSON을 많이 사용함   
보통 AJAX요청은 jQuery나 axios 같은 라이브러리를 이용함  
> [axios 참고 사이트 - 블로그](https://kyun2da.dev/%EB%9D%BC%EC%9D%B4%EB%B8%8C%EB%9F%AC%EB%A6%AC/axios-%EA%B0%9C%EB%85%90-%EC%A0%95%EB%A6%AC/) 

```HTML
<script src="http://unpkg.com/axios/dist/axios.min.js"></script>
<script>
    axios.get('http://www.zerocho.com/api/get')
        .then((result) => {
            console.log(result);
            console.log(result.data);
        })
        .catch((error) => {
            console.error(error);
        })
</script>
```
axios.get 내부에 new Promise가 들어 있으므로 then과 catch 사용 가능   
result.data에는 서버로부터 보낸 데이터가 들어 있음   
***익명 함수***이므로 즉시 실행을 위해 코드를 소괄호로 감싸서 호출 가능

<details>
<summary>익명함수란?</summary>
<div>
익명함수는 함수 리터럴 방식으로 만들어진 함수명이 없는 함수! 대신 변수명을 마치 함수명처럼 사용해서 함수를 호출하거나 변수값을 이동시키는데 사용할 수 있다
</div>
<ul>
<li>재사용을 할 필요가 없기 때문에</li>
<li>불필요한 시간동안 메모리를 차지하지 않도록 익명함수로 구현한다면, 정확히 해당 함수가 필요한 위치에서만 해당 함수가 구현되고 사라지면서 메모리를 아낄 수 있게 된다</li>
</ul>
</details>

> [익명함수 참고 사이트 - 블로그](https://dev-note-97.tistory.com/273)

```JAVASCRIPT
(async () => {
    try {
        const result = await axios.get('https://www.zerocho.com/api.get');
        console.log(result);
        console.log(result.data);
    } catch (error) {
        console.error(error);
    }
})
```

POST 요청이라면 axios.post('서버', {보낼 내용})으로 가능

### 2.2.2 FormData
HTML form 태그의 데이터를 동적으로 제어할 수 있음. 주로 AJAX와 함께 사용
```JAVASCRIPT
const formData = new FormData();
formData.append('name', 'zerocho');
formData.append('item', 'orange');
formData.append('item', 'melon');
formData.has('item'); // true
formData.has('money'); // false
formData.get('item'); // orange
formData.getAll('item'); // ['orange', 'melon']'
formaData.append('test', ['hi', 'zero']);
formData.get('test'); // hi, zero
formData.delete('test');
formData.get('test'); // null
formData.set('item', 'apple');
formData.getAll('item'); // ['apple']
```
- append: 키-값 형식의 데이터를 저장
- has: 주어진 키에 해당하는 값이 있는지 여부 확인
- get: 주어진 키에 해당하는 값 하나를 가져옴
- getAll: 해당하는 모든 값을 가져 옴
- delete: 현재 키 제거
- set: 현재 키 수정

```JAVASCRIPT
(async () => {
    try {
        const formData = new FormData();
        formData.append('name', 'zerocho');
        formData.append('birth', 1994);
        const result = await axios.post('https://www.zerocho.com/api/post/formdata', formData);
        console.log(result);
        console.log(result.data);
    } catch (error) {
        console.error(error);
    }
})();
```

### 2.2.3 encodeURIComponent, decodeURIComponet
주소에 한글이 들어가는 경우 서버가 한글 주소를 이해하지 못하는 경우가 있어, window 객체의 메서드인 encodeURIComponent 메서드를 사용
```JS
(async () => {
    try {
        const result = await axios.get(`https://www.zerocho.com/api/search/${encodeURIComponent('노드')}`);
        console.log(result);
        console.log(result.data);
    } catch (error) {
        console.error(error);
    }
})();
```
받는 쪽에서는 decodeURIComponent를 사용   

### 2.2.4 데이터 속성과 dataset
서버에서 보내준 데이터를 프론트엔드 어디에 넣어야 할 것인가??

첫 번째 고려사항: 보안 - 비밀번호같은건 내려보내면 안됨   
자바스크립트 변수에 저장도 가능하지만 `데이터 속성(data attribute) (data-*)`를 사용해서 저장도 가능
```HTML
<ul>
    <li data-id="1" data-user-job="programmer">ZERO</li>
</ul>
<script>
    console.log(document.querySelector('li').dataSet); // { id: '1', userJob: 'programmer'}
</script>
```
- 화면에 나타나지 않지만 웹 애플리케이션 구동에 필요한 데이터들
- 자바스크립트로 쉽게 접근 가능
- 다만 키 값이 data- 접두어가 사라지고 하이픈 뒤 연결되는 문자가 대문자로 변경됨

### 2.3 함께 보면 좋은 자료
- ES2015
- ES상세 후보군
- ES2015+ 브라우저/서버 호환 여부
- 브라우저별 기능 지원 여부 확인: https://caniuse.com
- 노드 버전별 ECMAScript 스펙
- AJAX
- FormData
- ESLint 툴
- 에어비앤비 코딩 스타일
- 제로초 블로그
- 모던 자바스크립트 튜토리얼: https://ko.javascript.info/