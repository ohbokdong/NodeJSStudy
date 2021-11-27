### 3.5.5 crypto

- 암호화를 도와주는 모듈

#### 3.5.5.1 단방향 암호화

```node
const crypto = require('crypto');

console.log('base64:', crypto.createHash('sha512').update('비밀번호').digest('base64'));
console.log('hex:', crypto.createHash('sha512').update('비밀번호').digest('hex'));
console.log('base64:', crypto.createHash('sha512').update('다른 비밀번호').digest('base64'));
```

```shell
결과 생략
```

<table>
    <caption>crypto 모듈 내장 함수</caption>
    <thead>
        <tr>
            <th>모듈</th>
            <th>함수</th>
            <th>설명</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td rowspan="3">crypto</td>
            <td>createHash(알고리즘)</td>
            <td>사용할 해시 알고리즘 입력</td>
        </tr>
        <tr>
            <td>update(문자열)</td>
            <td>변환할 문자열 입력</td>
        </tr>
        <tr>
            <td>digest(인코딩)</td>
            <td>인코딩할 알고리즘 입력 및 변환된 문자열 반환</td>
        </tr>
    </tbody>
</table>

#### 3.5.5.2 양방향 암호화

```shell
const crypto = require('crypto');

const algorithm = 'aes-256-cbc';
const key = 'abcdefghijklmnopqrstuvwxyz123456';
const iv = '1234567890123456';

const cipher = crypto.createCipheriv(algorithm, key, iv);
let result = cipher.update('암호화할 문장', 'utf8', 'base64');
result += cipher.final('base64');
console.log('암호화:', result);

const decipher = crypto.createDecipheriv(algorithm, key, iv);
let result2 = decipher.update(result, 'base64', 'utf8');
result2 += decipher.final('utf8');
console.log('복호화:', result2);
```

```shell
결과 생략
```

<table>
    <caption>chiper 객체 내장 함수</caption>
    <thead>
        <tr>
            <th>모듈</th>
            <th>함수</th>
            <th>설명</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td rowspan="2">crypto</td>
            <td>createChipher(알고리즘, 키, iv)</td>
            <td>암호화 알고리즘과 키, iv 입력. 입력된 알고리즘마다 키와 iv의 길이가 상이함. 따로 공부 해야한다 함. <code>crypto.getCiphers()</code>를 호출하면 사용가능한 알고리즘 목록을 볼 수 있음</td>
        </tr>
        <tr>
            <td>createDecipher(알고리즘, 키, iv)</td>
            <td>암호화할 때 사용했던 알고리즘, 키, iv를 똑같이 입력</td>
        </tr>
        <tr>
            <td rowspan="2">chipher</td>
            <td>update(문자열, 인코딩, 출력 인코딩)</td>
            <td>암호화할 대상과 대상의 인코딩, 출력 결과물의 인코딩 입력</td>
        </tr>
        <tr>
            <td>final(출력 인코딩)</td>
            <td>출력 결과물의 인코딩 입력 시 암호화가 완료됨</td>
        </tr>
        <tr>
            <td rowspan="2">dicipher</td>
            <td>update(문자열, 인코딩, 출력 인코딩)</td>
            <td>암호화된 문장, 그 문장의 인코딩, 복호화할 인코딩 입력</td>
        </tr>
        <tr>
            <td>final(출력 인코딩)</td>
            <td>복호화 결과물의 인코딩 입력</td>
        </tr>
    </tbody>
</table>

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

```node
const fs = require('fs');

console.log('시작');
fs.readFile('./readme.txt', (err, data) => {
    if (err) {
        throw err;
    }
    console.log('1번', data.toString());
});
fs.readFile('./readme.txt', (err, data) => {
    if (err) {
        throw err;
    }
    console.log('2번', data.toString());
});
fs.readFile('./readme.txt', (err, data) => {
    if (err) {
        throw err;
    }
    console.log('3번', data.toString());
});
console.log('끝');
```

```shell
# 결과
시작
끝
1번 저를 읽어주세요.
2번 저를 읽어주세요.
3번 저를 읽어주세요.
```

- 비동기 메서드들은 `백그라운드`에 해당 파일을 읽도록 요청만 하고 다음 작업으로 넘어감
    - 따라서 파일 읽기 요청만 세 번을 보내고 `console.log('끝')`을 찍음
- 파일 읽기가 완료되면 백그라운드가 다시 메인 스레드에 알리고 메인 스레드는 콜백 함수를 실행함
- 동기적으로 처리하려면 `fs.readFileSync()`를 사용하면 됨 (142p 참고)

> #### 동기와 비동기, 블로킹과 논 블로킹
> - 동기와 비동기 : `백그라운드` 작업 완료 확인 여부
> - 블로킹과 논 블로킹 : 함수가 바로 `return` 되는지 여부
> - 동기-블로킹 방식 : 백그라운드 작업 완료 여부를 계속 확인, 백그라운드 작업이 끝나면 호출한 함수 `return` 후 다음 작업 처리
> - 비동기-논 블로킹 방식 : 호출한 함수를 바로 `return` 시킨 후 다음 작업 처리, 백그라운드가 작업 완료를 알리면 해당 함수의 호출 결과 처리

#### 3.6.1.1 비동기 방식으로 하되 순서 유지

```node
const fs = require('fs');

console.log('시작');
fs.readFile('./readme2.txt', (err, data) => {
    if (err) {
        throw err;
    }
    console.log('1번', data.toString());
    fs.readFile('./readme2.txt', (err, data) => {
        if (err) {
            throw err;
        }
        console.log('2번', data.toString());
        fs.readFile('./readme2.txt', (err, data) => {
            if (err) {
                throw err;
            }
            console.log('3번', data.toString());
            console.log('끝');
        });
    });
});
```

- 이전 `readFile`의 콜백으로 다음 `readFile`을 넣으면 됨

#### 3.6.1.2 콜백 지옥 해결

- `async/await` 사용하면 어느정도 해결 가능

```node
const fs = require('fs').promises;

console.log('시작');
fs.readFile('./readme2.txt')
    .then((data) => {
        console.log('1번', data.toString());
        return fs.readFile('./readme2.txt');
    })
    .then((data) => {
        console.log('2번', data.toString());
        return fs.readFile('./readme2.txt');
    })
    .then((data) => {
        console.log('3번', data.toString());
        console.log('끝');
    })
    .catch((err) => {
        console.error(err);
    });
```

### 3.6.2 버퍼와 스트림 이해하기

- 버퍼링 : 데이터를 쓸 수 있을 때까지 모으는 동작 (영상 로딩)
- 스트리밍 : 데이터를 조금씩 전송하는 동작 (인터넷 영상 시청)
- `fs.readFile` 메서드는 버퍼 형식임
  - 메모리에 저장된 `파일 데이터`가 `버퍼`

> 스트리밍하는 과정에서 버퍼링을 할 수도 있음. 
> 전송이 너무 느리면 화면을 내보내기까지 최소한의 데이터를 모아야 하고 영상 데이터가 
> 재생 속도보다 빠르게 전송되어도 미리 전송받은 데이터를 저장할 공간이 필요하기 때문.

#### 3.6.2.1 Buffer 클래스

```node
const buffer = Buffer.from('저를 버퍼로 바꿔보세요');
console.log('from():', buffer);
console.log('length:', buffer.length);
console.log('toString():', buffer.toString());

const array = [Buffer.from('띄엄 '), Buffer.from('띄엄 '), Buffer.from('띄어쓰기')];
const buffer2 = Buffer.concat(array);
console.log('concat():', buffer2.toString());

const buffer3 = Buffer.alloc(5);
console.log('alloc():', buffer3);
```

```shell
# 결과
from(): <Buffer ec a0 80 eb a5 bc 20 eb b2 84 ed 8d bc eb a1 9c 20 eb b0 94 ea bf 94 eb b3 b4 ec 84 b8 ec 9a 94>
length: 32
toString(): 저를 버퍼로 바꿔보세요
concat(): 띄엄 띄엄 띄어쓰기
alloc(): <Buffer 00 00 00 00 00>
```

<table>
    <caption>Buffer 객체 제공 메서드</caption>
    <thead>
        <tr>
            <th>객체</th>
            <th>메서드</th>
            <th>설명</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td rowspan="4">Buffer</td>
            <td>from(문자열)</td>
            <td>문자열을 버퍼로 바꿀 수 있음. length 속성은 버퍼의 크기</td>
        </tr>
        <tr>
            <td>toString(버퍼)</td>
            <td>버퍼를 다시 문자열로 바꿔 표출</td>
        </tr>
        <tr>
            <td>concat(배열)</td>
            <td>배열 안에 든 버퍼들을 하나로 합침</td>
        </tr>
        <tr>
            <td>alloc(바이트)</td>
            <td>인자로 입력받은 크기의 빈 버퍼를 생성</td>
        </tr>
    </tbody>
</table>

> #### 버퍼의 문제점
> - 용량이 큰 파일을 읽을 때 메모리 비용이 증가함 (해당 작업이 많을 수록 더)
> - 그래서 버퍼의 크기를 작게 만든 후 여러 번으로 나눠 보내는 방식(스트림)이 등장

