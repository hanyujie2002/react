---
title: 对 state 进行保留和重置 | Preserving and Resetting State
translators:
  - YogaLin
  - Neo42
  - QC-L
---

<Intro>

State is isolated between components. React keeps track of which state belongs to which component based on their place in the UI tree. You can control when to preserve state and when to reset it between re-renders.
各个组件的 state 是各自独立的。根据组件在 UI 树中的位置，React 可以跟踪哪些 state 属于哪个组件。你可以控制在重新渲染过程中何时对 state 进行保留和重置。

</Intro>

<YouWillLearn>

* When React chooses to preserve or reset the state
* React 何时选择保留或重置状态
* How to force React to reset component's state
* 如何强制 React 重置组件的状态
* How keys and types affect whether the state is preserved
* 键和类型如何影响状态是否被保留

</YouWillLearn>

## 状态与渲染树中的位置相关 | State is tied to a position in the render tree {/*state-is-tied-to-a-position-in-the-tree*/}

React builds [render trees](learn/understanding-your-ui-as-a-tree#the-render-tree) for the component structure in your UI.
React 会为 UI 中的组件结构构建 [渲染树](learn/understanding-your-ui-as-a-tree#the-render-tree)。

When you give a component state, you might think the state "lives" inside the component. But the state is actually held inside React. React associates each piece of state it's holding with the correct component by where that component sits in the render tree.
当向一个组件添加状态时，那么可能会认为状态“存在”在组件内。但实际上，状态是由 React 保存的。React 通过组件在渲染树中的位置将它保存的每个状态与正确的组件关联起来。

Here, there is only one `<Counter />` JSX tag, but it's rendered at two different positions:
下面的例子中只有一个 `<Counter />` JSX 标签，但它会在两个不同的位置渲染：

<Sandpack>

```js
import { useState } from 'react';

export default function App() {
  const counter = <Counter />;
  return (
    <div>
      {counter}
      {counter}
    </div>
  );
}

function Counter() {
  const [score, setScore] = useState(0);
  const [hover, setHover] = useState(false);

  let className = 'counter';
  if (hover) {
    className += ' hover';
  }

  return (
    <div
      className={className}
      onPointerEnter={() => setHover(true)}
      onPointerLeave={() => setHover(false)}
    >
      <h1>{score}</h1>
      <button onClick={() => setScore(score + 1)}>
        加一
      </button>
    </div>
  );
}
```

```css
label {
  display: block;
  clear: both;
}

.counter {
  width: 100px;
  text-align: center;
  border: 1px solid gray;
  border-radius: 4px;
  padding: 20px;
  margin: 0 20px 20px 0;
  float: left;
}

.hover {
  background: #ffffd8;
}
```

</Sandpack>

Here's how these look as a tree:    
下面是它们的树形结构的样子：   

<DiagramGroup>

<Diagram name="preserving_state_tree" height={248} width={395} alt="Diagram of a tree of React components. The root node is labeled 'div' and has two children. Each of the children are labeled 'Counter' and both contain a state bubble labeled 'count' with value 0.">

React tree
React 树

</Diagram>

</DiagramGroup>

**These are two separate counters because each is rendered at its own position in the tree.** You don't usually have to think about these positions to use React, but it can be useful to understand how it works.
**这是两个独立的 counter，因为它们在树中被渲染在了各自的位置。** 一般情况下你不用去考虑这些位置来使用 React，但知道它们是如何工作会很有用。

In React, each component on the screen has fully isolated state. For example, if you render two `Counter` components side by side, each of them will get its own, independent, `score` and `hover` states.
在 React 中，屏幕中的每个组件都有完全独立的 state。举个例子，当你并排渲染两个 `Counter` 组件时，它们都会拥有各自独立的 `score` 和 `hover` state。

Try clicking both counters and notice they don't affect each other:
试试点击两个 counter 你会发现它们互不影响：

<Sandpack>

```js
import { useState } from 'react';

export default function App() {
  return (
    <div>
      <Counter />
      <Counter />
    </div>
  );
}

function Counter() {
  const [score, setScore] = useState(0);
  const [hover, setHover] = useState(false);

  let className = 'counter';
  if (hover) {
    className += ' hover';
  }

  return (
    <div
      className={className}
      onPointerEnter={() => setHover(true)}
      onPointerLeave={() => setHover(false)}
    >
      <h1>{score}</h1>
      <button onClick={() => setScore(score + 1)}>
        加一
      </button>
    </div>
  );
}
```

```css
.counter {
  width: 100px;
  text-align: center;
  border: 1px solid gray;
  border-radius: 4px;
  padding: 20px;
  margin: 0 20px 20px 0;
  float: left;
}

.hover {
  background: #ffffd8;
}
```

</Sandpack>

As you can see, when one counter is updated, only the state for that component is updated:
如你所见，当一个计数器被更新时，只有该组件的状态会被更新：


<DiagramGroup>

<Diagram name="preserving_state_increment" height={248} width={441} alt="Diagram of a tree of React components. The root node is labeled 'div' and has two children. The left child is labeled 'Counter' and contains a state bubble labeled 'count' with value 0. The right child is labeled 'Counter' and contains a state bubble labeled 'count' with value 1. The state bubble of the right child is highlighted in yellow to indicate its value has updated.">

Updating state

</Diagram>

</DiagramGroup>


React will keep the state around for as long as you render the same component at the same position in the tree. To see this, increment both counters, then remove the second component by unchecking "Render the second counter" checkbox, and then add it back by ticking it again:
只有当在树中相同的位置渲染相同的组件时，React 才会一直保留着组件的 state。想要验证这一点，可以将两个计数器的值递增，取消勾选 “渲染第二个计数器” 复选框，然后再次勾选它：

<Sandpack>

```js
import { useState } from 'react';

export default function App() {
  const [showB, setShowB] = useState(true);
  return (
    <div>
      <Counter />
      {showB && <Counter />} 
      <label>
        <input
          type="checkbox"
          checked={showB}
          onChange={e => {
            setShowB(e.target.checked)
          }}
        />
        渲染第二个计数器
      </label>
    </div>
  );
}

function Counter() {
  const [score, setScore] = useState(0);
  const [hover, setHover] = useState(false);

  let className = 'counter';
  if (hover) {
    className += ' hover';
  }

  return (
    <div
      className={className}
      onPointerEnter={() => setHover(true)}
      onPointerLeave={() => setHover(false)}
    >
      <h1>{score}</h1>
      <button onClick={() => setScore(score + 1)}>
        加一
      </button>
    </div>
  );
}
```

```css
label {
  display: block;
  clear: both;
}

.counter {
  width: 100px;
  text-align: center;
  border: 1px solid gray;
  border-radius: 4px;
  padding: 20px;
  margin: 0 20px 20px 0;
  float: left;
}

.hover {
  background: #ffffd8;
}
```

</Sandpack>

Notice how the moment you stop rendering the second counter, its state disappears completely. That's because when React removes a component, it destroys its state.
注意，当你停止渲染第二个计数器的那一刻，它的 state 完全消失了。这是因为 React 在移除一个组件时，也会销毁它的 state。

<DiagramGroup>

<Diagram name="preserving_state_remove_component" height={253} width={422} alt="Diagram of a tree of React components. The root node is labeled 'div' and has two children. The left child is labeled 'Counter' and contains a state bubble labeled 'count' with value 0. The right child is missing, and in its place is a yellow 'poof' image, highlighting the component being deleted from the tree.">

删除组件

</Diagram>

</DiagramGroup>

When you tick "Render the second counter", a second `Counter` and its state are initialized from scratch (`score = 0`) and added to the DOM.
当你重新勾选“渲染第二个计数器”复选框时，另一个计数器及其 state 将从头开始初始化（`score = 0`）并被添加到 DOM 中。

<DiagramGroup>

<Diagram name="preserving_state_add_component" height={258} width={500} alt="Diagram of a tree of React components. The root node is labeled 'div' and has two children. The left child is labeled 'Counter' and contains a state bubble labeled 'count' with value 0. The right child is labeled 'Counter' and contains a state bubble labeled 'count' with value 0. The entire right child node is highlighted in yellow, indicating that it was just added to the tree.">

Adding a component
添加组件

</Diagram>

</DiagramGroup>

**React preserves a component's state for as long as it's being rendered at its position in the UI tree.** If it gets removed, or a different component gets rendered at the same position, React discards its state.
**只要一个组件还被渲染在 UI 树的相同位置，React 就会保留它的 state。** 如果它被移除，或者一个不同的组件被渲染在相同的位置，那么 React 就会丢掉它的 state。

## 相同位置的相同组件会使得 state 被保留下来 | Same component at the same position preserves state {/*same-component-at-the-same-position-preserves-state*/}

In this example, there are two different `<Counter />` tags:
在这个例子中，有两个不同的 `<Counter />` 标签：

<Sandpack>

```js
import { useState } from 'react';

export default function App() {
  const [isFancy, setIsFancy] = useState(false);
  return (
    <div>
      {isFancy ? (
        <Counter isFancy={true} /> 
      ) : (
        <Counter isFancy={false} /> 
      )}
      <label>
        <input
          type="checkbox"
          checked={isFancy}
          onChange={e => {
            setIsFancy(e.target.checked)
          }}
        />
        使用好看的样式
      </label>
    </div>
  );
}

function Counter({ isFancy }) {
  const [score, setScore] = useState(0);
  const [hover, setHover] = useState(false);

  let className = 'counter';
  if (hover) {
    className += ' hover';
  }
  if (isFancy) {
    className += ' fancy';
  }

  return (
    <div
      className={className}
      onPointerEnter={() => setHover(true)}
      onPointerLeave={() => setHover(false)}
    >
      <h1>{score}</h1>
      <button onClick={() => setScore(score + 1)}>
        加一
      </button>
    </div>
  );
}
```

```css
label {
  display: block;
  clear: both;
}

.counter {
  width: 100px;
  text-align: center;
  border: 1px solid gray;
  border-radius: 4px;
  padding: 20px;
  margin: 0 20px 20px 0;
  float: left;
}

.fancy {
  border: 5px solid gold;
  color: #ff6767;
}

.hover {
  background: #ffffd8;
}
```

</Sandpack>

When you tick or clear the checkbox, the counter state does not get reset. Whether `isFancy` is `true` or `false`, you always have a `<Counter />` as the first child of the `div` returned from the root `App` component:
当你勾选或清空复选框的时候，计数器 state 并没有被重置。不管 `isFancy` 是 `true` 还是 `false`，根组件 `App` 返回的 `div` 的第一个子组件都是 `<Counter />`：

<DiagramGroup>

<Diagram name="preserving_state_same_component" height={461} width={600} alt="Diagram with two sections separated by an arrow transitioning between them. Each section contains a layout of components with a parent labeled 'App' containing a state bubble labeled isFancy. This component has one child labeled 'div', which leads to a prop bubble containing isFancy (highlighted in purple) passed down to the only child. The last child is labeled 'Counter' and contains a state bubble with label 'count' and value 3 in both diagrams. In the left section of the diagram, nothing is highlighted and the isFancy parent state value is false. In the right section of the diagram, the isFancy parent state value has changed to true and it is highlighted in yellow, and so is the props bubble below, which has also changed its isFancy value to true.">

Updating the `App` state does not reset the `Counter` because `Counter` stays in the same position
更新 `App` 的状态不会重置 `Counter`，因为 `Counter` 始终保持在同一位置。

</Diagram>

</DiagramGroup>


It's the same component at the same position, so from React's perspective, it's the same counter.
它是位于相同位置的相同组件，所以对 React 来说，它是同一个计数器。

<Pitfall>

Remember that **it's the position in the UI tree--not in the JSX markup--that matters to React!** This component has two `return` clauses with different `<Counter />` JSX tags inside and outside the `if`:
记住 **对 React 来说重要的是组件在 UI 树中的位置,而不是在 JSX 中的位置！** 这个组件在 `if` 内外有两个`return` 语句，它们带有不同的 `<Counter />` JSX 标签：

<Sandpack>

```js
import { useState } from 'react';

export default function App() {
  const [isFancy, setIsFancy] = useState(false);
  if (isFancy) {
    return (
      <div>
        <Counter isFancy={true} />
        <label>
          <input
            type="checkbox"
            checked={isFancy}
            onChange={e => {
              setIsFancy(e.target.checked)
            }}
          />
          使用好看的样式
        </label>
      </div>
    );
  }
  return (
    <div>
      <Counter isFancy={false} />
      <label>
        <input
          type="checkbox"
          checked={isFancy}
          onChange={e => {
            setIsFancy(e.target.checked)
          }}
        />
        使用好看的样式
      </label>
    </div>
  );
}

function Counter({ isFancy }) {
  const [score, setScore] = useState(0);
  const [hover, setHover] = useState(false);

  let className = 'counter';
  if (hover) {
    className += ' hover';
  }
  if (isFancy) {
    className += ' fancy';
  }

  return (
    <div
      className={className}
      onPointerEnter={() => setHover(true)}
      onPointerLeave={() => setHover(false)}
    >
      <h1>{score}</h1>
      <button onClick={() => setScore(score + 1)}>
        加一
      </button>
    </div>
  );
}
```

```css
label {
  display: block;
  clear: both;
}

.counter {
  width: 100px;
  text-align: center;
  border: 1px solid gray;
  border-radius: 4px;
  padding: 20px;
  margin: 0 20px 20px 0;
  float: left;
}

.fancy {
  border: 5px solid gold;
  color: #ff6767;
}

.hover {
  background: #ffffd8;
}
```

</Sandpack>

You might expect the state to reset when you tick checkbox, but it doesn't! This is because **both of these `<Counter />` tags are rendered at the same position.** React doesn't know where you place the conditions in your function. All it "sees" is the tree you return.
你可能以为当你勾选复选框的时候 state 会被重置，但它并没有！这是因为 **两个 `<Counter />` 标签被渲染在了相同的位置。** React 不知道你的函数里是如何进行条件判断的，它只会“看到”你返回的树。

In both cases, the `App` component returns a `<div>` with `<Counter />` as a first child. To React, these two counters have the same "address": the first child of the first child of the root. This is how React matches them up between the previous and next renders, regardless of how you structure your logic.
在这两种情况下，`App` 组件都会返回一个包裹着 `<Counter />` 作为第一个子组件的 `div`。你可以认为它们有相同的“地址”：根组件的第一个子组件的第一个子组件。不管你的逻辑是怎么组织的，这就是 React 在前后两次渲染之间将它们进行匹配的方式。

</Pitfall>

## 相同位置的不同组件会使 state 重置 | Different components at the same position reset state {/*different-components-at-the-same-position-reset-state*/}

In this example, ticking the checkbox will replace `<Counter>` with a `<p>`:
在这个例子中，勾选复选框会将 `<Counter>` 替换为一个 `<p>`：

<Sandpack>

```js
import { useState } from 'react';

export default function App() {
  const [isPaused, setIsPaused] = useState(false);
  return (
    <div>
      {isPaused ? (
        <p>待会见！</p> 
      ) : (
        <Counter /> 
      )}
      <label>
        <input
          type="checkbox"
          checked={isPaused}
          onChange={e => {
            setIsPaused(e.target.checked)
          }}
        />
        休息一下
      </label>
    </div>
  );
}

function Counter() {
  const [score, setScore] = useState(0);
  const [hover, setHover] = useState(false);

  let className = 'counter';
  if (hover) {
    className += ' hover';
  }

  return (
    <div
      className={className}
      onPointerEnter={() => setHover(true)}
      onPointerLeave={() => setHover(false)}
    >
      <h1>{score}</h1>
      <button onClick={() => setScore(score + 1)}>
        加一
      </button>
    </div>
  );
}
```

```css
label {
  display: block;
  clear: both;
}

.counter {
  width: 100px;
  text-align: center;
  border: 1px solid gray;
  border-radius: 4px;
  padding: 20px;
  margin: 0 20px 20px 0;
  float: left;
}

.hover {
  background: #ffffd8;
}
```

</Sandpack>

Here, you switch between _different_ component types at the same position. Initially, the first child of the `<div>` contained a `Counter`. But when you swapped in a `p`, React removed the `Counter` from the UI tree and destroyed its state.
示例中，你在相同位置对 **不同** 的组件类型进行切换。刚开始 `<div>` 的第一个子组件是一个 `Counter`。但是当你切换成 `p` 时，React 将 `Counter` 从 UI 树中移除了并销毁了它的状态。

<DiagramGroup>

<Diagram name="preserving_state_diff_pt1" height={290} width={753} alt="Diagram with three sections, with an arrow transitioning each section in between. The first section contains a React component labeled 'div' with a single child labeled 'Counter' containing a state bubble labeled 'count' with value 3. The middle section has the same 'div' parent, but the child component has now been deleted, indicated by a yellow 'proof' image. The third section has the same 'div' parent again, now with a new child labeled 'p', highlighted in yellow.">

When `Counter` changes to `p`, the `Counter` is deleted and the `p` is added
当 `Counter` 变为 `p` 时，`Counter` 会被移除，同时 `p` 被添加。

</Diagram>

</DiagramGroup>

<DiagramGroup>

<Diagram name="preserving_state_diff_pt2" height={290} width={753} alt="Diagram with three sections, with an arrow transitioning each section in between. The first section contains a React component labeled 'p'. The middle section has the same 'div' parent, but the child component has now been deleted, indicated by a yellow 'proof' image. The third section has the same 'div' parent again, now with a new child labeled 'Counter' containing a state bubble labeled 'count' with value 0, highlighted in yellow.">

When switching back, the `p` is deleted and the `Counter` is added
当切换回来时，`p` 会被删除，而 `Counter` 会被添加

</Diagram>

</DiagramGroup>

Also, **when you render a different component in the same position, it resets the state of its entire subtree.** To see how this works, increment the counter and then tick the checkbox:
并且，**当你在相同位置渲染不同的组件时，组件的整个子树都会被重置**。要验证这一点，可以增加计数器的值然后勾选复选框：

<Sandpack>

```js
import { useState } from 'react';

export default function App() {
  const [isFancy, setIsFancy] = useState(false);
  return (
    <div>
      {isFancy ? (
        <div>
          <Counter isFancy={true} /> 
        </div>
      ) : (
        <section>
          <Counter isFancy={false} />
        </section>
      )}
      <label>
        <input
          type="checkbox"
          checked={isFancy}
          onChange={e => {
            setIsFancy(e.target.checked)
          }}
        />
        使用好看的样式
      </label>
    </div>
  );
}

function Counter({ isFancy }) {
  const [score, setScore] = useState(0);
  const [hover, setHover] = useState(false);

  let className = 'counter';
  if (hover) {
    className += ' hover';
  }
  if (isFancy) {
    className += ' fancy';
  }

  return (
    <div
      className={className}
      onPointerEnter={() => setHover(true)}
      onPointerLeave={() => setHover(false)}
    >
      <h1>{score}</h1>
      <button onClick={() => setScore(score + 1)}>
        加一
      </button>
    </div>
  );
}
```

```css
label {
  display: block;
  clear: both;
}

.counter {
  width: 100px;
  text-align: center;
  border: 1px solid gray;
  border-radius: 4px;
  padding: 20px;
  margin: 0 20px 20px 0;
  float: left;
}

.fancy {
  border: 5px solid gold;
  color: #ff6767;
}

.hover {
  background: #ffffd8;
}
```

</Sandpack>

The counter state gets reset when you click the checkbox. Although you render a `Counter`, the first child of the `div` changes from a `div` to a `section`. When the child `div` was removed from the DOM, the whole tree below it (including the `Counter` and its state) was destroyed as well.
当你勾选复选框后计数器的 state 被重置了。虽然你渲染了一个 `Counter`，但是 `div` 的第一个子组件从 `div` 变成了 `section`。当子组件 `div` 从 DOM 中被移除的时候，它底下的整棵树（包含 `Counter` 以及它的 state）也都被销毁了。

<DiagramGroup>

<Diagram name="preserving_state_diff_same_pt1" height={350} width={794} alt="Diagram with three sections, with an arrow transitioning each section in between. The first section contains a React component labeled 'div' with a single child labeled 'section', which has a single child labeled 'Counter' containing a state bubble labeled 'count' with value 3. The middle section has the same 'div' parent, but the child components have now been deleted, indicated by a yellow 'proof' image. The third section has the same 'div' parent again, now with a new child labeled 'div', highlighted in yellow, also with a new child labeled 'Counter' containing a state bubble labeled 'count' with value 0, all highlighted in yellow.">

When `section` changes to `div`, the `section` is deleted and the new `div` is added
当 `section` 变为 `div` 时，`section` 会被删除，新的 `div` 被添加

</Diagram>

</DiagramGroup>

<DiagramGroup>

<Diagram name="preserving_state_diff_same_pt2" height={350} width={794} alt="Diagram with three sections, with an arrow transitioning each section in between. The first section contains a React component labeled 'div' with a single child labeled 'div', which has a single child labeled 'Counter' containing a state bubble labeled 'count' with value 0. The middle section has the same 'div' parent, but the child components have now been deleted, indicated by a yellow 'proof' image. The third section has the same 'div' parent again, now with a new child labeled 'section', highlighted in yellow, also with a new child labeled 'Counter' containing a state bubble labeled 'count' with value 0, all highlighted in yellow.">

When switching back, the `div` is deleted and the new `section` is added
当切换回来时，`div` 会被删除，新的 `section` 被添加

</Diagram>

</DiagramGroup>

As a rule of thumb, **if you want to preserve the state between re-renders, the structure of your tree needs to "match up"** from one render to another. If the structure is different, the state gets destroyed because React destroys state when it removes a component from the tree.
一般来说，**如果你想在重新渲染时保留 state，几次渲染中的树形结构就应该相互“匹配”**。结构不同就会导致 state 的销毁，因为 React 会在将一个组件从树中移除时销毁它的 state。

<Pitfall>

This is why you should not nest component function definitions.
以下是你不应该把组件函数的定义嵌套起来的原因。

Here, the `MyTextField` component function is defined *inside* `MyComponent`:
示例中， `MyTextField` 组件被定义在 `MyComponent` **内部**：

<Sandpack>

```js
import { useState } from 'react';

export default function MyComponent() {
  const [counter, setCounter] = useState(0);

  function MyTextField() {
    const [text, setText] = useState('');

    return (
      <input
        value={text}
        onChange={e => setText(e.target.value)}
      />
    );
  }

  return (
    <>
      <MyTextField />
      <button onClick={() => {
        setCounter(counter + 1)
      }}>点击了 {counter} 次</button>
    </>
  );
}
```

</Sandpack>


Every time you click the button, the input state disappears! This is because a *different* `MyTextField` function is created for every render of `MyComponent`. You're rendering a *different* component in the same position, so React resets all state below. This leads to bugs and performance problems. To avoid this problem, **always declare component functions at the top level, and don't nest their definitions.**
每次你点击按钮，输入框的 state 都会消失！这是因为每次 `MyComponent` 渲染时都会创建一个 **不同** 的 `MyTextField` 函数。你在相同位置渲染的是 **不同** 的组件，所以 React 将其下所有的 state 都重置了。这样会导致 bug 以及性能问题。为了避免这个问题， **永远要将组件定义在最上层并且不要把它们的定义嵌套起来。**

</Pitfall>

## 在相同位置重置 state | Resetting state at the same position {/*resetting-state-at-the-same-position*/}

By default, React preserves state of a component while it stays at the same position. Usually, this is exactly what you want, so it makes sense as the default behavior. But sometimes, you may want to reset a component's state. Consider this app that lets two players keep track of their scores during each turn:
默认情况下，React 会在一个组件保持在同一位置时保留它的 state。通常这就是你想要的，所以把它作为默认特性很合理。但有时候，你可能想要重置一个组件的 state。考虑一下这个应用，它可以让两个玩家在每个回合中记录他们的得分：

<Sandpack>

```js
import { useState } from 'react';

export default function Scoreboard() {
  const [isPlayerA, setIsPlayerA] = useState(true);
  return (
    <div>
      {isPlayerA ? (
        <Counter person="Taylor" />
      ) : (
        <Counter person="Sarah" />
      )}
      <button onClick={() => {
        setIsPlayerA(!isPlayerA);
      }}>
        下一位玩家！
      </button>
    </div>
  );
}

function Counter({ person }) {
  const [score, setScore] = useState(0);
  const [hover, setHover] = useState(false);

  let className = 'counter';
  if (hover) {
    className += ' hover';
  }

  return (
    <div
      className={className}
      onPointerEnter={() => setHover(true)}
      onPointerLeave={() => setHover(false)}
    >
      <h1>{person} 的分数：{score}</h1>
      <button onClick={() => setScore(score + 1)}>
        加一
      </button>
    </div>
  );
}
```

```css
h1 {
  font-size: 18px;
}

.counter {
  width: 100px;
  text-align: center;
  border: 1px solid gray;
  border-radius: 4px;
  padding: 20px;
  margin: 0 20px 20px 0;
}

.hover {
  background: #ffffd8;
}
```

</Sandpack>

Currently, when you change the player, the score is preserved. The two `Counter`s appear in the same position, so React sees them as *the same* `Counter` whose `person` prop has changed.
目前当你切换玩家时，分数会被保留下来。这两个 `Counter` 出现在相同的位置，所以 React 会认为它们是 **同一个** `Counter`，只是传了不同的 `person` prop。

But conceptually, in this app they should be two separate counters. They might appear in the same place in the UI, but one is a counter for Taylor, and another is a counter for Sarah.
但是从概念上讲，这个应用中的两个计数器应该是各自独立的。虽然它们在 UI 中的位置相同，但是一个是 Taylor 的计数器，一个是 Sarah 的计数器。

There are two ways to reset state when switching between them:
有两个方法可以在它们相互切换时重置 state：

1. Render components in different positions
- 将组件渲染在不同的位置
2. Give each component an explicit identity with `key`
- 使用 `key` 赋予每个组件一个明确的身份


### 方法一：将组件渲染在不同的位置 | Option 1: Rendering a component in different positions {/*option-1-rendering-a-component-in-different-positions*/}

If you want these two `Counter`s to be independent, you can render them in two different positions:
你如果想让两个 `Counter` 各自独立的话，可以将它们渲染在不同的位置：

<Sandpack>

```js
import { useState } from 'react';

export default function Scoreboard() {
  const [isPlayerA, setIsPlayerA] = useState(true);
  return (
    <div>
      {isPlayerA &&
        <Counter person="Taylor" />
      }
      {!isPlayerA &&
        <Counter person="Sarah" />
      }
      <button onClick={() => {
        setIsPlayerA(!isPlayerA);
      }}>
        下一位玩家！
      </button>
    </div>
  );
}

function Counter({ person }) {
  const [score, setScore] = useState(0);
  const [hover, setHover] = useState(false);

  let className = 'counter';
  if (hover) {
    className += ' hover';
  }

  return (
    <div
      className={className}
      onPointerEnter={() => setHover(true)}
      onPointerLeave={() => setHover(false)}
    >
      <h1>{person} 的分数：{score}</h1>
      <button onClick={() => setScore(score + 1)}>
        加一
      </button>
    </div>
  );
}
```

```css
h1 {
  font-size: 18px;
}

.counter {
  width: 100px;
  text-align: center;
  border: 1px solid gray;
  border-radius: 4px;
  padding: 20px;
  margin: 0 20px 20px 0;
}

.hover {
  background: #ffffd8;
}
```

</Sandpack>

* Initially, `isPlayerA` is `true`. So the first position contains `Counter` state, and the second one is empty.
* 起初 `isPlayerA` 的值是 `true`。所以第一个位置包含了 `Counter` 的 state，而第二个位置是空的。
* When you click the "Next player" button the first position clears but the second one now contains a `Counter`.
* 当你点击“下一位玩家”按钮时，第一个位置会被清空，而第二个位置现在包含了一个 `Counter`。

<DiagramGroup>

<Diagram name="preserving_state_diff_position_p1" height={375} width={504} alt="Diagram with a tree of React components. The parent is labeled 'Scoreboard' with a state bubble labeled isPlayerA with value 'true'. The only child, arranged to the left, is labeled Counter with a state bubble labeled 'count' and value 0. All of the left child is highlighted in yellow, indicating it was added.">

Initial state

</Diagram>

<Diagram name="preserving_state_diff_position_p2" height={375} width={504} alt="Diagram with a tree of React components. The parent is labeled 'Scoreboard' with a state bubble labeled isPlayerA with value 'false'. The state bubble is highlighted in yellow, indicating that it has changed. The left child is replaced with a yellow 'poof' image indicating that it has been deleted and there is a new child on the right, highlighted in yellow indicating that it was added. The new child is labeled 'Counter' and contains a state bubble labeled 'count' with value 0.">

Clicking "next"

</Diagram>

<Diagram name="preserving_state_diff_position_p3" height={375} width={504} alt="Diagram with a tree of React components. The parent is labeled 'Scoreboard' with a state bubble labeled isPlayerA with value 'true'. The state bubble is highlighted in yellow, indicating that it has changed. There is a new child on the left, highlighted in yellow indicating that it was added. The new child is labeled 'Counter' and contains a state bubble labeled 'count' with value 0. The right child is replaced with a yellow 'poof' image indicating that it has been deleted.">

Clicking "next" again

</Diagram>

</DiagramGroup>

Each `Counter`'s state gets destroyed each time it's removed from the DOM. This is why they reset every time you click the button.
每当 `Counter` 组件从 DOM 中移除时，它的 state 会被销毁。这就是每次点击按钮它们就会被重置的原因。

This solution is convenient when you only have a few independent components rendered in the same place. In this example, you only have two, so it's not a hassle to render both separately in the JSX.
这个解决方案在你只有少数几个独立的组件渲染在相同的位置时会很方便。这个例子中只有 2 个组件，所以在 JSX 里将它们分开进行渲染并不麻烦。

### 方法二：使用 key 来重置 state | Option 2: Resetting state with a key {/*option-2-resetting-state-with-a-key*/}

There is also another, more generic, way to reset a component's state.
还有另一种更通用的重置组件 state 的方法。

You might have seen `key`s when [rendering lists.](/learn/rendering-lists#keeping-list-items-in-order-with-key) Keys aren't just for lists! You can use keys to make React distinguish between any components. By default, React uses order within the parent ("first counter", "second counter") to discern between components. But keys let you tell React that this is not just a *first* counter, or a *second* counter, but a specific counter--for example, *Taylor's* counter. This way, React will know *Taylor's* counter wherever it appears in the tree!
你可能在 [渲染列表](/learn/rendering-lists#keeping-list-items-in-order-with-key) 时见到过 `key`。但 key 不只可以用于列表！你可以使用 key 来让 React 区分任何组件。默认情况下，React 使用父组件内部的顺序（“第一个计数器”、“第二个计数器”）来区分组件。但是 key 可以让你告诉 React 这不仅仅是 **第一个** 或者 **第二个** 计数器，而且还是一个特定的计数器——例如，**Taylor 的** 计数器。这样无论它出现在树的任何位置， React 都会知道它是 **Taylor 的** 计数器！

In this example, the two `<Counter />`s don't share state even though they appear in the same place in JSX:
在这个例子中，即使两个 `<Counter />` 会出现在 JSX 中的同一个位置，它们也不会共享 state：

<Sandpack>

```js
import { useState } from 'react';

export default function Scoreboard() {
  const [isPlayerA, setIsPlayerA] = useState(true);
  return (
    <div>
      {isPlayerA ? (
        <Counter key="Taylor" person="Taylor" />
      ) : (
        <Counter key="Sarah" person="Sarah" />
      )}
      <button onClick={() => {
        setIsPlayerA(!isPlayerA);
      }}>
        下一位玩家！
      </button>
    </div>
  );
}

function Counter({ person }) {
  const [score, setScore] = useState(0);
  const [hover, setHover] = useState(false);

  let className = 'counter';
  if (hover) {
    className += ' hover';
  }

  return (
    <div
      className={className}
      onPointerEnter={() => setHover(true)}
      onPointerLeave={() => setHover(false)}
    >
      <h1>{person} 的分数：{score}</h1>
      <button onClick={() => setScore(score + 1)}>
        加一
      </button>
    </div>
  );
}
```

```css
h1 {
  font-size: 18px;
}

.counter {
  width: 100px;
  text-align: center;
  border: 1px solid gray;
  border-radius: 4px;
  padding: 20px;
  margin: 0 20px 20px 0;
}

.hover {
  background: #ffffd8;
}
```

</Sandpack>

Switching between Taylor and Sarah does not preserve the state. This is because **you gave them different `key`s:**
在 Taylor 和 Sarah 之间切换不会使 state 被保留下来。因为 **你给他们赋了不同的 `key`：**

```js
{isPlayerA ? (
  <Counter key="Taylor" person="Taylor" />
) : (
  <Counter key="Sarah" person="Sarah" />
)}
```

Specifying a `key` tells React to use the `key` itself as part of the position, instead of their order within the parent. This is why, even though you render them in the same place in JSX, React sees them as two different counters, and so they will never share state. Every time a counter appears on the screen, its state is created. Every time it is removed, its state is destroyed. Toggling between them resets their state over and over.
指定一个 `key` 能够让 React 将 `key` 本身而非它们在父组件中的顺序作为位置的一部分。这就是为什么尽管你用 JSX 将组件渲染在相同位置，但在 React 看来它们是两个不同的计数器。因此它们永远都不会共享 state。每当一个计数器出现在屏幕上时，它的 state 会被创建出来。每当它被移除时，它的 state 就会被销毁。在它们之间切换会一次又一次地使它们的 state 重置。

<Note>

Remember that keys are not globally unique. They only specify the position *within the parent*.
请记住 key 不是全局唯一的。它们只能指定 **父组件内部** 的顺序。

</Note>

### 使用 key 重置表单 | Resetting a form with a key {/*resetting-a-form-with-a-key*/}

Resetting state with a key is particularly useful when dealing with forms.
使用 key 来重置 state 在处理表单时特别有用。

In this chat app, the `<Chat>` component contains the text input state:
在这个聊天应用中， `<Chat>` 组件包含文本输入 state：

<Sandpack>

```js src/App.js
import { useState } from 'react';
import Chat from './Chat.js';
import ContactList from './ContactList.js';

export default function Messenger() {
  const [to, setTo] = useState(contacts[0]);
  return (
    <div>
      <ContactList
        contacts={contacts}
        selectedContact={to}
        onSelect={contact => setTo(contact)}
      />
      <Chat contact={to} />
    </div>
  )
}

const contacts = [
  { id: 0, name: 'Taylor', email: 'taylor@mail.com' },
  { id: 1, name: 'Alice', email: 'alice@mail.com' },
  { id: 2, name: 'Bob', email: 'bob@mail.com' }
];
```

```js src/ContactList.js
export default function ContactList({
  selectedContact,
  contacts,
  onSelect
}) {
  return (
    <section className="contact-list">
      <ul>
        {contacts.map(contact =>
          <li key={contact.id}>
            <button onClick={() => {
              onSelect(contact);
            }}>
              {contact.name}
            </button>
          </li>
        )}
      </ul>
    </section>
  );
}
```

```js src/Chat.js
import { useState } from 'react';

export default function Chat({ contact }) {
  const [text, setText] = useState('');
  return (
    <section className="chat">
      <textarea
        value={text}
        placeholder={'跟 ' + contact.name + ' 聊一聊'}
        onChange={e => setText(e.target.value)}
      />
      <br />
      <button>发送到 {contact.email}</button>
    </section>
  );
}
```

```css
.chat, .contact-list {
  float: left;
  margin-bottom: 20px;
}
ul, li {
  list-style: none;
  margin: 0;
  padding: 0;
}
li button {
  width: 100px;
  padding: 10px;
  margin-right: 10px;
}
textarea {
  height: 150px;
}
```

</Sandpack>

Try entering something into the input, and then press "Alice" or "Bob" to choose a different recipient. You will notice that the input state is preserved because the `<Chat>` is rendered at the same position in the tree.
尝试在输入框中输入一些内容，然后点击 “Alice” 或 “Bob” 来选择不同的收件人。你会发现因为 `<Chat>` 被渲染在了树的相同位置，输入框的 state 被保留下来了。

**In many apps, this may be the desired behavior, but not in a chat app!** You don't want to let the user send a message they already typed to a wrong person due to an accidental click. To fix it, add a `key`:
**在很多应用里这可能会是大家所需要的特性，但在这个聊天应用里并不是！** 你不应该让用户因为一次偶然的点击而把他们已经输入的信息发送给一个错误的人。要修复这个问题，只需给组件添加一个 `key` ：

```js
<Chat key={to.id} contact={to} />
```

This ensures that when you select a different recipient, the `Chat` component will be recreated from scratch, including any state in the tree below it. React will also re-create the DOM elements instead of reusing them.
这样确保了当你选择一个不同的收件人时， `Chat` 组件——包括其下方树中的任何 state——都将从头开始重新创建。 React 还将重新创建 DOM 元素，而不是复用它们。

Now switching the recipient always clears the text field:
现在切换收件人就总会清除文本字段了：

<Sandpack>

```js src/App.js
import { useState } from 'react';
import Chat from './Chat.js';
import ContactList from './ContactList.js';

export default function Messenger() {
  const [to, setTo] = useState(contacts[0]);
  return (
    <div>
      <ContactList
        contacts={contacts}
        selectedContact={to}
        onSelect={contact => setTo(contact)}
      />
      <Chat key={to.id} contact={to} />
    </div>
  )
}

const contacts = [
  { id: 0, name: 'Taylor', email: 'taylor@mail.com' },
  { id: 1, name: 'Alice', email: 'alice@mail.com' },
  { id: 2, name: 'Bob', email: 'bob@mail.com' }
];
```

```js src/ContactList.js
export default function ContactList({
  selectedContact,
  contacts,
  onSelect
}) {
  return (
    <section className="contact-list">
      <ul>
        {contacts.map(contact =>
          <li key={contact.id}>
            <button onClick={() => {
              onSelect(contact);
            }}>
              {contact.name}
            </button>
          </li>
        )}
      </ul>
    </section>
  );
}
```

```js src/Chat.js
import { useState } from 'react';

export default function Chat({ contact }) {
  const [text, setText] = useState('');
  return (
    <section className="chat">
      <textarea
        value={text}
        placeholder={'跟 ' + contact.name + ' 聊一聊'}
        onChange={e => setText(e.target.value)}
      />
      <br />
      <button>发送到 {contact.email}</button>
    </section>
  );
}
```

```css
.chat, .contact-list {
  float: left;
  margin-bottom: 20px;
}
ul, li {
  list-style: none;
  margin: 0;
  padding: 0;
}
li button {
  width: 100px;
  padding: 10px;
  margin-right: 10px;
}
textarea {
  height: 150px;
}
```

</Sandpack>

<DeepDive>

#### 为被移除的组件保留 state | Preserving state for removed components {/*preserving-state-for-removed-components*/}

In a real chat app, you'd probably want to recover the input state when the user selects the previous recipient again. There are a few ways to keep the state "alive" for a component that's no longer visible:
在真正的聊天应用中，你可能会想在用户再次选择前一个收件人时恢复输入 state。对于一个不可见的组件，有几种方法可以让它的 state “活下去”：

- You could render _all_ chats instead of just the current one, but hide all the others with CSS. The chats would not get removed from the tree, so their local state would be preserved. This solution works great for simple UIs. But it can get very slow if the hidden trees are large and contain a lot of DOM nodes.
- 与其只渲染现在这一个聊天，你可以把 **所有** 聊天都渲染出来，但用 CSS 把其他聊天隐藏起来。这些聊天就不会从树中被移除了，所以它们的内部 state 会被保留下来。这种解决方法对于简单 UI 非常有效。但如果要隐藏的树形结构很大且包含了大量的 DOM 节点，那么性能就会变得很差。
- You could [lift the state up](/learn/sharing-state-between-components) and hold the pending message for each recipient in the parent component. This way, when the child components get removed, it doesn't matter, because it's the parent that keeps the important information. This is the most common solution.
- 你可以进行 [状态提升](/learn/sharing-state-between-components) 并在父组件中保存每个收件人的草稿消息。这样即使子组件被移除了也无所谓，因为保留重要信息的是父组件。这是最常见的解决方法。
- You might also use a different source in addition to React state. For example, you probably want a message draft to persist even if the user accidentally closes the page. To implement this, you could have the `Chat` component initialize its state by reading from the [`localStorage`](https://developer.mozilla.org/en-US/docs/Web/API/Window/localStorage), and save the drafts there too.
- 除了 React 的 state，你也可以使用其他数据源。例如，也许你希望即使用户不小心关闭页面也可以保存一份信息草稿。要实现这一点，你可以让 `Chat` 组件通过读取 [`localStorage`](https://developer.mozilla.org/zh-CN/docs/Web/API/Window/localStorage) 对其 state 进行初始化，并把草稿保存在那里。

No matter which strategy you pick, a chat _with Alice_ is conceptually distinct from a chat _with Bob_, so it makes sense to give a `key` to the `<Chat>` tree based on the current recipient.
无论采取哪种策略，**与 Alice** 的聊天在概念上都不同于 **与 Bob** 的聊天，因此根据当前收件人为 `<Chat>` 树指定一个 `key` 是合理的。

</DeepDive>

<Recap>

- React keeps state for as long as the same component is rendered at the same position.
- 只要在相同位置渲染的是相同组件， React 就会保留状态。
- State is not kept in JSX tags. It's associated with the tree position in which you put that JSX.
- state 不会被保存在 JSX 标签里。它与你在树中放置该 JSX 的位置相关联。
- You can force a subtree to reset its state by giving it a different key.
- 你可以通过为一个子树指定一个不同的 key 来重置它的 state。
- Don't nest component definitions, or you'll reset state by accident.
- 不要嵌套组件的定义，否则你会意外地导致 state 被重置。


</Recap>



<Challenges>

#### 修复丢失的输入框文本 | Fix disappearing input text {/*fix-disappearing-input-text*/}

This example shows a message when you press the button. However, pressing the button also accidentally resets the input. Why does this happen? Fix it so that pressing the button does not reset the input text.
这个例子在你按下按钮时会展示一条消息，但同时也会意外地重置输入框。为什么会发生这种情况？修复它，使按下按钮不再导致输入框文本重置。

<Sandpack>

```js src/App.js
import { useState } from 'react';

export default function App() {
  const [showHint, setShowHint] = useState(false);
  if (showHint) {
    return (
      <div>
        <p><i>提示：你最喜欢的城市？</i></p>
        <Form />
        <button onClick={() => {
          setShowHint(false);
        }}>隐藏提示</button>
      </div>
    );
  }
  return (
    <div>
      <Form />
      <button onClick={() => {
        setShowHint(true);
      }}>显示提示</button>
    </div>
  );
}

function Form() {
  const [text, setText] = useState('');
  return (
    <textarea
      value={text}
      onChange={e => setText(e.target.value)}
    />
  );
}
```

```css
textarea { display: block; margin: 10px 0; }
```

</Sandpack>

<Solution>

The problem is that `Form` is rendered in different positions. In the `if` branch, it is the second child of the `<div>`, but in the `else` branch, it is the first child. Therefore, the component type in each position changes. The first position changes between holding a `p` and a `Form`, while the second position changes between holding a `Form` and a `button`. React resets the state every time the component type changes.
问题在于 `Form` 被渲染在了不同的位置。在 `if` 分支里，`Form` 是 `<div>` 的第二个子组件，但在 `else` 分支里它是第一个子组件。所以相同位置的组件类型发生了变化。第一个位置时而包含一个 `p`，时而包含一个 `Form`；而第二个位置时而包含一个 `Form`，时而包含一个 `button`。每当组件类型发生变化时， React 都会重置 state。

The easiest solution is to unify the branches so that `Form` always renders in the same position:
最简单的解决方案是将各个分支合并，这样 `Form` 就总会在相同位置渲染：

<Sandpack>

```js src/App.js
import { useState } from 'react';

export default function App() {
  const [showHint, setShowHint] = useState(false);
  return (
    <div>
      {showHint &&
        <p><i>提示：你最喜欢的城市？</i></p>
      }
      <Form />
      {showHint ? (
        <button onClick={() => {
          setShowHint(false);
        }}>隐藏提示</button>
      ) : (
        <button onClick={() => {
          setShowHint(true);
        }}>显示提示</button>
      )}
    </div>
  );
}

function Form() {
  const [text, setText] = useState('');
  return (
    <textarea
      value={text}
      onChange={e => setText(e.target.value)}
    />
  );
}
```

```css
textarea { display: block; margin: 10px 0; }
```

</Sandpack>


Technically, you could also add `null` before `<Form />` in the `else` branch to match the `if` branch structure:
其实，你还可以在 `else` 分支的 `<Form />` 之前加个 `null`，以匹配 `if` 分支的结构：

<Sandpack>

```js src/App.js
import { useState } from 'react';

export default function App() {
  const [showHint, setShowHint] = useState(false);
  if (showHint) {
    return (
      <div>
        <p><i>提示：你最喜欢的城市？</i></p>
        <Form />
        <button onClick={() => {
          setShowHint(false);
        }}>隐藏提示</button>
      </div>
    );
  }
  return (
    <div>
      {null}
      <Form />
      <button onClick={() => {
        setShowHint(true);
      }}>显示提示</button>
    </div>
  );
}

function Form() {
  const [text, setText] = useState('');
  return (
    <textarea
      value={text}
      onChange={e => setText(e.target.value)}
    />
  );
}
```

```css
textarea { display: block; margin: 10px 0; }
```

</Sandpack>

This way, `Form` is always the second child, so it stays in the same position and keeps its state. But this approach is much less obvious and introduces a risk that someone else will remove that `null`.
这样 `Form` 就会始终是第二个子组件，所以它就会保持在相同位置并保留它的 state。但是这种方法特别不明显，并会引入一个风险因素——其他人可能会删除那个 `null`。

</Solution>

#### 交换两个表单字段 | Swap two form fields {/*swap-two-form-fields*/}

This form lets you enter first and last name. It also has a checkbox controlling which field goes first. When you tick the checkbox, the "Last name" field will appear before the "First name" field.
这个表单可以让你输入姓氏和名字。它还有一个复选框控制哪个字段放在前面。当你勾选复选框时，“姓氏”字段将出现在“名字”字段之前。

It almost works, but there is a bug. If you fill in the "First name" input and tick the checkbox, the text will stay in the first input (which is now "Last name"). Fix it so that the input text *also* moves when you reverse the order.
它几乎可以正常使用，但有一个 bug。如果你填写了“名字”输入框并勾选复选框，文本将保留在第一个输入框(也就是现在的“姓氏”)。修复它，使得输入框文本在你调换顺序时 **也** 会跟着移动。

<Hint>

It seems like for these fields, their position within the parent is not enough. Is there some way to tell React how to match up the state between re-renders?
似乎对于这两个字段来说，只知道它们在父组件中的位置并不足以实现功能。有没有办法告诉 React 如何匹配多次重新渲染中的 state？

</Hint>

<Sandpack>

```js src/App.js
import { useState } from 'react';

export default function App() {
  const [reverse, setReverse] = useState(false);
  let checkbox = (
    <label>
      <input
        type="checkbox"
        checked={reverse}
        onChange={e => setReverse(e.target.checked)}
      />
      调换顺序
    </label>
  );
  if (reverse) {
    return (
      <>
        <Field label="姓氏" /> 
        <Field label="名字" />
        {checkbox}
      </>
    );
  } else {
    return (
      <>
        <Field label="名字" /> 
        <Field label="姓氏" />
        {checkbox}
      </>
    );    
  }
}

function Field({ label }) {
  const [text, setText] = useState('');
  return (
    <label>
      {label}：
      <input
        type="text"
        value={text}
        placeholder={label}
        onChange={e => setText(e.target.value)}
      />
    </label>
  );
}
```

```css
label { display: block; margin: 10px 0; }
```

</Sandpack>

<Solution>

Give a `key` to both `<Field>` components in both `if` and `else` branches. This tells React how to "match up" the correct state for either `<Field>` even if their order within the parent changes:
为 `if` 和 `else` 分支中的两个 `<Field>` 组件都指定一个 `key`。这样可以告诉 React 如何为两个 `<Field>` “匹配”正确的状态——即使它们在父组件中的顺序会发生变化：

<Sandpack>

```js src/App.js
import { useState } from 'react';

export default function App() {
  const [reverse, setReverse] = useState(false);
  let checkbox = (
    <label>
      <input
        type="checkbox"
        checked={reverse}
        onChange={e => setReverse(e.target.checked)}
      />
      调换顺序
    </label>
  );
  if (reverse) {
    return (
      <>
        <Field key="lastName" label="姓氏" /> 
        <Field key="firstName" label="名字" />
        {checkbox}
      </>
    );
  } else {
    return (
      <>
        <Field key="firstName" label="名字" /> 
        <Field key="lastName" label="姓氏" />
        {checkbox}
      </>
    );    
  }
}

function Field({ label }) {
  const [text, setText] = useState('');
  return (
    <label>
      {label}：
      <input
        type="text"
        value={text}
        placeholder={label}
        onChange={e => setText(e.target.value)}
      />
    </label>
  );
}
```

```css
label { display: block; margin: 10px 0; }
```

</Sandpack>

</Solution>

#### 重置详情表单 | Reset a detail form {/*reset-a-detail-form*/}

This is an editable contact list. You can edit the selected contact's details and then either press "Save" to update it, or "Reset" to undo your changes.
这是一个可编辑的联系人列表。你可以编辑所选联系人的详细信息，然后点击“保存”进行更新或点击“重置”来撤消你的修改。

When you select a different contact (for example, Alice), the state updates but the form keeps showing the previous contact's details. Fix it so that the form gets reset when the selected contact changes.
当你选中另一个联系人（比如 Alice），状态就会更新，但表单会一直显示之前那个联系人的详细信息。修复它，使表单在所选联系人改变时重置。

<Sandpack>

```js src/App.js
import { useState } from 'react';
import ContactList from './ContactList.js';
import EditContact from './EditContact.js';

export default function ContactManager() {
  const [
    contacts,
    setContacts
  ] = useState(initialContacts);
  const [
    selectedId,
    setSelectedId
  ] = useState(0);
  const selectedContact = contacts.find(c =>
    c.id === selectedId
  );

  function handleSave(updatedData) {
    const nextContacts = contacts.map(c => {
      if (c.id === updatedData.id) {
        return updatedData;
      } else {
        return c;
      }
    });
    setContacts(nextContacts);
  }

  return (
    <div>
      <ContactList
        contacts={contacts}
        selectedId={selectedId}
        onSelect={id => setSelectedId(id)}
      />
      <hr />
      <EditContact
        initialData={selectedContact}
        onSave={handleSave}
      />
    </div>
  )
}

const initialContacts = [
  { id: 0, name: 'Taylor', email: 'taylor@mail.com' },
  { id: 1, name: 'Alice', email: 'alice@mail.com' },
  { id: 2, name: 'Bob', email: 'bob@mail.com' }
];
```

```js src/ContactList.js
export default function ContactList({
  contacts,
  selectedId,
  onSelect
}) {
  return (
    <section>
      <ul>
        {contacts.map(contact =>
          <li key={contact.id}>
            <button onClick={() => {
              onSelect(contact.id);
            }}>
              {contact.id === selectedId ?
                <b>{contact.name}</b> :
                contact.name
              }
            </button>
          </li>
        )}
      </ul>
    </section>
  );
}
```

```js src/EditContact.js
import { useState } from 'react';

export default function EditContact({ initialData, onSave }) {
  const [name, setName] = useState(initialData.name);
  const [email, setEmail] = useState(initialData.email);
  return (
    <section>
      <label>
        名称：
        <input
          type="text"
          value={name}
          onChange={e => setName(e.target.value)}
        />
      </label>
      <label>
        邮箱：
        <input
          type="email"
          value={email}
          onChange={e => setEmail(e.target.value)}
        />
      </label>
      <button onClick={() => {
        const updatedData = {
          id: initialData.id,
          name: name,
          email: email
        };
        onSave(updatedData);
      }}>
        保存
      </button>
      <button onClick={() => {
        setName(initialData.name);
        setEmail(initialData.email);
      }}>
        重置
      </button>
    </section>
  );
}
```

```css
ul, li {
  list-style: none;
  margin: 0;
  padding: 0;
}
li { display: inline-block; }
li button {
  padding: 10px;
}
label {
  display: block;
  margin: 10px 0;
}
button {
  margin-right: 10px;
  margin-bottom: 10px;
}
```

</Sandpack>

<Solution>

Give `key={selectedId}` to the `EditContact` component. This way, switching between different contacts will reset the form:
将 `key={selectedId}` 赋给 `EditContact` 组件。这样在不同联系人之间切换将重置表单：

<Sandpack>

```js src/App.js
import { useState } from 'react';
import ContactList from './ContactList.js';
import EditContact from './EditContact.js';

export default function ContactManager() {
  const [
    contacts,
    setContacts
  ] = useState(initialContacts);
  const [
    selectedId,
    setSelectedId
  ] = useState(0);
  const selectedContact = contacts.find(c =>
    c.id === selectedId
  );

  function handleSave(updatedData) {
    const nextContacts = contacts.map(c => {
      if (c.id === updatedData.id) {
        return updatedData;
      } else {
        return c;
      }
    });
    setContacts(nextContacts);
  }

  return (
    <div>
      <ContactList
        contacts={contacts}
        selectedId={selectedId}
        onSelect={id => setSelectedId(id)}
      />
      <hr />
      <EditContact
        key={selectedId}
        initialData={selectedContact}
        onSave={handleSave}
      />
    </div>
  )
}

const initialContacts = [
  { id: 0, name: 'Taylor', email: 'taylor@mail.com' },
  { id: 1, name: 'Alice', email: 'alice@mail.com' },
  { id: 2, name: 'Bob', email: 'bob@mail.com' }
];
```

```js src/ContactList.js
export default function ContactList({
  contacts,
  selectedId,
  onSelect
}) {
  return (
    <section>
      <ul>
        {contacts.map(contact =>
          <li key={contact.id}>
            <button onClick={() => {
              onSelect(contact.id);
            }}>
              {contact.id === selectedId ?
                <b>{contact.name}</b> :
                contact.name
              }
            </button>
          </li>
        )}
      </ul>
    </section>
  );
}
```

```js src/EditContact.js
import { useState } from 'react';

export default function EditContact({ initialData, onSave }) {
  const [name, setName] = useState(initialData.name);
  const [email, setEmail] = useState(initialData.email);
  return (
    <section>
      <label>
        名称：
        <input
          type="text"
          value={name}
          onChange={e => setName(e.target.value)}
        />
      </label>
      <label>
        邮箱：
        <input
          type="email"
          value={email}
          onChange={e => setEmail(e.target.value)}
        />
      </label>
      <button onClick={() => {
        const updatedData = {
          id: initialData.id,
          name: name,
          email: email
        };
        onSave(updatedData);
      }}>
        保存
      </button>
      <button onClick={() => {
        setName(initialData.name);
        setEmail(initialData.email);
      }}>
        重置
      </button>
    </section>
  );
}
```

```css
ul, li {
  list-style: none;
  margin: 0;
  padding: 0;
}
li { display: inline-block; }
li button {
  padding: 10px;
}
label {
  display: block;
  margin: 10px 0;
}
button {
  margin-right: 10px;
  margin-bottom: 10px;
}
```

</Sandpack>

</Solution>

#### 清除正在加载的图片 | Clear an image while it's loading {/*clear-an-image-while-its-loading*/}

When you press "Next", the browser starts loading the next image. However, because it's displayed in the same `<img>` tag, by default you would still see the previous image until the next one loads. This may be undesirable if it's important for the text to always match the image. Change it so that the moment you press "Next", the previous image immediately clears.
当你点击“下一张”时，浏览器会开始加载下一张图片。但因为它是在相同的 `<img>` 标签中显示的，所以默认情况下，你在下一张图片加载完成前都会看到上一张图片。如果文本必须始终与图片一一对应，那么这种特性可能并不是我们想要的。调整它使得上一张图片在你点击“下一张”时立即被清除。

<Hint>

Is there a way to tell React to re-create the DOM instead of reusing it?
有没有办法让 React 重新创建 DOM 而不是复用它？

</Hint>

<Sandpack>

```js
import { useState } from 'react';

export default function Gallery() {
  const [index, setIndex] = useState(0);
  const hasNext = index < images.length - 1;

  function handleClick() {
    if (hasNext) {
      setIndex(index + 1);
    } else {
      setIndex(0);
    }
  }

  let image = images[index];
  return (
    <>
      <button onClick={handleClick}>
        下一张
      </button>
      <h3>
        {images.length} 张图片中的第 {index + 1} 张
      </h3>
      <img src={image.src} />
      <p>
        {image.place}
      </p>
    </>
  );
}

let images = [{
  place: 'Penang, Malaysia',
  src: 'https://i.imgur.com/FJeJR8M.jpg'
}, {
  place: 'Lisbon, Portugal',
  src: 'https://i.imgur.com/dB2LRbj.jpg'
}, {
  place: 'Bilbao, Spain',
  src: 'https://i.imgur.com/z08o2TS.jpg'
}, {
  place: 'Valparaíso, Chile',
  src: 'https://i.imgur.com/Y3utgTi.jpg'
}, {
  place: 'Schwyz, Switzerland',
  src: 'https://i.imgur.com/JBbMpWY.jpg'
}, {
  place: 'Prague, Czechia',
  src: 'https://i.imgur.com/QwUKKmF.jpg'
}, {
  place: 'Ljubljana, Slovenia',
  src: 'https://i.imgur.com/3aIiwfm.jpg'
}];
```

```css
img { width: 150px; height: 150px; }
```

</Sandpack>

<Solution>

You can provide a `key` to the `<img>` tag. When that `key` changes, React will re-create the `<img>` DOM node from scratch. This causes a brief flash when each image loads, so it's not something you'd want to do for every image in your app. But it makes sense if you want to ensure the image always matches the text.
你可以为 `<img>` 提供一个 `key`。当 `key` 更改时，React 将从头开始重新创建 `<img>` DOM 节点。这样会导致在每张图片加载时出现一个短暂的闪白，所以你不应该对你应用里的每张图片都这样做。但是如果你想确保图片与文本始终匹配，那这么做就是合理的。

<Sandpack>

```js
import { useState } from 'react';

export default function Gallery() {
  const [index, setIndex] = useState(0);
  const hasNext = index < images.length - 1;

  function handleClick() {
    if (hasNext) {
      setIndex(index + 1);
    } else {
      setIndex(0);
    }
  }

  let image = images[index];
  return (
    <>
      <button onClick={handleClick}>
        下一张
      </button>
      <h3>
        {images.length} 张图片中的第 {index + 1} 张
      </h3>
      <img key={image.src} src={image.src} />
      <p>
        {image.place}
      </p>
    </>
  );
}

let images = [{
  place: 'Penang, Malaysia',
  src: 'https://i.imgur.com/FJeJR8M.jpg'
}, {
  place: 'Lisbon, Portugal',
  src: 'https://i.imgur.com/dB2LRbj.jpg'
}, {
  place: 'Bilbao, Spain',
  src: 'https://i.imgur.com/z08o2TS.jpg'
}, {
  place: 'Valparaíso, Chile',
  src: 'https://i.imgur.com/Y3utgTi.jpg'
}, {
  place: 'Schwyz, Switzerland',
  src: 'https://i.imgur.com/JBbMpWY.jpg'
}, {
  place: 'Prague, Czechia',
  src: 'https://i.imgur.com/QwUKKmF.jpg'
}, {
  place: 'Ljubljana, Slovenia',
  src: 'https://i.imgur.com/3aIiwfm.jpg'
}];
```

```css
img { width: 150px; height: 150px; }
```

</Sandpack>

</Solution>

#### 修复列表中错位的 state | Fix misplaced state in the list {/*fix-misplaced-state-in-the-list*/}

In this list, each `Contact` has state that determines whether "Show email" has been pressed for it. Press "Show email" for Alice, and then tick the "Show in reverse order" checkbox. You will notice that it's _Taylor's_ email that is expanded now, but Alice's--which has moved to the bottom--appears collapsed.
在这个列表中每个 `Contact` 都有个 state 表示它的“显示邮箱”按钮是否已经按过了。点击 Alice 的“显示邮箱”按钮，然后勾选“以相反的顺序显示”复选框。你会注意到现在展开的是 **Taylor 的** 邮箱，而 Alice 的邮箱已经被移到底部并被收起了。

Fix it so that the expanded state is associated with each contact, regardless of the chosen ordering.
修复它，使得不管选中的顺序如何，`expanded` state 都与各个联系人相关联。

<Sandpack>

```js src/App.js
import { useState } from 'react';
import Contact from './Contact.js';

export default function ContactList() {
  const [reverse, setReverse] = useState(false);

  const displayedContacts = [...contacts];
  if (reverse) {
    displayedContacts.reverse();
  }

  return (
    <>
      <label>
        <input
          type="checkbox"
          value={reverse}
          onChange={e => {
            setReverse(e.target.checked)
          }}
        />{' '}
        以相反的顺序显示
      </label>
      <ul>
        {displayedContacts.map((contact, i) =>
          <li key={i}>
            <Contact contact={contact} />
          </li>
        )}
      </ul>
    </>
  );
}

const contacts = [
  { id: 0, name: 'Alice', email: 'alice@mail.com' },
  { id: 1, name: 'Bob', email: 'bob@mail.com' },
  { id: 2, name: 'Taylor', email: 'taylor@mail.com' }
];
```

```js src/Contact.js
import { useState } from 'react';

export default function Contact({ contact }) {
  const [expanded, setExpanded] = useState(false);
  return (
    <>
      <p><b>{contact.name}</b></p>
      {expanded &&
        <p><i>{contact.email}</i></p>
      }
      <button onClick={() => {
        setExpanded(!expanded);
      }}>
        {expanded ? '隐藏' : '显示'}邮箱
      </button>
    </>
  );
}
```

```css
ul, li {
  list-style: none;
  margin: 0;
  padding: 0;
}
li {
  margin-bottom: 20px;
}
label {
  display: block;
  margin: 10px 0;
}
button {
  margin-right: 10px;
  margin-bottom: 10px;
}
```

</Sandpack>

<Solution>

The problem is that this example was using index as a `key`:
问题在于这个例子使用了 index 作为 `key`：

```js
{displayedContacts.map((contact, i) =>
  <li key={i}>
```

However, you want the state to be associated with _each particular contact_.
然而你应该让 state 与 **每个特定的联系人** 相关联。

Using the contact ID as a `key` instead fixes the issue:
使用联系人的 ID 作为 `key` 就会修复这个问题：

<Sandpack>

```js src/App.js
import { useState } from 'react';
import Contact from './Contact.js';

export default function ContactList() {
  const [reverse, setReverse] = useState(false);

  const displayedContacts = [...contacts];
  if (reverse) {
    displayedContacts.reverse();
  }

  return (
    <>
      <label>
        <input
          type="checkbox"
          value={reverse}
          onChange={e => {
            setReverse(e.target.checked)
          }}
        />{' '}
        以相反的顺序显示
      </label>
      <ul>
        {displayedContacts.map(contact =>
          <li key={contact.id}>
            <Contact contact={contact} />
          </li>
        )}
      </ul>
    </>
  );
}

const contacts = [
  { id: 0, name: 'Alice', email: 'alice@mail.com' },
  { id: 1, name: 'Bob', email: 'bob@mail.com' },
  { id: 2, name: 'Taylor', email: 'taylor@mail.com' }
];
```

```js src/Contact.js
import { useState } from 'react';

export default function Contact({ contact }) {
  const [expanded, setExpanded] = useState(false);
  return (
    <>
      <p><b>{contact.name}</b></p>
      {expanded &&
        <p><i>{contact.email}</i></p>
      }
      <button onClick={() => {
        setExpanded(!expanded);
      }}>
        {expanded ? '显示' : '隐藏'}邮箱
      </button>
    </>
  );
}
```

```css
ul, li {
  list-style: none;
  margin: 0;
  padding: 0;
}
li {
  margin-bottom: 20px;
}
label {
  display: block;
  margin: 10px 0;
}
button {
  margin-right: 10px;
  margin-bottom: 10px;
}
```

</Sandpack>

State is associated with the tree position. A `key` lets you specify a named position instead of relying on order.
state 与树中的位置相关联。 `key` 让你可以指定一个特定的位置，而不依赖于顺序。

</Solution>

</Challenges>
