## 프로젝트 초기 세팅
### public 디렉토리 & src 디렉토리
- public 디렉토리는 next server에서 전체 공개 설정이 되어있다. 따라서 누구나 접근 가능.
- src 디렉토리는 Optional하지만, 프로젝트 소스 파일을 패키징하는 용도로 활용 가능하다.
- 원래 app 디렉토리는 src 디렉토리 밖에 두는게 원칙이긴 하다.

## App Router 디렉토리 구조
### 레이아웃
- app router에서 처음 도입된 개념. page를 children으로 받는다.
- 모든 라우트에 기본적으로 적용되는 레이아웃을 설정하는 Root Layout이 있다.
- 개별 라우트에 해당하는 디렉토리 하위에 `layout.tsx`을 생성하면 해당 라우트는 루트 레이아웃 + 개별 레이아웃이 적용된다.
    - 개별 라우트에 해당하는 파일이 있으면 Root Layout의 children으로 해당 라우트의 page가 아닌 layout이 전달되기 때문

### Static Route
- Next.js는 파일 기반 라우팅 시스템을 채택해서 app 디렉토리 하위에 생성하는 디렉토리의 이름이 그대로 라우트를 표현하게 된다.
- app 디렉토리 하위에 디렉토리를 하나 생성하고 `page.tsx`를 하나 생성하면 특정 라우트를 가진 페이지가 하나 생성되는 것.
    - `/디렉토리명`으로 접속하면 해당 페이지를 볼 수 있다.

### Dynamic Route
- 라우트명을 대괄호(`[ ]`)로 감싸서 동적으로 변화하는 라우트를 구현할 수 있다.
- Pages Router 시절에도 존재했던 기능이다.

### Not Found
- app 디렉토리 하위에 `not-found.tsx`라는 이름의 파일을 만들고 페이지를 구성하면 next 서버에서 404 상태코드 반환 시 해당 페이지를 보여주게 된다.

### Route Groups
- 라우트의 관심사가 분리된다면, 해당 관심사를 소괄호(`( )`)로 감싸서 라우트들을 묶어줄 수 있다.
- 다른 디렉토리와 다르게 이 Group을 위한 디렉토리명은 주소에 반영되지 않는다.
- page는 생성할 수 없지만 layout 파일을 생성해서 개별 레이아웃 적용은 가능하다.
    - 그래서 그런 기능 때문에 Grouping의 기준은 레이아웃 적용 기준이 되기도 한다.

### Template
- Layout과 동일한 기능을 갖고 있지만, re-mount를 의도하고 싶다면 사용한다.
- layout과 같은 방식으로 `template.tsx`를 생성해서 활용한다. (GA 적용할 때 활용?)

### Link
- a태그의 대체재. a 태그의 href를 활용하는 방식은 새로고침을 유발하기 때문에 Next App을 SPA처럼 만들고 싶으면 사용을 자제하는게 좋음

### Image
- Next App에서 이미지 파일을 페이지에 그릴 때 사용하는 내장 컴포넌트
- 이미지 파일을 자동으로 최적화해준다. (그런데 sharp라는 라이브러리를 추가로 설치해줘야한다고 알고있음)

### CSS Module
- Next App에서 Server Component까지 감안했을 때 스타일링을 위한 도구를 선택한다면 아래와 같은 옵션들이 있음
    - Tailwind CSS
    - SASS
    - CSS Module
    - Vanilla Extract
- 저 도구들의 공통점은 '빌드 시점'에서 순수 CSS 코드로 변환되거나 적용된다는 것 (그래서 RSC의 매커니즘에서도 문제없이 스타일이 적용됨)

### Parallel Routes
- 하나의 라우트에서 둘 이상의 화면을 보여주고 싶을 때 활용(ex: modal)
- 디렉토리명 앞에 '@'를 붙여서 parallel route 생성 가능
- 특정 라우트 디렉토리 하위에 위치해야하며 (`layout.tsx`와 같은 레벨), `layout.tsx`에서 children과 같이 Prop으로 받아주면 된다.
- Parallel Route의 기본값은 `default.tsx`에서 정의해줄 수 있다.
    - 주소가 localhost:3000일때 children -> `page.tsx`, modal-> `@modal/default.tsx`
    - 주소가 localhost:3000/i/flow/login일 때는 children -> `i/flow/login/page.tsx`, modal -> `@modal/i/flow/login/page.tsx`

### Client Component
- app 디렉토리 하위의 컴포넌트는 기본적으로 모두 Server Component로 렌더링되지만, 페이지 상단에 `'use client'`라는 디렉티브를 추가해서 해당 컴포넌트를 Client Component로 선언해줄 수 있다.

### Intercepting Routes
- 현재 주소에서다른 주소로 이동할 때 해당 라우트로 이동하지 않고 현재 페이지의 컨텍스트를 유지하며 해당 라우트를 렌더링한다.
- Parallel Route와 같이 사용하면 라우트가 바뀌는 모달 같은거 만들 때 유용
- 인터셉팅 라우트의 컨벤션은 아래와 같다.
    - (.) : 동일 레벨 세그먼트
    - (..) : 상위 레벨 세그먼트
    - (..)(..) : 상위의 상위 레벨 세그먼트
    - (...) : 루트 레벨(app 디렉토리) 세그먼트
      ![intercepting route ex01](https://nextjs.org/_next/image?url=%2Fdocs%2Fdark%2Fintercepted-routes-files.png&w=3840&q=75&dpl=dpl_BgDMtkMC7Ys3CBykeL1toqez4tqp)

공식문서에서 가져온 이미지인데, 위의 경우처럼 feed 라우트에서 `photo/[id]`라우트로 이동(`Link` 컴포넌트, `navigate()` 등을 활용)하게되면 `(..photo)/[id]`가 이를 가로채서 렌더링하게 된다.
주의할 점은 이 컨벤션(..)은 라우팅 시스템 기준이 아니고 파일 시스템 기준이다. (Route Group, Parallel Route 등 실제 접속 요청에 활용되는 경로에 영향을 주지 못하는 것들은 무시)
- 그러면 인터셉팅 상황에 있는 본래의 라우트는 평생 못 들어가는것인가?
    - 그건 아님. 해당 라우트로 직접 접속(외부에서 접속) 또는 새로고침 시에는 인터셉팅이 발생하지 않기 때문에 원래의 페이지가 보여지게 됨.
    - 그래서 인터셉팅이 발생하는 상황과 그렇지 않은 상황을 잘 구별하는 것이 필요.

### Private Folder
- 라우팅에 영향을 주지 않는 디렉토리 컨벤션이다.
- 디렉토리명 앞에 `_`를 붙여서 디렉토리를 생성하면 Private Folder가 된다.
- UI 로직과 라우팅 로직을 분리할 수 있다는 이점이 있다. (공통 컴포넌트 같은거 여기 넣으면 좋을듯? 근데 서버 컴포넌트 선언이 default가 되니까 State를 활용하는 대부분의 공통 컴포넌트들은 활용하기 애매하지 않을까... 'use client'를 죄다 붙여줘야할지도 모른다.)

### useRouter, Redirect
- 클라이언트 컴포넌트와 서버 컴포넌트 둘 모두 특정 라우트로 리다이렉트 시키고 싶은 상황이 있을 수 있다.
- 서버 컴포넌트의 경우에는 `next/navigation`의 `redirect()` 메서드, 클라이언트 컴포넌트의 경우에는 useRouter 훅의 반환값, router 인스턴스에서 호출할 수 있는 `push()`, `replace()`를 활용해서 리다이렉트 시킬 수 있다.
    - push()는 히스토리를 남기고, replace()는 히스토리를 남기지 않는다는 차이점이 있다.
