# Section 04. Rendering Elements

## Elements의 정의와 생김새

createElement() 이름 그대로 Element를 생성해주는 함수

### Elements란?

리액트 앱을 구성하는 요소  
Elements ar the smallest building blocks of React apps  
(리액트 앱을 구성하는 가장 작은 블록들)

크롬 개발자도구의 Elements에서 확인할 수 있는 Elements들은 DOM Elements  
화면에 나타나는 내용을 기술하는 자바스크립트 객체 - Descriptor > DOM Elements

결국 React Elements는 DOM Elements의 가상 표현이라고 볼 수 있음  
DOM Elements는 React Elements에 비해 많은 정보를 담고 있기 때문에 상대적으로 크고 무거움

React Element는 화면에서 보이는 것들을 기술  
React Elements가 기술한 내용을 토대로 실제 우리가 화면에서 보게되는 DOM Elements가 만들어짐

```jsx
const element = <h1> Hello, World</h1>
```

### React Elements 생김새

React Elements는 자바스크립트 객체 형태로 존재

Elements는 컴포넌트 유형과 속성 및 내부의 모든 자식에 대한 정보를 포함하고 있는 일반적인 자바스크립트 객체  
이 객체는 마음대로 변경할 수 없는 불변성을 가지고 있음

```js
{
    type: 'button',
    props: {
        className: 'bg-green',
        children: {
            type: 'b',
            props: {
                children: 'Hello, element!'
            }
        }
    }
}
```

위처럼 type에 HTML 태그 이름이 문자열로 들어가는 경우  
엘리먼트는 해당 태그 이름을 가진 DOM 노드를 나타내고 props는 속성에 해당



## Elements의 특징 및 렌더링하기

## 시계 만들기