## 3.5 노드 내장 모듈 사용하기

- 노드는 JS 보다 더 많은 기능을 제공함
  - 운영체제 정보, 클라이언트가 요청한 주소에 대한 정보 등

### 3.5.1 os

- 운영체제 정보를 담고있는 모듈
- 노드로 컴퓨터 내부 자원에 빈번하게 접근할 경우 `os` 모듈을 사용 (일반적 웹 서비스는 잘 안 씀)
  - 운영체제 별로 다른 서비스를 제공하고 싶을 때 쓰자

```node
const os = require('os');

console.log('운영체제 정보---------------------------------');
console.log('os.arch():', os.arch());
console.log('os.platform():', os.platform());
console.log('os.type():', os.type());
console.log('os.uptime():', os.uptime());
console.log('os.hostname():', os.hostname());
console.log('os.release():', os.release());

console.log('경로------------------------------------------');
console.log('os.homedir():', os.homedir());
console.log('os.tmpdir():', os.tmpdir());

console.log('cpu 정보--------------------------------------');
console.log('os.cpus():', os.cpus());
console.log('os.cpus().length:', os.cpus().length);

console.log('메모리 정보-----------------------------------');
console.log('os.freemem():', os.freemem());
console.log('os.totalmem():', os.totalmem());
```
```shell
결과 생략
```

<table>
    <caption>os 모듈 내장 함수</caption>
    <thead>
        <tr>
            <th>모듈</th>
            <th>함수</th>
            <th>설명</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td rowspan="11">os</td>
            <td>arch()</td>
            <td><code>process.arch</code>와 동일</td>
        </tr>
        <tr>
            <td>platform()</td>
            <td><code>process.platform과 동일</code></td>
        </tr>
        <tr>
            <td>type()</td>
            <td>운영체제 종류 표출</td>
        </tr>
        <tr>
            <td>uptime()</td>
            <td>운영체제 부팅 이후 흐른 시간 표출</td>
        </tr>
        <tr>
            <td>hostname()</td>
            <td>컴퓨터의 이름 표출</td>
        </tr>
        <tr>
            <td>release()</td>
            <td>운영체제의 버전 표출</td>
        </tr>
        <tr>
            <td>homedir()</td>
            <td>홈 디렉터리 경로 표출</td>
        </tr>
        <tr>
            <td>tmpdir()</td>
            <td>임시 파일 저장 경로 표출</td>
        </tr>
        <tr>
            <td>cpus()</td>
            <td>컴퓨터의 코어 정보 표출</td>
        </tr>
        <tr>
            <td>freemem()</td>
            <td>사용 가능한 메모리(RAM) 표출</td>
        </tr>
        <tr>
            <td>totalmem()</td>
            <td>전체 메모리 용량 표출</td>
        </tr>
    </tbody>
</table>

> #### 코어 개수 확인하기
> ```node
> os.cpus().length
> ```

### 3.5.2 path

- 폴더, 파일 경로를 쉽게 조작하도록 도와주는 모듈
- 운영체제별로 `경로 구분자`가 달라서 필요

```node
const path = require('path');

const string = __filename;

console.log('path.sep:', path.sep);
console.log('path.delimiter:', path.delimiter);
console.log('------------------------------');
console.log('path.dirname():', path.dirname(string));
console.log('path.extname():', path.extname(string));
console.log('path.basename():', path.basename(string));
console.log('path.basename - extname:', path.basename(string, path.extname(string)));
console.log('------------------------------');
console.log('path.parse()', path.parse(string));
console.log('path.format():', path.format({
  dir: 'C:\\users\\zerocho',
  name: 'path',
  ext: '.js',
}));
console.log('path.normalize():', path.normalize('C://users\\\\zerocho\\\path.js'));
console.log('------------------------------');
console.log('path.isAbsolute(C:\\):', path.isAbsolute('C:\\'));
console.log('path.isAbsolute(./home):', path.isAbsolute('./home'));
console.log('------------------------------');
console.log('path.relative():', path.relative('C:\\users\\zerocho\\path.js', 'C:\\'));
console.log('path.join():', path.join(__dirname, '..', '..', '/users', '.', '/zerocho'));
console.log('path.resolve():', path.resolve(__dirname, '..', 'users', '.', '/zerocho'));
```

```shell
결과 생략
```

<table>
    <caption>path 모듈 내장 함수, 속성</caption>
    <thead>
        <tr>
            <th>모듈</th>
            <th>함수(속성)</th>
            <th>설명</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td rowspan="11">path</td>
            <td>sep</td>
            <td>경로 구분자 (윈도 : <code>\</code>, POSIX : <code>/</code>)</td>
        </tr>
        <tr>
            <td>delimiter</td>
            <td>환경 변수의 구분자. (윈도 : <code>;</code>, POSIX : <code>:</code>)</td>
        </tr>
        <tr>
            <td>dirname(경로)</td>
            <td>파일이 위치한 폴더 경로 표출</td>
        </tr>
        <tr>
            <td>basename(경로, 확장자)</td>
            <td>확장자 포함된 파일명 표출. 파일명만 표시할 땐 두 번째 인자로 파일 확장자 입력</td>
        </tr>
        <tr>
            <td>parse(경로)</td>
            <td>파일 경로를 root, dir, base, ext, name으로 분리한 객체로 가져옴</td>
        </tr>
        <tr>
            <td>format(객체)</td>
            <td><code>path.parse()</code>한 객체를 파일 경로로 합쳐서 표출</td>
        </tr>
        <tr>
            <td>normalize(경로)</td>
            <td><code>\</code>나 <code>/</code>를 여러번 사용했을 때 정상적인 경로로 변환해줌</td>
        </tr>
        <tr>
            <td>isAbsolute(경로)</td>
            <td>절대경로인지 boolean 값으로 반환</td>
        </tr>
        <tr>
            <td>relaitve(기준경로, 비교경로)</td>
            <td>기준경로 → 비교경로로 가는 경로값 반환</td>
        </tr>
        <tr>
            <td>join(경로, …)</td>
            <td>여러 인수를 넣으면 하나의 경로로 합침, 상대경로나 현위치도 알아서 처리</td>
        </tr>
        <tr>
            <td>resolve(경로, …)</td>
            <td><code>path.join()</code>과 비슷하지만 차이가 있음. 다음에 나오는 Node에서 설명!</td>
        </tr>
    </tbody>
</table>

> #### join과 resolve의 차이
> ```node
> path.join('/a', '/b', 'c'); /* /a/b/c/ */  
> path.resolve('/a', '/b', 'c'); /* /b/c/ */ 
> ```
> `/`가 붙으면 `resolve`는 절대 경로로 인식하고 `join`은 상대경로로 인식

### 3.5.3 url

- 인터넷 주소를 쉽게 조작하도록 도와주는 모듈
- url 처리의 두 가지 방식
  - `기존 노드` 방식
  - `WHATWG` 방식

```node
const url = require('url');

const { URL } = url;
const myURL = new URL('http://www.gilbut.co.kr/book/bookList.aspx?sercate1=001001000#anchor');
console.log('new URL():', myURL);
console.log('url.format():', url.format(myURL));
console.log('------------------------------');
const parsedUrl = url.parse('http://www.gilbut.co.kr/book/bookList.aspx?sercate1=001001000#anchor');
console.log('url.parse():', parsedUrl);
console.log('url.format():', url.format(parsedUrl));
```

```shell
결과 생략
```

<table>
    <caption>기존 노드방식의 url 모듈 내장 함수</caption>
    <thead>
        <tr>
            <th>모듈</th>
            <th>함수</th>
            <th>설명</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td rowspan="2">url</td>
            <td>parse(주소)</td>
            <td>주소를 분해하여 객체 형식으로 가져옴</td>
        </tr>
        <tr>
            <td>format</td>
            <td><code>WHATWG</code>, <code>기존 노드</code>방식의 url을 모두 사용할 수 있음</td>
        </tr>
    </tbody>
</table>

- `WHATWG`와 `노드`의 url은 취향에 따라 사용하지만 노드의 url 형식을 꼭 사용해야 하는 경우가 있음
  - `host` 부분없이 `pathname` 부분만 오는 주소(/book/bookList.apsx)는 `WHATWG` 방식으로 처리 불가능

#### WHATWG의 searchParams
- url의 search 부분을 특수한 객체로 반환

```node
const { URL } = require('url');

const myURL = new URL('http://www.gilbut.co.kr/?page=3&limit=10&category=nodejs&category=javascript');
console.log('searchParams:', myURL.searchParams);
console.log('searchParams.getAll():', myURL.searchParams.getAll('category'));
console.log('searchParams.get():', myURL.searchParams.get('limit'));
console.log('searchParams.has():', myURL.searchParams.has('page'));

console.log('searchParams.keys():', myURL.searchParams.keys());
console.log('searchParams.values():', myURL.searchParams.values());

myURL.searchParams.append('filter', 'es3');
myURL.searchParams.append('filter', 'es5');
console.log(myURL.searchParams.getAll('filter'));

myURL.searchParams.set('filter', 'es6');
console.log(myURL.searchParams.getAll('filter'));

myURL.searchParams.delete('filter');
console.log(myURL.searchParams.getAll('filter'));

console.log('searchParams.toString():', myURL.searchParams.toString());
myURL.search = myURL.searchParams.toString();
```

```shell
결과 생략
```

<table>
    <caption>searchParams 객체 내장 함수</caption>
    <thead>
        <tr>
            <th>모듈</th>
            <th>함수</th>
            <th>설명</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td rowspan="9">searchParams</td>
            <td>getAll(키)</td>
            <td>키에 해당하는 모든 값들을 배열 형식으로 가져옴</td>
        </tr>
        <tr>
            <td>get(키)</td>
            <td>키에 해당하는 첫 번째 값 가져옴</td>
        </tr>
        <tr>
            <td>has(키)</td>
            <td>해당 키가 있는지 boolean 반환</td>
        </tr>
        <tr>
            <td>keys()</td>
            <td><code>searchParams</code>의 모든 키를 반복기(<code>iterator</code>) 객체로 가져옴 - ES2015 문법</td>
        </tr>
        <tr>
            <td>values()</td>
            <td><code>searchParams</code>의 모든 값을 반복기 객체로 가져옴</td>
        </tr>
        <tr>
            <td>append(키, 값)</td>
            <td>해당 키를 추가. 같은키의 값이 있다면 유지하고 하나 더 추가</td>
        </tr>
        <tr>
            <td>set(키, 값)</td>
            <td><code>append</code>와 비슷하지만 중복된 키 값을 지우고 새로 추가</td>
        </tr>
        <tr>
            <td>delete(키)</td>
            <td>해당 키 제거</td>
        </tr>
        <tr>
            <td>toString()</td>
            <td>조작한 <code>searchParams</code> 객체를 다시 문자열로 만들어 가져옴</td>
        </tr>
    </tbody>
</table>

- query 같은 문자열보다 `searchParams`가 더 유용한 이유는 query의 경우 아래 `queryString` 모듈을 한번 더 사용해야 하기 때문

### 3.5.4 queryString

- `WHATWG` 방식의 url 대신 `기존 노드`의 url을 사용할 때, search 부분을 사용하기 쉽게 객체로 만드는 모듈

```node
const url = require('url');
const querystring = require('querystring');

const parsedUrl = url.parse('http://www.gilbut.co.kr/?page=3&limit=10&category=nodejs&category=javascript');
const query = querystring.parse(parsedUrl.query);
console.log('querystring.parse():', query);
console.log('querystring.stringify():', querystring.stringify(query));
```

```shell
결과 생략
```

<table>
    <caption>queryString 모듈 내장 함수</caption>
    <thead>
        <tr>
            <th>모듈</th>
            <th>함수</th>
            <th>설명</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td rowspan="2">queryString</td>
            <td>parse</td>
            <td>url의 query 부분을 JS 객체로 분해</td>
        </tr>
        <tr>
            <td>stringify</td>
            <td>분해된 query 객체를 문자열로 다시 조립</td>
        </tr>
    </tbody>
</table>

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
            <td>콜백 패턴의 함수를 프로미스 패턴으로 바꿈. 반대로 프로미스를 콜백으로 바꾸는 <code>callbackify()</code>도 있지만 잘 사용되지는 않음</td>
        </tr>
    </tbody>
</table>

### 3.5.7 worker_threads

- 노드에서 멀티 스레드 방식으로 작업할 수 있도록하는 모듈