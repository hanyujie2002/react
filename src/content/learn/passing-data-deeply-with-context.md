---
title: 使用 Context 深层传递参数 | Passing Data Deeply with Context
translators: 
  - KeanuoLeung
  - rottenpen
---

<Intro>

Usually, you will pass information from a parent component to a child component via props. But passing props can become verbose and inconvenient if you have to pass them through many components in the middle, or if many components in your app need the same information. *Context* lets the parent component make some information available to any component in the tree below it—no matter how deep—without passing it explicitly through props.
通常来说，你会通过 props 将信息从父组件传递到子组件。但是，如果你必须通过许多中间组件向下传递 props，或是在你应用中的许多组件需要相同的信息，传递 props 会变的十分冗长和不便。**Context** 允许父组件向其下层无论多深的任何组件提供信息，而无需通过 props 显式传递。

</Intro>

<YouWillLearn>

- What "prop drilling" is
- 什么是 “prop 逐级透传”
- How to replace repetitive prop passing with context
- 如何使用 context 代替重复的参数传递
- Common use cases for context
- Context 的常见用法
- Common alternatives to context
- Context 的常见替代方案

</YouWillLearn>

## 传递 props 带来的问题 | The problem with passing props {/*the-problem-with-passing-props*/}

[Passing props](/learn/passing-props-to-a-component) is a great way to explicitly pipe data through your UI tree to the components that use it.
[传递 props](/learn/passing-props-to-a-component) 是将数据通过 UI 树显式传递到使用它的组件的好方法。

But passing props can become verbose and inconvenient when you need to pass some prop deeply through the tree, or if many components need the same prop. The nearest common ancestor could be far removed from the components that need data, and [lifting state up](/learn/sharing-state-between-components) that high can lead to a situation called "prop drilling".
但是当你需要在组件树中深层传递参数以及需要在组件间复用相同的参数时，传递 props 就会变得很麻烦。最近的根节点父组件可能离需要数据的组件很远，[状态提升](/learn/sharing-state-between-components) 到太高的层级会导致 “逐层传递 props” 的情况。

<DiagramGroup>

<Diagram name="passing_data_lifting_state" height={160} width={608} captionPosition="top" alt="Diagram with a tree of three components. The parent contains a bubble representing a value highlighted in purple. The value flows down to each of the two children, both highlighted in purple." >

Lifting state up
状态提升

</Diagram>
<Diagram name="passing_data_prop_drilling" height={430} width={608} captionPosition="top" alt="Diagram with a tree of ten nodes, each node with two children or less. The root node contains a bubble representing a value highlighted in purple. The value flows down through the two children, each of which pass the value but do not contain it. The left child passes the value down to two children which are both highlighted purple. The right child of the root passes the value through to one of its two children - the right one, which is highlighted purple. That child passed the value through its single child, which passes it down to both of its two children, which are highlighted purple.">

Prop drilling
Prop 逐级透传

</Diagram>

</DiagramGroup>

Wouldn't it be great if there were a way to "teleport" data to the components in the tree that need it without passing props? With React's context feature, there is!
要是有一种方法可以在组件树中不需要 props 将数据“直达”到所需的组件中，那可就太好了。React 的 context 功能可以满足我们的这个心愿。

## Context：传递 props 的另一种方法 | Context: an alternative to passing props {/*context-an-alternative-to-passing-props*/}

Context lets a parent component provide data to the entire tree below it. There are many uses for context. Here is one example. Consider this `Heading` component that accepts a `level` for its size:
Context 让父组件可以为它下面的整个组件树提供数据。Context 有很多种用途。这里就有一个示例。思考一下这个 `Heading` 组件接收一个 `level` 参数来决定它标题尺寸的场景：

<Sandpack>

```js
import Heading from './Heading.js';
import Section from './Section.js';

export default function Page() {
  return (
    <Section>
      <Heading level={1}>主标题</Heading>
      <Heading level={2}>副标题</Heading>
      <Heading level={3}>子标题</Heading>
      <Heading level={4}>子子标题</Heading>
      <Heading level={5}>子子子标题</Heading>
      <Heading level={6}>子子子子标题</Heading>
    </Section>
  );
}
```

```js src/Section.js
export default function Section({ children }) {
  return (
    <section className="section">
      {children}
    </section>
  );
}
```

```js src/Heading.js
export default function Heading({ level, children }) {
  switch (level) {
    case 1:
      return <h1>{children}</h1>;
    case 2:
      return <h2>{children}</h2>;
    case 3:
      return <h3>{children}</h3>;
    case 4:
      return <h4>{children}</h4>;
    case 5:
      return <h5>{children}</h5>;
    case 6:
      return <h6>{children}</h6>;
    default:
      throw Error('未知的 level：' + level);
  }
}
```

```css
.section {
  padding: 10px;
  margin: 5px;
  border-radius: 5px;
  border: 1px solid #aaa;
}
```

</Sandpack>

Let's say you want multiple headings within the same `Section` to always have the same size:
假设你想让相同 `Section` 中的多个 Heading 具有相同的尺寸：

<Sandpack>

```js
import Heading from './Heading.js';
import Section from './Section.js';

export default function Page() {
  return (
    <Section>
      <Heading level={1}>主标题</Heading>
      <Section>
        <Heading level={2}>副标题</Heading>
        <Heading level={2}>副标题</Heading>
        <Heading level={2}>副标题</Heading>
        <Section>
          <Heading level={3}>子标题</Heading>
          <Heading level={3}>子标题</Heading>
          <Heading level={3}>子标题</Heading>
          <Section>
            <Heading level={4}>子子标题</Heading>
            <Heading level={4}>子子标题</Heading>
            <Heading level={4}>子子标题</Heading>
          </Section>
        </Section>
      </Section>
    </Section>
  );
}
```

```js src/Section.js
export default function Section({ children }) {
  return (
    <section className="section">
      {children}
    </section>
  );
}
```

```js src/Heading.js
export default function Heading({ level, children }) {
  switch (level) {
    case 1:
      return <h1>{children}</h1>;
    case 2:
      return <h2>{children}</h2>;
    case 3:
      return <h3>{children}</h3>;
    case 4:
      return <h4>{children}</h4>;
    case 5:
      return <h5>{children}</h5>;
    case 6:
      return <h6>{children}</h6>;
    default:
      throw Error('未知的 level：' + level);
  }
}
```

```css
.section {
  padding: 10px;
  margin: 5px;
  border-radius: 5px;
  border: 1px solid #aaa;
}
```

</Sandpack>

Currently, you pass the `level` prop to each `<Heading>` separately:
目前，你将 `level` 参数分别传递给每个 `<Heading>`：

```js
<Section>
  <Heading level={3}>关于</Heading>
  <Heading level={3}>照片</Heading>
  <Heading level={3}>视频</Heading>
</Section>
```

It would be nice if you could pass the `level` prop to the `<Section>` component instead and remove it from the `<Heading>`. This way you could enforce that all headings in the same section have the same size:
将 `level` 参数传递给 `<Section>` 组件而不是传给 `<Heading>` 组件看起来更好一些。这样的话你可以强制使同一个 section 中的所有标题都有相同的尺寸：

```js
<Section level={3}>
  <Heading>关于</Heading>
  <Heading>照片</Heading>
  <Heading>视频</Heading>
</Section>
```

But how can the `<Heading>` component know the level of its closest `<Section>`? **That would require some way for a child to "ask" for data from somewhere above in the tree.**
但是 `<Heading>` 组件是如何知道离它最近的 `<Section>` 的 level 的呢？**这需要子组件可以通过某种方式“访问”到组件树中某处在其上层的数据。**

You can't do it with props alone. This is where context comes into play. You will do it in three steps:
你不能只通过 props 来实现它。这就是 context 大显身手的地方。你可以通过以下三个步骤来实现它：

1. **Create** a context. (You can call it `LevelContext`, since it's for the heading level.)
- **创建** 一个 context。（你可以将其命名为 `LevelContext`, 因为它表示的是标题级别。）
2. **Use** that context from the component that needs the data. (`Heading` will use `LevelContext`.)
- 在需要数据的组件内 **使用** 刚刚创建的 context。（`Heading` 将会使用 `LevelContext`。）
3. **Provide** that context from the component that specifies the data. (`Section` will provide `LevelContext`.)
- 在指定数据的组件中 **提供** 这个 context。 （`Section` 将会提供 `LevelContext`。）

Context lets a parent--even a distant one!--provide some data to the entire tree inside of it.
Context 可以让父节点，甚至是很远的父节点都可以为其内部的整个组件树提供数据。

<DiagramGroup>

<Diagram name="passing_data_context_close" height={160} width={608} captionPosition="top" alt="Diagram with a tree of three components. The parent contains a bubble representing a value highlighted in orange which projects down to the two children, each highlighted in orange." >

Using context in close children
同级子组件使用 context

</Diagram>

<Diagram name="passing_data_context_far" height={430} width={608} captionPosition="top" alt="Diagram with a tree of ten nodes, each node with two children or less. The root parent node contains a bubble representing a value highlighted in orange. The value projects down directly to four leaves and one intermediate component in the tree, which are all highlighted in orange. None of the other intermediate components are highlighted.">

Using context in distant children
远亲组件使用 context

</Diagram>

</DiagramGroup>

### Step 1：创建 context | Step 1: Create the context {/*step-1-create-the-context*/}

First, you need to create the context. You'll need to **export it from a file** so that your components can use it:
首先，你需要创建这个 context，并 **将其从一个文件中导出**，这样你的组件才可以使用它：

<Sandpack>

```js
import Heading from './Heading.js';
import Section from './Section.js';

export default function Page() {
  return (
    <Section>
      <Heading level={1}>主标题</Heading>
      <Section>
        <Heading level={2}>副标题</Heading>
        <Heading level={2}>副标题</Heading>
        <Heading level={2}>副标题</Heading>
        <Section>
          <Heading level={3}>子标题</Heading>
          <Heading level={3}>子标题</Heading>
          <Heading level={3}>子标题</Heading>
          <Section>
            <Heading level={4}>子子标题</Heading>
            <Heading level={4}>子子标题</Heading>
            <Heading level={4}>子子标题</Heading>
          </Section>
        </Section>
      </Section>
    </Section>
  );
}
```

```js src/Section.js
export default function Section({ children }) {
  return (
    <section className="section">
      {children}
    </section>
  );
}
```

```js src/Heading.js
export default function Heading({ level, children }) {
  switch (level) {
    case 1:
      return <h1>{children}</h1>;
    case 2:
      return <h2>{children}</h2>;
    case 3:
      return <h3>{children}</h3>;
    case 4:
      return <h4>{children}</h4>;
    case 5:
      return <h5>{children}</h5>;
    case 6:
      return <h6>{children}</h6>;
    default:
      throw Error('未知的 level：' + level);
  }
}
```

```js src/LevelContext.js active
import { createContext } from 'react';

export const LevelContext = createContext(1);
```

```css
.section {
  padding: 10px;
  margin: 5px;
  border-radius: 5px;
  border: 1px solid #aaa;
}
```

</Sandpack>

The only argument to `createContext` is the _default_ value. Here, `1` refers to the biggest heading level, but you could pass any kind of value (even an object). You will see the significance of the default value in the next step.
`createContext` 只需**默认值**这么一个参数。在这里, `1` 表示最大的标题级别，但是你可以传递任何类型的值（甚至可以传入一个对象）。你将在下一个步骤中见识到默认值的意义。

### Step 2：使用 Context | Step 2: Use the context {/*step-2-use-the-context*/}

Import the `useContext` Hook from React and your context:
从 React 中引入 `useContext` Hook 以及你刚刚创建的 context:

```js
import { useContext } from 'react';
import { LevelContext } from './LevelContext.js';
```

Currently, the `Heading` component reads `level` from props:
目前，`Heading` 组件从 props 中读取 `level`：

```js
export default function Heading({ level, children }) {
  // ...
}
```

Instead, remove the `level` prop and read the value from the context you just imported, `LevelContext`:
删掉 `level` 参数并从你刚刚引入的 `LevelContext` 中读取值：

```js {2}
export default function Heading({ children }) {
  const level = useContext(LevelContext);
  // ...
}
```

`useContext` is a Hook. Just like `useState` and `useReducer`, you can only call a Hook immediately inside a React component (not inside loops or conditions). **`useContext` tells React that the `Heading` component wants to read the `LevelContext`.**
`useContext` 是一个 Hook。和 `useState` 以及 `useReducer`一样，你只能在 React 组件中（不是循环或者条件里）立即调用 Hook。**`useContext` 告诉 React `Heading` 组件想要读取 `LevelContext`。**

Now that the `Heading` component doesn't have a `level` prop, you don't need to pass the level prop to `Heading` in your JSX like this anymore:
现在 `Heading` 组件没有 `level` 参数，你不需要再像这样在你的 JSX 中将 level 参数传递给 `Heading`：

```js
<Section>
  <Heading level={4}>子子标题</Heading>
  <Heading level={4}>子子标题</Heading>
  <Heading level={4}>子子标题</Heading>
</Section>
```

修改一下 JSX，让 `Section` 组件代替 `Heading` 组件接收 level 参数：

```jsx
<Section level={4}>
  <Heading>子子标题</Heading>
  <Heading>子子标题</Heading>
  <Heading>子子标题</Heading>
</Section>
```

你将修改下边的代码直到它正常运行：

<Sandpack>

```js
import Heading from './Heading.js';
import Section from './Section.js';

export default function Page() {
  return (
    <Section level={1}>
      <Heading>主标题</Heading>
      <Section level={2}>
        <Heading>副标题</Heading>
        <Heading>副标题</Heading>
        <Heading>副标题</Heading>
        <Section level={3}>
          <Heading>子标题</Heading>
          <Heading>子标题</Heading>
          <Heading>子标题</Heading>
          <Section level={4}>
            <Heading>子子标题</Heading>
            <Heading>子子标题</Heading>
            <Heading>子子标题</Heading>
          </Section>
        </Section>
      </Section>
    </Section>
  );
}
```

```js src/Section.js
export default function Section({ children }) {
  return (
    <section className="section">
      {children}
    </section>
  );
}
```

```js src/Heading.js
import { useContext } from 'react';
import { LevelContext } from './LevelContext.js';

export default function Heading({ children }) {
  const level = useContext(LevelContext);
  switch (level) {
    case 1:
      return <h1>{children}</h1>;
    case 2:
      return <h2>{children}</h2>;
    case 3:
      return <h3>{children}</h3>;
    case 4:
      return <h4>{children}</h4>;
    case 5:
      return <h5>{children}</h5>;
    case 6:
      return <h6>{children}</h6>;
    default:
      throw Error('未知的 level：' + level);
  }
}
```

```js src/LevelContext.js
import { createContext } from 'react';

export const LevelContext = createContext(1);
```

```css
.section {
  padding: 10px;
  margin: 5px;
  border-radius: 5px;
  border: 1px solid #aaa;
}
```

</Sandpack>

Notice this example doesn't quite work, yet! All the headings have the same size because **even though you're *using* the context, you have not *provided* it yet.** React doesn't know where to get it!
注意！这个示例还不能运行。所有 headings 的尺寸都一样，因为 **即使你正在使用 context，但是你还没有提供它。** React 不知道从哪里获取这个 context！

If you don't provide the context, React will use the default value you've specified in the previous step. In this example, you specified `1` as the argument to `createContext`, so `useContext(LevelContext)` returns `1`, setting all those headings to `<h1>`. Let's fix this problem by having each `Section` provide its own context.
如果你不提供 context，React 会使用你在上一步指定的默认值。在这个例子中，你为 `createContext` 传入了 `1` 这个参数，所以 `useContext(LevelContext)` 会返回 `1`，把所有的标题都设置为`<h1>`。我们通过让每个 `Section` 提供它自己的 context 来修复这个问题。

### Step 3：提供 context | Step 3: Provide the context {/*step-3-provide-the-context*/}

The `Section` component currently renders its children:
`Section` 组件目前渲染传入它的子组件：

```js
export default function Section({ children }) {
  return (
    <section className="section">
      {children}
    </section>
  );
}
```

**Wrap them with a context provider** to provide the `LevelContext` to them:
**把它们用 context provider 包裹起来** 以提供 `LevelContext` 给它们：

```js {1,6,8}
import { LevelContext } from './LevelContext.js';

export default function Section({ level, children }) {
  return (
    <section className="section">
      <LevelContext.Provider value={level}>
        {children}
      </LevelContext.Provider>
    </section>
  );
}
```

This tells React: "if any component inside this `<Section>` asks for `LevelContext`, give them this `level`." The component will use the value of the nearest `<LevelContext.Provider>` in the UI tree above it.
这告诉 React：“如果在 `<Section>` 组件中的任何子组件请求 `LevelContext`，给他们这个 `level`。”组件会使用 UI 树中在它上层最近的那个 `<LevelContext.Provider>` 传递过来的值。

<Sandpack>

```js
import Heading from './Heading.js';
import Section from './Section.js';

export default function Page() {
  return (
    <Section level={1}>
      <Heading>主标题</Heading>
      <Section level={2}>
        <Heading>副标题</Heading>
        <Heading>副标题</Heading>
        <Heading>副标题</Heading>
        <Section level={3}>
          <Heading>子标题</Heading>
          <Heading>子标题</Heading>
          <Heading>子标题</Heading>
          <Section level={4}>
            <Heading>子子标题</Heading>
            <Heading>子子标题</Heading>
            <Heading>子子标题</Heading>
          </Section>
        </Section>
      </Section>
    </Section>
  );
}
```

```js src/Section.js
import { LevelContext } from './LevelContext.js';

export default function Section({ level, children }) {
  return (
    <section className="section">
      <LevelContext.Provider value={level}>
        {children}
      </LevelContext.Provider>
    </section>
  );
}
```

```js src/Heading.js
import { useContext } from 'react';
import { LevelContext } from './LevelContext.js';

export default function Heading({ children }) {
  const level = useContext(LevelContext);
  switch (level) {
    case 1:
      return <h1>{children}</h1>;
    case 2:
      return <h2>{children}</h2>;
    case 3:
      return <h3>{children}</h3>;
    case 4:
      return <h4>{children}</h4>;
    case 5:
      return <h5>{children}</h5>;
    case 6:
      return <h6>{children}</h6>;
    default:
      throw Error('未知的 level：' + level);
  }
}
```

```js src/LevelContext.js
import { createContext } from 'react';

export const LevelContext = createContext(1);
```

```css
.section {
  padding: 10px;
  margin: 5px;
  border-radius: 5px;
  border: 1px solid #aaa;
}
```

</Sandpack>

It's the same result as the original code, but you did not need to pass the `level` prop to each `Heading` component! Instead, it "figures out" its heading level by asking the closest `Section` above:
这与原始代码的运行结果相同，但是你不需要向每个 `Heading` 组件传递 `level` 参数了！取而代之的是，它通过访问上层最近的 `Section` 来“断定”它的标题级别：

1. You pass a `level` prop to the `<Section>`.
- 你将一个 `level` 参数传递给 `<Section>`。
2. `Section` wraps its children into `<LevelContext.Provider value={level}>`.
- `Section` 把它的子元素包在 `<LevelContext.Provider value={level}>` 里面。
3. `Heading` asks the closest value of `LevelContext` above with `useContext(LevelContext)`.
- `Heading` 使用 `useContext(LevelContext)` 访问上层最近的 `LevelContext` 提供的值。

## 在相同的组件中使用并提供 context | Using and providing context from the same component {/*using-and-providing-context-from-the-same-component*/}

Currently, you still have to specify each section's `level` manually:
目前，你仍需要手动指定每个 section 的 `level`：

```js
export default function Page() {
  return (
    <Section level={1}>
      ...
      <Section level={2}>
        ...
        <Section level={3}>
          ...
```

Since context lets you read information from a component above, each `Section` could read the `level` from the `Section` above, and pass `level + 1` down automatically. Here is how you could do it:
由于 context 让你可以从上层的组件读取信息，每个 `Section` 都会从上层的 `Section` 读取 `level`，并自动向下层传递 `level + 1`。
你可以像下面这样做：

```js src/Section.js {5,8}
import { useContext } from 'react';
import { LevelContext } from './LevelContext.js';

export default function Section({ children }) {
  const level = useContext(LevelContext);
  return (
    <section className="section">
      <LevelContext.Provider value={level + 1}>
        {children}
      </LevelContext.Provider>
    </section>
  );
}
```

With this change, you don't need to pass the `level` prop *either* to the `<Section>` or to the `<Heading>`:
这样修改之后，你不用将 `level` 参数传给 `<Section>` **或者是** `<Heading>` 了：

<Sandpack>

```js
import Heading from './Heading.js';
import Section from './Section.js';

export default function Page() {
  return (
    <Section>
      <Heading>主标题</Heading>
      <Section>
        <Heading>副标题</Heading>
        <Heading>副标题</Heading>
        <Heading>副标题</Heading>
        <Section>
          <Heading>子标题</Heading>
          <Heading>子标题</Heading>
          <Heading>子标题</Heading>
          <Section>
            <Heading>子子标题</Heading>
            <Heading>子子标题</Heading>
            <Heading>子子标题</Heading>
          </Section>
        </Section>
      </Section>
    </Section>
  );
}
```

```js src/Section.js
import { useContext } from 'react';
import { LevelContext } from './LevelContext.js';

export default function Section({ children }) {
  const level = useContext(LevelContext);
  return (
    <section className="section">
      <LevelContext.Provider value={level + 1}>
        {children}
      </LevelContext.Provider>
    </section>
  );
}
```

```js src/Heading.js
import { useContext } from 'react';
import { LevelContext } from './LevelContext.js';

export default function Heading({ children }) {
  const level = useContext(LevelContext);
  switch (level) {
    case 0:
      throw Error('Heading 必须在 Section 内部！');
    case 1:
      return <h1>{children}</h1>;
    case 2:
      return <h2>{children}</h2>;
    case 3:
      return <h3>{children}</h3>;
    case 4:
      return <h4>{children}</h4>;
    case 5:
      return <h5>{children}</h5>;
    case 6:
      return <h6>{children}</h6>;
    default:
      throw Error('未知的 level：' + level);
  }
}
```

```js src/LevelContext.js
import { createContext } from 'react';

export const LevelContext = createContext(0);
```

```css
.section {
  padding: 10px;
  margin: 5px;
  border-radius: 5px;
  border: 1px solid #aaa;
}
```

</Sandpack>

Now both `Heading` and `Section` read the `LevelContext` to figure out how "deep" they are. And the `Section` wraps its children into the `LevelContext` to specify that anything inside of it is at a "deeper" level.
现在，`Heading` 和 `Section` 都通过读取 `LevelContext` 来判断它们的深度。而且 `Section` 把它的子组件都包在 `LevelContext` 中来指定其中的任何内容都处于一个“更深”的级别。

<Note>

This example uses heading levels because they show visually how nested components can override context. But context is useful for many other use cases too. You can pass down any information needed by the entire subtree: the current color theme, the currently logged in user, and so on.
本示例使用标题级别来展示，因为它们直观地显示了嵌套组件如何覆盖 context。但是 context 对于许多其他的场景也很有用。你可以用它来传递整个子树需要的任何信息：当前的颜色主题、当前登录的用户等。

</Note>

## Context 会穿过中间层级的组件 | Context passes through intermediate components {/*context-passes-through-intermediate-components*/}

You can insert as many components as you like between the component that provides context and the one that uses it. This includes both built-in components like `<div>` and components you might build yourself.
你可以在提供 context 的组件和使用它的组件之间的层级插入任意数量的组件。这包括像 `<div>` 这样的内置组件和你自己创建的组件。

In this example, the same `Post` component (with a dashed border) is rendered at two different nesting levels. Notice that the `<Heading>` inside of it gets its level automatically from the closest `<Section>`:
在这个示例中，相同的 `Post` 组件（带有虚线边框）在两个不同的嵌套层级上渲染。注意，它内部的 `<Heading>` 会自动从最近的 `<Section>` 获取它的级别：

<Sandpack>

```js
import Heading from './Heading.js';
import Section from './Section.js';

export default function ProfilePage() {
  return (
    <Section>
      <Heading>My Profile</Heading>
      <Post
        title="旅行者，你好！"
        body="来看看我的冒险。"
      />
      <AllPosts />
    </Section>
  );
}

function AllPosts() {
  return (
    <Section>
      <Heading>帖子</Heading>
      <RecentPosts />
    </Section>
  );
}

function RecentPosts() {
  return (
    <Section>
      <Heading>最近的帖子</Heading>
      <Post
        title="里斯本的味道"
        body="...那些蛋挞！"
      />
      <Post
        title="探戈节奏中的布宜诺斯艾利斯"
        body="我爱它！"
      />
    </Section>
  );
}

function Post({ title, body }) {
  return (
    <Section isFancy={true}>
      <Heading>
        {title}
      </Heading>
      <p><i>{body}</i></p>
    </Section>
  );
}
```

```js src/Section.js
import { useContext } from 'react';
import { LevelContext } from './LevelContext.js';

export default function Section({ children, isFancy }) {
  const level = useContext(LevelContext);
  return (
    <section className={
      'section ' +
      (isFancy ? 'fancy' : '')
    }>
      <LevelContext.Provider value={level + 1}>
        {children}
      </LevelContext.Provider>
    </section>
  );
}
```

```js src/Heading.js
import { useContext } from 'react';
import { LevelContext } from './LevelContext.js';

export default function Heading({ children }) {
  const level = useContext(LevelContext);
  switch (level) {
    case 0:
      throw Error('Heading 必须在 Section 内部！');
    case 1:
      return <h1>{children}</h1>;
    case 2:
      return <h2>{children}</h2>;
    case 3:
      return <h3>{children}</h3>;
    case 4:
      return <h4>{children}</h4>;
    case 5:
      return <h5>{children}</h5>;
    case 6:
      return <h6>{children}</h6>;
    default:
      throw Error('未知的 level：' + level);
  }
}
```

```js src/LevelContext.js
import { createContext } from 'react';

export const LevelContext = createContext(0);
```

```css
.section {
  padding: 10px;
  margin: 5px;
  border-radius: 5px;
  border: 1px solid #aaa;
}

.fancy {
  border: 4px dashed pink;
}
```

</Sandpack>

You didn't do anything special for this to work. A `Section` specifies the context for the tree inside it, so you can insert a `<Heading>` anywhere, and it will have the correct size. Try it in the sandbox above!
你不需要做任何特殊的操作。`Section` 为它内部的树指定一个 context，所以你可以在任何地方插入一个 `<Heading>`，而且它会有正确的尺寸。在上边的沙箱中尝试一下！

**Context lets you write components that "adapt to their surroundings" and display themselves differently depending on _where_ (or, in other words, _in which context_) they are being rendered.**
**Context 让你可以编写“适应周围环境”的组件，并且根据 在哪 （或者说 在哪个 context 中）来渲染它们不同的样子。**

How context works might remind you of [CSS property inheritance.](https://developer.mozilla.org/en-US/docs/Web/CSS/inheritance) In CSS, you can specify `color: blue` for a `<div>`, and any DOM node inside of it, no matter how deep, will inherit that color unless some other DOM node in the middle overrides it with `color: green`. Similarly, in React, the only way to override some context coming from above is to wrap children into a context provider with a different value.
Context 的工作方式可能会让你想起 [CSS 属性继承](https://developer.mozilla.org/zh-CN/docs/Web/CSS/inheritance)。在 CSS 中，你可以为一个 `<div>` 手动指定 `color: blue`，并且其中的任何 DOM 节点，无论多深，都会继承那个颜色，除非中间的其他 DOM 节点用 `color: green` 来覆盖它。类似地，在 React 中，覆盖来自上层的某些 context 的唯一方法是将子组件包裹到一个提供不同值的 context provider 中。

In CSS, different properties like `color` and `background-color` don't override each other. You can set all  `<div>`'s `color` to red without impacting `background-color`. Similarly, **different React contexts don't override each other.** Each context that you make with `createContext()` is completely separate from other ones, and ties together components using and providing *that particular* context. One component may use or provide many different contexts without a problem.
在 CSS 中，诸如 `color` 和 `background-color` 之类的不同属性不会覆盖彼此。你可以设置所有 `<div>` 的 `color` 为红色，而不会影响 `background-color`。类似地，**不同的 React context 不会覆盖彼此**。你通过 `createContext()` 创建的每个 context 都和其他 context 完全分离，只有使用和提供 **那个特定的** context 的组件才会联系在一起。一个组件可以轻松地使用或者提供许多不同的 context。

## 写在你使用 context 之前 | Before you use context {/*before-you-use-context*/}

Context is very tempting to use! However, this also means it's too easy to overuse it. **Just because you need to pass some props several levels deep doesn't mean you should put that information into context.**
使用 Context 看起来非常诱人！然而，这也意味着它也太容易被过度使用了。**如果你只想把一些 props 传递到多个层级中，这并不意味着你需要把这些信息放到 context 里。**

Here's a few alternatives you should consider before using context:
在使用 context 之前，你可以考虑以下几种替代方案：

1. **Start by [passing props.](/learn/passing-props-to-a-component)** If your components are not trivial, it's not unusual to pass a dozen props down through a dozen components. It may feel like a slog, but it makes it very clear which components use which data! The person maintaining your code will be glad you've made the data flow explicit with props.
- **从 [传递 props](/learn/passing-props-to-a-component) 开始。** 如果你的组件看起来不起眼，那么通过十几个组件向下传递一堆 props 并不罕见。这有点像是在埋头苦干，但是这样做可以让哪些组件用了哪些数据变得十分清晰！维护你代码的人会很高兴你用 props 让数据流变得更加清晰。
2. **Extract components and [pass JSX as `children`](/learn/passing-props-to-a-component#passing-jsx-as-children) to them.** If you pass some data through many layers of intermediate components that don't use that data (and only pass it further down), this often means that you forgot to extract some components along the way. For example, maybe you pass data props like `posts` to visual components that don't use them directly, like `<Layout posts={posts} />`. Instead, make `Layout` take `children` as a prop, and render `<Layout><Posts posts={posts} /></Layout>`. This reduces the number of layers between the component specifying the data and the one that needs it.
- **抽象组件并 [将 JSX 作为 `children` 传递](/learn/passing-props-to-a-component#passing-jsx-as-children) 给它们。** 如果你通过很多层不使用该数据的中间组件（并且只会向下传递）来传递数据，这通常意味着你在此过程中忘记了抽象组件。举个例子，你可能想传递一些像 `posts` 的数据 props 到不会直接使用这个参数的组件，类似 `<Layout posts={posts} />`。取而代之的是，让 `Layout` 把 `children` 当做一个参数，然后渲染 `<Layout><Posts posts={posts} /></Layout>`。这样就减少了定义数据的组件和使用数据的组件之间的层级。

If neither of these approaches works well for you, consider context.
如果这两种方法都不适合你，再考虑使用 context。

## Context 的使用场景 | Use cases for context {/*use-cases-for-context*/}

* **Theming:** If your app lets the user change its appearance (e.g. dark mode), you can put a context provider at the top of your app, and use that context in components that need to adjust their visual look.
* **主题：** 如果你的应用允许用户更改其外观（例如暗夜模式），你可以在应用顶层放一个 context provider，并在需要调整其外观的组件中使用该 context。
* **Current account:** Many components might need to know the currently logged in user. Putting it in context makes it convenient to read it anywhere in the tree. Some apps also let you operate multiple accounts at the same time (e.g. to leave a comment as a different user). In those cases, it can be convenient to wrap a part of the UI into a nested provider with a different current account value.
* **当前账户：** 许多组件可能需要知道当前登录的用户信息。将它放到 context 中可以方便地在树中的任何位置读取它。某些应用还允许你同时操作多个账户（例如，以不同用户的身份发表评论）。在这些情况下，将 UI 的一部分包裹到具有不同账户数据的 provider 中会很方便。
* **Routing:** Most routing solutions use context internally to hold the current route. This is how every link "knows" whether it's active or not. If you build your own router, you might want to do it too.
* **路由：** 大多数路由解决方案在其内部使用 context 来保存当前路由。这就是每个链接“知道”它是否处于活动状态的方式。如果你创建自己的路由库，你可能也会这么做。
* **Managing state:** As your app grows, you might end up with a lot of state closer to the top of your app. Many distant components below may want to change it. It is common to [use a reducer together with context](/learn/scaling-up-with-reducer-and-context) to manage complex state and pass it down to distant components without too much hassle.
* **状态管理：** 随着你的应用的增长，最终在靠近应用顶部的位置可能会有很多 state。许多遥远的下层组件可能想要修改它们。通常 [将 reducer 与 context 搭配使用](/learn/scaling-up-with-reducer-and-context)来管理复杂的状态并将其传递给深层的组件来避免过多的麻烦。
  
Context is not limited to static values. If you pass a different value on the next render, React will update all the components reading it below! This is why context is often used in combination with state.
Context 不局限于静态值。如果你在下一次渲染时传递不同的值，React 将会更新读取它的所有下层组件！这就是 context 经常和 state 结合使用的原因。

In general, if some information is needed by distant components in different parts of the tree, it's a good indication that context will help you.
一般而言，如果树中不同部分的远距离组件需要某些信息，context 将会对你大有帮助。

<Recap>

* Context lets a component provide some information to the entire tree below it.
* Context 使组件向其下方的整个树提供信息。
* To pass context:
* 传递 Context 的方法:
  1. Create and export it with `export const MyContext = createContext(defaultValue)`.
  - 通过 `export const MyContext = createContext(defaultValue)` 创建并导出 context。
  2. Pass it to the `useContext(MyContext)` Hook to read it in any child component, no matter how deep.
  - 在无论层级多深的任何子组件中，把 context 传递给 `useContext(MyContext)` Hook 来读取它。
  3. Wrap children into `<MyContext.Provider value={...}>` to provide it from a parent.
  - 在父组件中把 children 包在 `<MyContext.Provider value={...}>` 中来提供 context。
* Context passes through any components in the middle.
* Context 会穿过中间的任何组件。
* Context lets you write components that "adapt to their surroundings".
* Context 可以让你写出 “较为通用” 的组件。
* Before you use context, try passing props or passing JSX as `children`.
* 在使用 context 之前，先试试传递 props 或者将 JSX 作为 `children` 传递。

</Recap>

<Challenges>

#### 用 context 替代逐层 props | Replace prop drilling with context {/*replace-prop-drilling-with-context*/}

In this example, toggling the checkbox changes the `imageSize` prop passed to each `<PlaceImage>`. The checkbox state is held in the top-level `App` component, but each `<PlaceImage>` needs to be aware of it.
在这个示例中，切换复选框状态会修改传入每个 `<PlaceImage>` 的 `imageSize` 参数。复选框的 state 保存在顶层的 `App` 组件中，但是每个 `<PlaceImage>` 都需要注意它。

Currently, `App` passes `imageSize` to `List`, which passes it to each `Place`, which passes it to the `PlaceImage`. Remove the `imageSize` prop, and instead pass it from the `App` component directly to `PlaceImage`.
目前，`App` 将 `imageSize` 传递给 `List`，`List` 再将其传递给每个 `Place`，`Place` 又将其传递给 `PlaceImage`。移除 `imageSize` 参数，并在 `App` 组件中直接将其传递给 `PlaceImage`。

You can declare context in `Context.js`.
你可以在 `Context.js` 中声明 context。

<Sandpack>

```js src/App.js
import { useState } from 'react';
import { places } from './data.js';
import { getImageUrl } from './utils.js';

export default function App() {
  const [isLarge, setIsLarge] = useState(false);
  const imageSize = isLarge ? 150 : 100;
  return (
    <>
      <label>
        <input
          type="checkbox"
          checked={isLarge}
          onChange={e => {
            setIsLarge(e.target.checked);
          }}
        />
        Use large images
      </label>
      <hr />
      <List imageSize={imageSize} />
    </>
  )
}

function List({ imageSize }) {
  const listItems = places.map(place =>
    <li key={place.id}>
      <Place
        place={place}
        imageSize={imageSize}
      />
    </li>
  );
  return <ul>{listItems}</ul>;
}

function Place({ place, imageSize }) {
  return (
    <>
      <PlaceImage
        place={place}
        imageSize={imageSize}
      />
      <p>
        <b>{place.name}</b>
        {': ' + place.description}
      </p>
    </>
  );
}

function PlaceImage({ place, imageSize }) {
  return (
    <img
      src={getImageUrl(place)}
      alt={place.name}
      width={imageSize}
      height={imageSize}
    />
  );
}
```

```js src/Context.js

```

```js src/data.js
export const places = [{
  id: 0,
  name: '南非开普敦的波卡普区',
  description: '为房屋选择亮色的传统始于 20 世纪后期。',
  imageId: 'K9HVAGH'
}, {
  id: 1, 
  name: '中国台湾省台中市彩虹村',
  description: '1924 年，当地居民黄永福为了避免拆迁，将 1200 间房屋全部粉刷。',
  imageId: '9EAYZrt'
}, {
  id: 2, 
  name: '墨西哥的帕丘卡宏观壁画',
  description: '世界上最大的壁画之一，覆盖了山坡上的房屋。',
  imageId: 'DgXHVwu'
}, {
  id: 3, 
  name: '巴西里约热内卢的塞拉龙楼梯',
  description: '这个地标由智利出生的艺术家 Jorge Selarón 创作，以“向巴西人民致敬”。',
  imageId: 'aeO3rpI'
}, {
  id: 4, 
  name: '意大利布拉诺',
  description: '这些房屋按照一个可追溯到 16 世纪的特定颜色系统进行粉刷。',
  imageId: 'kxsph5C'
}, {
  id: 5, 
  name: '摩洛哥舍夫沙万',
  description: '关于为什么房屋被涂成蓝色，有几种理论，包括这几种颜色可以驱蚊或者它象征着天空和天堂。',
  imageId: 'rTqKo46'
}, {
  id: 6,
  name: '韩国釜山甘川文化村',
  description: '2009 年，该村通过粉刷房屋并举办展览和艺术装置而转变为文化中心。',
  imageId: 'ZfQOOzf'
}];
```

```js src/utils.js
export function getImageUrl(place) {
  return (
    'https://i.imgur.com/' +
    place.imageId +
    'l.jpg'
  );
}
```

```css
ul { list-style-type: none; padding: 0px 10px; }
li { 
  margin-bottom: 10px; 
  display: grid; 
  grid-template-columns: auto 1fr;
  gap: 20px;
  align-items: center;
}
```

</Sandpack>

<Solution>

Remove `imageSize` prop from all the components.
移除掉所有组件中的 `imageSize` 参数。

Create and export `ImageSizeContext` from `Context.js`. Then wrap the List into `<ImageSizeContext.Provider value={imageSize}>` to pass the value down, and `useContext(ImageSizeContext)` to read it in the `PlaceImage`:
在 `Context.js` 中创建并导出 `ImageSizeContext`。然后用 `<ImageSizeContext.Provider value={imageSize}>` 包裹住整个列表来向下传递值，最后在 `PlaceImage` 中使用 `useContext(ImageSizeContext)` 来读取它。

<Sandpack>

```js src/App.js
import { useState, useContext } from 'react';
import { places } from './data.js';
import { getImageUrl } from './utils.js';
import { ImageSizeContext } from './Context.js';

export default function App() {
  const [isLarge, setIsLarge] = useState(false);
  const imageSize = isLarge ? 150 : 100;
  return (
    <ImageSizeContext.Provider
      value={imageSize}
    >
      <label>
        <input
          type="checkbox"
          checked={isLarge}
          onChange={e => {
            setIsLarge(e.target.checked);
          }}
        />
        Use large images
      </label>
      <hr />
      <List />
    </ImageSizeContext.Provider>
  )
}

function List() {
  const listItems = places.map(place =>
    <li key={place.id}>
      <Place place={place} />
    </li>
  );
  return <ul>{listItems}</ul>;
}

function Place({ place }) {
  return (
    <>
      <PlaceImage place={place} />
      <p>
        <b>{place.name}</b>
        {': ' + place.description}
      </p>
    </>
  );
}

function PlaceImage({ place }) {
  const imageSize = useContext(ImageSizeContext);
  return (
    <img
      src={getImageUrl(place)}
      alt={place.name}
      width={imageSize}
      height={imageSize}
    />
  );
}
```

```js src/Context.js
import { createContext } from 'react';

export const ImageSizeContext = createContext(500);
```

```js src/data.js
export const places = [{
  id: 0,
  name: '南非开普敦的波卡普区',
  description: '为房屋选择亮色的传统始于 20 世纪后期。',
  imageId: 'K9HVAGH'
}, {
  id: 1, 
  name: '中国台湾省台中市彩虹村',
  description: '1924 年，当地居民黄永福为了避免拆迁，将 1200 间房屋全部粉刷。',
  imageId: '9EAYZrt'
}, {
  id: 2, 
  name: '墨西哥的帕丘卡宏观壁画',
  description: '世界上最大的壁画之一，覆盖了山坡上的房屋。',
  imageId: 'DgXHVwu'
}, {
  id: 3, 
  name: '巴西里约热内卢的塞拉龙楼梯',
  description: '这个地标由智利出生的艺术家 Jorge Selarón 创作，以“向巴西人民致敬”。',
  imageId: 'aeO3rpI'
}, {
  id: 4, 
  name: '意大利布拉诺',
  description: '这些房屋按照一个可追溯到 16 世纪的特定颜色系统进行粉刷。',
  imageId: 'kxsph5C'
}, {
  id: 5, 
  name: '摩洛哥舍夫沙万',
  description: '关于为什么房屋被涂成蓝色，有几种理论，包括这几种颜色可以驱蚊或者它象征着天空和天堂。',
  imageId: 'rTqKo46'
}, {
  id: 6,
  name: '韩国釜山甘川文化村',
  description: '2009 年，该村通过粉刷房屋并举办展览和艺术装置而转变为文化中心。',
  imageId: 'ZfQOOzf'
}];
```

```js src/utils.js
export function getImageUrl(place) {
  return (
    'https://i.imgur.com/' +
    place.imageId +
    'l.jpg'
  );
}
```

```css
ul { list-style-type: none; padding: 0px 10px; }
li { 
  margin-bottom: 10px; 
  display: grid; 
  grid-template-columns: auto 1fr;
  gap: 20px;
  align-items: center;
}
```

</Sandpack>

Note how components in the middle don't need to pass `imageSize` anymore.
请注意中间的组件是怎样实现不用传入 `imageSize` 参数的。

</Solution>

</Challenges>
