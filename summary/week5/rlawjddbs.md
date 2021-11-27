### 3.5.6 util

- 각종 편의 기능을 모아둔 모듈

```node
const util = require('util');
const crypto = require('crypto');

const dontUseMe = util.deprecate((x, y) => {
  console.log(x + y);
}, 'dontUseMe 함수는 deprecated되었으니 더 이상 사용하지 마세요!');
dontUseMe(1, 2);

const randomBytesPromise = util.promisify(crypto.randomBytes);
randomBytesPromise(64)
  .then((buf) => {
    console.log(buf.toString('base64'));
  })
  .catch((error) => {
    console.error(error);
  });
```

```shell
결과 생략
```

<table>
    <caption>util 모듈 내장 함수</caption>
    <thead>
        <tr>
            <th>모듈</th>
            <th>함수</th>
            <th>설명</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td rowspan="2">util</td>
            <td>deprecate(deprecated 처리할 함수, 경고 메시지)</td>
            <td>함수가 deprecated 처리 되었음을 알림</td>
        </tr>
        <tr>
            <td>promisify(프로미스 패턴으로 바꿀 함수)</td>
            <td>콜백 패턴의 함수를 프로미스 패턴으로 바꿈. 반대로 프로미스를 콜백으로 바꾸는 <code>callbackify()</code>도 있지만 잘 사용되지는 않음</td>
        </tr>
    </tbody>
</table>

### 3.5.7 worker_threads

- 노드에서 멀티 스레드 방식으로 작업할 수 있도록하는 모듈
- 