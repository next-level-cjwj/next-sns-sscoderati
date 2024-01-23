### MSW 세팅

```bash
npm i -D msw
npx msw init public/ --save
```

### MSW 장점
- 실제 api 서버로 요청을 보내도 중간에 가로채서 응답을 주기 때문에 개발서버랑 프로덕션 분리해서 주소를 나눠줄 필요 없음
- 정상 응답 외에 에러도 리턴 가능
- BE에서 API 만든 상황에서도 놔둔 채로 에러나 신규로 도입되는 api에 대한 작업을 지연없이 시작 가능

### MSW 사용법
- 보통 /mocks 하위에 아래와 같은 파일들 생성
    - `handlers.ts`
    - `browsers.ts` -> Client Side handlers 등록 설정
    - `http.ts` -> 익스프레스 미들웨어로 초기 요청 주소, 포트 설정하고 msw의 handler를 등록 (Next.js가 Client, Server Side 둘 다 돌기 때문에 처리해줌)

### Next.js 용 MSW 컴포넌트
- 클라이언트 컴포넌트임
```js
"use client";
import { useEffect } from "react";

export const MSWComponent = () => {
  useEffect(() => {
    if (typeof window !== 'undefined') { 
    // 개발 환경에서 클라이언트 사이드(브라우저 환경) 보장
      if (process.env.NEXT_PUBLIC_API_MOCKING === "enabled") {
        require("@/mocks/browser");
      }
    }
  }, []);

  return null;
};
```

### .env 관련 메모
- .env와 .env.local 차이: `.env`는 프로덕션과 개발환경 둘 다, `.env.local`은 개발환경에서 돌아감
- .env 키에 `NEXT_PUBLIC` 접두사를 붙이면 브라우저에서 접근 가능하게 됨. 없으면 브라우저에서 접근 불가능(보안과 관련된 중요한 정보를 보관하는 데 사용)

### Server Actions (Next.js 14 Stabled)

Server Actions는 Next.js 14버전에서 안정된 기능이다. Next.js 공식문서에서는 Server / Client Component 모두 활용할 수 있고 서버에서 실행되는 비동기 함수로서, 폼 제출 및 data mutation을 위해 활용할 수 있다고 설명하고 있다.
기본적인 활용 형태는 아래와 같다.

```js
// Server Component
export default function Page() {
  // Server Action
  async function create() {
    'use server'
 
    // ...
  }
 
  return (
    // ...
  )
}
```

우선 서버 컴포넌트의 경우 파일 내에서 함수 스코프 최상단에 `'use server'` 디렉티브를 선언해서 Server Action을 선언할 수 있고, 클라이언트 컴포넌트는 Server Action을 모듈로서 import해서 사용하는 방식만 허용한다.

한 가지 주의할 점은, Server Action에서 외부 서버로 Data Fetching을 시도할 때, 결과값에 따라 특정 주소로 redirect하도록 로직을 구성할 필요가 있는 경우가 있는데, 이 때 절대로 try-catch 문 내부에서 `redirect()`를 사용하면 안된다. (공식문서 왈)

Server Action의 도입 이유는폼 처리 로직의 실행을 클라이언트에서 서버 사이드로 이관하려는 것이라고 하는데, (JS가 로딩되기 이전이나 비활성화된 상황에서도 폼 처리가 이루어질 수 있도록) 각종 이벤트 핸들러, 리액트의 Side Effect, `<button>`과 같은 다른 폼 엘리먼트와도 활용될 수 있다. 그리고 '함수'이기 때문에 어디선가 재활용되어 생산성을 높이려는 의도도 있다. 그렇지만 여전히 '이러한 움직임이 옳은가'에 대해서는 찬반논쟁이 한창이라고 한다.

### NextAuth
Next.js 프로젝트에서 인증에 관한 부분을 구현할 때 빈번히 활용되는 라이브러리이다.
사용법은 `app/` 디렉토리와 동일한 레벨에서 `auth.ts`와 `middleware.ts` 두 파일을 생성해주고, auth.ts에서는 인증 로직, middleware.ts는 auth.ts에서 auth 함수를 불러와 미들웨어로서 동작하도록 export해줌으로써 인증 로직을 완성할 수 있다.
```js
// auth.ts 예시
import NextAuth from "next-auth"
import GithubProvider from "next-auth/providers/github"

export const authOptions = {
  // Configure one or more authentication providers
  providers: [
    GithubProvider({
      clientId: process.env.GITHUB_ID,
      clientSecret: process.env.GITHUB_SECRET,
    }),
    // ...add more providers here
  ],
}

export default NextAuth(authOptions)
```

```js
// middleware.ts
export { auth as middleware } from './auth';

export const config = {
  matcher: ['/route1', '/route2', ...] // Private Routes
}
```
Middleware는 Next.js app router에서 지원하는 기능이다. 요청이나 응답을 가로채서 전/후처리가 가능하다. config의 matcher 속성에 특정 라우트에서만 동작하도록 설정해줄 수도 있어서 페이지별 인가 처리가 매우 간편해졌다.

### API Routes
API Routes는 프론트 서버 상에서 특정 라우트의 GET, POST 요청을 처리하는 API 함수를 작성할 수 있는 기능이다. `route.ts`라는 네임 컨벤션을 갖는다.
NextAuth에서 GET, POST 핸들러를 꺼내올 수 있는데, 이걸 특정 API Route 파일에서 import 해오면 해당 라우트 주소에서 NextAuth의 핸들러 함수가 동작하도록 할 수 있다.


### NextAuth를 활용한 세션 관리
NextAuth는 세션을 사용할 App을 `<SessionProvider />`로 감싸줘야한다. Provider는 Client Component이므로 아래와 같이 children으로 App을 Prop으로 받아서 감싸는 형태로 적용해주면 된다.
```tsx
import { SessionProvider } from 'next-auth/react';

type Props = ({
  children: React.ReactNode;
});

export default function AuthSession({ children }: Props) {
  return <SessionProvider>{children}</SessionProvider>
}
```

Server Side에서 세션을 사용할 때와 Client Side에서 세션을 사용할 때가 살짝 다른데, 이것도 기억해두자.
```js
// Server Side
import { auth } from "@/auth"; // auth.ts에서 가져옴

export default async function Home() {
  const session = await auth();
  if (session?.user) {
    redirect('/home');
    return null;
  }
  // ...
}
```

```js
// Client Side
import { useSession } from 'next-auth/react';

export default function Login() {
  const router = useRouter();
  const session = useSession();
  if (session) {
    router.replace('/home')
    return null;
  }
  // ...
}
```

### React Query: SSR
React Query는 Server State를 관리하는 데 많은 도움을 주는 라이브러리이다. Provider 영역 내에서 QueryClient를 통한 상태 공유가 가능하며, useQuery, useMutation 등 유용한 Hooks들도 제공해준다. (Client Side)

역시 Client Component와 Server Component에서 사용하는 방법이 살짝 다른데, Server Component에서 데이터를 Fetching해서 활용하는 방법에 대해 알아보자.
```jsx
// Home.tsx (Server Component)
import { dehydrate, HydrationBoundary, QueryClient } from "@tanstack/react-query";

async function getPostRecommneds() {
  // fetch code
}

export default async function Home() {
  const queryClient = new QueryClient();
  await queryClient.prefetchQuery({ queryKey: ["post", "recommned"], queryFn: getPostRecommends });
  const dehydrateState = dehydrate(queryClient);

  return (
    <main className={style.main}>
      <HydrationBoundary state={dehydrateState}>
        // ...
      </HydrationBoundary>
    </main>
  )
}
```

Server Component에서 Query Client를 불러오고, 인스턴스의 `prefetchQuery()`메서드를 호출해서 데이터를 가져온 뒤, 그렇게 데이터를 불러온 queryClient를 `dehydrate()` 메서드로 해당 데이터를 Client에서 활용 가능한 상태로 변환해준 뒤 이것을 `<HydrationBoundary />` 안에서 불러올 수 있다. (불러올 때는 `queryClient.getQueryData()`, 임의의 값으로 수정할 때는 `queryClient.setQueryData()` 활용)

### revalidateTag, revalidatePath
React Query도 queryKey 캐싱 데이터 관리가 가능하지만, Next.js에서도 기본적인 기능으로 Tag 기반의 캐싱 데이터 관리가 가능하다. 아래 예시 코드를 보자.

```tsx
async function getData() {
  const res = await fetch('https://api.example.com/...')
  // The return value is *not* serialized
  // You can return Date, Map, Set, etc.
 
  if (!res.ok) {
    // This will activate the closest `error.js` Error Boundary
    throw new Error('Failed to fetch data')
  }
 
  return res.json()
}
 
export default async function Page() {
  const data = await getData()
 
  return <main></main>
}
```
Next.js Server Component에서 데이터를 불러온다면 이런 형태일 것이다.
이때 fetch 메서드에 다음과 같은 옵션이 생략돼있다.
```js
// 'force-cache' is the default, and can be omitted
fetch('https://...', { cache: 'force-cache' })
```
preFetch를 통한 build-time 요청이나 runtime 요청은 최초 1회 요청 이후에 계속해서 캐싱되어 재사용되는것이다. (GET 뿐만 아니라 POST 요청도 적용)
그런데 이 캐싱이 모든 요청에 적용되면 최신 정보를 받아올 필요가 있을 때는 문제가 된다.
그래서 Revalidate 과정이 필요하게 되는데, fetch 메서드에 아래와 같은 옵션을 설정해주면 된다.
```js
fetch('https://...', { next: { revalidate: 3600 } })
```
revalidate에 할당한 값의 단위는 second이며, 위 경우 1시간마다 캐싱을 무효화해서 최신 데이터를 불러오게 된다.

이 설정은 페이지 라우트나 레이아웃별로도 적용할 수 있다.
```js
// layout.js | page.js
export const revalidate = 3600 // revalidate at most every hour
```

그리고 조금 더 세밀한 관리가 필요한 경우, `revalidateTag()`, `revalidatePath()`라는 함수에 의해서도 revalidate가 가능하다.

우선 특정 캐시 데이터에 대한 식별자로서, 아래 코드와 같이 태그를 설정할 수 있다.
```js
export default async function Page() {
  const res = await fetch('https://...', { next: { tags: ['collection'] } })
  const data = await res.json()
  // ...
}
```

그리고 아래와 같이 `revalidateTag()`를 통해 `'collection'`이라는 태그에 의해 식별되는 데이터를 만료시켜 줄 수 있다. React Query의 `queryClient.invalidateQuery()`와 같은 역할을 하는 것이다.
```js
'use server'
 
import { revalidateTag } from 'next/cache'
 
export default async function action() {
  revalidateTag('collection')
}
```

그리고 revalidatePath()의 경우는 특정 라우트 주소에서 캐싱되었던 모든 데이터를 만료시키는 메서드이다.
```js
'use server'
 
import { revalidateTag } from 'next/cache'
 
export default async function action() {
  revalidatePath('/home')
}
```

추가로, 특정 요청에 대해 캐싱을 하고 싶지 않은 경우가 있을텐데, 아래와 같이 `no-store`옵션을 주면 된다.
```js
fetch('https://...', { cache: 'no-store' })
```

### React Query를 사용하는 이유와 fresh, stale, inactive
React Query가 나오기 이전에 Server State를 관리한다고 하면, 작은 프로젝트는 Fetching해온 데이터를 Redux 등의 전역 상태 관리 라이브러리를 통해 필요한 컴포넌트가 가져다 쓰도록 하는 방식이 주를 이뤘다.

그런데 애초에 그 라이브러리들이 '전역 상태 관리' 라이브러리이지, '서버 상태 관리'를 위해 만들어진 것이 아니었다보니, 여러 곳에서 비효율성이 드러나기 시작했다. Redux의 경우, 원래 라이브러리 철학 자체가 전역 상태를 '직접' 변경하는 것을 허용하지 않다보니 mutation이 필요한 상태마다 리듀서 생성이 불가피하고, 그로 인한 전체적인 Store의 크기도 비대해지는 문제가 있었다. 특히, 서버 리소스 절약 및 UX 향상을 위한 캐싱 도입 시에도 전역 상태 관리 라이브러리는 복잡한 설정들이 많이 추가돼야했다고 한다.

하지만 React Query는 Provider의 영역 내에서 서버에서 불러온 데이터를 상태로서 공유할 수 있고, 서버의 데이터와 이미 캐싱된 데이터의 변경사항을 감지할 수 있고, Redux 시절에 일일이 구현해야했던 반환값들(data, isLoading, error, ...)을 훅으로 간단하게 제공해준다. Server State를 관리하는 데 매우 최적화된 도구라고 할 수 있다.

같은 목적을 위해 태어난 Vercel의 SWR(공식문서: Stail-While-Revalidate 전략에서 유래)이라는 도구도 있다.

React Query에서는 Server State의 상태(status)를 Fresh, Fetching, Paused, Stale, Inactive의 5가지로 구분한다. 이 각각의 상태는 어떤 의미를 갖는지 알아보자.
- Fresh: 이제 막 서버에서 불러온 데이터(응답 값). staleTime동안 Fresh 상태가 유지된다.
- Stale: 서버에 있는 데이터와 차이가 있을 수도 있는 데이터(최신화 필요)
- Fetching: 서버에서 가져오고 있는 데이터(로딩 중)
- Paused: Fetching이 중단된 상태(네트워크 연결 끊김 등에 의해)
- Inactive: 캐싱되었지만 현재 페이지에서 사용되고 있지 않은 데이터

핵심은 Fresh와 Stale이다. 모든 서버 데이터는 Fresh -> Stale이 되는데, Fresh 상태의 데이터는 새로고침 시에도 캐싱된 상태에서 불러와지기 때문에 네트워크 요청이 발생하지 않는데, staleTime 이후에는 Stale 상태가 되어 새로고침 또는 Window Focus, 컴포넌트 Mount 등에 의해 서버에서 새로운 데이터를 불러오게 된다.

이 staleTime은 기본값이 0으로 설정되어 있으며, queryClient 또는 queryOption으로 설정해줄 수 있다.

gcTime도 있는데, 이건 **g**arbage **c**ollection **Time** 즉, Inactive 상태의 data가 gc에 의해 정리되는 주기를 설정하는 옵션이다.
무조건 staleTime보다는 길게 설정해줘야한다. 왜냐면 staleTime보다 gcTime이 짧게 설정될 경우, staleTime 주기가 돌아오기 전에 잠깐 Inactive 상태가 되었던 데이터가 gc에 의해 삭제돼서 의도한 staleTime동안 데이터가 캐싱되지 않을 수도 있기 때문이다.

### refetch, invalidate, reset의 차이
React Query Devtools에 보면 특정 queryData에 대해 실행할 수 있는 Refetch, Invalidate, Reset 등의 Action이 있다. 이들 Action의 기능이 무엇인지 알아보자.
- Refetch: 해당 데이터를 네트워크 요청을 통해 새로 받아온다.
- Invalidate: 해당 데이터를 '새로 받아와야하는 상태'로 만든다. Inactive 상태일 때는 네트워크 요청을 하지 않다가 컴포넌트 Mount 등에 의해 해당 데이터를 사용하는 Observer가 1 이상 존재하게 되면 그 때 네트워크 요청을 통해 새로운 데이터를 불러온다.
- Reset: reset은 queryOptions에 initialState를 설정한 경우 queryData를 initialData로 초기화하고, 따로 설정값이 없으면 네트워크 요청을 통해 데이터를 불러온다.
- Remove: 해당 데이터를 제거한다.
- Trigger Loading / Restore Loading: 해당 데이터를 로딩 상태로 만들거나 복구시킨다.
- Trigger Error / Restore Error: 해당 데이터를 불러오지 못한 상태로 만들거나 복구시킨다.

### React v18: use 훅
React 18버전에서 나온 `use`라는 훅이 있다.
```js
const value = use(resource);
```
use의 인자로 promise, context를 포함한 읽을 수 있는 값을 넣을 수 있는데, promise의 경우 이 promise가 resolve 될 때까지 use가 기다려준다는 특징이 있다.
Next.js의 loading이라는 특수한 페이지는 해당 라우트의 페이지에 이 use가 사용되고 있고, 만약 use가 pending 상태라면 알아서 loading 페이지를 보여줌으로써 로딩 상태를 쉽게 보여줄 수 있도록 한다.
use의 한 가지 특별한 점은, 다른 리액트 훅과 다르게 if문 내부에서 조건부로 호출될 수 있다는 점이다.

### Lazy Query
useQuery의 enabled 옵션은 쿼리를 조건에 따라 아예 비활성화할 때도 사용되지만, 특정 상황에서 쿼리를 활성화/비활성화 시키는 데에도 사용된다. 아래 예시를 보자.
```jsx
function Todos() {
  const [filter, setFilter] = React.useState('')

  const { data } = useQuery({
      queryKey: ['todos', filter],
      queryFn: () => fetchTodos(filter),
      // ⬇️ disabled as long as the filter is empty
      enabled: !!filter
  })

  return (
      <div>
        // 🚀 applying the filter will enable and execute the query
        <FiltersForm onApply={setFilter} />
        {data && <TodosTable data={data}} />
      </div>
  )
}
```
필터를 적용할 때에만 쿼리가 작동하도록 되어있다.
그런데 여기서 필터를 적용한 데이터를 불러오는 상황의 로딩 처리를 하려면 어떻게 해야할까? isLoading? isPending? isFetching?

사실 React Query의 isLoading은 isPending && isFetching이다.
isPending은 서버로부터 불러온 데이터가 아직 없다는 뜻이고, 따라서 초기값은 true이다.
isFetching은 서버로부터 데이터를 불러오고 있다는 뜻이고, 쿼리가 활성화될 때 true가 될 것이다. 그렇기에 공식 문서에서는 이렇게 enabled 옵션을 활용해 비활성화되거나 Lazy Query인 경우에 로딩 처리를 할 때는 isLoading을 활용하라고 권장하고 있다.

### 이외 메모
- MSW Handler 함수에서 조건부로 반환값의 타입이 달라지는 상황에서 타이핑이 복잡한 경우, `StrictResponse<any>`타입을 줄 수 있다. 임시방편이므로 급할때만 쓰자.
- useSearchParams()의 반환값인 searchParams는 읽기전용이라 수정이 불가능하다. 수정하고 싶으면 `new URLSearchParams()`의 인자로 넣어서 수정 가능한 상태로 만들어주면 된다.
- prefetchInfiniteQuery와 다르게 useInfiniteQuery는 queryOption에 initialPageParam 이외에 getNextPageParams, 다음 페이지를 어떻게 설정할것인지에 대한 기준을 넣어줘야한다.
    - 그런데 그냥 `() => 5`로 하드코딩하면 기존 게시글 삭제 등의 상황에 대응하기 어렵기 때문에, `(lastPage) => lastPage.at(-1)?.postId`처럼 항상 마지막 데이터를 기준으로 데이터를 불러올 수 있도록 하자.
    - useSuspenseQuery, useSuspenseInfiniteQuery는 상위 컴포넌트를 감싼 Suspense에 로딩 처리를 위임한다.


