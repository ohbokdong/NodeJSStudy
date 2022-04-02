# 서버리스 = Serverless
* 서버를 클라우드 서비스가 대신 관리해주므로, 개발자나 운영자가 서버를 관리하는데 드는 부담이 줄어듦
### 장점
* 서버리스 컴퓨팅을 할 때에는 AWS EC2나 구글 컴퓨트 엔진과는 다르게 VM 인스턴스를 미리 구매하지 않아도 됨. (코드를 업로드한 뒤, 사용량에 따라 요금 지불하면 됨)
* 24시간 작동할 필요가 없는 서버인 경우, 서버리스 컴퓨팅을 사용하면 필요한 경우에만 실행되어 요금을 절약할 수도 있음
#### AWS: 람다, API 게이트웨이, S3 유명
* 람다, 클라우드 펑션스
	* 특정한 동작을 수행하는 로직 저장 -> 요청이 들어올 때 로직 실행
	* 함수처럼 호출할 때 실행되므로 FaaS (Function as a Service)라고 불림
	* 노드가 하기 버거운 작업 (이미지 리사이징)을 함수로 만들어 클라우드에 올리고, 리사이징이 필요할 때마다 FaaS 호출
#### GCP: 앱 엔진, 파이어베이스, 클라우드 펑션스, 클라우드 스토리지 등이 유명
* S3, 클라우드 스토리지
	* 이미지 같은 데이터를 저장하고, 다른 사람에게 보여줄 수 있음.
    * 노드 서버가 다른 서버보다 정적 파일을 제공하는데 더 유리하지 않으므로, 클라우드 데이터 저장소가 대신 정적 파일을 제공하도록 위임함
 
## AWS S3 사용하기
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AddPerm",
      "Effect": "Allow",
      "Principal": "*",
      "Action": [
          "s3:GetObject", // 데이터 가져오는 권한
          "s3:PutObject", // 데이터 넣는 권한
      ],
      "Resource": "arn:aws:s3:::nodebird/*" // -> nodebird를 버킷명으로 수정
    }
  ]
}
```
```js
const AWS = required('aws-sdk')

// 1. AWS.config.update (AWS 관한 설정)
AWS.config.update({
	accessKeyId: process.env.S3_ACCESS_KEY_ID,
  	secretAccessKey: process.env.S3_SECRET_ACCESS_KEY,
  	region: 'ap-northeast-2',
});

// 2. multer의 storage 옵션을 multerS3로 교체
// 버킷 내부에서 original 폴더 아래에 파일 생성
const upload = multer({
	storage: multerS3({
    	s3: new AWS.S3(),
      	bucket: 'nodebird',
      	key(req, file, cb) {
          cb(null, `original/${Date.now()}${path.basename(file.originalname)}`);
        },
    }),
  	limits: {fileSize: 5 * 1024 * 1024},
});

// 3. req.file.location에 S3 버킷 이미지 주소 담김. 이 주소를 클라이언트로 보냄
router.post('/img', isLoggedIn, upload.single('img'), (req, res) => {
  console.log(req.file);
  res.json({url: req.file.location});
});
```

## AWS 람다 사용하기
1. asw-upload 폴더 생성 
2. package 작성
3. 람다가 실행할 index.js 작성
4. 깃허브에 리포지터리 생성
5. 콘솔에서 aws-upload 폴더 이동 후, 소스 코드를 깃허브에 push
6. 클론 받고, aws-upload 폴더 아래의 모든 파일을 압축하여 aws-upload.zip 파일 생성
```
$ sudo zip -r aws-upload.zip ./*
$ ls
```
7. Lightsail에서 S3로 파일 업로드 (aws-cli 설치)
```
$ aws s3 cp "aws-upload.zip" s3://버킷명
```

---
#### AWS 기본 설정 섹션
* 핸들러는 반드시 실행할 '파일명.함수명'이어야 함
(예제 파일명: index.js -> index.handler)
---
## 구글 클라드 스토리지 사용하기
## 구글 클라우드 펑션스 사용하기
![](https://media.vlpt.us/images/developerelen/post/599cd7ae-9a60-4c63-91e3-b78c9c6d3a7a/62B9CCDC-0CBA-44B6-957A-18E35BC1CA97.jpeg)
1. gcp-upload 폴더 생성 
2. package 작성
3. 람다가 실행할 index.js 작성
4. 깃허브에 리포지터리 생성
5. 콘솔에서 gcp-upload 폴더 이동 후, 소스 코드를 깃허브에 push
6. 클론 받고, gcp-upload 폴더 아래의 모든 파일을 압축하여 gcp-upload.zip 파일 생성

```js
router.post('/img', isLoggedIn, upload.single('img'), (req, res) => {
	console.log(req.file);
  	const filePath = req.file.path.split('/').splice(0, 3).join('/');
  	const originalUrl = `${filePath}/${req.file.filename}`;
  	const url = originalUrl.replace(/\/original\//, '/thumb/');
  	res.json({url, originalUrl});
});
```
