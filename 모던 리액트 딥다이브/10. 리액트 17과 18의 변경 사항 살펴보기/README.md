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