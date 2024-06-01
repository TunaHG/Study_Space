# Section 02. 리액트 시작하기

## 직접 리액트 연동하기

HTML 코드를 사용하여 리액트 연동

```html
<!-- index.html -->
<html>
    <head>
        <title>블로그</title>
        <!-- index.html 파일과 style.css 파일이 동일한 디렉토리에 위치해야 함 -->
        <link rel="stylesheet" href="style.css">
    </head>
    <body>
        <h1>블로그에 오신 여러분을 환영합니다!</h1>

        <!-- DOM Container (Root DOM Node) -->
        <div id="root"></div>

        <!-- 리액트 가져오기 -->
        <script src="https://unpkg.com/react@17/umd/react.development.js" crossorigin></script>
        <script src="https://unpkg.com/react-dom@17/umd/react-dom.development.js" crossorigin></script>

        <!-- 리액트 컴포넌트 가져오기 -->
        <script src="MyButton.js"></script>
    </body>
</html>
```

```css
/* style.css */
h1 {
    color: green;
    font-style: italic;
}
```

```js
// MyButton.js
function MyButton(props) {
    const [isClicked, setIsClicked] = React.useState(false);

    return React.createElement(
        'button',
        { onClick: () => setIsClicked(true) },
        isClicked ? 'Clicked!' : 'Click here!'
    )
}

const domContainer = document.querySelector('#root');
ReactDOM.render(React.createElement(MyButton), domContainer);
```

## create-react-app

```bash
# npm 라이브러리 설치
brew install npm

# create-react-app
npx create-react-app my-app

# app 실행
cd my-app
npm start
```