---
title: '教程：井字棋游戏 | Tutorial: Tic-Tac-Toe'
---

<Intro>

You will build a small tic-tac-toe game during this tutorial. This tutorial does not assume any existing React knowledge. The techniques you'll learn in the tutorial are fundamental to building any React app, and fully understanding it will give you a deep understanding of React.
本教程将引导你逐步实现一个简单的井字棋游戏，并且不需要你对 React 有任何了解。在此过程中你会学习到一些编写 React 程序的基本知识，完全理解它们可以让你对 React 有比较深入的理解。

</Intro>

<Note>

This tutorial is designed for people who prefer to **learn by doing** and want to quickly try making something tangible. If you prefer learning each concept step by step, start with [Describing the UI.](/learn/describing-the-ui)
本教程专为喜欢 **理论与实战相结合** 以及希望快速看见成果的人而设计。如果你喜欢逐步学习每个概念，请从 [描述 UI](/learn/describing-the-ui) 开始。

</Note>

The tutorial is divided into several sections:
教程分成以下几个部分：

- [Setup for the tutorial](#setup-for-the-tutorial) will give you **a starting point** to follow the tutorial.
- [配置](#setup-for-the-tutorial) 是一些准备工作。
- [Overview](#overview) will teach you **the fundamentals** of React: components, props, and state.
- [概览](#overview) 介绍了 React 的 **基础知识**：组件、props 和 state。
- [Completing the game](#completing-the-game) will teach you **the most common techniques** in React development.
- [完成游戏](#completing-the-game) 介绍了 React 开发中 **最常用的技术**。
- [Adding time travel](#adding-time-travel) will give you **a deeper insight** into the unique strengths of React.
- [添加“时间旅行”](#adding-time-travel) 可以让你更深入地了解 React 的独特优势。

### 实现的是什么程序？| What are you building? {/*what-are-you-building*/}

In this tutorial, you'll build an interactive tic-tac-toe game with React.
本教程将使用 React 实现一个交互式的井字棋游戏。

You can see what it will look like when you're finished here:
你可以在下面预览最终成果：

<Sandpack>

```js src/App.js
import { useState } from 'react';

function Square({ value, onSquareClick }) {
  return (
    <button className="square" onClick={onSquareClick}>
      {value}
    </button>
  );
}

function Board({ xIsNext, squares, onPlay }) {
  function handleClick(i) {
    if (calculateWinner(squares) || squares[i]) {
      return;
    }
    const nextSquares = squares.slice();
    if (xIsNext) {
      nextSquares[i] = 'X';
    } else {
      nextSquares[i] = 'O';
    }
    onPlay(nextSquares);
  }

  const winner = calculateWinner(squares);
  let status;
  if (winner) {
    status = 'Winner: ' + winner;
  } else {
    status = 'Next player: ' + (xIsNext ? 'X' : 'O');
  }

  return (
    <>
      <div className="status">{status}</div>
      <div className="board-row">
        <Square value={squares[0]} onSquareClick={() => handleClick(0)} />
        <Square value={squares[1]} onSquareClick={() => handleClick(1)} />
        <Square value={squares[2]} onSquareClick={() => handleClick(2)} />
      </div>
      <div className="board-row">
        <Square value={squares[3]} onSquareClick={() => handleClick(3)} />
        <Square value={squares[4]} onSquareClick={() => handleClick(4)} />
        <Square value={squares[5]} onSquareClick={() => handleClick(5)} />
      </div>
      <div className="board-row">
        <Square value={squares[6]} onSquareClick={() => handleClick(6)} />
        <Square value={squares[7]} onSquareClick={() => handleClick(7)} />
        <Square value={squares[8]} onSquareClick={() => handleClick(8)} />
      </div>
    </>
  );
}

export default function Game() {
  const [history, setHistory] = useState([Array(9).fill(null)]);
  const [currentMove, setCurrentMove] = useState(0);
  const xIsNext = currentMove % 2 === 0;
  const currentSquares = history[currentMove];

  function handlePlay(nextSquares) {
    const nextHistory = [...history.slice(0, currentMove + 1), nextSquares];
    setHistory(nextHistory);
    setCurrentMove(nextHistory.length - 1);
  }

  function jumpTo(nextMove) {
    setCurrentMove(nextMove);
  }

  const moves = history.map((squares, move) => {
    let description;
    if (move > 0) {
      description = 'Go to move #' + move;
    } else {
      description = 'Go to game start';
    }
    return (
      <li key={move}>
        <button onClick={() => jumpTo(move)}>{description}</button>
      </li>
    );
  });

  return (
    <div className="game">
      <div className="game-board">
        <Board xIsNext={xIsNext} squares={currentSquares} onPlay={handlePlay} />
      </div>
      <div className="game-info">
        <ol>{moves}</ol>
      </div>
    </div>
  );
}

function calculateWinner(squares) {
  const lines = [
    [0, 1, 2],
    [3, 4, 5],
    [6, 7, 8],
    [0, 3, 6],
    [1, 4, 7],
    [2, 5, 8],
    [0, 4, 8],
    [2, 4, 6],
  ];
  for (let i = 0; i < lines.length; i++) {
    const [a, b, c] = lines[i];
    if (squares[a] && squares[a] === squares[b] && squares[a] === squares[c]) {
      return squares[a];
    }
  }
  return null;
}
```

```css src/styles.css
* {
  box-sizing: border-box;
}

body {
  font-family: sans-serif;
  margin: 20px;
  padding: 0;
}

.square {
  background: #fff;
  border: 1px solid #999;
  float: left;
  font-size: 24px;
  font-weight: bold;
  line-height: 34px;
  height: 34px;
  margin-right: -1px;
  margin-top: -1px;
  padding: 0;
  text-align: center;
  width: 34px;
}

.board-row:after {
  clear: both;
  content: '';
  display: table;
}

.status {
  margin-bottom: 10px;
}
.game {
  display: flex;
  flex-direction: row;
}

.game-info {
  margin-left: 20px;
}
```

</Sandpack>

If the code doesn't make sense to you yet, or if you are unfamiliar with the code's syntax, don't worry! The goal of this tutorial is to help you understand React and its syntax.
如果你还不是很明白上面的代码，不用担心！本教程的目的就是帮你理解 React 及其语法。

We recommend that you check out the tic-tac-toe game above before continuing with the tutorial. One of the features that you'll notice is that there is a numbered list to the right of the game's board. This list gives you a history of all of the moves that have occurred in the game, and it is updated as the game progresses.
我们建议你在继续本教程之前，先看看上面的井字棋游戏。我们会注意到的一项功能是，棋盘右侧有一个编号列表，它记录了游戏中落子的历史，并随着游戏的进行而更新。

Once you've played around with the finished tic-tac-toe game, keep scrolling. You'll start with a simpler template in this tutorial. Our next step is to set you up so that you can start building the game.
体验完游戏以后，继续阅读本教程吧！我们将从一个更简单的模板开始。下一步将介绍相关配置，以便于你着手实现这个游戏。

## 配置 | Setup for the tutorial {/*setup-for-the-tutorial*/}

In the live code editor below, click **Fork** in the top-right corner to open the editor in a new tab using the website CodeSandbox. CodeSandbox lets you write code in your browser and preview how your users will see the app you've created. The new tab should display an empty square and the starter code for this tutorial.
在下面的实时代码编辑器中，单击右上角的 **Fork** 来在新选项卡中打开 CodeSandbox 编辑器。CodeSandbox 让你能够在浏览器中编写代码并预览效果。一切顺利的话，你应该会看见一个空方块和本教程的初始代码。

<Sandpack>

```js src/App.js
export default function Square() {
  return <button className="square">X</button>;
}
```

```css src/styles.css
* {
  box-sizing: border-box;
}

body {
  font-family: sans-serif;
  margin: 20px;
  padding: 0;
}

.square {
  background: #fff;
  border: 1px solid #999;
  float: left;
  font-size: 24px;
  font-weight: bold;
  line-height: 34px;
  height: 34px;
  margin-right: -1px;
  margin-top: -1px;
  padding: 0;
  text-align: center;
  width: 34px;
}

.board-row:after {
  clear: both;
  content: '';
  display: table;
}

.status {
  margin-bottom: 10px;
}
.game {
  display: flex;
  flex-direction: row;
}

.game-info {
  margin-left: 20px;
}
```

</Sandpack>

<Note>

You can also follow this tutorial using your local development environment. To do this, you need to:
如果你想要使用本地开发环境来学习这个教程，需要按照下面的流程进行：

1. Install [Node.js](https://nodejs.org/en/)
- 安装 [Node.js](https://nodejs.org/zh-cn/)
2. In the CodeSandbox tab you opened earlier, press the top-left corner button to open the menu, and then choose **Download Sandbox** in that menu to download an archive of the files locally
- 在之前打开的 CodeSandbox 选项卡中，按左上角的按钮打开菜单，然后选择 **Download Sandbox**，将代码压缩包下载到本地。
3. Unzip the archive, then open a terminal and `cd` to the directory you unzipped
- 将压缩包解压，打开终端并使用 `cd` 命令切换到你解压后的目录。
4. Install the dependencies with `npm install`
- 使用 `npm install` 安装依赖。
5. Run `npm start` to start a local server and follow the prompts to view the code running in a browser
- 运行 `npm start` 启动本地服务器，按照提示在浏览器中查看运行效果。

If you get stuck, don't let this stop you! Follow along online instead and try a local setup again later.

如果你遇到了困难，不要花费时间去找解决方案。请改为在线进行操作，稍后再尝试本地配置。

</Note>

## 概览 | Overview {/*overview*/}

Now that you're set up, let's get an overview of React!
完成配置以后，我们先来大致了解一下 React 吧！

### 看一下刚刚的代码 | Inspecting the starter code {/*inspecting-the-starter-code*/}

In CodeSandbox you'll see three main sections:
在 CodeSandbox 中，你将看到三个主要的部分：

![CodeSandbox with starter code | CodeSandbox 的初始代码](../images/tutorial/react-starter-code-codesandbox.png)

1. The _Files_ section with a list of files like `App.js`, `index.js`, `styles.css` and a folder called `public`
- _Files_ 部分列出了一些文件：`App.js`、`index.js`、`styles.css` 和 `public` 文件夹。
2. The _code editor_ where you'll see the source code of your selected file
- _code editor_ 部分可以看到你所选中文件的源码。
3. The _browser_ section where you'll see how the code you've written will be displayed
- _browser_ 部分可以预览代码的实时结果。

The `App.js` file should be selected in the _Files_ section. The contents of that file in the _code editor_ should be:

`App.js` 文件里面的内容应该是这样的：

```jsx
export default function Square() {
  return <button className="square">X</button>;
}
```

The _browser_ section should be displaying a square with a X in it like this:
_browser_ 部分应该会像下面这样在方块里面显示一个 X：

![x-filled square | 里面是“X”的方块](../images/tutorial/x-filled-square.png)

Now let's have a look at the files in the starter code.
现在，让我们仔细研究一下这些文件吧。

#### `App.js` {/*appjs*/}

The code in `App.js` creates a _component_. In React, a component is a piece of reusable code that represents a part of a user interface. Components are used to render, manage, and update the UI elements in your application. Let's look at the component line by line to see what's going on:
`App.js` 的代码创建了一个 **组件**。在 React 中，组件是一段可重用代码，它通常作为 UI 界面的一部分。组件用于渲染、管理和更新应用中的 UI 元素。让我们逐行查看这段代码，看看发生了什么：

```js {1}
export default function Square() {
  return <button className="square">X</button>;
}
```

The first line defines a function called `Square`. The `export` JavaScript keyword makes this function accessible outside of this file. The `default` keyword tells other files using your code that it's the main function in your file.
第一行定义了一个名为 `Square` 的函数。JavaScript 的 `export` 关键字使此函数可以在此文件之外访问。`default` 关键字表明它是文件中的主要函数。

```js {2}
export default function Square() {
  return <button className="square">X</button>;
}
```

The second line returns a button. The `return` JavaScript keyword means whatever comes after is returned as a value to the caller of the function. `<button>` is a *JSX element*. A JSX element is a combination of JavaScript code and HTML tags that describes what you'd like to display. `className="square"` is a button property or *prop* that tells CSS how to style the button. `X` is the text displayed inside of the button and `</button>` closes the JSX element to indicate that any following content shouldn't be placed inside the button.
第二行返回一个按钮。JavaScript 的 `return` 关键字意味着后面的内容都作为值返回给函数的调用者。`<button>` 是一个 JSX 元素。JSX 元素是 JavaScript 代码和 HTML 标签的组合，用于描述要显示的内容。`className="square"` 是一个 button 属性，它决定 CSS 如何设置按钮的样式。`X` 是按钮内显示的文本，`</button>` 闭合 JSX 元素以表示不应将任何后续内容放置在按钮内。

#### `styles.css` {/*stylescss*/}

Click on the file labeled `styles.css` in the _Files_ section of CodeSandbox. This file defines the styles for your React app. The first two _CSS selectors_ (`*` and `body`) define the style of large parts of your app while the `.square` selector defines the style of any component where the `className` property is set to `square`. In your code, that would match the button from your Square component in the `App.js` file.
单击 CodeSandbox 中的 `styles.css` 文件。该文件定义了 React 应用的样式。前两个 **CSS 选择器**（`*` 和 `body`）定义了应用大部分的样式，而 `.square` 选择器定义了 `className` 属性设置为 `square` 的组件的样式。这与 `App.js` 文件中的 `Square` 组件中的按钮是相匹配的。

#### `index.js` {/*indexjs*/}

Click on the file labeled `index.js` in the _Files_ section of CodeSandbox. You won't be editing this file during the tutorial but it is the bridge between the component you created in the `App.js` file and the web browser.
单击 CodeSandbox 中的 `index.js` 的文件。在本教程中我们不会编辑此文件，但它是 `App.js` 文件中创建的组件与 Web 浏览器之间的桥梁。

```jsx
import { StrictMode } from 'react';
import { createRoot } from 'react-dom/client';
import './styles.css';

import App from './App';
```

Lines 1-5 bring all the necessary pieces together: 
第 1-5 行将所有必要的部分组合在一起：

* React
* React
* React's library to talk to web browsers (React DOM)
* React 与 Web 浏览器对话的库（React DOM）
* the styles for your components
* 组件的样式
* the component you created in `App.js`.
* `App.js` 里面创建的组件

其他文件将它们组合在一起，并将最终成果注入 `public` 文件夹里面的 `index.html` 中。

### 构建棋盘 | Building the board {/*building-the-board*/}

Let's get back to `App.js`. This is where you'll spend the rest of the tutorial.
让我们回到 `App.js`。接下来我们将专注于这个文件。

Currently the board is only a single square, but you need nine! If you just try and copy paste your square to make two squares like this:
目前棋盘只有一个方块，但你需要九个！如果你只是想着复制粘贴来制作两个像这样的方块：

```js {2}
export default function Square() {
  return <button className="square">X</button><button className="square">X</button>;
}
```

You'll get this error:
你将会得到如下错误：

<ConsoleBlock level="error">

/src/App.js: Adjacent JSX elements must be wrapped in an enclosing tag. Did you want a JSX Fragment `<>...</>`?

</ConsoleBlock>

React components need to return a single JSX element and not multiple adjacent JSX elements like two buttons. To fix this you can use *Fragments* (`<>` and `</>`) to wrap multiple adjacent JSX elements like this:
React 组件必须返回单个 JSX 元素，不能像两个按钮那样返回多个相邻的 JSX 元素。要解决此问题，可以使用 Fragment（`<>` 与 `</>`）包裹多个相邻的 JSX 元素，如下所示：

```js {3-6}
export default function Square() {
  return (
    <>
      <button className="square">X</button>
      <button className="square">X</button>
    </>
  );
}
```

Now you should see:
现在你应该可以看见：

![two x-filled squares | 两个“X”方块](../images/tutorial/two-x-filled-squares.png)

great! now you just need to copy-paste a few times to add nine squares and...
非常棒！现在你只需要通过复制粘贴来添加九个方块，然后……

![nine x-filled squares in a line | 9 个在同一行的方块](../images/tutorial/nine-x-filled-squares.png)

Oh no! The squares are all in a single line, not in a grid like you need for our board. To fix this you'll need to group your squares into rows with `div`s and add some CSS classes. While you're at it, you'll give each square a number to make sure you know where each square is displayed.
但事与愿违的是这些方块并没有排列成网格，而是都在一条线上。要解决此问题，需要使用 `div` 将方块分到每一行中并添加一些 CSS 样式。当你这样做的时候，需要给每个方块一个数字，以确保你知道每个方块的位置。

In the `App.js` file, update the `Square` component to look like this:
`App.js` 文件中，`Square` 组件看起来像这样：

```js {3-19}
export default function Square() {
  return (
    <>
      <div className="board-row">
        <button className="square">1</button>
        <button className="square">2</button>
        <button className="square">3</button>
      </div>
      <div className="board-row">
        <button className="square">4</button>
        <button className="square">5</button>
        <button className="square">6</button>
      </div>
      <div className="board-row">
        <button className="square">7</button>
        <button className="square">8</button>
        <button className="square">9</button>
      </div>
    </>
  );
}
```

The CSS defined in `styles.css` styles the divs with the `className` of `board-row`. Now that you've grouped your components into rows with the styled `div`s you have your tic-tac-toe board:
借助 `styles.css` 中定义的 `board-row` 样式，我们将组件分到每一行的 `div` 中。最终完成了井字棋棋盘：

![tic-tac-toe board filled with numbers 1 through 9 | 有着数字 1 到 9 的井字棋棋盘](../images/tutorial/number-filled-board.png)

But you now have a problem. Your component named `Square`, really isn't a square anymore. Let's fix that by changing the name to `Board`:
但是现在有个问题，名为 `Square` 的组件实际上不再是方块了。让我们通过将名称更改为 Board 来解决这个问题：

```js {1}
export default function Board() {
  //...
}
```

At this point your code should look something like this:
此时你的代码应如下所示：

<Sandpack>

```js
export default function Board() {
  return (
    <>
      <div className="board-row">
        <button className="square">1</button>
        <button className="square">2</button>
        <button className="square">3</button>
      </div>
      <div className="board-row">
        <button className="square">4</button>
        <button className="square">5</button>
        <button className="square">6</button>
      </div>
      <div className="board-row">
        <button className="square">7</button>
        <button className="square">8</button>
        <button className="square">9</button>
      </div>
    </>
  );
}
```

```css src/styles.css
* {
  box-sizing: border-box;
}

body {
  font-family: sans-serif;
  margin: 20px;
  padding: 0;
}

.square {
  background: #fff;
  border: 1px solid #999;
  float: left;
  font-size: 24px;
  font-weight: bold;
  line-height: 34px;
  height: 34px;
  margin-right: -1px;
  margin-top: -1px;
  padding: 0;
  text-align: center;
  width: 34px;
}

.board-row:after {
  clear: both;
  content: '';
  display: table;
}

.status {
  margin-bottom: 10px;
}
.game {
  display: flex;
  flex-direction: row;
}

.game-info {
  margin-left: 20px;
}
```

</Sandpack>

<Note>

Psssst... That's a lot to type! It's okay to copy and paste code from this page. However, if you're up for a little challenge, we recommend only copying code that you've manually typed at least once yourself.
嘶……要改的内容也太多了！从该页面复制和粘贴代码是很好的办法。不过如果你愿意挑战一下自己，可以只复制手动输入过的代码。

</Note>

### 通过 props 传递数据 | Passing data through props {/*passing-data-through-propss*/}

Next, you'll want to change the value of a square from empty to "X" when the user clicks on the square. With how you've built the board so far you would need to copy-paste the code that updates the square nine times (once for each square you have)! Instead of copy-pasting, React's component architecture allows you to create a reusable component to avoid messy, duplicated code.
接下来，当用户单击方块时，我们要将方块的值从空更改为“X”。根据目前构建的棋盘，你需要复制并粘贴九次更新方块的代码（每个方块都需要一次）！但是，React 的组件架构可以创建可重用的组件，以避免混乱、重复的代码。

First, you are going to copy the line defining your first square (`<button className="square">1</button>`) from your `Board` component into a new `Square` component:
首先，要将定义第一个方块（`<button className="square">1</button>`）的这行代码从 `Board` 组件复制到新的 `Square` 组件中：

```js {1-3}
function Square() {
  return <button className="square">1</button>;
}

export default function Board() {
  // ...
}
```

Then you'll update the Board component to render that `Square` component using JSX syntax:
然后，更新 Board 组件并使用 JSX 语法渲染 `Square` 组件：

```js {5-19}
// ...
export default function Board() {
  return (
    <>
      <div className="board-row">
        <Square />
        <Square />
        <Square />
      </div>
      <div className="board-row">
        <Square />
        <Square />
        <Square />
      </div>
      <div className="board-row">
        <Square />
        <Square />
        <Square />
      </div>
    </>
  );
}
```

Note how unlike the browser `div`s, your own components `Board` and `Square` must start with a capital letter. 
需要注意的是，这并不像 `div`，这些你自己的组件如 `Board` 和 `Square`，必须以大写字母开头。

让我们来看一看效果：
Let's take a look:

![one-filled board | 都是 1 的方块](../images/tutorial/board-filled-with-ones.png)

Oh no! You lost the numbered squares you had before. Now each square says "1". To fix this, you will use *props* to pass the value each square should have from the parent component (`Board`) to its child (`Square`).
哦不！你失去了你以前有正确编号的方块。现在每个方块都写着“1”。要解决此问题，需要使用 *props* 将每个方块应有的值从父组件（`Board`）传递到其子组件（`Square`）。

Update the `Square` component to read the `value` prop that you'll pass from the `Board`:
更新 `Square` 组件，读取从 `Board` 传递的 `value` props：

```js {1}
function Square({ value }) {
  return <button className="square">1</button>;
}
```

`function Square({ value })` indicates the Square component can be passed a prop called `value`.
`function Square({ value })` 表示可以向 Square 组件传递一个名为 `value` 的 props。

Now you want to display that `value` instead of `1` inside every square. Try doing it like this:
现在你如果想要显示对应的 `value` 而不是 `1`，可以试一下像下面这样：

```js {2}
function Square({ value }) {
  return <button className="square">value</button>;
}
```

Oops, this is not what you wanted:
糟糕！这还不是你想要的：

![value-filled board | 都是“value”的棋盘](../images/tutorial/board-filled-with-value.png)

You wanted to render the JavaScript variable called `value` from your component, not the word "value". To "escape into JavaScript" from JSX, you need curly braces. Add curly braces around `value` in JSX like so:
我们需要从组件中渲染名为 `value` 的 JavaScript 变量，而不是“value”这个词。要从 JSX“转义到 JavaScript”，你需要使用大括号。在 JSX 中的 `value` 周围添加大括号，如下所示：

```js {2}
function Square({ value }) {
  return <button className="square">{value}</button>;
}
```

For now, you should see an empty board:
现在，你应该会看到一个空的棋盘了：

![empty board | 空棋盘](../images/tutorial/empty-board.png)

This is because the `Board` component hasn't passed the `value` prop to each `Square` component it renders yet. To fix it you'll add the `value` prop to each `Square` component rendered by the `Board` component:
这是因为 `Board` 组件还没有将 `value` props 传递给它渲染的每个 `Square` 组件。要修复这个问题，需要向 `Board` 组件里面的每个 `Square` 组件添加 `value` props：

```js {5-7,10-12,15-17}
export default function Board() {
  return (
    <>
      <div className="board-row">
        <Square value="1" />
        <Square value="2" />
        <Square value="3" />
      </div>
      <div className="board-row">
        <Square value="4" />
        <Square value="5" />
        <Square value="6" />
      </div>
      <div className="board-row">
        <Square value="7" />
        <Square value="8" />
        <Square value="9" />
      </div>
    </>
  );
}
```

Now you should see a grid of numbers again:
现在你应该能再次看到数字网格：

![tic-tac-toe board filled with numbers 1 through 9 | 有着数字 1 到 9 的井字棋棋盘](../images/tutorial/number-filled-board.png)

Your updated code should look like this:
更新后的代码应该是这样：

<Sandpack>

```js src/App.js
function Square({ value }) {
  return <button className="square">{value}</button>;
}

export default function Board() {
  return (
    <>
      <div className="board-row">
        <Square value="1" />
        <Square value="2" />
        <Square value="3" />
      </div>
      <div className="board-row">
        <Square value="4" />
        <Square value="5" />
        <Square value="6" />
      </div>
      <div className="board-row">
        <Square value="7" />
        <Square value="8" />
        <Square value="9" />
      </div>
    </>
  );
}
```

```css src/styles.css
* {
  box-sizing: border-box;
}

body {
  font-family: sans-serif;
  margin: 20px;
  padding: 0;
}

.square {
  background: #fff;
  border: 1px solid #999;
  float: left;
  font-size: 24px;
  font-weight: bold;
  line-height: 34px;
  height: 34px;
  margin-right: -1px;
  margin-top: -1px;
  padding: 0;
  text-align: center;
  width: 34px;
}

.board-row:after {
  clear: both;
  content: '';
  display: table;
}

.status {
  margin-bottom: 10px;
}
.game {
  display: flex;
  flex-direction: row;
}

.game-info {
  margin-left: 20px;
}
```

</Sandpack>

### 创建一个具有交互性的组件 | Making an interactive component {/*making-an-interactive-component*/}

Let's fill the `Square` component with an `X` when you click it. Declare a function called `handleClick` inside of the `Square`. Then, add `onClick` to the props of the button JSX element returned from the `Square`:
当你单击它的时候，`Square` 组件需要显示“X”。在 `Square` 内部声明一个名为 `handleClick` 的函数。然后，将 `onClick` 添加到由 `Square` 返回的 JSX 元素的 button 的 props 中：

```js {2-4,9}
function Square({ value }) {
  function handleClick() {
    console.log('clicked!');
  }

  return (
    <button
      className="square"
      onClick={handleClick}
    >
      {value}
    </button>
  );
}
```

If you click on a square now, you should see a log saying `"clicked!"` in the _Console_ tab at the bottom of the _Browser_ section in CodeSandbox. Clicking the square more than once will log `"clicked!"` again. Repeated console logs with the same message will not create more lines in the console. Instead, you will see an incrementing counter next to your first `"clicked!"` log.
如果现在单击一个方块，你应该会看到一条日志，上面写着 `"clicked!"`。在 CodeSandbox 中 *Browser* 部分底部的 *Console* 选项卡中。多次单击方块将再次记录 `"clicked!"`。具有相同消息的重复控制台日志不会在控制台中重复创建。而你会在第一次 `"clicked!"` 旁边看到一个递增的计数器。

<Note>

If you are following this tutorial using your local development environment, you need to open your browser's Console. For example, if you use the Chrome browser, you can view the Console with the keyboard shortcut **Shift + Ctrl + J** (on Windows/Linux) or **Option + ⌘ + J** (on macOS).
如果使用本地开发环境学习本教程，则需要打开浏览器的控制台。例如，如果使用 Chrome 浏览器，则可以使用键盘快捷键 **Shift + Ctrl + J**（在 Windows/Linux 上）或 **Option + ⌘ + J**（在 macOS 上）查看控制台。

</Note>

As a next step, you want the Square component to "remember" that it got clicked, and fill it with an "X" mark. To "remember" things, components use *state*.
下一步，我们希望 Square 组件能够“记住”它被单击过，并用“X”填充它。为了“记住”一些东西，组件使用 *state*。

React provides a special function called `useState` that you can call from your component to let it "remember" things. Let's store the current value of the `Square` in state, and change it when the `Square` is clicked.
React 提供了一个名为 `useState` 的特殊函数，可以从组件中调用它来让它“记住”一些东西。让我们将 `Square` 的当前值存储在 state 中，并在单击 `Square` 时更改它。

Import `useState` at the top of the file. Remove the `value` prop from the `Square` component. Instead, add a new line at the start of the `Square` that calls `useState`. Have it return a state variable called `value`:
在文件的顶部导入 `useState`。从 `Square` 组件中移除 `value` props。在调用 `useState` 的 `Square` 的开头添加一个新行。让它返回一个名为 value 的 state 变量：

```js {1,3,4}
import { useState } from 'react';

function Square() {
  const [value, setValue] = useState(null);

  function handleClick() {
    //...
```

`value` stores the value and `setValue` is a function that can be used to change the value. The `null` passed to `useState` is used as the initial value for this state variable, so `value` here starts off equal to `null`.
`value` 存储值，而 `setValue` 是可用于更改值的函数。传递给 `useState` 的 `null` 用作这个 state 变量的初始值，因此此处 `value` 的值开始时等于 `null`。

Since the `Square` component no longer accepts props anymore, you'll remove the `value` prop from all nine of the Square components created by the Board component:
由于 `Square` 组件不再接受 props，我们从 Board 组件创建的所有九个 Square 组件中删除 `value` props：

```js {6-8,11-13,16-18}
// ...
export default function Board() {
  return (
    <>
      <div className="board-row">
        <Square />
        <Square />
        <Square />
      </div>
      <div className="board-row">
        <Square />
        <Square />
        <Square />
      </div>
      <div className="board-row">
        <Square />
        <Square />
        <Square />
      </div>
    </>
  );
}
```

Now you'll change `Square` to display an "X" when clicked. Replace the `console.log("clicked!");` event handler with `setValue('X');`. Now your `Square` component looks like this:
现在将更改 `Square` 以在单击时显示“X”。不再使用 `console.log("clicked!");` 而使用 `setValue('X');` 的事件处理程序。现在你的 `Square` 组件看起来像这样：

```js {5}
function Square() {
  const [value, setValue] = useState(null);

  function handleClick() {
    setValue('X');
  }

  return (
    <button
      className="square"
      onClick={handleClick}
    >
      {value}
    </button>
  );
}
```

By calling this `set` function from an `onClick` handler, you're telling React to re-render that `Square` whenever its `<button>` is clicked. After the update, the `Square`'s `value` will be `'X'`, so you'll see the "X" on the game board. Click on any Square, and "X" should show up:
通过从 `onClick` 处理程序调用此 `set` 函数，你告诉 `React` 在单击其 `<button>` 时要重新渲染该 `Square`。更新后，方块的值将为“X”，因此会在棋盘上看到“X”。点击任意方块，“X”应该出现：

![adding xes to board | 向棋盘上添加“X”](../images/tutorial/tictac-adding-x-s.gif)

Each Square has its own state: the `value` stored in each Square is completely independent of the others. When you call a `set` function in a component, React automatically updates the child components inside too.
每个 Square 都有自己的 state：存储在每个 Square 中的 `value` 完全独立于其他的 Square。当你在组件中调用 set 函数时，React 也会自动更新内部的子组件。

After you've made the above changes, your code will look like this:
完成上述更改后，代码将如下所示：

<Sandpack>

```js src/App.js
import { useState } from 'react';

function Square() {
  const [value, setValue] = useState(null);

  function handleClick() {
    setValue('X');
  }

  return (
    <button
      className="square"
      onClick={handleClick}
    >
      {value}
    </button>
  );
}

export default function Board() {
  return (
    <>
      <div className="board-row">
        <Square />
        <Square />
        <Square />
      </div>
      <div className="board-row">
        <Square />
        <Square />
        <Square />
      </div>
      <div className="board-row">
        <Square />
        <Square />
        <Square />
      </div>
    </>
  );
}
```

```css src/styles.css
* {
  box-sizing: border-box;
}

body {
  font-family: sans-serif;
  margin: 20px;
  padding: 0;
}

.square {
  background: #fff;
  border: 1px solid #999;
  float: left;
  font-size: 24px;
  font-weight: bold;
  line-height: 34px;
  height: 34px;
  margin-right: -1px;
  margin-top: -1px;
  padding: 0;
  text-align: center;
  width: 34px;
}

.board-row:after {
  clear: both;
  content: '';
  display: table;
}

.status {
  margin-bottom: 10px;
}
.game {
  display: flex;
  flex-direction: row;
}

.game-info {
  margin-left: 20px;
}
```

</Sandpack>

### React 开发者工具 | React Developer Tools {/*react-developer-tools*/}

React DevTools let you check the props and the state of your React components. You can find the React DevTools tab at the bottom of the _browser_ section in CodeSandbox:
React 开发者工具可以检查 React 组件的 props 和 state。可以在 CodeSandbox 的 *Browser* 部分底部找到 React DevTools 选项卡：

![React DevTools in CodeSandbox | CodeSandbox 中的 React 开发者工具](../images/tutorial/codesandbox-devtools.png)

To inspect a particular component on the screen, use the button in the top left corner of React DevTools:
要检查屏幕上的特定组件，请使用 React 开发者工具左上角的按钮：

![Selecting components on the page with React DevTools | 用 React 开发者工具选中组件](../images/tutorial/devtools-select.gif)

<Note>

For local development, React DevTools is available as a [Chrome](https://chrome.google.com/webstore/detail/react-developer-tools/fmkadmapgofadopljbjfkapdkoienihi?hl=en), [Firefox](https://addons.mozilla.org/en-US/firefox/addon/react-devtools/), and [Edge](https://microsoftedge.microsoft.com/addons/detail/react-developer-tools/gpphkfbcpidddadnkolkpfckpihlkkil) browser extension. Install it, and the *Components* tab will appear in your browser Developer Tools for sites using React.
对于本地开发，React 开发工具可作为 [Chrome](https://chrome.google.com/webstore/detail/react-developer-tools/fmkadmapgofadopljbjfkapdkoienihi?hl=en)、[Firefox](https://addons.mozilla.org/zh-CN/firefox/addon/react-devtools/) 和 [Edge](https://microsoftedge.microsoft.com/addons/detail/react-developer-tools/gpphkfbcpidddadnkolkpfckpihlkkil) 的浏览器扩展来使用。安装它，*Component* 选项卡将出现在你的浏览器开发者工具中，将被用于使用 React 的站点。

</Note>

## 完成这个游戏 | Completing the game {/*completing-the-game*/}

By this point, you have all the basic building blocks for your tic-tac-toe game. To have a complete game, you now need to alternate placing "X"s and "O"s on the board, and you need a way to determine a winner.
至此，你已经拥有井字棋游戏的所有基本构建块。要玩完整的游戏，你现在需要在棋盘上交替放置“X”和“O”，并且你需要一种确定获胜者的方法。

### 状态提升 | Lifting state up {/*lifting-state-up*/}

Currently, each `Square` component maintains a part of the game's state. To check for a winner in a tic-tac-toe game, the `Board` would need to somehow know the state of each of the 9 `Square` components.
目前，每个 `Square` 组件都维护着游戏 state 的一部分。要检查井字棋游戏中的赢家，`Board` 需要以某种方式知道 9 个 `Square` 组件中每个组件的 state。

How would you approach that? At first, you might guess that the `Board` needs to "ask" each `Square` for that `Square`'s state. Although this approach is technically possible in React, we discourage it because the code becomes difficult to understand, susceptible to bugs, and hard to refactor. Instead, the best approach is to store the game's state in the parent `Board` component instead of in each `Square`. The `Board` component can tell each `Square` what to display by passing a prop, like you did when you passed a number to each Square.
你会如何处理？起初，你可能会猜测 `Board` 需要向每个 `Square`“询问”`Square` 的 state。尽管这种方法在 React 中在技术上是可行的，但我们不鼓励这样做，因为代码变得难以理解、容易出现错误并且难以重构。相反，最好的方法是将游戏的 state 存储在 `Board` 父组件中，而不是每个 `Square` 中。`Board` 组件可以通过传递一个 props 来告诉每个 `Square` 显示什么，就像你将数字传递给每个 Square 时所做的那样。

**To collect data from multiple children, or to have two child components communicate with each other, declare the shared state in their parent component instead. The parent component can pass that state back down to the children via props. This keeps the child components in sync with each other and with their parent.**
**要从多个子组件收集数据，或让两个子组件相互通信，请改为在其父组件中声明共享 state。父组件可以通过 props 将该 state 传回给子组件。这使子组件彼此同步并与其父组件保持同步。**

Lifting state into a parent component is common when React components are refactored.
重构 React 组件时，将状态提升到父组件中很常见。

Let's take this opportunity to try it out. Edit the `Board` component so that it declares a state variable named `squares` that defaults to an array of 9 nulls corresponding to the 9 squares:
让我们借此机会尝试一下。编辑 `Board` 组件，使其声明一个名为 `squares` 的 state 变量，该变量默认为对应于 9 个方块的 9 个空值数组：

```js {3}
// ...
export default function Board() {
  const [squares, setSquares] = useState(Array(9).fill(null));
  return (
    // ...
  );
}
```

`Array(9).fill(null)` creates an array with nine elements and sets each of them to `null`. The `useState()` call around it declares a `squares` state variable that's initially set to that array. Each entry in the array corresponds to the value of a square. When you fill the board in later, the `squares` array will look like this:
`Array(9).fill(null)` 创建了一个包含九个元素的数组，并将它们中的每一个都设置为 `null`。包裹它的 `useState()` 声明了一个初始设置为该数组的 `squares` state 变量。数组中的每个元素对应于一个 square 的值。当你稍后填写棋盘时，`squares` 数组将如下所示：

```jsx
['O', null, 'X', 'X', 'X', 'O', 'O', null, null]
```

Now your `Board` component needs to pass the `value` prop down to each `Square` that it renders:
现在你的 `Board` 组件需要将 `value` props 向下传递给它渲染的每个 `Square`：

```js {6-8,11-13,16-18}
export default function Board() {
  const [squares, setSquares] = useState(Array(9).fill(null));
  return (
    <>
      <div className="board-row">
        <Square value={squares[0]} />
        <Square value={squares[1]} />
        <Square value={squares[2]} />
      </div>
      <div className="board-row">
        <Square value={squares[3]} />
        <Square value={squares[4]} />
        <Square value={squares[5]} />
      </div>
      <div className="board-row">
        <Square value={squares[6]} />
        <Square value={squares[7]} />
        <Square value={squares[8]} />
      </div>
    </>
  );
}
```

Next, you'll edit the `Square` component to receive the `value` prop from the Board component. This will require removing the Square component's own stateful tracking of `value` and the button's `onClick` prop:
接下来，你将编辑 `Square` 组件，以从 Board 组件接收 `value` props。这将需要删除 Square 组件自己的 `value` state 和按钮的 `onClick` props：

```js {1,2}
function Square({value}) {
  return <button className="square">{value}</button>;
}
```

At this point you should see an empty tic-tac-toe board:
此时你应该看到一个空的井字棋棋盘：

![empty board | 空的井字棋棋盘](../images/tutorial/empty-board.png)

And your code should look like this:
你的代码应该是这样的：

<Sandpack>

```js src/App.js
import { useState } from 'react';

function Square({ value }) {
  return <button className="square">{value}</button>;
}

export default function Board() {
  const [squares, setSquares] = useState(Array(9).fill(null));
  return (
    <>
      <div className="board-row">
        <Square value={squares[0]} />
        <Square value={squares[1]} />
        <Square value={squares[2]} />
      </div>
      <div className="board-row">
        <Square value={squares[3]} />
        <Square value={squares[4]} />
        <Square value={squares[5]} />
      </div>
      <div className="board-row">
        <Square value={squares[6]} />
        <Square value={squares[7]} />
        <Square value={squares[8]} />
      </div>
    </>
  );
}
```

```css src/styles.css
* {
  box-sizing: border-box;
}

body {
  font-family: sans-serif;
  margin: 20px;
  padding: 0;
}

.square {
  background: #fff;
  border: 1px solid #999;
  float: left;
  font-size: 24px;
  font-weight: bold;
  line-height: 34px;
  height: 34px;
  margin-right: -1px;
  margin-top: -1px;
  padding: 0;
  text-align: center;
  width: 34px;
}

.board-row:after {
  clear: both;
  content: '';
  display: table;
}

.status {
  margin-bottom: 10px;
}
.game {
  display: flex;
  flex-direction: row;
}

.game-info {
  margin-left: 20px;
}
```

</Sandpack>

Each Square will now receive a `value` prop that will either be `'X'`, `'O'`, or `null` for empty squares.
现在，每个 Square 都会收到一个 `value` props，对于空方块，该 props 将是 `'X'`、`'O'` 或 `null`。

Next, you need to change what happens when a `Square` is clicked. The `Board` component now maintains which squares are filled. You'll need to create a way for the `Square` to update the `Board`'s state. Since state is private to a component that defines it, you cannot update the `Board`'s state directly from `Square`.
接下来，你需要更改单击 `Square` 时发生的情况。`Board` 组件现在维护已经填充过的方块。你需要为 `Square` 创建一种更新 `Board` state 的方法。由于 state 对于定义它的组件是私有的，因此你不能直接从 `Square` 更新 `Board` 的 state。

Instead, you'll pass down a function from the `Board` component to the `Square` component, and you'll have `Square` call that function when a square is clicked. You'll start with the function that the `Square` component will call when it is clicked. You'll call that function `onSquareClick`:
你将从 `Board` 组件向下传递一个函数到 `Square` 组件，然后让 `Square` 在单击方块时调用该函数。我们将从单击 `Square` 组件时将调用的函数开始。调用该函数 `onSquareClick`：

```js {3}
function Square({ value }) {
  return (
    <button className="square" onClick={onSquareClick}>
      {value}
    </button>
  );
}
```

Next, you'll add the `onSquareClick` function to the `Square` component's props:
接下来，将 `onSquareClick` 函数添加到 `Square` 组件的 props 中：

```js {1}
function Square({ value, onSquareClick }) {
  return (
    <button className="square" onClick={onSquareClick}>
      {value}
    </button>
  );
}
```

Now you'll connect the `onSquareClick` prop to a function in the `Board` component that you'll name `handleClick`. To connect `onSquareClick` to `handleClick` you'll pass a function to the `onSquareClick` prop of the first `Square` component: 
现在，你将把 `onSquareClick` props 连接到 `Board` 组件中的一个函数，命名为 `handleClick`。要将 `onSquareClick` 连接到 `handleClick`，需要将一个函数传递给第一个 `Square` 组件的 `onSquareClick` props：

```js {7}
export default function Board() {
  const [squares, setSquares] = useState(Array(9).fill(null));

  return (
    <>
      <div className="board-row">
        <Square value={squares[0]} onSquareClick={handleClick} />
        //...
  );
}
```

Lastly, you will define the `handleClick` function inside the Board component to update the `squares` array holding your board's state:
最后，你将在 Board 组件内定义 `handleClick` 函数来更新并保存棋盘 state 的 `squares` 数组：

```js {4-8}
export default function Board() {
  const [squares, setSquares] = useState(Array(9).fill(null));

  function handleClick() {
    const nextSquares = squares.slice();
    nextSquares[0] = "X";
    setSquares(nextSquares);
  }

  return (
    // ...
  )
}
```

The `handleClick` function creates a copy of the `squares` array (`nextSquares`) with the JavaScript `slice()` Array method. Then, `handleClick` updates the `nextSquares` array to add `X` to the first (`[0]` index) square.
`handleClick` 函数使用 JavaScript 数组的 `slice()` 方法创建 `squares` 数组（`nextSquares`）的副本。然后，`handleClick` 更新 `nextSquares` 数组，将 `X` 添加到第一个（`[0]` 索引）方块。

Calling the `setSquares` function lets React know the state of the component has changed. This will trigger a re-render of the components that use the `squares` state (`Board`) as well as its child components (the `Square` components that make up the board).
调用 `setSquares` 函数让 React 知道组件的 state 已经改变。这将触发使用 `squares` state 的组件（`Board`）及其子组件（构成棋盘的 `Square` 组件）的重新渲染。

<Note>

JavaScript supports [closures](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Closures) which means an inner function (e.g. `handleClick`) has access to variables and functions defined in an outer function (e.g. `Board`). The `handleClick` function can read the `squares` state and call the `setSquares` method because they are both defined inside of the `Board` function.
JavaScript 支持 [闭包](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Closures)，这意味着内部函数（例如 `handleClick`）可以访问外部函数（例如 `Board`）中定义的变量和函数。`handleClick` 函数可以读取 `squares` state 并调用 `setSquares` 方法，因为它们都是在 `Board` 函数内部定义的。

</Note>

Now you can add X's to the board...  but only to the upper left square. Your `handleClick` function is hardcoded to update the index for the upper left square (`0`). Let's update `handleClick` to be able to update any square. Add an argument `i` to the `handleClick` function that takes the index of the square to update:
现在你可以将 X 添加到棋盘上……但只能添加到左上角的方块。你的 `handleClick` 函数被硬编码为更新左上角方块（ `0`）的索引。让我们更新 `handleClick` 以便能够更新任何方块。将参数 `i` 添加到 `handleClick` 函数，该函数采用要更新的 square 索引：

```js {4,6}
export default function Board() {
  const [squares, setSquares] = useState(Array(9).fill(null));

  function handleClick(i) {
    const nextSquares = squares.slice();
    nextSquares[i] = "X";
    setSquares(nextSquares);
  }

  return (
    // ...
  )
}
```

Next, you will need to pass that `i` to `handleClick`. You could try to set the `onSquareClick` prop of square to be `handleClick(0)` directly in the JSX like this, but it won't work:
接下来，你需要将 `i` 传递给 `handleClick`。你可以尝试像这样在 JSX 中直接将 square 的 `onSquareClick` props 设置为 `handleClick(0)`，但这是行不通的：

```jsx
<Square value={squares[0]} onSquareClick={handleClick(0)} />
```

Here is why this doesn't work. The `handleClick(0)` call will be a part of rendering the board component. Because `handleClick(0)` alters the state of the board component by calling `setSquares`, your entire board component will be re-rendered again. But this runs `handleClick(0)` again, leading to an infinite loop:
为什么会是这样呢？`handleClick(0)` 调用将成为渲染 Board 组件的一部分。因为 `handleClick(0)` 通过调用 `setSquares` 改变了棋盘组件的 state，所以你的整个棋盘组件将再次重新渲染。但这再次运行了 `handleClick(0)`，导致无限循环：

<ConsoleBlock level="error">

Too many re-renders. React limits the number of renders to prevent an infinite loop.

</ConsoleBlock>

Why didn't this problem happen earlier?
为什么这个问题没有早点发生？

When you were passing `onSquareClick={handleClick}`, you were passing the `handleClick` function down as a prop. You were not calling it! But now you are *calling* that function right away--notice the parentheses in `handleClick(0)`--and that's why it runs too early. You don't *want* to call `handleClick` until the user clicks!
当你传递 `onSquareClick={handleClick}` 时，你将 `handleClick` 函数作为 props 向下传递。你不是在调用它！但是现在你立即调用了该函数——注意 `handleClick(0)` 中的括号——这就是它运行得太早的原因。你不想在用户点击之前调用 `handleClick`！

You could fix this by creating a function like `handleFirstSquareClick` that calls `handleClick(0)`, a function like `handleSecondSquareClick` that calls `handleClick(1)`, and so on. You would pass (rather than call) these functions down as props like `onSquareClick={handleFirstSquareClick}`. This would solve the infinite loop.
你可以通过创建调用 `handleClick(0)` 的函数（如 `handleFirstSquareClick`）、调用 `handleClick(1)` 的函数（如 `handleSecondSquareClick`）等来修复。你可以将这些函数作为 `onSquareClick={handleFirstSquareClick}` 之类的 props 传递（而不是调用）。这将解决无限循环的问题。

However, defining nine different functions and giving each of them a name is too verbose. Instead, let's do this:
但是，定义九个不同的函数并为每个函数命名过于冗余。让我们这样做：

```js {6}
export default function Board() {
  // ...
  return (
    <>
      <div className="board-row">
        <Square value={squares[0]} onSquareClick={() => handleClick(0)} />
        // ...
  );
}
```

Notice the new `() =>` syntax. Here, `() => handleClick(0)` is an *arrow function,* which is a shorter way to define functions. When the square is clicked, the code after the `=>` "arrow" will run, calling `handleClick(0)`.
注意新的 `() =>` 语法。这里，`() => handleClick(0)` 是一个箭头函数，它是定义函数的一种较短的方式。单击方块时，`=>`“箭头”之后的代码将运行，调用 `handleClick(0)`。

Now you need to update the other eight squares to call `handleClick` from the arrow functions you pass. Make sure that the argument for each call of the `handleClick` corresponds to the index of the correct square:
现在你需要更新其他八个方块以从你传递的箭头函数中调用 `handleClick`。确保 `handleClick` 的每次调用的参数对应于正确的 square 索引：

```js {6-8,11-13,16-18}
export default function Board() {
  // ...
  return (
    <>
      <div className="board-row">
        <Square value={squares[0]} onSquareClick={() => handleClick(0)} />
        <Square value={squares[1]} onSquareClick={() => handleClick(1)} />
        <Square value={squares[2]} onSquareClick={() => handleClick(2)} />
      </div>
      <div className="board-row">
        <Square value={squares[3]} onSquareClick={() => handleClick(3)} />
        <Square value={squares[4]} onSquareClick={() => handleClick(4)} />
        <Square value={squares[5]} onSquareClick={() => handleClick(5)} />
      </div>
      <div className="board-row">
        <Square value={squares[6]} onSquareClick={() => handleClick(6)} />
        <Square value={squares[7]} onSquareClick={() => handleClick(7)} />
        <Square value={squares[8]} onSquareClick={() => handleClick(8)} />
      </div>
    </>
  );
};
```

Now you can again add X's to any square on the board by clicking on them:
现在你可以再次通过单击将 X 添加到棋盘的方块上：

![filling the board with X | 用 X 填充棋盘](../images/tutorial/tictac-adding-x-s.gif)

But this time all the state management is handled by the `Board` component!
但是这次所有的 state 管理都由 `Board` 组件处理！

This is what your code should look like:
你的代码应该是这样的：

<Sandpack>

```js src/App.js
import { useState } from 'react';

function Square({ value, onSquareClick }) {
  return (
    <button className="square" onClick={onSquareClick}>
      {value}
    </button>
  );
}

export default function Board() {
  const [squares, setSquares] = useState(Array(9).fill(null));

  function handleClick(i) {
    const nextSquares = squares.slice();
    nextSquares[i] = 'X';
    setSquares(nextSquares);
  }

  return (
    <>
      <div className="board-row">
        <Square value={squares[0]} onSquareClick={() => handleClick(0)} />
        <Square value={squares[1]} onSquareClick={() => handleClick(1)} />
        <Square value={squares[2]} onSquareClick={() => handleClick(2)} />
      </div>
      <div className="board-row">
        <Square value={squares[3]} onSquareClick={() => handleClick(3)} />
        <Square value={squares[4]} onSquareClick={() => handleClick(4)} />
        <Square value={squares[5]} onSquareClick={() => handleClick(5)} />
      </div>
      <div className="board-row">
        <Square value={squares[6]} onSquareClick={() => handleClick(6)} />
        <Square value={squares[7]} onSquareClick={() => handleClick(7)} />
        <Square value={squares[8]} onSquareClick={() => handleClick(8)} />
      </div>
    </>
  );
}
```

```css src/styles.css
* {
  box-sizing: border-box;
}

body {
  font-family: sans-serif;
  margin: 20px;
  padding: 0;
}

.square {
  background: #fff;
  border: 1px solid #999;
  float: left;
  font-size: 24px;
  font-weight: bold;
  line-height: 34px;
  height: 34px;
  margin-right: -1px;
  margin-top: -1px;
  padding: 0;
  text-align: center;
  width: 34px;
}

.board-row:after {
  clear: both;
  content: '';
  display: table;
}

.status {
  margin-bottom: 10px;
}
.game {
  display: flex;
  flex-direction: row;
}

.game-info {
  margin-left: 20px;
}
```

</Sandpack>

Now that your state handling is in the `Board` component, the parent `Board` component passes props to the child `Square` components so that they can be displayed correctly. When clicking on a `Square`, the child `Square` component now asks the parent `Board` component to update the state of the board. When the `Board`'s state changes, both the `Board` component and every child `Square` re-renders automatically. Keeping the state of all squares in the `Board` component will allow it to determine the winner in the future.
现在，我们在 `Board` 组件中处理 state， `Board` 父组件将 props 传递给 `Square` 子组件，以便它们可以正确显示。单击 `Square` 时， `Square` 子组件现在要求 `Board` 父组件更新棋盘的 state。当 `Board` 的 state 改变时，`Board` 组件和每个子 `Square` 都会自动重新渲染。保存 `Board` 组件中所有方块的 state 将使得它可以确定未来的赢家。

Let's recap what happens when a user clicks the top left square on your board to add an `X` to it:
让我们回顾一下当用户单击你的棋盘左上角的方块以向其添加 `X` 时会发生什么：

1. Clicking on the upper left square runs the function that the `button` received as its `onClick` prop from the `Square`. The `Square` component received that function as its `onSquareClick` prop from the `Board`. The `Board` component defined that function directly in the JSX. It calls `handleClick` with an argument of `0`.
- 单击左上角的方块运行 `button` 从 `Square` 接收到的 `onClick` props 的函数。`Square` 组件从 `Board` 通过 `onSquareClick` props 接收到该函数。`Board` 组件直接在 JSX 中定义了该函数。它使用参数 `0` 调用 `handleClick`。
2. `handleClick` uses the argument (`0`) to update the first element of the `squares` array from `null` to `X`.
- `handleClick` 使用参数（`0`）将 `squares` 数组的第一个元素从 `null` 更新为 `X`。
3. The `squares` state of the `Board` component was updated, so the `Board` and all of its children re-render. This causes the `value` prop of the `Square` component with index `0` to change from `null` to `X`.
- `Board` 组件的 `squares` state 已更新，因此 `Board` 及其所有子组件都将重新渲染。这会导致索引为 `0` 的 `Square` 组件的 `value` props 从 `null` 更改为 `X`。

In the end the user sees that the upper left square has changed from empty to having a `X` after clicking it.
最后，用户看到左上角的方块在单击后从空变为 `X`。

<Note>

The DOM `<button>` element's `onClick` attribute has a special meaning to React because it is a built-in component. For custom components like Square, the naming is up to you. You could give any name to the `Square`'s `onSquareClick` prop or `Board`'s `handleClick` function, and the code would work the same. In React, it's conventional to use `onSomething` names for props which represent events and `handleSomething` for the function definitions which handle those events.

DOM `<button>` 元素的 `onClick` props 对 React 有特殊意义，因为它是一个内置组件。对于像 Square 这样的自定义组件，命名由你决定。你可以给 `Square` 的 `onSquareClick` props 或 `Board` 的 `handleClick` 函数起任何名字，代码还是可以运行的。在 React 中，通常使用 `onSomething` 命名代表事件的 props，使用 `handleSomething` 命名处理这些事件的函数。

</Note>

### 为什么不变性很重要 | Why immutability is important {/*why-immutability-is-important*/}

Note how in `handleClick`, you call `.slice()` to create a copy of the `squares` array instead of modifying the existing array. To explain why, we need to discuss immutability and why immutability is important to learn.
请注意在 `handleClick` 中，你调用了 `.slice()` 来创建 `squares` 数组的副本而不是修改现有数组。为了解释原因，我们需要讨论不变性以及为什么学习不变性很重要。

There are generally two approaches to changing data. The first approach is to _mutate_ the data by directly changing the data's values. The second approach is to replace the data with a new copy which has the desired changes. Here is what it would look like if you mutated the `squares` array:
通常有两种更改数据的方法。第一种方法是通过直接更改数据的值来改变数据。第二种方法是使用具有所需变化的新副本替换数据。如果你改变 `squares` 数组，它会是这样的：

```jsx
const squares = [null, null, null, null, null, null, null, null, null];
squares[0] = 'X';
// Now `squares` is ["X", null, null, null, null, null, null, null, null];
```

And here is what it would look like if you changed data without mutating the `squares` array:
如果你在不改变 `squares` 数组的情况下更改数据，它会是这样的：

```jsx
const squares = [null, null, null, null, null, null, null, null, null];
const nextSquares = ['X', null, null, null, null, null, null, null, null];
// Now `squares` is unchanged, but `nextSquares` first element is 'X' rather than `null`
```

The result is the same but by not mutating (changing the underlying data) directly, you gain several benefits.
结果是一样的，但通过不直接改变（改变底层数据），你可以获得几个好处。

Immutability makes complex features much easier to implement. Later in this tutorial, you will implement a "time travel" feature that lets you review the game's history and "jump back" to past moves. This functionality isn't specific to games--an ability to undo and redo certain actions is a common requirement for apps. Avoiding direct data mutation lets you keep previous versions of the data intact, and reuse them later.
不变性使复杂的功能更容易实现。在本教程的后面，你将实现一个“时间旅行”功能，让你回顾游戏的历史并“跳回”到过去的动作。此功能并非特定于游戏——撤消和重做某些操作的能力是应用程序的常见要求。避免数据直接突变可以让你保持以前版本的数据完好无损，并在以后重用它们。

There is also another benefit of immutability. By default, all child components re-render automatically when the state of a parent component changes. This includes even the child components that weren't affected by the change. Although re-rendering is not by itself noticeable to the user (you shouldn't actively try to avoid it!), you might want to skip re-rendering a part of the tree that clearly wasn't affected by it for performance reasons. Immutability makes it very cheap for components to compare whether their data has changed or not. You can learn more about how React chooses when to re-render a component in [the `memo` API reference](/reference/react/memo).
不变性还有另一个好处。默认情况下，当父组件的 state 发生变化时，所有子组件都会自动重新渲染。这甚至包括未受变化影响的子组件。尽管重新渲染本身不会引起用户注意（你不应该主动尝试避免它！），但出于性能原因，你可能希望跳过重新渲染显然不受其影响的树的一部分。不变性使得组件比较其数据是否已更改的成本非常低。你可以在 [`memo` API 参考](/reference/react/memo) 中了解更多关于 React 如何选择何时重新渲染组件的信息。

### 交替落子 | Taking turns {/*taking-turns*/}

It's now time to fix a major defect in this tic-tac-toe game: the "O"s cannot be marked on the board.
现在是时候修复这个井字棋游戏的一个主要缺陷了：棋盘上无法标记“O”。

You'll set the first move to be "X" by default. Let's keep track of this by adding another piece of state to the Board component:
默认情况下，你会将第一步设置为“X”。让我们通过向 Board 组件添加另一个 state 来跟踪这一点：

```js {2}
function Board() {
  const [xIsNext, setXIsNext] = useState(true);
  const [squares, setSquares] = useState(Array(9).fill(null));

  // ...
}
```

Each time a player moves, `xIsNext` (a boolean) will be flipped to determine which player goes next and the game's state will be saved. You'll update the `Board`'s `handleClick` function to flip the value of `xIsNext`:
每次玩家落子时，`xIsNext`（一个布尔值）将被翻转以确定下一个玩家，游戏 state 将被保存。你将更新 `Board` 的 `handleClick` 函数以翻转 `xIsNext` 的值：

```js {7,8,9,10,11,13}
export default function Board() {
  const [xIsNext, setXIsNext] = useState(true);
  const [squares, setSquares] = useState(Array(9).fill(null));

  function handleClick(i) {
    const nextSquares = squares.slice();
    if (xIsNext) {
      nextSquares[i] = "X";
    } else {
      nextSquares[i] = "O";
    }
    setSquares(nextSquares);
    setXIsNext(!xIsNext);
  }

  return (
    //...
  );
}
```

Now, as you click on different squares, they will alternate between `X` and `O`, as they should!
现在，当你点击不同的方块时，它们会在 `X` 和 `O` 之间交替，这是它们应该做的！

But wait, there's a problem. Try clicking on the same square multiple times:
但是等等，有一个问题。尝试多次点击同一个方块：

![O overwriting an X | O 覆盖了 X](../images/tutorial/o-replaces-x.gif)

The `X` is overwritten by an `O`! While this would add a very interesting twist to the game, we're going to stick to the original rules for now.

 `X` 被 `O` 覆盖！虽然这会给游戏带来非常有趣的变化，但我们现在将坚持原来的规则。

When you mark a square with a `X` or an `O` you aren't first checking to see if the square already has a `X` or `O` value. You can fix this by *returning early*. You'll check to see if the square already has a `X` or an `O`. If the square is already filled, you will `return` in the `handleClick` function early--before it tries to update the board state.
当你用 `X` 或 `O` 标记方块时，你没有检查该方块是否已经具有 `X` 或 `O` 值。你可以通过提早返回来解决此问题。我们将检查方块是否已经有 `X` 或 `O`。如果方块已经填满，你将尽早在 `handleClick` 函数中 `return`——在它尝试更新棋盘 state 之前。

```js {2,3,4}
function handleClick(i) {
  if (squares[i]) {
    return;
  }
  const nextSquares = squares.slice();
  //...
}
```

Now you can only add `X`'s or `O`'s to empty squares! Here is what your code should look like at this point:
现在你只能将 `X` 或 `O` 添加到空方块中！此时你的代码应该如下所示：

<Sandpack>

```js src/App.js
import { useState } from 'react';

function Square({value, onSquareClick}) {
  return (
    <button className="square" onClick={onSquareClick}>
      {value}
    </button>
  );
}

export default function Board() {
  const [xIsNext, setXIsNext] = useState(true);
  const [squares, setSquares] = useState(Array(9).fill(null));

  function handleClick(i) {
    if (squares[i]) {
      return;
    }
    const nextSquares = squares.slice();
    if (xIsNext) {
      nextSquares[i] = 'X';
    } else {
      nextSquares[i] = 'O';
    }
    setSquares(nextSquares);
    setXIsNext(!xIsNext);
  }

  return (
    <>
      <div className="board-row">
        <Square value={squares[0]} onSquareClick={() => handleClick(0)} />
        <Square value={squares[1]} onSquareClick={() => handleClick(1)} />
        <Square value={squares[2]} onSquareClick={() => handleClick(2)} />
      </div>
      <div className="board-row">
        <Square value={squares[3]} onSquareClick={() => handleClick(3)} />
        <Square value={squares[4]} onSquareClick={() => handleClick(4)} />
        <Square value={squares[5]} onSquareClick={() => handleClick(5)} />
      </div>
      <div className="board-row">
        <Square value={squares[6]} onSquareClick={() => handleClick(6)} />
        <Square value={squares[7]} onSquareClick={() => handleClick(7)} />
        <Square value={squares[8]} onSquareClick={() => handleClick(8)} />
      </div>
    </>
  );
}
```

```css src/styles.css
* {
  box-sizing: border-box;
}

body {
  font-family: sans-serif;
  margin: 20px;
  padding: 0;
}

.square {
  background: #fff;
  border: 1px solid #999;
  float: left;
  font-size: 24px;
  font-weight: bold;
  line-height: 34px;
  height: 34px;
  margin-right: -1px;
  margin-top: -1px;
  padding: 0;
  text-align: center;
  width: 34px;
}

.board-row:after {
  clear: both;
  content: '';
  display: table;
}

.status {
  margin-bottom: 10px;
}
.game {
  display: flex;
  flex-direction: row;
}

.game-info {
  margin-left: 20px;
}
```

</Sandpack>

### 宣布获胜者 | Declaring a winner {/*declaring-a-winner*/}

Now that the players can take turns, you'll want to show when the game is won and there are no more turns to make. To do this you'll add a helper function called `calculateWinner` that takes an array of 9 squares, checks for a winner and returns `'X'`, `'O'`, or `null` as appropriate. Don't worry too much about the `calculateWinner` function; it's not specific to React:
现在你可以轮流对战了，接下来我们应该显示游戏何时获胜。为此，你将添加一个名为 `calculateWinner` 的辅助函数，它接受 9 个方块的数组，检查获胜者并根据需要返回 `'X'`、`'O'` 或 `null`。不要太担心 `calculateWinner` 函数；它不是 React 才会有的：

```js src/App.js
export default function Board() {
  //...
}

function calculateWinner(squares) {
  const lines = [
    [0, 1, 2],
    [3, 4, 5],
    [6, 7, 8],
    [0, 3, 6],
    [1, 4, 7],
    [2, 5, 8],
    [0, 4, 8],
    [2, 4, 6]
  ];
  for (let i = 0; i < lines.length; i++) {
    const [a, b, c] = lines[i];
    if (squares[a] && squares[a] === squares[b] && squares[a] === squares[c]) {
      return squares[a];
    }
  }
  return null;
}
```

<Note>

It does not matter whether you define `calculateWinner` before or after the `Board`. Let's put it at the end so that you don't have to scroll past it every time you edit your components.
在 `Board` 之前还是之后定义 `calculateWinner` 并不重要。让我们把它放在最后，这样你就不必在每次编辑组件时都滚动过去。

</Note>

You will call `calculateWinner(squares)` in the `Board` component's `handleClick` function to check if a player has won. You can perform this check at the same time you check if a user has clicked a square that already has a `X` or and `O`. We'd like to return early in both cases:
你将在 `Board` 组件的 `handleClick` 函数中调用 `calculateWinner(squares)` 来检查玩家是否获胜。你可以在检查用户是否单击了已经具有 `X` 或 `O` 的方块的同时执行此检查。在这两种情况下，我们都希望尽早返回：

```js {2}
function handleClick(i) {
  if (squares[i] || calculateWinner(squares)) {
    return;
  }
  const nextSquares = squares.slice();
  //...
}
```

To let the players know when the game is over, you can display text such as "Winner: X" or "Winner: O". To do that you'll add a `status` section to the `Board` component. The status will display the winner if the game is over and if the game is ongoing you'll display which player's turn is next:
为了让玩家知道游戏何时结束，你可以显示“获胜者：X”或“获胜者：O”等文字。为此，你需要将 `status` 部分添加到 `Board` 组件。如果游戏结束，将显示获胜者，如果游戏正在进行，你将显示下一轮将会是哪个玩家：

```js {3-9,13}
export default function Board() {
  // ...
  const winner = calculateWinner(squares);
  let status;
  if (winner) {
    status = "Winner: " + winner;
  } else {
    status = "Next player: " + (xIsNext ? "X" : "O");
  }

  return (
    <>
      <div className="status">{status}</div>
      <div className="board-row">
        // ...
  )
}
```

Congratulations! You now have a working tic-tac-toe game. And you've just learned the basics of React too. So _you_ are the real winner here. Here is what the code should look like:
恭喜！你现在有一个可以运行的井字棋游戏。你也学习了 React 的基础知识。所以你是这里真正的赢家。代码应该如下所示：

<Sandpack>

```js src/App.js
import { useState } from 'react';

function Square({value, onSquareClick}) {
  return (
    <button className="square" onClick={onSquareClick}>
      {value}
    </button>
  );
}

export default function Board() {
  const [xIsNext, setXIsNext] = useState(true);
  const [squares, setSquares] = useState(Array(9).fill(null));

  function handleClick(i) {
    if (calculateWinner(squares) || squares[i]) {
      return;
    }
    const nextSquares = squares.slice();
    if (xIsNext) {
      nextSquares[i] = 'X';
    } else {
      nextSquares[i] = 'O';
    }
    setSquares(nextSquares);
    setXIsNext(!xIsNext);
  }

  const winner = calculateWinner(squares);
  let status;
  if (winner) {
    status = 'Winner: ' + winner;
  } else {
    status = 'Next player: ' + (xIsNext ? 'X' : 'O');
  }

  return (
    <>
      <div className="status">{status}</div>
      <div className="board-row">
        <Square value={squares[0]} onSquareClick={() => handleClick(0)} />
        <Square value={squares[1]} onSquareClick={() => handleClick(1)} />
        <Square value={squares[2]} onSquareClick={() => handleClick(2)} />
      </div>
      <div className="board-row">
        <Square value={squares[3]} onSquareClick={() => handleClick(3)} />
        <Square value={squares[4]} onSquareClick={() => handleClick(4)} />
        <Square value={squares[5]} onSquareClick={() => handleClick(5)} />
      </div>
      <div className="board-row">
        <Square value={squares[6]} onSquareClick={() => handleClick(6)} />
        <Square value={squares[7]} onSquareClick={() => handleClick(7)} />
        <Square value={squares[8]} onSquareClick={() => handleClick(8)} />
      </div>
    </>
  );
}

function calculateWinner(squares) {
  const lines = [
    [0, 1, 2],
    [3, 4, 5],
    [6, 7, 8],
    [0, 3, 6],
    [1, 4, 7],
    [2, 5, 8],
    [0, 4, 8],
    [2, 4, 6],
  ];
  for (let i = 0; i < lines.length; i++) {
    const [a, b, c] = lines[i];
    if (squares[a] && squares[a] === squares[b] && squares[a] === squares[c]) {
      return squares[a];
    }
  }
  return null;
}
```

```css src/styles.css
* {
  box-sizing: border-box;
}

body {
  font-family: sans-serif;
  margin: 20px;
  padding: 0;
}

.square {
  background: #fff;
  border: 1px solid #999;
  float: left;
  font-size: 24px;
  font-weight: bold;
  line-height: 34px;
  height: 34px;
  margin-right: -1px;
  margin-top: -1px;
  padding: 0;
  text-align: center;
  width: 34px;
}

.board-row:after {
  clear: both;
  content: '';
  display: table;
}

.status {
  margin-bottom: 10px;
}
.game {
  display: flex;
  flex-direction: row;
}

.game-info {
  margin-left: 20px;
}
```

</Sandpack>

##  添加时间旅行 | Adding time travel {/*adding-time-travel*/}

As a final exercise, let's make it possible to "go back in time" to the previous moves in the game.
作为最后的练习，让我们能够“回到”到游戏中之前的动作。

### 存储落子历史 | Storing a history of moves {/*storing-a-history-of-moves*/}

If you mutated the `squares` array, implementing time travel would be very difficult.
如果你改变了 `squares` 数组，实现时间旅行将非常困难。

However, you used `slice()` to create a new copy of the `squares` array after every move, and treated it as immutable. This will allow you to store every past version of the `squares` array, and navigate between the turns that have already happened.
但是，你在每次落子后都使用 `slice()` 创建 `squares` 数组的新副本，并将其视为不可变的。这将允许你存储 `squares` 数组的每个过去的版本，并在已经发生的轮次之间“来回”。

You'll store the past `squares` arrays in another array called `history`, which you'll store as a new state variable. The `history` array represents all board states, from the first to the last move, and has a shape like this:
把过去的 `squares` 数组存储在另一个名为 `history` 的数组中，把它存储为一个新的 state 变量。`history` 数组表示所有棋盘的 state，从第一步到最后一步，其形状如下：

```jsx
[
  // Before first move
  [null, null, null, null, null, null, null, null, null],
  // After first move
  [null, null, null, null, 'X', null, null, null, null],
  // After second move
  [null, null, null, null, 'X', null, null, null, 'O'],
  // ...
]
```

### 再一次“状态提升” | Lifting state up, again {/*lifting-state-up-again*/}

You will now write a new top-level component called `Game` to display a list of past moves. That's where you will place the `history` state that contains the entire game history.
你现在将编写一个名为 `Game` 的新顶级组件来显示过去的着法列表。这就是放置包含整个游戏历史的 `history` state 的地方。

Placing the `history` state into the `Game` component will let you remove the `squares` state from its child `Board` component. Just like you "lifted state up" from the `Square` component into the `Board` component, you will now lift it up from the `Board` into the top-level `Game` component. This gives the `Game` component full control over the `Board`'s data and lets it instruct the `Board` to render previous turns from the `history`.
将 `history` state 放入 `Game` 组件将使你可以从其 `Board` 子组件中删除 `squares` state。就像你将 state 从 `Square` 组件“提升”到 `Board` 组件一样，你现在将把它从 `Board` 提升到顶层 `Game` 组件。这使 `Game` 组件可以完全控制 `Board` 的数据，并使它让 `Board` 渲染来自 `history` 的之前的回合。

First, add a `Game` component with `export default`. Have it render the `Board` component and some markup:
首先，添加一个带有 `export default` 的 `Game` 组件。让它渲染 `Board` 组件和一些标签：

```js {1,5-16}
function Board() {
  // ...
}

export default function Game() {
  return (
    <div className="game">
      <div className="game-board">
        <Board />
      </div>
      <div className="game-info">
        <ol>{/*TODO*/}</ol>
      </div>
    </div>
  );
}
```

Note that you are removing the `export default` keywords before the `function Board() {` declaration and adding them before the `function Game() {` declaration. This tells your `index.js` file to use the `Game` component as the top-level component instead of your `Board` component. The additional `div`s returned by the `Game` component are making room for the game information you'll add to the board later.
请注意，你要删除 `function Board() {` 声明之前的 `export default` 关键字，并将它们添加到 `function Game() {` 声明之前。这会告诉你的 `index.js` 文件使用 `Game` 组件而不是你的 `Board` 组件作为顶层组件。`Game` 组件返回的额外 `div` 正在为你稍后添加到棋盘的游戏信息腾出空间。

Add some state to the `Game` component to track which player is next and the history of moves:
向 `Game` 组件添加一些 state 以跟踪下一个玩家和落子历史：

```js {2-3}
export default function Game() {
  const [xIsNext, setXIsNext] = useState(true);
  const [history, setHistory] = useState([Array(9).fill(null)]);
  // ...
```

Notice how `[Array(9).fill(null)]` is an array with a single item, which itself is an array of 9 `null`s.
请注意，`[Array(9).fill(null)]` 是一个包含单个元素的数组，它本身是一个包含 9 个 `null` 的数组。

To render the squares for the current move, you'll want to read the last squares array from the `history`. You don't need `useState` for this--you already have enough information to calculate it during rendering:
要渲染当前落子的方块，你需要从 `history` 中读取最后一个 squares 数组。你不需要 `useState`——你已经有足够的信息可以在渲染过程中计算它：

```js {4}
export default function Game() {
  const [xIsNext, setXIsNext] = useState(true);
  const [history, setHistory] = useState([Array(9).fill(null)]);
  const currentSquares = history[history.length - 1];
  // ...
```

Next, create a `handlePlay` function inside the `Game` component that will be called by the `Board` component to update the game. Pass `xIsNext`, `currentSquares` and `handlePlay` as props to the `Board` component:
接下来，在 `Game` 组件中创建一个 `handlePlay` 函数，`Board` 组件将调用该函数来更新游戏。将 `xIsNext`、`currentSquares` 和 `handlePlay` 作为 props 传递给 `Board` 组件：

```js {6-8,13}
export default function Game() {
  const [xIsNext, setXIsNext] = useState(true);
  const [history, setHistory] = useState([Array(9).fill(null)]);
  const currentSquares = history[history.length - 1];

  function handlePlay(nextSquares) {
    // TODO
  }

  return (
    <div className="game">
      <div className="game-board">
        <Board xIsNext={xIsNext} squares={currentSquares} onPlay={handlePlay} />
        //...
  )
}
```

Let's make the `Board` component fully controlled by the props it receives. Change the `Board` component to take three props: `xIsNext`, `squares`, and a new `onPlay` function that `Board` can call with the updated squares array when a player makes a move. Next, remove the first two lines of the `Board` function that call `useState`:
让 `Board` 组件完全由它接收到的 props 控制。更改 `Board` 组件以采用三个 props：`xIsNext`、`squares`和一个新的 `onPlay` 函数，当玩家落子时，`Board` 可以使用更新的 square 数组调用该函数。接下来，删除调用 `useState` 的 `Board` 函数的前两行：

```js {1}
function Board({ xIsNext, squares, onPlay }) {
  function handleClick(i) {
    //...
  }
  // ...
}
```

Now replace the `setSquares` and `setXIsNext` calls in `handleClick` in the `Board` component with a single call to your new `onPlay` function so the `Game` component can update the `Board` when the user clicks a square:
现在，将 `Board` 组件里面的 `handleClick` 中的 `setSquares` 和 `setXIsNext` 调用替换为对新 `onPlay` 函数的一次调用，这样 `Game` 组件就可以在用户单击方块时更新 `Board`：

```js {12}
function Board({ xIsNext, squares, onPlay }) {
  function handleClick(i) {
    if (calculateWinner(squares) || squares[i]) {
      return;
    }
    const nextSquares = squares.slice();
    if (xIsNext) {
      nextSquares[i] = "X";
    } else {
      nextSquares[i] = "O";
    }
    onPlay(nextSquares);
  }
  //...
}
```

The `Board` component is fully controlled by the props passed to it by the `Game` component. You need to implement the `handlePlay` function in the `Game` component to get the game working again.
`Board` 组件完全由 `Game` 组件传递给它的 props 控制。你需要在 `Game` 组件中实现 `handlePlay` 函数才能使游戏重新运行。

What should `handlePlay` do when called? Remember that Board used to call `setSquares` with an updated array; now it passes the updated `squares` array to `onPlay`.
`handlePlay` 被调用应该做什么？请记住，Board 以前使用更新后的数组调用 `setSquares`；现在它将更新后的 `squares` 数组传递给 `onPlay`。

The `handlePlay` function needs to update `Game`'s state to trigger a re-render, but you don't have a `setSquares` function that you can call any more--you're now using the `history` state variable to store this information. You'll want to update `history` by appending the updated `squares` array as a new history entry. You also want to toggle `xIsNext`, just as Board used to do:
`handlePlay` 函数需要更新 `Game` 的 state 以触发重新渲染，但是你没有可以再调用的 `setSquares` 函数——你现在正在使用 `history` state 变量来存储这些信息。你需要追加更新后的 `squares` 数组来更新 `history` 作为新的历史入口。你还需要切换 `xIsNext`，就像 Board 过去所做的那样：

```js {4-5}
export default function Game() {
  //...
  function handlePlay(nextSquares) {
    setHistory([...history, nextSquares]);
    setXIsNext(!xIsNext);
  }
  //...
}
```

Here, `[...history, nextSquares]` creates a new array that contains all the items in `history`, followed by `nextSquares`. (You can read the `...history` [*spread syntax*](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Spread_syntax) as "enumerate all the items in `history`".)
在这里，`[...history, nextSquares]` 创建了一个新数组，其中包含 `history` 中的所有元素，后跟 `nextSquares`。（你可以将 `...history` 展开语法理解为“枚举 `history` 中的所有元素”。）

For example, if `history` is `[[null,null,null], ["X",null,null]]` and `nextSquares` is `["X",null,"O"]`, then the new `[...history, nextSquares]` array will be `[[null,null,null], ["X",null,null], ["X",null,"O"]]`.
例如，如果 `history` 是 `[[null,null,null], ["X",null,null]]`，`nextSquares` 是 `["X",null,"O"]`，那么新的 `[...history, nextSquares]` 数组就是 `[[null,null,null], ["X",null,null], ["X",null,"O"]]`。

At this point, you've moved the state to live in the `Game` component, and the UI should be fully working, just as it was before the refactor. Here is what the code should look like at this point:
此时，你已将 state 移至 `Game` 组件中，并且 UI 应该完全正常工作，就像重构之前一样。这是此时代码的样子：

<Sandpack>

```js src/App.js
import { useState } from 'react';

function Square({ value, onSquareClick }) {
  return (
    <button className="square" onClick={onSquareClick}>
      {value}
    </button>
  );
}

function Board({ xIsNext, squares, onPlay }) {
  function handleClick(i) {
    if (calculateWinner(squares) || squares[i]) {
      return;
    }
    const nextSquares = squares.slice();
    if (xIsNext) {
      nextSquares[i] = 'X';
    } else {
      nextSquares[i] = 'O';
    }
    onPlay(nextSquares);
  }

  const winner = calculateWinner(squares);
  let status;
  if (winner) {
    status = 'Winner: ' + winner;
  } else {
    status = 'Next player: ' + (xIsNext ? 'X' : 'O');
  }

  return (
    <>
      <div className="status">{status}</div>
      <div className="board-row">
        <Square value={squares[0]} onSquareClick={() => handleClick(0)} />
        <Square value={squares[1]} onSquareClick={() => handleClick(1)} />
        <Square value={squares[2]} onSquareClick={() => handleClick(2)} />
      </div>
      <div className="board-row">
        <Square value={squares[3]} onSquareClick={() => handleClick(3)} />
        <Square value={squares[4]} onSquareClick={() => handleClick(4)} />
        <Square value={squares[5]} onSquareClick={() => handleClick(5)} />
      </div>
      <div className="board-row">
        <Square value={squares[6]} onSquareClick={() => handleClick(6)} />
        <Square value={squares[7]} onSquareClick={() => handleClick(7)} />
        <Square value={squares[8]} onSquareClick={() => handleClick(8)} />
      </div>
    </>
  );
}

export default function Game() {
  const [xIsNext, setXIsNext] = useState(true);
  const [history, setHistory] = useState([Array(9).fill(null)]);
  const currentSquares = history[history.length - 1];

  function handlePlay(nextSquares) {
    setHistory([...history, nextSquares]);
    setXIsNext(!xIsNext);
  }

  return (
    <div className="game">
      <div className="game-board">
        <Board xIsNext={xIsNext} squares={currentSquares} onPlay={handlePlay} />
      </div>
      <div className="game-info">
        <ol>{/*TODO*/}</ol>
      </div>
    </div>
  );
}

function calculateWinner(squares) {
  const lines = [
    [0, 1, 2],
    [3, 4, 5],
    [6, 7, 8],
    [0, 3, 6],
    [1, 4, 7],
    [2, 5, 8],
    [0, 4, 8],
    [2, 4, 6],
  ];
  for (let i = 0; i < lines.length; i++) {
    const [a, b, c] = lines[i];
    if (squares[a] && squares[a] === squares[b] && squares[a] === squares[c]) {
      return squares[a];
    }
  }
  return null;
}
```

```css src/styles.css
* {
  box-sizing: border-box;
}

body {
  font-family: sans-serif;
  margin: 20px;
  padding: 0;
}

.square {
  background: #fff;
  border: 1px solid #999;
  float: left;
  font-size: 24px;
  font-weight: bold;
  line-height: 34px;
  height: 34px;
  margin-right: -1px;
  margin-top: -1px;
  padding: 0;
  text-align: center;
  width: 34px;
}

.board-row:after {
  clear: both;
  content: '';
  display: table;
}

.status {
  margin-bottom: 10px;
}
.game {
  display: flex;
  flex-direction: row;
}

.game-info {
  margin-left: 20px;
}
```

</Sandpack>

### 显示过去的落子 | Showing the past moves {/*showing-the-past-moves*/}

Since you are recording the tic-tac-toe game's history, you can now display a list of past moves to the player.
由于你正在记录井字棋游戏的历史，因此你现在可以向玩家显示过去的动作列表。

React elements like `<button>` are regular JavaScript objects; you can pass them around in your application. To render multiple items in React, you can use an array of React elements.
像 `<button>` 这样的 React 元素是常规的 JavaScript 对象；你可以在你的应用程序中传递它们。要在 React 中渲染多个项目，你可以使用 React 元素数组。

You already have an array of `history` moves in state, so now you need to transform it to an array of React elements. In JavaScript, to transform one array into another, you can use the [array `map` method:](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/map)
你已经有一组 state 为 `history` 的数组，所以现在你需要将其转换为一组 React 元素。在 JavaScript 中，要将一个数组转换为另一个数组，可以使用 [数组的 `map` 方法](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/map)。

```jsx
[1, 2, 3].map((x) => x * 2) // [2, 4, 6]
```

You'll use `map` to transform your `history` of moves into React elements representing buttons on the screen, and display a list of buttons to "jump" to past moves. Let's `map` over the `history` in the Game component:
你将使用 `map` 将你的 `history` 动作转换为代表屏幕上按钮的 React 元素，并显示一个按钮列表以“跳转”到过去的动作。让我们在 Game 组件中用 `map` 代替 `history`：

```js {11-13,15-27,35}
export default function Game() {
  const [xIsNext, setXIsNext] = useState(true);
  const [history, setHistory] = useState([Array(9).fill(null)]);
  const currentSquares = history[history.length - 1];

  function handlePlay(nextSquares) {
    setHistory([...history, nextSquares]);
    setXIsNext(!xIsNext);
  }

  function jumpTo(nextMove) {
    // TODO
  }

  const moves = history.map((squares, move) => {
    let description;
    if (move > 0) {
      description = 'Go to move #' + move;
    } else {
      description = 'Go to game start';
    }
    return (
      <li>
        <button onClick={() => jumpTo(move)}>{description}</button>
      </li>
    );
  });

  return (
    <div className="game">
      <div className="game-board">
        <Board xIsNext={xIsNext} squares={currentSquares} onPlay={handlePlay} />
      </div>
      <div className="game-info">
        <ol>{moves}</ol>
      </div>
    </div>
  );
}
```

You can see what your code should look like below. Note that you should see an error in the developer tools console that says: 
你可以在下面看到你的代码应该是什么样子。请注意，你应该会在开发者工具控制台中看到一条错误消息：

<ConsoleBlock level="warning">
Warning: Each child in an array or iterator should have a unique "key" prop. Check the render method of &#96;Game&#96;.
</ConsoleBlock>
  
You'll fix this error in the next section.
你将在下一节中修复此错误。

<Sandpack>

```js src/App.js
import { useState } from 'react';

function Square({ value, onSquareClick }) {
  return (
    <button className="square" onClick={onSquareClick}>
      {value}
    </button>
  );
}

function Board({ xIsNext, squares, onPlay }) {
  function handleClick(i) {
    if (calculateWinner(squares) || squares[i]) {
      return;
    }
    const nextSquares = squares.slice();
    if (xIsNext) {
      nextSquares[i] = 'X';
    } else {
      nextSquares[i] = 'O';
    }
    onPlay(nextSquares);
  }

  const winner = calculateWinner(squares);
  let status;
  if (winner) {
    status = 'Winner: ' + winner;
  } else {
    status = 'Next player: ' + (xIsNext ? 'X' : 'O');
  }

  return (
    <>
      <div className="status">{status}</div>
      <div className="board-row">
        <Square value={squares[0]} onSquareClick={() => handleClick(0)} />
        <Square value={squares[1]} onSquareClick={() => handleClick(1)} />
        <Square value={squares[2]} onSquareClick={() => handleClick(2)} />
      </div>
      <div className="board-row">
        <Square value={squares[3]} onSquareClick={() => handleClick(3)} />
        <Square value={squares[4]} onSquareClick={() => handleClick(4)} />
        <Square value={squares[5]} onSquareClick={() => handleClick(5)} />
      </div>
      <div className="board-row">
        <Square value={squares[6]} onSquareClick={() => handleClick(6)} />
        <Square value={squares[7]} onSquareClick={() => handleClick(7)} />
        <Square value={squares[8]} onSquareClick={() => handleClick(8)} />
      </div>
    </>
  );
}

export default function Game() {
  const [xIsNext, setXIsNext] = useState(true);
  const [history, setHistory] = useState([Array(9).fill(null)]);
  const currentSquares = history[history.length - 1];

  function handlePlay(nextSquares) {
    setHistory([...history, nextSquares]);
    setXIsNext(!xIsNext);
  }

  function jumpTo(nextMove) {
    // TODO
  }

  const moves = history.map((squares, move) => {
    let description;
    if (move > 0) {
      description = 'Go to move #' + move;
    } else {
      description = 'Go to game start';
    }
    return (
      <li>
        <button onClick={() => jumpTo(move)}>{description}</button>
      </li>
    );
  });

  return (
    <div className="game">
      <div className="game-board">
        <Board xIsNext={xIsNext} squares={currentSquares} onPlay={handlePlay} />
      </div>
      <div className="game-info">
        <ol>{moves}</ol>
      </div>
    </div>
  );
}

function calculateWinner(squares) {
  const lines = [
    [0, 1, 2],
    [3, 4, 5],
    [6, 7, 8],
    [0, 3, 6],
    [1, 4, 7],
    [2, 5, 8],
    [0, 4, 8],
    [2, 4, 6],
  ];
  for (let i = 0; i < lines.length; i++) {
    const [a, b, c] = lines[i];
    if (squares[a] && squares[a] === squares[b] && squares[a] === squares[c]) {
      return squares[a];
    }
  }
  return null;
}
```

```css src/styles.css
* {
  box-sizing: border-box;
}

body {
  font-family: sans-serif;
  margin: 20px;
  padding: 0;
}

.square {
  background: #fff;
  border: 1px solid #999;
  float: left;
  font-size: 24px;
  font-weight: bold;
  line-height: 34px;
  height: 34px;
  margin-right: -1px;
  margin-top: -1px;
  padding: 0;
  text-align: center;
  width: 34px;
}

.board-row:after {
  clear: both;
  content: '';
  display: table;
}

.status {
  margin-bottom: 10px;
}

.game {
  display: flex;
  flex-direction: row;
}

.game-info {
  margin-left: 20px;
}
```

</Sandpack>

As you iterate through `history` array inside the function you passed to `map`, the `squares` argument goes through each element of `history`, and the `move` argument goes through each array index: `0`, `1`, `2`, …. (In most cases, you'd need the actual array elements, but to render a list of moves you will only need indexes.)
当你在传递给 `map` 的函数中遍历 `history` 数组时，`squares` 参数遍历 `history` 的每个元素，`move` 参数遍历每个数组索引：`0` 、`1`、`2`……（在大多数情况下，你需要数组元素，但要渲染落子列表，你只需要索引。）

For each move in the tic-tac-toe game's history, you create a list item `<li>` which contains a button `<button>`. The button has an `onClick` handler which calls a function called `jumpTo` (that you haven't implemented yet).
对于井字棋游戏历史中的每一步，你创建一个列表项 `<li>`，其中包含一个按钮 `<button>`。该按钮有一个 `onClick` 处理程序，它调用一个名为 `jumpTo` 的函数（你尚未实现）。

For now, you should see a list of the moves that occurred in the game and an error in the developer tools console. Let's discuss what the "key" error means.
现在，你应该会看到游戏中发生的动作列表和开发人员工具控制台中的错误。让我们讨论一下“关键”错误的含义。

### 选择 key | Picking a key {/*picking-a-key*/}

when you render a list, react stores some information about each rendered list item. when you update a list, react needs to determine what has changed. you could have added, removed, re-arranged, or updated the list's items.
当你渲染一个列表时，React 会存储一些关于每个渲染列表项的信息。当你更新一个列表时，React 需要确定发生了什么变化。你可以添加、删除、重新排列或更新列表的项目。

Imagine transitioning from
想象一下从

```html
<li>Alexa: 7 tasks left</li>
<li>Ben: 5 tasks left</li>
```

to
到

```html
<li>Ben: 9 tasks left</li>
<li>Claudia: 8 tasks left</li>
<li>Alexa: 5 tasks left</li>
```

In addition to the updated counts, a human reading this would probably say that you swapped Alexa and Ben's ordering and inserted Claudia between Alexa and Ben. However, React is a computer program and does not know what you intended, so you need to specify a _key_ property for each list item to differentiate each list item from its siblings. If your data was from a database, Alexa, Ben, and Claudia's database IDs could be used as keys.
除了更新的计数之外，阅读本文的人可能会说你交换了 Alexa 和 Ben 的顺序，并在 Alexa 和 Ben 之间插入了 Claudia。然而，React 是一个计算机程序，不知道你的意图，因此你需要为每个列表项指定一个 key 属性，以将每个列表项与其兄弟项区分开来。如果你的数据来自数据库，Alexa、Ben 和 Claudia 的数据库 ID 可以用作 key：

```js {1}
<li key={user.id}>
  {user.name}: {user.taskCount} tasks left
</li>
```

When a list is re-rendered, React takes each list item's key and searches the previous list's items for a matching key. If the current list has a key that didn't exist before, React creates a component. If the current list is missing a key that existed in the previous list, React destroys the previous component. If two keys match, the corresponding component is moved.
重新渲染列表时，React 获取每个列表项的 key 并搜索前一个列表的项以查找匹配的 key。如果当前列表有一个之前不存在的 key，React 会创建一个组件。如果当前列表缺少前一个列表中存在的 key，React 会销毁前一个组件。如果两个 key 匹配，则落子相应的组件。

Keys tell React about the identity of each component, which allows React to maintain state between re-renders. If a component's key changes, the component will be destroyed and re-created with a new state.
key 告诉 React 每个组件的身份，这使得 React 可以在重新渲染时保持 state。如果组件的 key 发生变化，组件将被销毁，新 state 将重新创建。

`key` is a special and reserved property in React. When an element is created, React extracts the `key` property and stores the key directly on the returned element. Even though `key` may look like it is passed as props, React automatically uses `key` to decide which components to update. There's no way for a component to ask what `key` its parent specified.
`key` 是 React 中一个特殊的保留属性。创建元素时，React 提取 `key` 属性并将 key 直接存储在返回的元素上。尽管 `key` 看起来像是作为 props 传递的，但 React 会自动使用 `key` 来决定要更新哪些组件。组件无法询问其父组件指定的 `key`。

**It's strongly recommended that you assign proper keys whenever you build dynamic lists.** If you don't have an appropriate key, you may want to consider restructuring your data so that you do.
**强烈建议你在构建动态列表时分配适当的 key**。如果你没有合适的 key，你可能需要考虑重组你的数据，以便你这样做。

If no key is specified, React will report an error and use the array index as a key by default. Using the array index as a key is problematic when trying to re-order a list's items or inserting/removing list items. Explicitly passing `key={i}` silences the error but has the same problems as array indices and is not recommended in most cases.
如果没有指定 key，React 会报错，默认使用数组索引作为 key。在尝试重新排序列表项或插入/删除列表项时，使用数组索引作为 key 是有问题的。显式传递 `key={i}` 可以消除错误，但与数组索引有相同的问题，在大多数情况下不推荐使用。

Keys do not need to be globally unique; they only need to be unique between components and their siblings.
key 不需要是全局唯一的；它们只需要在组件及其同级组件之间是唯一的。

### 实现时间旅行 | Implementing time travel {/*implementing-time-travel*/}

In the tic-tac-toe game's history, each past move has a unique ID associated with it: it's the sequential number of the move. Moves will never be re-ordered, deleted, or inserted in the middle, so it's safe to use the move index as a key.
在井字棋游戏的历史中，过去的每一步都有一个唯一的 ID 与之相关联：它是动作的序号。落子永远不会被重新排序、删除或从中间插入，因此使用落子的索引作为 key 是安全的。

In the `Game` function, you can add the key as `<li key={move}>`, and if you reload the rendered game, React's "key" error should disappear:
在 `Game` 函数中，你可以将 key 添加为 `<li key={move}>`，如果你重新加载渲染的游戏，React 的“key”错误应该会消失：

```js {4}
const moves = history.map((squares, move) => {
  //...
  return (
    <li key={move}>
      <button onClick={() => jumpTo(move)}>{description}</button>
    </li>
  );
});
```

<Sandpack>

```js src/App.js
import { useState } from 'react';

function Square({ value, onSquareClick }) {
  return (
    <button className="square" onClick={onSquareClick}>
      {value}
    </button>
  );
}

function Board({ xIsNext, squares, onPlay }) {
  function handleClick(i) {
    if (calculateWinner(squares) || squares[i]) {
      return;
    }
    const nextSquares = squares.slice();
    if (xIsNext) {
      nextSquares[i] = 'X';
    } else {
      nextSquares[i] = 'O';
    }
    onPlay(nextSquares);
  }

  const winner = calculateWinner(squares);
  let status;
  if (winner) {
    status = 'Winner: ' + winner;
  } else {
    status = 'Next player: ' + (xIsNext ? 'X' : 'O');
  }

  return (
    <>
      <div className="status">{status}</div>
      <div className="board-row">
        <Square value={squares[0]} onSquareClick={() => handleClick(0)} />
        <Square value={squares[1]} onSquareClick={() => handleClick(1)} />
        <Square value={squares[2]} onSquareClick={() => handleClick(2)} />
      </div>
      <div className="board-row">
        <Square value={squares[3]} onSquareClick={() => handleClick(3)} />
        <Square value={squares[4]} onSquareClick={() => handleClick(4)} />
        <Square value={squares[5]} onSquareClick={() => handleClick(5)} />
      </div>
      <div className="board-row">
        <Square value={squares[6]} onSquareClick={() => handleClick(6)} />
        <Square value={squares[7]} onSquareClick={() => handleClick(7)} />
        <Square value={squares[8]} onSquareClick={() => handleClick(8)} />
      </div>
    </>
  );
}

export default function Game() {
  const [xIsNext, setXIsNext] = useState(true);
  const [history, setHistory] = useState([Array(9).fill(null)]);
  const currentSquares = history[history.length - 1];

  function handlePlay(nextSquares) {
    setHistory([...history, nextSquares]);
    setXIsNext(!xIsNext);
  }

  function jumpTo(nextMove) {
    // TODO
  }

  const moves = history.map((squares, move) => {
    let description;
    if (move > 0) {
      description = 'Go to move #' + move;
    } else {
      description = 'Go to game start';
    }
    return (
      <li key={move}>
        <button onClick={() => jumpTo(move)}>{description}</button>
      </li>
    );
  });

  return (
    <div className="game">
      <div className="game-board">
        <Board xIsNext={xIsNext} squares={currentSquares} onPlay={handlePlay} />
      </div>
      <div className="game-info">
        <ol>{moves}</ol>
      </div>
    </div>
  );
}

function calculateWinner(squares) {
  const lines = [
    [0, 1, 2],
    [3, 4, 5],
    [6, 7, 8],
    [0, 3, 6],
    [1, 4, 7],
    [2, 5, 8],
    [0, 4, 8],
    [2, 4, 6],
  ];
  for (let i = 0; i < lines.length; i++) {
    const [a, b, c] = lines[i];
    if (squares[a] && squares[a] === squares[b] && squares[a] === squares[c]) {
      return squares[a];
    }
  }
  return null;
}

```

```css src/styles.css
* {
  box-sizing: border-box;
}

body {
  font-family: sans-serif;
  margin: 20px;
  padding: 0;
}

.square {
  background: #fff;
  border: 1px solid #999;
  float: left;
  font-size: 24px;
  font-weight: bold;
  line-height: 34px;
  height: 34px;
  margin-right: -1px;
  margin-top: -1px;
  padding: 0;
  text-align: center;
  width: 34px;
}

.board-row:after {
  clear: both;
  content: '';
  display: table;
}

.status {
  margin-bottom: 10px;
}

.game {
  display: flex;
  flex-direction: row;
}

.game-info {
  margin-left: 20px;
}
```

</Sandpack>

Before you can implement `jumpTo`, you need the `Game` component to keep track of which step the user is currently viewing. To do this, define a new state variable called `currentMove`, defaulting to `0`:
在你可以实现 `jumpTo` 之前，你需要 `Game` 组件来跟踪用户当前正在查看的步骤。为此，定义一个名为 `currentMove` 的新 state 变量，默认为 `0`：

```js {4}
export default function Game() {
  const [xIsNext, setXIsNext] = useState(true);
  const [history, setHistory] = useState([Array(9).fill(null)]);
  const [currentMove, setCurrentMove] = useState(0);
  const currentSquares = history[history.length - 1];
  //...
}
```

Next, update the `jumpTo` function inside `Game` to update that `currentMove`. You'll also set `xIsNext` to `true` if the number that you're changing `currentMove` to is even.
接下来，更新 `Game` 中的 `jumpTo` 函数来更新 `currentMove`。如果你将 `currentMove` 更改为偶数，你还将设置 `xIsNext` 为 `true`。

```js {4-5}
export default function Game() {
  // ...
  function jumpTo(nextMove) {
    setCurrentMove(nextMove);
    setXIsNext(nextMove % 2 === 0);
  }
  //...
}
```

You will now make two changes to the `Game`'s `handlePlay` function which is called when you click on a square.
你现在将对 `Game` 的 `handlePlay` 函数进行两处更改，该函数在你单击方块时调用。

- If you "go back in time" and then make a new move from that point, you only want to keep the history up to that point. Instead of adding `nextSquares` after all items (`...` spread syntax) in `history`, you'll add it after all items in `history.slice(0, currentMove + 1)` so that you're only keeping that portion of the old history.
- 如果你“回到过去”然后从那一点开始采取新的行动，你只想保持那一点的历史。不是在 `history` 中的所有项目（`...` 扩展语法）之后添加 `nextSquares`，而是在 `history.slice(0, currentMove + 1)` 中的所有项目之后添加它，这样你就只保留旧历史的那部分。
- Each time a move is made, you need to update `currentMove` to point to the latest history entry.
- 每次落子时，你都需要更新 `currentMove` 以指向最新的历史条目。

```js {2-4}
function handlePlay(nextSquares) {
  const nextHistory = [...history.slice(0, currentMove + 1), nextSquares];
  setHistory(nextHistory);
  setCurrentMove(nextHistory.length - 1);
  setXIsNext(!xIsNext);
}
```

Finally, you will modify the `Game` component to render the currently selected move, instead of always rendering the final move:
最后，你将修改 `Game` 组件以渲染当前选定的着法，而不是始终渲染最后的着法：

```js {5}
export default function Game() {
  const [xIsNext, setXIsNext] = useState(true);
  const [history, setHistory] = useState([Array(9).fill(null)]);
  const [currentMove, setCurrentMove] = useState(0);
  const currentSquares = history[currentMove];

  // ...
}
```

If you click on any step in the game's history, the tic-tac-toe board should immediately update to show what the board looked like after that step occurred.
如果你点击游戏历史中的任何一步，井字棋棋盘应立即更新以显示该步骤发生后棋盘的样子。

<Sandpack>

```js src/App.js
import { useState } from 'react';

function Square({value, onSquareClick}) {
  return (
    <button className="square" onClick={onSquareClick}>
      {value}
    </button>
  );
}

function Board({ xIsNext, squares, onPlay }) {
  function handleClick(i) {
    if (calculateWinner(squares) || squares[i]) {
      return;
    }
    const nextSquares = squares.slice();
    if (xIsNext) {
      nextSquares[i] = 'X';
    } else {
      nextSquares[i] = 'O';
    }
    onPlay(nextSquares);
  }

  const winner = calculateWinner(squares);
  let status;
  if (winner) {
    status = 'Winner: ' + winner;
  } else {
    status = 'Next player: ' + (xIsNext ? 'X' : 'O');
  }

  return (
    <>
      <div className="status">{status}</div>
      <div className="board-row">
        <Square value={squares[0]} onSquareClick={() => handleClick(0)} />
        <Square value={squares[1]} onSquareClick={() => handleClick(1)} />
        <Square value={squares[2]} onSquareClick={() => handleClick(2)} />
      </div>
      <div className="board-row">
        <Square value={squares[3]} onSquareClick={() => handleClick(3)} />
        <Square value={squares[4]} onSquareClick={() => handleClick(4)} />
        <Square value={squares[5]} onSquareClick={() => handleClick(5)} />
      </div>
      <div className="board-row">
        <Square value={squares[6]} onSquareClick={() => handleClick(6)} />
        <Square value={squares[7]} onSquareClick={() => handleClick(7)} />
        <Square value={squares[8]} onSquareClick={() => handleClick(8)} />
      </div>
    </>
  );
}

export default function Game() {
  const [xIsNext, setXIsNext] = useState(true);
  const [history, setHistory] = useState([Array(9).fill(null)]);
  const [currentMove, setCurrentMove] = useState(0);
  const currentSquares = history[currentMove];

  function handlePlay(nextSquares) {
    const nextHistory = [...history.slice(0, currentMove + 1), nextSquares];
    setHistory(nextHistory);
    setCurrentMove(nextHistory.length - 1);
    setXIsNext(!xIsNext);
  }

  function jumpTo(nextMove) {
    setCurrentMove(nextMove);
    setXIsNext(nextMove % 2 === 0);
  }

  const moves = history.map((squares, move) => {
    let description;
    if (move > 0) {
      description = 'Go to move #' + move;
    } else {
      description = 'Go to game start';
    }
    return (
      <li key={move}>
        <button onClick={() => jumpTo(move)}>{description}</button>
      </li>
    );
  });

  return (
    <div className="game">
      <div className="game-board">
        <Board xIsNext={xIsNext} squares={currentSquares} onPlay={handlePlay} />
      </div>
      <div className="game-info">
        <ol>{moves}</ol>
      </div>
    </div>
  );
}

function calculateWinner(squares) {
  const lines = [
    [0, 1, 2],
    [3, 4, 5],
    [6, 7, 8],
    [0, 3, 6],
    [1, 4, 7],
    [2, 5, 8],
    [0, 4, 8],
    [2, 4, 6],
  ];
  for (let i = 0; i < lines.length; i++) {
    const [a, b, c] = lines[i];
    if (squares[a] && squares[a] === squares[b] && squares[a] === squares[c]) {
      return squares[a];
    }
  }
  return null;
}
```

```css src/styles.css
* {
  box-sizing: border-box;
}

body {
  font-family: sans-serif;
  margin: 20px;
  padding: 0;
}

.square {
  background: #fff;
  border: 1px solid #999;
  float: left;
  font-size: 24px;
  font-weight: bold;
  line-height: 34px;
  height: 34px;
  margin-right: -1px;
  margin-top: -1px;
  padding: 0;
  text-align: center;
  width: 34px;
}

.board-row:after {
  clear: both;
  content: '';
  display: table;
}

.status {
  margin-bottom: 10px;
}
.game {
  display: flex;
  flex-direction: row;
}

.game-info {
  margin-left: 20px;
}
```

</Sandpack>

### 最后清理 | Final cleanup {/*final-cleanup*/}

If you look at the code very closely, you may notice that `xIsNext === true` when `currentMove` is even and `xIsNext === false` when `currentMove` is odd. In other words, if you know the value of `currentMove`, then you can always figure out what `xIsNext` should be.
如果仔细查看代码，你可能会注意到当 `currentMove` 为偶数时为 `xIsNext === true`，而当 `currentMove` 为奇数时为 `xIsNext === false`。换句话说，如果你知道 `currentMove` 的值，那么你总能算出 `xIsNext` 应该是什么。

There's no reason for you to store both of these in state. In fact, always try to avoid redundant state. Simplifying what you store in state reduces bugs and makes your code easier to understand. Change `Game` so that it doesn't store `xIsNext` as a separate state variable and instead figures it out based on the `currentMove`:
你没有理由将这两者都存储在 state 中。事实上，总是尽量避免冗余的 state。简化你在 state 中存储的内容可以减少错误并使你的代码更易于理解。更改 `Game` 使其不将 `xIsNext` 存储为单独的 state 变量，而是根据 `currentMove` 计算出来：

```js {4,11,15}
export default function Game() {
  const [history, setHistory] = useState([Array(9).fill(null)]);
  const [currentMove, setCurrentMove] = useState(0);
  const xIsNext = currentMove % 2 === 0;
  const currentSquares = history[currentMove];

  function handlePlay(nextSquares) {
    const nextHistory = [...history.slice(0, currentMove + 1), nextSquares];
    setHistory(nextHistory);
    setCurrentMove(nextHistory.length - 1);
  }

  function jumpTo(nextMove) {
    setCurrentMove(nextMove);
  }
  // ...
}
```

You no longer need the `xIsNext` state declaration or the calls to `setXIsNext`. Now, there's no chance for `xIsNext` to get out of sync with `currentMove`, even if you make a mistake while coding the components.
你不再需要 `xIsNext` state 声明或对 `setXIsNext` 的调用。现在，`xIsNext` 不可能与 `currentMove` 不同步，即使你的代码写错了。

### 收尾 | Wrapping up {/*wrapping-up*/}

Congratulations! You've created a tic-tac-toe game that:
祝贺！你已经创建了一个井字棋游戏，你实现了：

- lets you play tic-tac-toe,
- 你现在可以玩的井字棋游戏
- Indicates when a player has won the game,
- 玩家在赢的时候有提示
- Stores a game's history as a game progresses,
- 随着游戏的进行存储游戏的历史
- Allows players to review a game's history and see previous versions of a game's board.
- 允许玩家回顾游戏的历史并查看棋盘的以前的版本

Nice work! We hope you now feel like you have a decent grasp of how React works.
干得好！我们希望你现在觉得你对 React 的工作原理有了很好的了解。

Check out the final result here:
在这里对照一下最终的结果吧：

<Sandpack>

```js src/App.js
import { useState } from 'react';

function Square({ value, onSquareClick }) {
  return (
    <button className="square" onClick={onSquareClick}>
      {value}
    </button>
  );
}

function Board({ xIsNext, squares, onPlay }) {
  function handleClick(i) {
    if (calculateWinner(squares) || squares[i]) {
      return;
    }
    const nextSquares = squares.slice();
    if (xIsNext) {
      nextSquares[i] = 'X';
    } else {
      nextSquares[i] = 'O';
    }
    onPlay(nextSquares);
  }

  const winner = calculateWinner(squares);
  let status;
  if (winner) {
    status = 'Winner: ' + winner;
  } else {
    status = 'Next player: ' + (xIsNext ? 'X' : 'O');
  }

  return (
    <>
      <div className="status">{status}</div>
      <div className="board-row">
        <Square value={squares[0]} onSquareClick={() => handleClick(0)} />
        <Square value={squares[1]} onSquareClick={() => handleClick(1)} />
        <Square value={squares[2]} onSquareClick={() => handleClick(2)} />
      </div>
      <div className="board-row">
        <Square value={squares[3]} onSquareClick={() => handleClick(3)} />
        <Square value={squares[4]} onSquareClick={() => handleClick(4)} />
        <Square value={squares[5]} onSquareClick={() => handleClick(5)} />
      </div>
      <div className="board-row">
        <Square value={squares[6]} onSquareClick={() => handleClick(6)} />
        <Square value={squares[7]} onSquareClick={() => handleClick(7)} />
        <Square value={squares[8]} onSquareClick={() => handleClick(8)} />
      </div>
    </>
  );
}

export default function Game() {
  const [history, setHistory] = useState([Array(9).fill(null)]);
  const [currentMove, setCurrentMove] = useState(0);
  const xIsNext = currentMove % 2 === 0;
  const currentSquares = history[currentMove];

  function handlePlay(nextSquares) {
    const nextHistory = [...history.slice(0, currentMove + 1), nextSquares];
    setHistory(nextHistory);
    setCurrentMove(nextHistory.length - 1);
  }

  function jumpTo(nextMove) {
    setCurrentMove(nextMove);
  }

  const moves = history.map((squares, move) => {
    let description;
    if (move > 0) {
      description = 'Go to move #' + move;
    } else {
      description = 'Go to game start';
    }
    return (
      <li key={move}>
        <button onClick={() => jumpTo(move)}>{description}</button>
      </li>
    );
  });

  return (
    <div className="game">
      <div className="game-board">
        <Board xIsNext={xIsNext} squares={currentSquares} onPlay={handlePlay} />
      </div>
      <div className="game-info">
        <ol>{moves}</ol>
      </div>
    </div>
  );
}

function calculateWinner(squares) {
  const lines = [
    [0, 1, 2],
    [3, 4, 5],
    [6, 7, 8],
    [0, 3, 6],
    [1, 4, 7],
    [2, 5, 8],
    [0, 4, 8],
    [2, 4, 6],
  ];
  for (let i = 0; i < lines.length; i++) {
    const [a, b, c] = lines[i];
    if (squares[a] && squares[a] === squares[b] && squares[a] === squares[c]) {
      return squares[a];
    }
  }
  return null;
}
```

```css src/styles.css
* {
  box-sizing: border-box;
}

body {
  font-family: sans-serif;
  margin: 20px;
  padding: 0;
}

.square {
  background: #fff;
  border: 1px solid #999;
  float: left;
  font-size: 24px;
  font-weight: bold;
  line-height: 34px;
  height: 34px;
  margin-right: -1px;
  margin-top: -1px;
  padding: 0;
  text-align: center;
  width: 34px;
}

.board-row:after {
  clear: both;
  content: '';
  display: table;
}

.status {
  margin-bottom: 10px;
}
.game {
  display: flex;
  flex-direction: row;
}

.game-info {
  margin-left: 20px;
}
```

</Sandpack>

If you have extra time or want to practice your new React skills, here are some ideas for improvements that you could make to the tic-tac-toe game, listed in order of increasing difficulty:
如果你有额外的时间或想练习新的 React 技能，这里有一些你可以改进井字棋游戏的想法，按难度递增的顺序列出：

1. For the current move only, show "You are at move #..." instead of a button.
- 仅针对当前着手，显示“You are at move #…”而不是按钮。
1. Rewrite `Board` to use two loops to make the squares instead of hardcoding them.
- 重写 `Board` 以使用两个循环来制作方块而不是对它们进行硬编码。
3. Add a toggle button that lets you sort the moves in either ascending or descending order.
- 添加一个切换按钮，使可以按升序或降序对落子的步数进行排序。
4. When someone wins, highlight the three squares that caused the win (and when no one wins, display a message about the result being a draw).
- 当有人获胜时，突出显示致使获胜的三个方块（当没有人获胜时，显示一条关于结果为平局的消息）。
5. Display the location for each move in the format (row, col) in the move history list.
- 在“落子”的历史列表中以 (row, col) 格式显示每步的位置。

Throughout this tutorial, you've touched on React concepts including elements, components, props, and state. Now that you've seen how these concepts work when building a game, check out [Thinking in React](/learn/thinking-in-react) to see how the same React concepts work when building an app's UI.
在本教程中，你已经接触到了 React 概念，包括元素、组件、props 和 state。现在你已经了解了这些概念在构建游戏时是如何工作的，请查看 [React 哲学](/learn/thinking-in-react) 以了解这些 React 概念在构建应用的 UI 时是如何工作的。
