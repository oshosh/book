## 14.1 XSS
  - XSS란 웹사이트 개발자가 아닌 제3자가 웹사이트에 악성 스크립트를 삽입해 실행할 수 있는 취약점을 의미
  - `dangerouslySetInnerHTML` prop
    - 인수로 받는 문자열에 제한이 없다는 것
  ```
  function App() {
    return <div dangerouslySetInnerHTML={{ __html: 'First &middot; Second' }} />
  }
  ```
  - useRef를 활용한 직접 삽입
  ```
  const html = `<span><svg/onload=alert(origin)></span>'

  function App() {
    const divRef = useRef<HTMLDivElement>(null)
    useEffect(() => {
      if (divRef.current) {
        divRef.current.innerHTML = html
      }
    })      
    return <div ref={divRef} />
    }
  ```
### 14.1.3 XSS 문제 피하기
  - `DOMpurify`
  - `sanitize-html`
  - `js-xss`

## 14.2 getServerSideProps와 서버 컴포넌트를 주의하자
  - 쿠키를 가져와 클라이언트에서 유효성 확인 하는 일부 코드
    - 보안 관점에서는 좋지 않음
  ```
  export default function App({ cookie }: { cookie: string }) {
    if (!validateCookie(cookie)) {
      Router.replace()
      return null
    }
  }

   export const getServerSideProps = async (ctx: GetServerSidePropsContext) => {
     const cookie = ctx.req.headers.cookie || ''
      return {
        props: {
          cookie,
        }
      }
    }
  ```

  - 토큰 검증을 SSR에서 하자
    - 유효하지 않으면 컴포넌트 렌더링 자체를 하지 않고 바로 redirect
  ```
  export default function App({ token }: { token: string }) { 
    const user = JSON.parse(window.atob(token.split('.')[1]))
    const user_id = user.id
  }

  export const getServerSideProps = async (ctx: GetServerSidePropsContext) => {
    const cookie = ctx.req.headers.cookie || ''
  
    const token = validateCookie(cookie)

    if (!token) {
      return {
        redirect: {
          destination: '/404',
          permanet: false,
        },
      }
    }
    return {
      props: { token },
    }
  }
  ```

# 14.3 <a> 태그의 값에 적절한 제한을 둬야 한다.
  - a 태그의 기본 기능을 막아 href의 url 이동을 막고 onClick 이벤트와 같이 별도 이벤트 핸들러 작동 시키는 구버전 코드를 확인 해보자
    - 페이지 이동이 없이 핸들러만 작동 시키고 싶다면 button 태그 사용하자
    - href가 작동하지 않은 것이 아니라 `javascript:;`만 실행된 것이다. 즉 href내에 다른 자바스크립트 코드가 존재한다면 이를 실행한다는 뜻
  ```
  function App() {
    const handleClick = () => {
      console.log('hi')
    }

    return (
      <>
        <a href="javascript:;" onClick={handleClick}>
          링크
        </a>
      </>
    )
  }
  ```
## 14.4 HTTP 보안 헤더 설정하기
  - `Strict-Transport-Security`
    -  모드 사이트가 HTTPS를 통해 접근해야 하며, 만약 HTTP로 접근하는 경우 이러한 모든 시도는 HTTPS로 변경되게 한다.
  - `X-XSS-Protection`
    - 이는 비표준 기술로, 구형 브라우저와 사파리에서만 제공되는 기능이다.
    - XSS 취약점이 발견되면 페이지 로딩을 중단하는 헤더다. 이 헤더를 전적으로 믿어서는 안되며, 반드시 페이지 내부에서 XSS에 대한 처리가 존재하는 것이 좋다.
  - `X-Frame-Options`
    - 페이지를 frame, iframe, embed, object 내부에서 렌더링을 허용할지를 나타낼 수 있다.
  - `Permissions-Policy`
    - 웹사이트에서 사용할 수 있는 기능과 사용할 수 없는 기능을 명시적으로 선언하는 헤더다.
  - `X-Content-Type-Options`
    - MIME을 먼저 알아야하는데, MIME(Multipurpose Internet Mail Extensions)는 Content-type의 값으로 사용된다. 원래는 메일을 전송할 때 사용하던 인코딩 방식으로, 현재는 Content-type에서 대표적으로 사용되고 있다.
    - Content-type 헤더에서 제공하는 MIME 유형이 브라우저에 의해 임의로 변경되지 않게 하는 헤더이다. 즉, Content-type: text/css 헤더가 없는 파일은 브라우저가 임의로 CSS로 사용할 수 없다. 웹서버가 브라우저에 강제로 이 파일을 읽는 방식을 지정하는 것이 바로 이 헤더다.
  - `Referrer-Policy`
    - Referer라는 헤더는 현재 요청을 보낸 페이지의 주소가 나타난다. 만약 링크를 통해 들어왔다면 해당 링크를 포함하고 있는 페이지 주소가 다른 도메인에 요청을 보낸다면 해당 리소스를 사용하는 페이지 주소가 포함된다.
  - `Content-Security-Policy`
    - `*-src`
        - font-src, img-src, script-src 등 다양한 src를 제어할 수 있는 지시문이다.
    - `form-action`
        - form-action은 폼 양식으로 제출할 수 있는 URL을 제한할 수 있다.
  ### 14.4.8 HTTP 보안 헤더 설정하기
    - Next.js에서는 애플리케이션 보안을 위해 HTTP 경로별로 보안 헤더를 적용할 수 있다.
    - 정적인 파일을 제공하는 NGINX의 경우 다음과 같이 경로별로 add_header지시자를 사용해 원하는 응답헤더를 더 추가할 수 있다.