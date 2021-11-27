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
            <td>콜백 패턴의 함수를 프로미스 패턴으로 바꿈. 반대로 프로미스를 콜백으로 바꾸는<br /><code>callbackify()</code>도 있지만 잘 사용되지는 않음</td>
        </tr>
    </tbody>
</table>

### 3.5.7 worker_threads

- 노드에서 멀티 스레드 방식으로 작업할 수 있도록하는 모듈

```node
const {
    Worker, isMainThread, parentPort,
} = require('worker_threads');

if (isMainThread) { // 부모일 때
    const worker = new Worker(__filename);
    worker.on('message', message => console.log('from worker', message));
    worker.on('exit', () => console.log('worker exit'));
    worker.postMessage('ping');
} else { // 워커일 때
    parentPort.on('message', (value) => {
        console.log('from parent', value);
        parentPort.postMessage('pong');
        parentPort.close();
    });
}
```

```shell
# 결과
from parent ping
from worker pong
worker exit
```

- `isMainThread`를 통해 현재 코드가 `메인 스레드(싱글 스레드)`에서 실행되는지 개발자가 생성한 `워커 스레드`에서 실행되는지 구분 가능
- 메인 스레드에서는 `new Worker`를 통해 위 소스 코드를 `워커 스레드`에서 실행시키고 있음
    - 실상 위 소스에서 `else` 부분만 워커 스레드에서 실행됨

#### 3.5.7.1 워커 스레드로 데이터 보내기

- `new Worker`를 호출 할 때 두 번째 인수의 workerData 속성으로 원하는 데이터를 보낼 수 있음

```node
const {
    Worker, isMainThread, parentPort, workerData,
} = require('worker_threads');

if (isMainThread) { // 부모일 때

    const threads = new Set();

    threads.add(new Worker(__filename, {
        workerData: {start: 1},
    }));

    threads.add(new Worker(__filename, {
        workerData: {start: 2},
    }));

    for (let worker of threads) {
        worker.on('message', message => console.log('from worker', message));
        worker.on('exit', () => {
            threads.delete(worker);
            if (threads.size === 0) {
                console.log('job done');
            }
        });
    }

} else { // 워커일 때
    const data = workerData;
    parentPort.postMessage(data.start + 100);
}
```

```shell
# 결과
from worker 101
from worker 102
job done
```

### 3.5.8 child_process

- 노드에서 다른 프로그램을 실행하고 싶거나 명령어를 수행하고 싶을 때 사용

```node
const exec = require('child_process').exec;

var process = exec('ls');

process.stdout.on('data', function (data) {
    console.log(data.toString());
}); // 실행 결과

process.stderr.on('data', function (data) {
    console.error(data.toString());
}); // 실행 에러
```

```shell
# 실행
zeongyun nodejsstudy % node test/exec.js
# 결과
README.md
summary
test
```

- 결과는 `stdout(표준출력)`과 `stderr(표준에러)`에 붙여둔 `data 이벤트 리스너`에 `버퍼` 형태로 전달됨

### 3.5.9 기타 모듈들

<table>
    <caption>기타 모듈 모듈 내장 함수</caption>
    <thead>
        <tr>
            <th>모듈</th>
            <th>설명</th>
        </tr>
    </thead>
    <tbody>
        <tr><td>assert</td>
            <td>값을 비교하여 프로그램이 제대로 동작하는지 테스트할 때 사용</td></tr>
        <tr><td>dns</td>
            <td>도메인 이름에 대한 IP 주소를 얻어내는 데 사용</td></tr>
        <tr><td>net</td>
            <td>HTTP보다 로우 레벨인 TCP나 IPC 통신을 할 때 사용</td></tr>
        <tr><td>string_decoder</td>
            <td>버퍼 데이터를 문자열로 바꾸는데 사용</td></tr>
        <tr><td>tls</td>
            <td>TLS와 SSL에 관련된 작업을 할 때 사용</td></tr>
        <tr><td>tty</td>
            <td>터미널과 관련된 작업을 할 때 사용</td></tr>
        <tr><td>dgram</td>
            <td>UDP와 관련된 작업을 할 때 사용</td></tr>
        <tr><td>v8</td>
            <td>V8 엔진에 직접 접근할 때 사용</td></tr>
        <tr><td>vm</td>
            <td>가상 머신에 직접 접근할 때 사용</td></tr>
    </tbody>
</table>

### 3.6 파일 시스템 접근하기

- fs 모듈은 파일 시스템에 접근 가능

```node
const fs = require('fs');

fs.readFile('./readme.txt', (err, data) => {
    if (err) {
        throw err;
    }
    console.log(data);
    console.log(data.toString());
});
```

```shell
# 결과
<Buffer ec a0 80 eb a5 bc 20 ec 9d bd ec 96 b4 ec a3 bc ec 84 b8 ec 9a 94 2e>
저를 읽어주세요.
```

- `fs.readFile()`의 결과물은 Buffer 형식
- `toString() `함수로 문자열 형식으로 변환 가능

```node
const fs = require('fs').promises;

fs.readFile('./readme.txt')
    .then((data) => {
        console.log(data);
        console.log(data.toString());
    })
    .catch((err) => {
        console.error(err);
    });
```

- `fs` 모듈에서 `promise` 속성을 불러오면 프로미스 기반으로 사용 가능

```node
const fs = require('fs').promises;

fs.writeFile('./writeme.txt', '글이 입력됩니다')
    .then(() => {
        return fs.readFile('./writeme.txt');
    })
    .then((data) => {
        console.log(data.toString());
    })
    .catch((err) => {
        console.error(err);
    });
```

- `fs.writeFile()`을 통해 파일 쓰기 또한 가능

### 3.6.1 동기 메서드와 비동기 메서드

