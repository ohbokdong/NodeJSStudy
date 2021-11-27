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

#### 3.6.2.2 Stream 클래스

#### 버퍼의 문제점

- 용량이 큰 파일을 읽을 때 메모리 비용이 증가함 (해당 작업이 많을 수록 더)
- 그래서 버퍼의 크기를 작게 만든 후 여러 번으로 나눠 보내는 방식(스트림)이 등장

```node
const fs = require('fs');

const readStream = fs.createReadStream('./readme3.txt', {highWaterMark: 16});
const data = [];

readStream.on('data', (chunk) => {
    data.push(chunk);
    console.log('data :', chunk, chunk.length);
});

readStream.on('end', () => {
    console.log('end :', Buffer.concat(data).toString());
});

readStream.on('error', (err) => {
    #
    #
    # 3.5
    .5
    crypto

    - 암호화를
    도와주는
    모듈

    #
    #
    #
    # 3.5
    .5
    .1
    단방향
    암호화

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

#### 3.6.2.2 Stream 클래스

#### 버퍼의 문제점

- 용량이 큰 파일을 읽을 때 메모리 비용이 증가함 (해당 작업이 많을 수록 더)
- 그래서 버퍼의 크기를 작게 만든 후 여러 번으로 나눠 보내는 방식(스트림)이 등장

```node
const fs = require('fs');

const readStream = fs.createReadStream('./readme.txt', {highWaterMark: 16});
const data = [];

readStream.on('data', (chunk) => {
    data.push(chunk);
    console.log('data :', chunk, chunk.length);
});

readStream.on('end', () => {
    console.log('end :', Buffer.concat(data).toString());
});

readStream.on('error', (err) => {
    console.log('error :', err);
});
```

```shell
# 결과
data : <Buffer ec a0 80 eb a5 bc 20 ec 9d bd ec 96 b4 ec a3 bc> 16
data : <Buffer ec 84 b8 ec 9a 94 2e> 7
end : 저를 읽어주세요.
```

- `createReadStream(읽을 파일 경로, 옵션 객체)` 형식으로 사용
    - 옵션 객체의 속성 값으로 `highWatermark`를 사용하여 바이트 단위로 버퍼 크기 설정 가능(기본값 64KB)
- 자바처럼 `while`문 쓸 필요 없음. on("data/end/error 등등...")으로 이벤트 등록하면 됨 (편의성 무엇...)
- `createWriteStream` 으로 파일 쓰기도 가능 (149p)

#### 3.6.2.3 zlib 모듈

- 압축 모듈

```node
const zlib = require('zlib');
const fs = require('fs');

const readStream = fs.createReadStream('./readme4.txt');
const zlibStream = zlib.createGzip();
const writeStream = fs.createWriteStream('./readme4.txt.gz');
readStream.pipe(zlibStream).pipe(writeStream);
```

### 3.6.3 기타 fs 메서드 알아보기

- 파일 생성/삭제, 폴더 생성/삭제

```node
const fs = require('fs');

fs.access('./folder', fs.constants.F_OK | fs.constants.R_OK | fs.constants.W_OK, (err) => {
    if (err) {
        if (err.code === 'ENOENT') {
            console.log('폴더 없음');
            fs.mkdir('./folder', (err) => {
                if (err) {
                    throw err;
                }
                console.log('폴더 만들기 성공');
                fs.open('./folder/file.js', 'w', (err, fd) => {
                    if (err) {
                        throw err;
                    }
                    console.log('빈 파일 만들기 성공', fd);
                    fs.rename('./folder/file.js', './folder/newfile.js', (err) => {
                        if (err) {
                            throw err;
                        }
                        console.log('이름 바꾸기 성공');
                    });
                });
            });
        } else {
            throw err;
        }
    } else {
        console.log('이미 폴더 있음');
    }
});
```

```shell
결과 생략
```

<table>
    <caption>fs 모듈 제공 함수</caption>
    <thead>
        <tr>
            <th>모듈</th>
            <th>함수</th>
            <th>설명</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td rowspan="9">fs</td>
            <td>access(경로, 옵션, 콜백)</td>
            <td>폴더나 파일에 접근할 수 있는지 체크. 두 번째 인자는 constants에서 가져온 상수(F_OK : 파일 존재 여부, R_OK : 읽기 권한 여부, W_OK : 쓰기 권한 여부). 파일/폴더나 권한이 없으면 에러 메시지 "ENOENT"</td>
        </tr>
        <tr>
            <td>mkdir(경로, 콜백)</td>
            <td>폴더를 만드는 메서드. 폴더가 있으면 에러 발생함. <code>access</code> 메서드 호출해서 분기 만들어야 함</td>
        </tr>
        <tr>
            <td>open(경로, 옵션, 콜백)</td>
            <td>
                파일의 아이디(fd 변수)를 가져오는 메서드. 
                파일이 없으면 파일을 생성한 후 그 아이디를 가져옴. 
                가져온 아이디로 <code>fs.read</code>, 
                <code>fs.write</code>하여 읽거나 쓸 수 있음.
                옵션값으로 쓰러면 <code>w</code>, 읽으려면 <code>r</code>, 기존 파일에 추가하려면 <code>a</code>를 쓰면 됨
            </td>
        </tr>
        <tr>
            <td>rname(기존 경로, 새 경로, 콜백)</td>
            <td>파일의 이름을 바꾸거나 잘라내기 할 때 사용</td>
        </tr>
        <tr>
            <td>copyFile(대상 경로, 복사할 경로)</td>
            <td>파일 복사. 프로미스로 후처리 가능(155p)</td>
        </tr>
        <tr>
            <td>readDir(경로, 콜백)</td>
            <td>폴더 안의 내용물 확인 가능 (["파일명.확장자", ...])</td>
        </tr>
        <tr>
            <td>unlink(경로, 콜백)</td>
            <td>파일 삭제. 프로미스로 후처리 가능(154p)</td>
        </tr>
        <tr>
            <td>rmdir(경로, 콜백)</td>
            <td>폴더 삭제. 프로미스로 후처리 가능(154p)</td>
        </tr>
        <tr>
            <td>watch(경로, 콜백)</td>
            <td>파일/폴더 변경 사항 감시. 콜백에 (eventType, filename) => { ... } 형태로 이벤트(change, rename 등)와 파일명 표출 가능</td>
        </tr>
    </tbody>
</table>

### 3.6.4 스레드풀 알아보기

- fs 메서드를 여러 번 실행해도 백그라운드에서 동시에 처리되는데 스레드풀이 있기 때문
- 스레드풀을 직접 컨트롤할 수는 없지만 `SET UV_THREADPOOL_SIZE=개수`(윈도), `UV_THREADPOOL_SIZE=개수`(리눅스, 맥) 명령어를 통해 스레드풀의 개수를 설정할 수 있음

## 3.7 이벤트 이해하기

- `Node`에서 등록한 이벤트 외 `events` 모듈을 사용하면 개발자가 직접 이벤트를 만들 수 있음

```node
const EventEmitter = require('events');

const myEvent = new EventEmitter();
myEvent.addListener('event1', () => {
  console.log('이벤트 1');
});
myEvent.on('event2', () => {
  console.log('이벤트 2');
});
myEvent.on('event2', () => {
  console.log('이벤트 2 추가');
});
myEvent.once('event3', () => {
  console.log('이벤트 3');
}); // 한 번만 실행됨

myEvent.emit('event1'); // 이벤트 호출
myEvent.emit('event2'); // 이벤트 호출

myEvent.emit('event3');
myEvent.emit('event3'); // 실행 안 됨

myEvent.on('event4', () => {
  console.log('이벤트 4');
});
myEvent.removeAllListeners('event4');
myEvent.emit('event4'); // 실행 안 됨

const listener = () => {
  console.log('이벤트 5');
};
myEvent.on('event5', listener);
myEvent.removeListener('event5', listener);
myEvent.emit('event5'); // 실행 안 됨

console.log(myEvent.listenerCount('event2'));
```

```shell
결과 생략
```

<table>
    <caption>events 모듈 제공 함수</caption>
    <thead>
        <tr>
            <th>모듈</th>
            <th>함수</th>
            <th>설명</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td rowspan="8">events</td>
            <td>on(이벤트명, 콜백)</td>
            <td>이벤트 이름과 이벤트 발생 시 콜백 연결. 이렇게 연결하는 동작을 <code>이벤트 리스닝</code>이라고 함. 하나의 이벤트에 이벤트 리스너를 여러개 달아줄 수도 있음</td>
        </tr>
        <tr>
            <td>addListener(이벤트명, 콜백)</td>
            <td><code>on</code>과 기능 동일</td>
        </tr>
        <tr>
            <td>emit(이벤트명)</td>
            <td>등록한 이벤트를 호출</td>
        </tr>
        <tr>
            <td>once(이벤트명, 콜백)</td>
            <td>한 번만 실행되는 이벤트</td>
        </tr>
        <tr>
            <td>removeAllListeners(이벤트명)</td>
            <td>이벤트에 연결된 모든 이벤트 리스너를 제거</td>
        </tr>
        <tr>
            <td>removeListener(이벤트명, 이벤트리스너)</td>
            <td>이벤트에서 이벤트 리스너를 제거</td>
        </tr>
        <tr>
            <td>off(이벤트명, 콜백)</td>
            <td><code>removeListener</code>와 기능 동일. 짧으니까 이걸 쓰자(?)</td>
        </tr>
        <tr>
            <td>listenerCount(이벤트명)</td>
            <td>현재 리스너가 몇 개 연결되어 있는지 개수 표출</td>
        </tr>
    </tbody>
</table>
