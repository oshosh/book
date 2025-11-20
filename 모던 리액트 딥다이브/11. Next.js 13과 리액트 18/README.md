## `app` 디렉터리의 등장

### 11.1.1 라우팅
  - 기존 `/pages`로 정의하던 라우팅 방식이 `/app` 디렉터리로 이동
  - 파일명으로 라우팅이 불가능
    - Next.js 12 이하: `/pages/a/b.tsx` or `/pages/a/b/index.tsx`는 모두 동일한 주소로 변경
    - Next.js 13 이상 app: `/app/a/b`는 `/a/b`로 변환되며 파일명은 무시가 되고 폴더명까지만 주소로 변환
  - `layout.js`
    - 페이지의 미치는 공통 기본적인 레이아웃 구성 요소
    - layout은 해당 주소 하위에만 적용
    - 예약어로서 `layout.{js|jsx|ts|tsx}`로 사용
  - `page.js`
    - props로는 params는 옵셔널 값으로서 `[...id]`와 같은 동적 파라미터를 사용할 경우 해당 파라미터 값이 들어와있음
    - searchParams는 URLSearchParams를 의미하며 layout에서는 제공하지 않음
  - `error.js`
    - 라우팅 영역에서 사용되는 공통 에러 컴포넌트다.
    - 에러 바운더리는 클라이언트에서만 작동하므로 클라리언트 컴포넌트여야한다.
  - `not-found.js`
    - 특정 라우팅 하위 주소를 찾지 못할 경우
  - `loading.js`
    - 리액트 `Suspense`를 기반으로 해당 컴포넌트가 불러오는 중임을 나타낼 때 사용
  - `route.js`
    - `/pages/api`와 동일하게 `/app/api`를 기준으로 디렉터리 라우팅을 지원
      - request: API 요청과 관련된 cookie, headers 등뿐만 아니라 nextUrl 같은 주소 객체도 확인
      - context: params만 가지고 있는 객체

### 11.2 리액트 서버 컴포넌트
### 11.2.1 기존 리액트 컴포넌트와 서버 사이드 렌더링의 한계
  - 자바스크립트 번들 크기가 0인 컴포넌트를 만들 수 없다: 게시판 등 사용자가 작성한 HTML에 위험한 태그를 제거하기 위해 사용되는 유명한 sanitize-html라이브러리를 사용한다고 가정할때, 컴포넌트를 서버에서 렌더링하고 클라이언트는 결과만 받는다면 sanitize-html 라이브러리를 다운로드해 실행하지 않고도 사용자에게 렌더링 할 수 있을 것이다.
  ```
  import sanitizeHtml from "sanitize-html"; // 206k (63.3K gzipped)

  function Board({ text }: { text: string }) {
    const html = useMemo(() => sanitizeHtml(text), [text]);

    return <div dangerouslySetInnerHtml={{ __html: html }} />;
  }
  ```
  - 백엔드 리소스에 대한 직접적인 접근이 불가능하다
  - 자동 코드 분할이 불가능하다: 개발자는 항상 코드 분할을 해도 되는 컴포넌트인지를 유념하고 개발해야 하기 때문에 이를 누락하는 경우가 발생할 수 있고 if문 전까지 어떤 컴포넌트를 불러올지 결정할 수 없다.
  - 연쇄적으로 발생하는 클라이언트와 서버의 요청을 대응하기 어렵다: 요청이 연달아 컴포넌트를 렌더링한다고 할때 최초 컴포넌트의 요청과 렌더링이 끝나기 전까지는 하위 컴포넌트의 요청과 렌더링이 끝나지 않는다는 큰 단점이 있다.
  - 추상화에 드는 비용이 증가한다: 리액트는 탬플릿 언어로 설계되지 않았다. 탬플릿 언어란 HTML에 특정 언어의 문법을 집어넣어 사용할 수 있는 것을 의미한다. 복잡한 추상화에 따른 결과물을 연산하는 작업을 서버에서 수행하게 된다면, 클라이언트에서는 속도가 빨라지고 서버에서는 클라이언트로 전송되는 결과물도 가벼워질 것이다

### 11.2.2 서버 컴포넌트란?
  - 서버 컴포넌트란 하나의 언어, 하나의 프레임워크, 그리고 하나의 API와 개념을 사용하면서 서버와 클라이언트 모두에서 컴포넌트를 렌더링할 수 있는 기법을 의미한다.
  - 클라이언트 컴포넌트는 서버 컴포넌트를 import 할 수 없다. 그 이유는 서버 컴포넌트를 불러오게 된다면 서버 컴포넌트를 실행할 방법이 없기 때문이다.
  - 브라우저에서 실행되지 않고 서버에서만 실행되기 때문에 DOM API나 window.document 등 접근할 수 없다.
  ```
  <!-- 불가 -->
  "use client";
  import ServerCard from "./ServerCard"; // 에러

  export default function Client() {
    return <ServerCard />;  {/* 서버 컴포넌트 */}
  }

  <!-- 가능 -->
  // app/page.tsx  (서버 컴포넌트)
  import Profile from "./Profile";
  import ProfileChart from "./ProfileChart"; // 클라 컴포넌트

  export default async function Page() {
    const user = await fetchUser(); // DB, API, 비밀키 가능 (서버)

    return (
      <div>
        <Profile user={user} />        {/* 서버 컴포넌트 */}
        <ProfileChart userId={user.id} /> {/* 클라이언트 컴포넌트 */}
      </div>
    );
  }
  ```

### 11.2.3 서버 사이드 렌더링과 서버 컴포넌트의 차이
  - 서버 사이드 렌더링
    - 정적인 HTML을 빠르게 내려주는 데 초점을 두고 있다.
    - 클라이언트에서 하이드레이션을 통해 재구성을 해야 인터랙션이 가능하다.
    - 여전히 HTML이 로딩된 이후에는 자바스크립트 코드를 다운로드하고 파싱하고 실행하는데 비용이 든다.

  - 서버 컴포넌트
    - 서버에서 실행 가능한 컴포넌트는 서버에서 React가 직접 실행되어 그 결과가 ‘RSC Payload(직렬화된 컴포넌트 트리 데이터)’로 브라우저에 전달된다. Payload는 HTML이 아니라, 클라이언트에서 최종 HTML을 만들기 위한 React 전용 데이터다.

### 11.2.4 서버 컴포넌트는 어떻게 작동하는가?
  - 서버가 렌더링 요청을 받는다. 서버가 렌더링 과정을 수행해야 하므로 서버에서 시작된다.
  - 서버에서 받은 요청에 따라 컴포넌트를 JSON으로 직렬화 한다. 이때 서버에서 렌더링할 수 있는 것은 직렬화해서 내보내고, 클라이언트로 표시된 부분은 해당 공간을 플레이스홀더 형식으로 비워두고 나타낸다. 브라우저는 이를 역질렬화해서 렌더링을 수행한다.
  - 브라우저가 리액트 컴포넌트 트리를 구성한다. 브라우저가 서버로 스트리밍으로 JSON 결과물을 받았다면 이 구문을 다시 파싱한 결과물을 바탕으로 트리를 재구성해 컴포넌트를 만들어 나간다.

## 11.3 Next.js에서의 리액트 서버 컴포넌트
### 11.3.1 새로운 fetch 도입과 getServerSideProps, getStaticProps, getInitialProps의 삭제
  - 13 버전 이후로는 `fetch` 기반으로 요청하도록 변경
  - 서버 컴포넌트 트리내에 동일한 요청이 있다면 재요청이 발생하지 않도록 요청 중복을 방지

### 11.3.2 정적 렌더링과 동적 렌더링
  - 13 버전에서는 정적 라우팅에 대해 기본적으로 빌드 타임에 렌더링을 미리하고 캐싱해 재사용할 수 있게 되었음.
  - 동적 라우팅은 서버에 매번 요청 올때 컴포넌트를 렌더링하도록 변경
  - 동적인 렌더링을 원할 경우 { cache: 'no-cache' } 옵션을 추가해서 요청이 올때 마다 렌더링을 수행하게 하면되고, 정적일 경우엔 제거하면 된다.
    ```
    async function getData() {
      // 
      const res = await fetch("https://api.example.com/...", { cache: "no-cache" });
      // The return value is *not* serialized
      // You can return Date, Map, Set, etc.

      if (!res.ok) {
        // This will activate the closest `error.js` Error Boundary
        throw new Error("Failed to fetch data");
      }

      return res.json();
    }

    export default async function Page() {
      const data = await getData();

      return <main></main>;
    }
    ```
  - `generateStaticParams`: 동적인 주소이지만 특정 주소에 대해서만 캐싱하고 싶을 경우
    - https://nextjs.org/docs/app/api-reference/functions/generate-static-params

### 11.3.3 캐시와 mutating, 그리고 revalidating
  - 페이지에 `revalidate`라는 변수를 선언해서 페이지 레벨로 정의하는 것도 가능
    - 최초로 해당 라우트 요청이 올 때는 미리 정적으로 캐시해 둔 데이터를 보여준다.
    - 이 캐시된 초기 요청은 `revalidate`에 선언된 값만큼 유지된다.
    - 만약 해당 시간이 지나도 일단은 캐시된 데이터를 보여준다.
    - `Next.js`는 캐시된 데이터를 보여주는 한편, 시간이 경과했으므로 백그라운드에서 다시 데이터를 불러온다.
    - 작업이 성공적으로 끝나면 캐시된 데이터를 갱신하고, 그렇지 않다면 과거 데이터를 보여준다.

### 11.3.4 스트리밍을 활용한 점진적인 페이지 불러오기
  - 과거 서버 사이드 렌더링 요청
    - 페이지를 모두 렌더링 해서 내려줄때 까지 사용자에게 보여줄 수 있는게 없음
    - 페이지를 전달 받아도 사용자는 인터랙션이 불가한 정적 페이지
    - 리액트에서 하이드레이션을 거쳐 사용자가 사용할 수 있는 페이지가 됨
  - RSC의 경우
    - 하나의 페이지를 다 완성될때 까지 기다리지 않고 HTML을 작은 단위로 쪼개서 클라이언트로 점진적으로 보내는 방식이 도입됨
    - 최초 바이트까지의 시간(TTFB)과 최초 콘텐츠풀 페인팅(FCP) 개선에 도움이 됨

## 11.5 서버 액션
  - 매커니즘
    - form의 action
      - 서버 액션은 클라이언트에서 직접 실행되지 않고 서버로 Action_id를 form으로 전송
      - 서버 액션은 번들에 포함되지 않고 서버에서만 실행 할 수 있도록 프론트 JS 네트워크를 통해 함수 소스가 아니라 함수 ID만 전달함
      - 서버 액션은 빌드 결과물이 클라이언트에 노출되지 않는다. 즉, 빌드 시점에 `use server`를 보고 빌드 시 자동으로 클라이언트로부터 분리가 됨
      - 서버 액션을 사용하면 요청이 서버 측에서 직접 실행
  - `startTransition`과의 연동
    - client 컴포넌트에서 서버 액션 호출 시에도 중복 호출을 막을 수 있는 장점이 있음
      - 예전 클라이언트 컴포넌트에서 굳이 쓰로틀링을 할 필요가 없어짐
    ```
    "use server"
    export async function updateData({
      id: string,
      data: {name: string, age: number}
    }) { 
      ...
      revalidatePath(`/server-action/form/${id}`)  
    }

    "use client"
    const [isPending, startTransition] = useTransition()

    const handleClick = () => {
      // 서버 액션 동작
      startTransition(() => updateData(id, { name: '기본값', age: 0 }))
    }

    return isPending ? <SkeletonBtn /> : (
      <button onClick={handleClick}>기본값으로 돌리기</button>
    )
    ```