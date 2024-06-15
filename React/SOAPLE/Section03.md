# Section 03. JSX

## JSX의 정의와 역할

### 정의

JSX란? A syntax extension to JavaScript  
자바스크림트의 문법을 확장시킨것

JavaScript + XML / HTML  
JSX의 X는 extension이라고 볼수도 있지만 XML의 앞글자를 따서 JSX라고 함

```jsx
const element = <h1>Hello, world!</h1>;
```

특이하게 대입 연산자 오른족에 HTML 코드가 존재  
Javascript 코드와 HTML 코드가 결합되어 있는 JSX 코드

### 역할

JSX는 내부적으로 XML, HTML 코드를 JS로 변환하는 과정을 거치게 됨  
실제로 JSX로 작성해도 최종적으로는 JS 코드가 나오게 되는 것

JSX 코드를 JS 코드로 변환하는 역할을 하는 것이 React의 `createElement()`

JSX 코드와 해당 JSX 코드를 JS로만 작성하면 어떻게 작성되는지 살펴봄

```jsx
// jsx
class Hello extends React.Component {
    render() {
        return <div>Hello {this.props.toWhat}</div>;
    }
}

ReactDOM.render(
    <Hello toWhat="World" />
    document.getElementById('root')
);

// javascript
class Hello extends React.Component {
    render() {
        return React.createElement('div', null, `Hello ${this.props.toWhat}`);
    }
}

ReactDOM.render(
    React.createElement(Hello, { toWhat: 'World' }, null),
    document.getElementById('root')
);
```

React.createElement()의 파라미터를 알아봄

```js
React.createElement(
    type,
    [props],
    [...children]
)
```

* type
  * element 유형
  * div, span 같은 HTML 태그 혹은 다른 React 컴포넌트가 들어갈 수 있음
* props
  * 속성
* children
  * 현재 엘리먼트가 포함하고 있는 자식 엘리먼트

그렇다고 React에서 JSX를 사용하는 것이 필수는 아님. (`createElement()`를 사용하면 되므로)  
하지만 JSX를 사용하면 장점이 많기 때문에 편리함

## JSX의 장점 및 사용법

### 장점

* 코드가 간결해짐
* 가독성이 향상됨
  * 버그를 발견하기 쉬움

```jsx
// jsx
<div>Hello, {name}</div>

// javascript
React.createElement('div', null, 'Hello, ${name});
```

* Injection Attacks 방어
  * Injection Attacks - 입력창에 소스 코드를 입력하여 해당 코드를 실행되게 만드는 해킹 방법

```jsx
const title = response.potentiallyMaliciousInput;

// 이 코드는 안전함
const element = <h1>{title}</h1>;
```

명시적으로 선언되지 않은 값은 괄호 사이에 들어갈 수 없으며,  
XSS라고 불리는 크로스 사이트 스크립팅 어택을 방어할 수 있음

### 사용법

```jsx
const name = '이름'
const element = <h1>안녕, {name}</h1>;

ReactDOM.render(
    element,
    document.getElementById('root')
);
```

XML, HTML 코드를 사용하다가 중간에 JS 코드를 사용하고 싶으면 `{ }`를 사용

```jsx
// 큰따옴표 사이에 문자열
const element = <div tabIndex="0"></div>

// 중괄호 사이에 JS 코드
const element = <img src={user.avatarUrl}></img>
```

태그의 속성에 값을 넣고 싶을 때에는 큰 따옴표 사이에 문자열을 넣거나 중괄호 사이에 JS 코드를 넣으면 됨  
JSX에서는 중괄호를 사용하면 무조건 JS 코드가 들어감

```jsx
const element = (
    <div>
        <h1>안녕하세요</h1>
        <h2>열심히 리액트를 공부해 봅시다!</h2>
    </div>
);
```

JSX를 사용해서 children을 정의하려면  
위처럼 평소에 HTML을 사용하듯이 상위 태그가 하위 태그를 둘러싸도록 하면 자연스럽게 children이 정의됨

## JSX 코드 작성해보기

Section 02에서 create-react-app을 진행한 부분에서 추가로 실습을 진행

src 폴더에 chapter_03 폴더를 생성하고 아래의 jsx 파일들을 생성

```jsx
// ./chapter_03/Book.jsx
import React from "react";

function Book(props) {
    return (
        <div>
            <h1>{`이 책의 이름은 ${props.name}입니다.`}</h1>
            <h2>{`이 책은 총 ${props.numOfPage}페이지로 이루어져 있습니다.`}</h2>
        </div>
    );
}

export default Book;

// ./chapter_03/Library.jsx
import React from "react";
import Book from "./Book";

function Library(props) {
    return (
        <div>
            <Book name="처음 만난 파이썬" numOfPage={300} />
            <Book name="처음 만난 AWS" numOfPage={400} />
            <Book name="처음 만난 리액트" numOfPage={500} />
        </div>
    );
}

export default Library;
```

Root 디렉토리의 index.js 파일을 아래와 같이 일부분 수정

```js
// index.js
import Library from './chapter_03/Library';

const root = ReactDOM.createRoot(document.getElementById('root'));
root.render(
  <React.StrictMode>
    <Library />
  </React.StrictMode>
);
```

Root 디렉토리의 터미널에서 아래 명령어를 실행하여 결과를 확인

```bash
$ npm start
```

JSX 를 사용하지 않아도 동일한 결과물을 만들 수 있지만 JSX를 사용하지 않으면 아래의 코드처럼 알아보기 어려움

```jsx
import React from "react";

function Book(props) {
    return React.createElement(
        'div',
        null,
        [
            React.createElement(
                'h1',
                null,
                `이 책의 이름은 ${props.name}입니다.`
            ),
            React.createElement(
                'h2',
                null,
                `이 책은 총 ${props.numOfPage}페이지로 이루어져 있습니다.`
            )
        ]
    )
}

export default Book;
```

React를 사용함에 있어 JSX는 필수적인 요소에 가깝고 공식 웹사이트에서도 JSX의 사용을 권장하고 있기 때문에 JSX의 사용법을 잘 익혀서 사용하길 권장
