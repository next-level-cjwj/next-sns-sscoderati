## Next.js에서의 다양한 Cache
Next.js는 최대한의 성능 향상을 위해 다양한 상황에서 사용할 수 있는 여러 캐시를 지원한다.

| 캐시 종류           | 무엇을          | 어디서   | 무엇을 위해                              | 지속 기간          |
| ------------------- | --------------- | -------- | ---------------------------------------- | ------------- |
| Request Memoization | 함수의 반환값 | 서버에서 | React Component Tree에서 데이터의 재사용 | 요청 주기마다 갱신 |
| Data Cache          | 데이터        | 서버에서         | 사용자의 요청에 따른 데이터의 저장과 배포                                         | 영구(임의 만료 가능)              |
| Full Route Cache    | HTML과 RSC 페이로드                | 서버에서         | 렌더링 비용의 절감과 성능 향상                                         | 영구(임의 만료 가능)              |
| Router Cache        | RSC 페이로드                | 클라이언트에서         | 페이지 탐색 시 서버 요청 감소                                         | 사용자 세션 or 임의 시간              |
![Next.js_Cahce](https://nextjs.org/_next/image?url=%2Fdocs%2Fdark%2Fcaching-overview.png&w=3840&q=75&dpl=dpl_BgBz72DvgwwwWh4xq5inDH2WcEcU)
위 도식도에서 Build Time과 Request Time이라는 타이밍이 나오는데, 이 시기에 대한 설명은 아래와 같다.
- Build Time: 배포 직전에 프로젝트 빌드할 때
- Request Time: 배포된 서비스에 사용자가 실제 요청을 날릴 때

그리고 각 캐시에 대해 조금 더 자세히 알아보자.
- Request Memoization: 단일 또는 복수 개의 컴포넌트에서 동일한 엔드 포인트로 중복해서 api 요청을 날리는 경우가 있는데, Next.js 서버에서 이 중복 요청을 Memoization을 통해 중복되지 않게 처리해준다. (AABBCC -> ABC)
- Data Cache: 사용자의 요청에 따라 클라이언트로 보내주는 데이터를 자동으로 캐시한다. 이 때 캐시의 식별자는 태그이고, 시간이 지나면 자동으로 만료시키거나 revalidateTag(), revalidatePath()를 통해 임의로 만료시킬 수 있다.
- Full Route Cache: 정적 페이지가 많은 경우 효용이 있는 캐시. 페이지 렌더링 시 이것을 캐시해두고 사용자 요청 시 반환해준다. dynamic function, data fetching이 일어나면 무효화되므로 적재적소에 사용 필요.
- Router Cache: 클라이언트에 30초, 특정 라우트에 대한 캐시를 적용한다. 애초에 dynamic한 페이지의 캐싱을 위해 사용하도록 만들어졌다.

## SSG 모드, 사라진 ISR 모드
Next.js에서 배포 시 두 가지 모드를 활용할 수 있는데, 하나는 static, 나머지는 dynamic이다. static 모드는 뉴스, 블로그 포스트 같이 정적인 페이지들을 serving하는 용도로 사용하고, 아래와 같이 next.config.js 내부에 `output: 'export'` 설정을 추가해서 사용할 수 있다.
```js
const nextConfig = {
// ...
  output: 'export'
// ...
}
```
이렇게되면 모든 라우트에 대한 정적인 html 문서를 생성하게 되며, 쿠키, Redirect, 미들웨어 등 몇 가지 기능을 사용할 수 없게 된다.

Pages 라우터 시절에는 ISR 모드(평소에는 static 페이지를 serving하다가 서버에 변경사항이 감지되면 정적인 페이지를 생성해서 serving하는 방식)가 있었는데, App 라우터로 들어서고 나서는 Full Router Cache, Revalidate 기능으로 구현이 가능해져서 모드로 따로 분리하지는 않게되었다.

## Next 앱 EC2 배포 과정
EC2에 node, git, sharp 설치 ->
git에서 프로젝트 소스 clone ->
환경변수 설정 ->
next build ->
next start

### 카톡 공유용 데이터
opengraph 메타데이터를 제공하면 SNS에 링크 공유할 때 다양한 정보를 미리 제공해줄 수 있다. title, desctiption, image 등의 프로퍼티를 통해 성정해줄 수 있고, 서버 컴포넌트에 아래 두 방법 중 하나를 선택해서 제공하면 된다.
```js
// 정적인 메타데이터
export const metadata: Metadata = { title: '...', };
```

```js
// 동적인 메타데이터
export async function generateMetadata({ params }) {
  return {
    title: '...',
  }
}
```

## WebSocket과 Socket.io
웹소켓은 양방향 통신을 위한 프로토콜이라서 서버 -> 클라이언트 or 클라이언트 -> 서버로 정보를 보내줄 수 있다.
서버 컴포넌트에서 Socket.io를 활용하면 메모리 릭이 생길 수 있다. 주의하자.

Socket.io를 통한 WebSocket 연결 활성화 코드
```js
// ...

const disconnect = useCallback(() => {
  socket?.disconnect();
  socket = null;
}, [socket]);

useEffect(() => {
  const socketResult = io('~~경로', {
    transports: ['websocket'] // 웹소켓 미지원 브라우저에 대한 폴리필
  });
  socketResult.on('connect_error', (err) => {
    console.error(err);
    console.log(`connect error due to ${err.message}`);
  });
  socket = socketResult;
}, []);
// ...
```


## 추가 메모
어떤 페이지에 아래 코드를 추가하면 해당 페이지에서 보내는 요청에 대한 캐시를 모두 적용하지 않는다.
```js
export const dynamic = 'force-dynamic';
```
서드 파티 라이브러리 사용 시 의도하지 않은 데이터 fetching 및 캐시가 작동해서 오류가 발생하는 것을 막기 위해 사용한다.

next build를 수행하면 라우트 별로 bundle된 js 크기가 나오는데 300kb 이내로 최적화하는게 권장됨.

