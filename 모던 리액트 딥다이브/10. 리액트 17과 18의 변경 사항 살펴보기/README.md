## 10.1 리액트 17버전 살펴보기
  - 버전 16 과 다르게 새롭게 추가된 기능 없음
  - 기존 코드의 수정에 필요로하는 변경사항을 최소화
  
### 10.1.1 리액트의 점진적인 업그레이드
  - 이전 버전까지는 새로운 주 버전 릴리스 시 이전 버전의 API 제공이 완전히 중단되었으나 17 버전부터 점진적인 업그레이드가 가능해짐

### 10.1.2 이벤트 위임 방식의 변경
 - [07. 크롬 개발자 도구를 활용한 애플리케이션 분석](../07.%20크롬%20개발자%20도구를%20활용한%20애플리케이션%20분석/)에서 상세하게 설명했으니 참고
 - react 16은 이벤트가 `document`로 전부 부착된다.
  - 그렇기 때문에 16 버전에서 다른 버전의 리액트 컴포넌트를 부착 시켰다고 가정 할때 예를 들어 e.stopPropagation을 해도 document에 등록된 다른 React 버전의 이벤트가 실행되는 부작용이 생길 수 있다. 
  - 17 버전 이후로는 각 React root마다 이벤트 위임 루트가 독립적으로 존재하고, SyntheticEvent도 해당 root 내부에서만 처리된다.

 ```
 import React from 'react' // 16.14
import ReactDOM from 'react-dom' // 16.14

function React1614() {
  function App() {
    function 안녕하세요() {
      alert('안녕하세요! 16.14')
    }
    return <button onClick={안녕하세요}>리액트 버튼</button>
  }
  return ReactDOM.render(<App />, document.getElementById('React-16-14'))
}

import React from 'react' // 16.8
import ReactDOM from 'react-dom' // 16.8

function React168() {
  function App() {
    function 안녕하세요() {
      alert('안녕하세요! 16.8')
    }
    return <button onClick={안녕하세요}>리액트 버튼</button>
  }
  return ReactDOM.render(<App />, document.getElementById('React-16-8'))
}

<!-- 만약 다른 버전의 리액트 컴포넌트가 들어오게 되면 문제가 발생......... -->
<html>
  <body>
    <div id='React-16-14'>
      <div id='React-16-8' />
    </div>
  </body>
</html>
 ```

 ### 10.1.3 import React from 'react'가 더 이상 필요 없다: 새로운 JSX transform
  - 리액트 17부터 `babel`과 협력을 통해 import 구문 없이 JSX를 변환할 수 있음.
    - 불필요한 번들링 크기를 줄일 수 있음.
    - 정확히는 `babel 7.9.0이상`이며 17 버전 보다 먼저 릴리즈가 되어 있다.
  - `JSX` → `React.createElement` 방식의 문제점
    - `import React from ‘react’`이 필수이다.
    - 성능과 기술부채 등 많은 제약이 있다. (create-element-change 관련 - RFC)[https://github.com/reactjs/rfcs/blob/createlement-rfc/text/0000-create-element-changes.md]
      - `React.createElement`는 엘리먼트 생성시 마다 `.defaultProps`가 있는지 동적 검사를 해야함 
        - 치명적인 문제로 보이는데 `React.createElement` 보다 `.defaultProps`가 먼저 판단이 되는 문제로 보인다.
      - React.lazy 동작의 어려움과 props의 구조등 가변 인자의 변이 객체 판단이 어려운점
        - 이 문제 또한 class component 구조에서 `defaultProps`를 사용하면서 아직 컴포넌트가 뭔지도 모르는 상태에서 먼저 판단되기 때문에 어려운점이 해당 기능을 삭제 하기위함으로 보인다.
      - 새로운 JSX transform: children을 항상 props.children에 넣어 전달
        - 런타임에서 children을 해석해 실제 얼마나 많은 children을 처리해야하는지 알수가 없는 것으로 보임
        ```
        <div>Hello</div>

        React.createElement("div", null, "Hello");
        jsx("div", { children: "Hello" });

        <div>{a}{b}{c}</div>
        React.createElement("div", null, a, b, c);
        jsxs("div", { children: [a, b, c] });
        ```
      - 등등..... 여럿 문제를 야기하는 듯 함....
    => 결론은 react/jsx-runtime의 jsx함수로 변환하여 babel이 inject를 하는 방식으로 변경되었다.
  ```
  const Component = (
    <div>
      <span>hello world</span>
    </div>
  )

  // 리액트 16에서 변환되는 코드
  var Component = React.createElement(
    'div',
    null,
    React.createElement('span',null, 'hello world'),
  )

  // 리액트 17에서 변환되는 코드
  'use strict'
  var _jsxRuntime = require('react/jsx-runtime')

  var Component (0, _jsxRuntime.jsx)('div', {
    children: (0, _jsxRuntime.jsx)('span', {
      children: 'hellow world',
    })
  })
  ```
  - `babel/repl`에서 @babel/preset-react의 runtime을 각각 변경해 비교해보자
  ```
  // React 일반 코드
  import {forwardRef} from 'react'

  function Foo() {
    return (
        <h1>
            <div>hi</div>
            <div>hi2</div>
        </h1>  
      )
  }

  function Foo2() {
    return (
        <h1>
            <div>hi</div>
        </h1>  
      )
  }


  // 바벨 설정 "runtime": "classic" vs "runtime": "automatic"
  {
    "filename": "repl.jsx",
    "presets": [
      [
        "env",
        {
          "targets": "defaults, not ie 11, not ie_mob 11",
          "modules": false,
          "bugfixes": true
        }
      ],
      [
        "react",
        {
          "runtime": "classic"
        }
      ]
    ],
    "sourceType": "module"
  }
  ```

  - `automatic` 결과
  ```
  import { forwardRef } from 'react';
  import { jsx as _jsx } from "react/jsx-runtime";
  import { jsxs as _jsxs } from "react/jsx-runtime";

  function Foo() {
    return /*#__PURE__*/_jsxs("h1", {
      children: [/*#__PURE__*/_jsx("div", {
        children: "hi"
      }), /*#__PURE__*/_jsx("div", {
        children: "hi2"
      })]
    });
  }

  function Foo2() {
    return /*#__PURE__*/_jsx("h1", {
      children: /*#__PURE__*/_jsx("div", {
        children: "hi"
      })
    });
  }
  ```

  - `classic` 결과
  ```
  import { forwardRef } from 'react';

  function Foo() {
    return /*#__PURE__*/React.createElement("h1", null, /*#__PURE__*/React.createElement("div", null, "hi"), /*#__PURE__*/React.createElement("div", null, "hi2"));
  }

  function Foo2() {
    return /*#__PURE__*/React.createElement("h1", null, /*#__PURE__*/React.createElement("div", null, "hi"));
  }
  ```

### 10.1.4 그 밖의 주요 변경 사항
  - 이벤트 폴링 제거
    - 리액트는 이벤트 처리를 위해 `SyntheticEvent`를 통해 브라우저의 기본 이벤트가 아니라 한번 더 래핑한 이벤트를 사용하는데 이벤트 발생 시 이벤트를 새로 만들어야하고 메모리 할당 작업이 일어나 메모리 누수를 방지 하기 위해서 주기적으로 해제를 하게 되는데 이러한 작업을 16버전에서는 이벤트 폴링을 발생 시켰다.
  - 이벤트 폴링 시스템
    1. 이벤트 핸들러가 이벤트를 발생
    2. 합성 이벤트 풀에서 합성 이벤트 객체에 대한 참조를 가져옴
    3. 이벤트 정보를 합성 이벤트 객체에 넣는다.
    4. 지정된 이벤트 리스너 실행
    5. 이벤트 객체가 초기화 되고 다시 이벤트 풀로 돌아감.
  - 이벤트 폴링 시스템의 단점
    - 이벤트 폴링 시스템은 어떻게 보면 합리적인 것 처럼 보인다. 예를 들어 16버전 이하의 코드를 살펴보자.
    - 코드
      - 아래와 같은 문제가 발생 하는 이유는 서로 다른 이벤트 간에 이벤트 객체를 재사용하고 그 사이에 모든 이벤트 필드를 null로 변경한다. 즉, 이벤트 핸들러 호출 시 `SyntheticEvent`(합성 이벤트) 이후 재사용을 위해 null로 초기화 된다. 그렇기 때문에 e로 접근 시 초기화 되어 아래와 같은 문제가 발생한다.

    ```
    function handleChange(e) {
      // This won't work because the event object gets reused.
      setTimeout(() => {
        console.log(e.target.value); // Too late!
      }, 100);
    }

    function handleChange(e) {
      // Prevents React from resetting its properties:
      e.persist();

      setTimeout(() => {
        console.log(e.target.value); // Works
      }, 100);
    }
    ```
  - `useEffect` 클린업 함수의 비동기 실행
    - 16버전에서는 클린업 시에 동기적 처리에서 비동기로 변경되었다.
      1. 렌더(update) 이후
      2. 커밋 단계(commit) 직전/직후에 클린업이 동기 실행됨
        - 여기서 시간이 길어질 수 있음..........
      3. 화면 paint 
    - 17 버전에서 변화
      1. 렌더(update) 이후
      2. 커밋 단계(commit)에서 DOM 변경 후 바로 화면 paint
      4. 비동기 클린업 실행
  - 컴포넌트 `undefined` 반환에 대한 일관적인 처리
    - React 16 / 17
      - 컴포넌트가 `undefined`를 반환하면 dev 모드에서 에러를 던진다.
        - 메시지: "Nothing was returned from render. This usually means a return statement is missing..."
      - (책에서 말하는 것처럼) forwardRef/memo 같은 일부 케이스는
        버전에 따라 에러를 제대로 못 잡던 버그가 있었다고 설명하는 자료도 있음.
        하지만 "정상 스펙" 기준으론 `undefined` = 에러.
    - React 18
      - 컴포넌트가 `undefined`를 반환해도 에러/경고를 **더 이상 내지 않는다.**
      - 그냥 "아무것도 렌더링하지 않음"으로 취급한다.

## 10.2 리액트 18버전 살펴보기
  - 가장 큰 변경점은 동시성 지원이다.
    - 해당 내용은 2장에서 정리를 했었음.
    - 16버전에서 `Stack Reconciler`에 대한 문제점으로 인하여 렌더링(재귀 호출) 발생 시 중단이 불가능한 문제로 무조건 동기 방식으로 작동하였다.
    - 16.8 버전 부터 fiber 아키텍처가 나오게 되었고 그 가반으로 렌더링 흐름을 직접 제어가 가능하도록 17 버전에서 준비기간을 가지고 나온것으로 보임.
  - `useId`
    - 유니크한 값을 생성하는 새로운 훅
  - `useTransition`
    - 무거운 렌더링 작업을 조금 미룰 수 있어 좋은 사용자 경험을 제공
  - `useDeferredValue`
    - 리렌더링이 급하지 않은 부분을 지연할 수 있게 도와주는 훅
  - `useSyncExternalStore`
    - 테어링(tearing)현상을 방지하기 위해 나타난 훅 렌더링을 일시 중지하거나 뒤로 미루는 등의 최적화가 가능해지면서 동시성 이슈가 발생

### 10.2.2 react-dom/client
  - `createRoot`
    ```
    // 18v 이전의 루트
    ReactDOM.render(, document.getElementById('root'));

    // 18v 이후의 루트 API
    import * as ReactDOMClient from ‘react-dom/client’;
    const container = document.getElementById('‘app');
    const root = ReactDOMClient.createRoot(container);
    root.render();
    ```

### 10.2.4 자동 배치
  - 리액트가 여러 상태 업데이트를 한번의 리렌더링으로 묶어서 업데이트를 한다.
  - 17 이하 버전에서도 이벤트 핸들러 내부에 자동 배치가 되나 비동기 이벤트에서는 자동 배치가 이뤄지지 않았다.
  ```
  const [count, setCount] = useState(0);
  function increase() {
    setCount(count + 1);
    setCount(count + 1);
    setCount(count + 1);
    // count: 1
  }

  // prev 사용
  function another() {
    setCount((prev) => prev + 1);
    setCount((prev) => prev + 1);
    setCount((prev) => prev + 1);
    // count: 3
  }

  import { flushSync } from "react-dom";
  // ReactDOM.flushSync() 사용
  function flush() {
    flushSync(() => {
        setCount(count + 1);
    });
    flushSync(() => {
        setCount(count + 1);
    });
    flushSync(() => {
        setCount(count + 1);
    });
    // count: 3
  }
  ```

### 10.2.6 Suspense 기능 강화
  - 16.6 버전에서 도입된 기능으로, 동적 컴포넌트를 가져오는데 있어 초기 렌더링 속도 향상 및 지연 로딩을 통해 많은 기능을 하고 있다.
  - 18 버전 이하에서는 useEffect에서 Suspense 컴포넌트가 보이기도 전에 실행 되는 문제가 있다. 그렇기 때문에 next.js에서는 client 컴포넌트로 랩핑을 해야하는 부분이 존재 했는데 현재는 해결이 되었다.
  - 현 18버전에서는 React.lazy 및 promise와 같은 비동기 처리에 대해 자연스러운 지원이 가능하다.
  - Suspense가 로딩 중인 경우 effect가 실행되지 않고 Hydration 중에도 Suspense fallback이 제대로 동작한다.
    - `use` 나 `await`를 붙혀 서버 컴포넌트에서 Promise를 “기다리지 않고” 바로 throw 하여 fallback 표시를 하도록 동작이 되고 데이터가 반영이 되면서 resolve 되며 UI가 교체가 됩니다.

