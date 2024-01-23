### 레이아웃 클론
- 두 영역의 너비가 다르지만 각 영역 외부에 동일한 여백을 주면서 가운데 정렬을 하고 싶은 경우 `flex-grow` 스타일을 활용할 수 있다.

### useSelectedLayoutSegment()
- Next.js에서 기본적으로 제공하는 훅
```ts
const segment: string | null = useSelectedLayoutSegment();
// ~~/home 라우트로 가면 segment는 "home"
```
- usePathname()과 어떤 차이가 있지?
    - 루트 경로 기준으로 usePathname은 `'/'`, useSelectedLayoutSegment는 null 반환
    - nested routes,` /blog/hello-world` 기준으로 usePathname은 `/blog/hello-world`, useSelectedLayoutSegment는 `hello-world` 반환
- nested routes에서 전체 segment를 얻고 싶으면 `useSelectedLayoutSegments()` 사용

### 개발자 도구에서 svg 따는 법
- 요소 선택 -> svg 클릭 -> html 문서에서 해당 svg 태그 찾고 -> 우클릭 -> 복사 -> outerHTML 복사

### faker.js
- 랜덤한 더미데이터를 생성해주는 라이브러리?

### day.js
- 시간 관련 데이터를 편하게 다룰 수 있게 해주는 라이브러리
- day.js 외에 moment, date-fns 등의 경쟁자가 있는데 day.js의 성장세가 꽤 좋음

### className 합성 라이브러리
- 잘 알려진 라이브러리에는 classnames, clsx가 있음
    - 취향껏 고르면 되는데 clsx를 이미 사용중이었어서 앞으로도 clsx 채택 예정

### 클라이언트 컴포넌트 선언 기준?
- 기본적으로 이벤트(클릭 등), 상태를 필요로 하면 선언해주자.
- 클라이언트 컴포넌트로 선언할 이유가 없는데도 클라이언트 컴포넌트로 선언하는 것을 지양해야하는 것이지, 이유가 분명하면 선언에 주저하지 말자.
- 클라이언트 컴포넌트도 SSR이 된다.

### usePathname
- useSelectedLayoutSegment()와 비슷하지만 살짝 다른 Next.js 기본 훅
- 라우트 주소 전체를 문자열로 반환한다.
- "use client" 선언 필요

### SearchParams
- 특정 라우트 주소 뒤에 쿼리스트링으로 입력되는 값들을 꺼내올 수 있다.
- 라우트에 해당하는 페이지의 Prop에 담겨온다
- 예시: `/test-query?q1="abc"&q2="def"`로 요청하면
```js
// /test-query/page.tsx
type Props = {
  searchParams: {
    q1: string, 
    q2: string
  }
}

const TestQueryPage = ({ searchParams }: Props) => {
  const a = searchParams.q1; // "abc"
  const b = searchParams.q2; // "def"

//...
}
```


### 이벤트 캡쳐링
공식문서에서 합성 이벤트(Synthetic Event)로 부르는 것이 있는데, 이벤트를 버블링이 아닌 캡쳐단계에서 핸들링할 수 있다. 버블링 방지할 때 쓰면 좋을 느낌?
- onClick => onClickCapture

### npm 설치시 유의할 점
- 악성 패키지들도 아직 npm에 떠돌고 있기 때문에 사용하려는 패키지의 패키지명을 제대로 알고 있는게 중요하다.
- ex) faker는 npm i faker하면 안되고, @fakerjs/faker로 설치해야함
