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
# 14.4 HTTP 보안 헤더 설정하기
  - `Strict-Transport-Security`
    -  모드 사이트가 HTTPS를 통해 접근해야 하며, 만약 HTTP로 접근하는 경우 이러한 모든 시도는 HTTPS로 변경되게 한다.