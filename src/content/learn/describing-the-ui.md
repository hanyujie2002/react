---
title: 描述 UI | Describing the UI
translators:
  - ChelesteWang
  - QC-L
  - KnowsCount
  - lyj505
---

<Intro>

React is a JavaScript library for rendering user interfaces (UI). UI is built from small units like buttons, text, and images. React lets you combine them into reusable, nestable *components.* From web sites to phone apps, everything on the screen can be broken down into components. In this chapter, you'll learn to create, customize, and conditionally display React components.
React 是一个用于构建用户界面（UI）的 JavaScript 库，用户界面由按钮、文本和图像等小单元内容构建而成。React 帮助你把它们组合成可重用、可嵌套的 *组件*。从 web 端网站到移动端应用，屏幕上的所有内容都可以被分解成组件。在本章节中，你将学习如何创建、定制以及有条件地显示 React 组件。

</Intro>

<YouWillLearn isChapter={true}>

* [How to write your first React component](/learn/your-first-component)
* [如何创建你的第一个组件](/learn/your-first-component)
* [When and how to create multi-component files](/learn/importing-and-exporting-components)
* [在什么时候以及如何创建多文件组件](/learn/importing-and-exporting-components)
* [How to add markup to JavaScript with JSX](/learn/writing-markup-with-jsx)
* [如何使用 JSX 为 JavaScript 添加标签](/learn/writing-markup-with-jsx)
* [How to use curly braces with JSX to access JavaScript functionality from your components](/learn/javascript-in-jsx-with-curly-braces)
* [如何在 JSX 中使用花括号来从组件中使用 JavaScript 功能](/learn/javascript-in-jsx-with-curly-braces)
* [How to configure components with props](/learn/passing-props-to-a-component)
* [如何用 props 配置组件](/learn/passing-props-to-a-component)
* [How to conditionally render components](/learn/conditional-rendering)
* [如何有条件地渲染组件](/learn/conditional-rendering)
* [How to render multiple components at a time](/learn/rendering-lists)
* [如何在同一时间渲染多个组件](/learn/rendering-lists)
* [How to avoid confusing bugs by keeping components pure](/learn/keeping-components-pure)
* [How to avoid confusing bugs by keeping components pure](/learn/keeping-components-pure)
* [如何通过保持组件的纯粹性来避免令人困惑的错误](/learn/keeping-components-pure)
* [Why understanding your UI as trees is useful](/learn/understanding-your-ui-as-a-tree)
* [为什么将 UI 理解为树是有用的](/learn/understanding-your-ui-as-a-tree)

</YouWillLearn>

## 你的第一个组件 | Your first component {/*your-first-component*/}

React applications are built from isolated pieces of UI called *components*. A React component is a JavaScript function that you can sprinkle with markup. Components can be as small as a button, or as large as an entire page. Here is a `Gallery` component rendering three `Profile` components:
React 应用是由被称为 **组件** 的独立 UI 片段构建而成。React 组件本质上是可以任意添加标签的 JavaScript 函数。组件可以小到一个按钮，也可以大到是整个页面。这是一个 `Gallery` 组件，用于渲染三个 `Profile` 组件：

<Sandpack>

```js
function Profile() {
  return (
    <img
      src="https://i.imgur.com/MK3eW3As.jpg"
      alt="Katherine Johnson"
    />
  );
}

export default function Gallery() {
  return (
    <section>
      <h1>Amazing scientists</h1>
      <Profile />
      <Profile />
      <Profile />
    </section>
  );
}
```

```css
img { margin: 0 10px 10px 0; height: 90px; }
```

</Sandpack>

<LearnMore path="/learn/your-first-component">

Read **[Your First Component](/learn/your-first-component)** to learn how to declare and use React components.
请参阅 **[你的第一个组件](/learn/your-first-component)** 以学习如何声明并使用 React 组件

</LearnMore>

## 组件的导入与导出 | Importing and exporting components {/*importing-and-exporting-components*/}

You can declare many components in one file, but large files can get difficult to navigate. To solve this, you can *export* a component into its own file, and then *import* that component from another file:
你可以在一个文件中声明许多组件，但文件的体积过大会变得难以浏览。为了解决这个问题，你可以在一个文件中只*导出*一个组件，然后再从另一个文件中*导入*该组件：

<Sandpack>

```js src/App.js hidden
import Gallery from './Gallery.js';

export default function App() {
  return (
    <Gallery />
  );
}
```

```js src/Gallery.js active
import Profile from './Profile.js';

export default function Gallery() {
  return (
    <section>
      <h1>Amazing scientists</h1>
      <Profile />
      <Profile />
      <Profile />
    </section>
  );
}
```

```js src/Profile.js
export default function Profile() {
  return (
    <img
      src="https://i.imgur.com/QIrZWGIs.jpg"
      alt="Alan L. Hart"
    />
  );
}
```

```css
img { margin: 0 10px 10px 0; }
```

</Sandpack>

<LearnMore path="/learn/importing-and-exporting-components">

Read **[Importing and Exporting Components](/learn/importing-and-exporting-components)** to learn how to split components into their own files.
请参阅 **[组件的导入与导出](/learn/importing-and-exporting-components)** 以学习如何切分组件。
</LearnMore>

## 使用 JSX 书写标签语言 | Writing markup with JSX {/*writing-markup-with-jsx*/}

Each React component is a JavaScript function that may contain some markup that React renders into the browser. React components use a syntax extension called JSX to represent that markup. JSX looks a lot like HTML, but it is a bit stricter and can display dynamic information.
每个 React 组件都是一个 JavaScript 函数，它可能包含一些标签，React 会将其渲染到浏览器中。React 组件使用一种叫做 JSX 的语法扩展来表示该标签。JSX 看起来很像 HTML，但它更为严格，可以显示动态信息。

If we paste existing HTML markup into a React component, it won't always work:
如果我们把现有的 HTML 标签粘贴到 React 组件中，它并不一定能成功运行：

<Sandpack>

```js
export default function TodoList() {
  return (
    // This doesn't quite work!
    <h1>Hedy Lamarr's Todos</h1>
    <img
      src="https://i.imgur.com/yXOvdOSs.jpg"
      alt="Hedy Lamarr"
      class="photo"
    >
    <ul>
      <li>Invent new traffic lights
      <li>Rehearse a movie scene
      <li>Improve spectrum technology
    </ul>
  );
}
```

```css
img { height: 90px; }
```

</Sandpack>

If you have existing HTML like this, you can fix it using a [converter](https://transform.tools/html-to-jsx):
如果你有像这样的现有的 HTML 片段，你可以使用它进行语法转换 [converter](https://transform.tools/html-to-jsx)：

<Sandpack>

```js
export default function TodoList() {
  return (
    <>
      <h1>Hedy Lamarr's Todos</h1>
      <img
        src="https://i.imgur.com/yXOvdOSs.jpg"
        alt="Hedy Lamarr"
        className="photo"
      />
      <ul>
        <li>Invent new traffic lights</li>
        <li>Rehearse a movie scene</li>
        <li>Improve spectrum technology</li>
      </ul>
    </>
  );
}
```

```css
img { height: 90px; }
```

</Sandpack>

<LearnMore path="/learn/writing-markup-with-jsx">

Read **[Writing Markup with JSX](/learn/writing-markup-with-jsx)** to learn how to write valid JSX.
请参阅 **[使用 JSX 书写标签语言](/learn/writing-markup-with-jsx)** 以学习如何编写有效的 JSX。

</LearnMore>

## 在 JSX 中通过大括号使用 JavaScript | JavaScript in JSX with curly braces {/*javascript-in-jsx-with-curly-braces*/}

JSX lets you write HTML-like markup inside a JavaScript file, keeping rendering logic and content in the same place. Sometimes you will want to add a little JavaScript logic or reference a dynamic property inside that markup. In this situation, you can use curly braces in your JSX to "open a window" to JavaScript:
JSX 可以让你在 JavaScript 文件中编写类似 HTML 的标签语法，使渲染逻辑和内容展示维护在同一个地方。有时你会想在标签中添加一点 JavaScript 逻辑或引用一个动态属性。在这种情况下，你可以在 JSX 中使用花括号来为 JavaScript "开辟通道"：

<Sandpack>

```js
const person = {
  name: 'Gregorio Y. Zara',
  theme: {
    backgroundColor: 'black',
    color: 'pink'
  }
};

export default function TodoList() {
  return (
    <div style={person.theme}>
      <h1>{person.name}'s Todos</h1>
      <img
        className="avatar"
        src="https://i.imgur.com/7vQD0fPs.jpg"
        alt="Gregorio Y. Zara"
      />
      <ul>
        <li>Improve the videophone</li>
        <li>Prepare aeronautics lectures</li>
        <li>Work on the alcohol-fuelled engine</li>
      </ul>
    </div>
  );
}
```

```css
body { padding: 0; margin: 0 }
body > div > div { padding: 20px; }
.avatar { border-radius: 50%; height: 90px; }
```

</Sandpack>

<LearnMore path="/learn/javascript-in-jsx-with-curly-braces">

Read **[JavaScript in JSX with Curly Braces](/learn/javascript-in-jsx-with-curly-braces)** to learn how to access JavaScript data from JSX.
请参阅 **[在 JSX 中通过大括号使用 JavaScript](/learn/javascript-in-jsx-with-curly-braces)** 以学习如何从 JSX 中访问 JavaScript 数据。

</LearnMore>

## 将 Props 传递给组件 | Passing props to a component {/*passing-props-to-a-component*/}

React components use *props* to communicate with each other. Every parent component can pass some information to its child components by giving them props. Props might remind you of HTML attributes, but you can pass any JavaScript value through them, including objects, arrays, functions, and even JSX!
React 组件使用 *props* 来进行组件之间的通讯。每个父组件都可以通过为子组件提供 props 的方式来传递信息。props 可能会让你想起 HTML 属性，但你可以通过它们传递任何 JavaScript 的值，包括对象、数组、函数、甚至是 JSX!

<Sandpack>

```js
import { getImageUrl } from './utils.js'

export default function Profile() {
  return (
    <Card>
      <Avatar
        size={100}
        person={{
          name: 'Katsuko Saruhashi',
          imageId: 'YfeOqp2'
        }}
      />
    </Card>
  );
}

function Avatar({ person, size }) {
  return (
    <img
      className="avatar"
      src={getImageUrl(person)}
      alt={person.name}
      width={size}
      height={size}
    />
  );
}

function Card({ children }) {
  return (
    <div className="card">
      {children}
    </div>
  );
}

```

```js src/utils.js
export function getImageUrl(person, size = 's') {
  return (
    'https://i.imgur.com/' +
    person.imageId +
    size +
    '.jpg'
  );
}
```

```css
.card {
  width: fit-content;
  margin: 5px;
  padding: 5px;
  font-size: 20px;
  text-align: center;
  border: 1px solid #aaa;
  border-radius: 20px;
  background: #fff;
}
.avatar {
  margin: 20px;
  border-radius: 50%;
}
```

</Sandpack>

<LearnMore path="/learn/passing-props-to-a-component">

Read **[Passing Props to a Component](/learn/passing-props-to-a-component)** to learn how to pass and read props.
请参阅 **[将 Props 传递给组件](/learn/passing-props-to-a-component)** 以了解如何传递并读取 props。

</LearnMore>

## 条件渲染 | Conditional rendering {/*conditional-rendering*/}

Your components will often need to display different things depending on different conditions. In React, you can conditionally render JSX using JavaScript syntax like `if` statements, `&&`, and `? :` operators.
你的组件经常需要根据不同的条件来显示不同的东西。在 React 中，你可以使用 JavaScript 语法，如 `if` 语句、`&&` 和 `? :` 操作符有条件地渲染 JSX。

In this example, the JavaScript `&&` operator is used to conditionally render a checkmark:
在这个示例中，JavaScript 的 `&&` 操作符将会条件渲染一个复选标签：

<Sandpack>

```js
function Item({ name, isPacked }) {
  return (
    <li className="item">
      {name} {isPacked && '✅'}
    </li>
  );
}

export default function PackingList() {
  return (
    <section>
      <h1>Sally Ride's Packing List</h1>
      <ul>
        <Item
          isPacked={true}
          name="Space suit"
        />
        <Item
          isPacked={true}
          name="Helmet with a golden leaf"
        />
        <Item
          isPacked={false}
          name="Photo of Tam"
        />
      </ul>
    </section>
  );
}
```

</Sandpack>

<LearnMore path="/learn/conditional-rendering">

Read **[Conditional Rendering](/learn/conditional-rendering)** to learn the different ways to render content conditionally.
请参阅 **[条件渲染](/learn/conditional-rendering)** 以学习使用不同的方法对内容进行条件渲染。

</LearnMore>

## 渲染列表 | Rendering lists {/*rendering-lists*/}

You will often want to display multiple similar components from a collection of data. You can use JavaScript's `filter()` and `map()` with React to filter and transform your array of data into an array of components.
通常，你需要根据数据集合来渲染多个较为类似的组件。你可以在 React 中使用 JavaScript 的 `filter()` 和 `map()` 来实现数组的过滤和转换，将数据数组转换为组件数组。

For each array item, you will need to specify a `key`. Usually, you will want to use an ID from the database as a `key`. Keys let React keep track of each item's place in the list even if the list changes.
对于数组的每个元素项，你需要指定一个 `key`。通常你需要使用数据库中的 ID 作为 `key`。即使列表发生了变化，React 也可以通过 key 来跟踪每个元素在列表中的位置。

<Sandpack>

```js src/App.js
import { people } from './data.js';
import { getImageUrl } from './utils.js';

export default function List() {
  const listItems = people.map(person =>
    <li key={person.id}>
      <img
        src={getImageUrl(person)}
        alt={person.name}
      />
      <p>
        <b>{person.name}:</b>
        {' ' + person.profession + ' '}
        known for {person.accomplishment}
      </p>
    </li>
  );
  return (
    <article>
      <h1>Scientists</h1>
      <ul>{listItems}</ul>
    </article>
  );
}
```

```js src/data.js
export const people = [{
  id: 0,
  name: 'Creola Katherine Johnson',
  profession: 'mathematician',
  accomplishment: 'spaceflight calculations',
  imageId: 'MK3eW3A'
}, {
  id: 1,
  name: 'Mario José Molina-Pasquel Henríquez',
  profession: 'chemist',
  accomplishment: 'discovery of Arctic ozone hole',
  imageId: 'mynHUSa'
}, {
  id: 2,
  name: 'Mohammad Abdus Salam',
  profession: 'physicist',
  accomplishment: 'electromagnetism theory',
  imageId: 'bE7W1ji'
}, {
  id: 3,
  name: 'Percy Lavon Julian',
  profession: 'chemist',
  accomplishment: 'pioneering cortisone drugs, steroids and birth control pills',
  imageId: 'IOjWm71'
}, {
  id: 4,
  name: 'Subrahmanyan Chandrasekhar',
  profession: 'astrophysicist',
  accomplishment: 'white dwarf star mass calculations',
  imageId: 'lrWQx8l'
}];
```

```js src/utils.js
export function getImageUrl(person) {
  return (
    'https://i.imgur.com/' +
    person.imageId +
    's.jpg'
  );
}
```

```css
ul { list-style-type: none; padding: 0px 10px; }
li {
  margin-bottom: 10px;
  display: grid;
  grid-template-columns: 1fr 1fr;
  align-items: center;
}
img { width: 100px; height: 100px; border-radius: 50%; }
h1 { font-size: 22px; }
h2 { font-size: 20px; }
```

</Sandpack>

<LearnMore path="/learn/rendering-lists">

Read **[Rendering Lists](/learn/rendering-lists)** to learn how to render a list of components, and how to choose a key.
请参阅 **[渲染列表](/learn/rendering-lists)** 以学习如何渲染一个组件列表，以及如何选择一个 `key`。

</LearnMore>

## 保持组件纯粹 | Keeping components pure {/*keeping-components-pure*/}

Some JavaScript functions are *pure.* A pure function:
有些 JavaScript 函数是 **纯粹** 的。纯函数的基本定义：

* **Minds its own business.** It does not change any objects or variables that existed before it was called.
* **只负责自己的任务**。 它不会更改在该函数调用前就已存在的对象或变量。
* **Same inputs, same output.** Given the same inputs, a pure function should always return the same result.
* **输入相同，输出也相同**。 在输入相同的情况下，对纯函数来说应总是返回相同的结果。

By strictly only writing your components as pure functions, you can avoid an entire class of baffling bugs and unpredictable behavior as your codebase grows. Here is an example of an impure component:
严格遵循纯函数的定义编写组件，可以让代码库体量增长时，避免一些令人困惑的错误和不可预测的行为。下面是一个非纯函数组件的示例：

<Sandpack>

```js
let guest = 0;

function Cup() {
  // Bad: changing a preexisting variable!
  guest = guest + 1;
  return <h2>Tea cup for guest #{guest}</h2>;
}

export default function TeaSet() {
  return (
    <>
      <Cup />
      <Cup />
      <Cup />
    </>
  );
}
```

</Sandpack>

You can make this component pure by passing a prop instead of modifying a preexisting variable:
你可以通过传递一个 `props` 来使这个组件变得纯粹，而非修改已经存在的变量：

<Sandpack>

```js
function Cup({ guest }) {
  return <h2>Tea cup for guest #{guest}</h2>;
}

export default function TeaSet() {
  return (
    <>
      <Cup guest={1} />
      <Cup guest={2} />
      <Cup guest={3} />
    </>
  );
}
```

</Sandpack>

<LearnMore path="/learn/keeping-components-pure">

Read **[Keeping Components Pure](/learn/keeping-components-pure)** to learn how to write components as pure, predictable functions.
请参阅 **[保持组件纯粹](/learn/keeping-components-pure)** 以学习如何将组件写成纯粹且可预测的函数。

</LearnMore>

## 将 UI 视为树 | Your UI as a tree {/*your-ui-as-a-tree*/}

React uses trees to model the relationships between components and modules. 
React 使用树形关系建模以展示组件和模块之间的关系。

A React render tree is a representation of the parent and child relationship between components. 
React 渲染树是组件之间父子关系的表示。

<Diagram name="generic_render_tree" height={250} width={500} alt="一个树状图，有五个节点，每个节点代表一个组件。树状图的顶部有一个名为 Root Component 的根节点，它有两个向下延伸的箭头，分别标有 renders。两个箭头指向标有 Component A 和 Component C 的节点。Component A 有一条 renders 箭头指向标有 Component B 的节点。Component C 有一条 renders 箭头指向标有 Component D 的节点。 | A tree graph with five nodes, with each node representing a component. The root node is located at the top the tree graph and is labelled 'Root Component'. It has two arrows extending down to two nodes labelled 'Component A' and 'Component C'. Each of the arrows is labelled with 'renders'. 'Component A' has a single 'renders' arrow to a node labelled 'Component B'. 'Component C' has a single 'renders' arrow to a node labelled 'Component D'.">

An example React render tree.
示例的 React 渲染树

</Diagram>

Components near the top of the tree, near the root component, are considered top-level components. Components with no child components are leaf components. This categorization of components is useful for understanding data flow and rendering performance.
位于树顶部、靠近根组件的组件被视为顶层组件。没有子组件的组件被称为叶子组件。对组件的这种分类对于理解数据流和渲染性能非常有用。

Modelling the relationship between JavaScript modules is another useful way to understand your app. We refer to it as a module dependency tree. 
对 JavaScript 模块之间的关系进行建模是了解应用程序的另一种有用方式。我们将其称为模块依赖树。

<Diagram name="generic_dependency_tree" height={250} width={500} alt="一个树状图，有五个节点。每个节点代表一个 JavaScript 模块。最顶部的节点标有 RootModule.js。它有三条箭头指向节点：ModuleA.js、ModuleB.js 和 ModuleC.js。每个箭头标有 imports。ModuleC.js 节点有一条 imports 箭头指向标有 ModuleD.js的节点。 | A tree graph with five nodes. Each node represents a JavaScript module. The top-most node is labelled 'RootModule.js'. It has three arrows extending to the nodes: 'ModuleA.js', 'ModuleB.js', and 'ModuleC.js'. Each arrow is labelled as 'imports'. 'ModuleC.js' node has a single 'imports' arrow that points to a node labelled 'ModuleD.js'.">

An example module dependency tree.
示例的模块依赖树

</Diagram>

A dependency tree is often used by build tools to bundle all the relevant JavaScript code for the client to download and render. A large bundle size regresses user experience for React apps. Understanding the module dependency tree is helpful to debug such issues. 
构建工具经常使用依赖树来捆绑客户端下载和渲染所需的所有 JavaScript 代码。对于 React 应用程序，打包大小会导致用户体验退化。了解模块依赖树有助于调试此类问题。

<LearnMore path="/learn/understanding-your-ui-as-a-tree">

Read **[Your UI as a Tree](/learn/understanding-your-ui-as-a-tree)** to learn how to create a render and module dependency trees for a React app and how they're useful mental models for improving user experience and performance.
阅读 **[将 UI 视为树](/learn/understanding-your-ui-as-a-tree)** 以了解如何为 React 应用程序创建渲染和模块依赖树，以及它们如何成为有效改善用户体验和性能的心理模型。

</LearnMore>


## 接下来应该…… | What's next? {/*whats-next*/}

Head over to [Your First Component](/learn/your-first-component) to start reading this chapter page by page!
请访问 [你的第一个组件](/learn/your-first-component) 这个章节并开始阅读！

Or, if you're already familiar with these topics, why not read about [Adding Interactivity](/learn/adding-interactivity)?
如若你已熟悉这些主题，可直接阅读 [添加交互](/learn/adding-interactivity) 一节。
