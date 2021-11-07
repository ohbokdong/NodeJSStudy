# 2장 알아두어야 할 자바스크립트 (2/2)

## 2.2.1 AJAX(Asynchronous Javascript And XML)
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

## 2.2.2 FormData
HTML form 태그의 데이터를 동적으로 제어할 수 있음. 주로 AJAX와 함께 사용
``JAVASCRIPT

```