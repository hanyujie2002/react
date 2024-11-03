---
title: '你可能不需要 Effect | You Might Not Need an Effect'
---

<Intro>

Effects are an escape hatch from the React paradigm. They let you "step outside" of React and synchronize your components with some external system like a non-React widget, network, or the browser DOM. If there is no external system involved (for example, if you want to update a component's state when some props or state change), you shouldn't need an Effect. Removing unnecessary Effects will make your code easier to follow, faster to run, and less error-prone.
Effect 是 React 范式中的一种脱围机制。它们让你可以 “逃出” React 并使组件和一些外部系统同步，比如非 React 组件、网络和浏览器 DOM。如果没有涉及到外部系统（例如，你想根据 props 或 state 的变化来更新一个组件的 state），你就不应该使用 Effect。移除不必要的 Effect 可以让你的代码更容易理解，运行得更快，并且更少出错。

</Intro>

<YouWillLearn>

* Why and how to remove unnecessary Effects from your components
* 为什么以及如何从你的组件中移除 Effect
* How to cache expensive computations without Effects
* 如何在没有 Effect 的情况下缓存昂贵的计算
* How to reset and adjust component state without Effects
* 如何在没有 Effect 的情况下重置和调整组件的 state
* How to share logic between event handlers
* 如何在事件处理函数之间共享逻辑
* Which logic should be moved to event handlers
* 应该将哪些逻辑移动到事件处理函数中
* How to notify parent components about changes
* 如何将发生的变动通知到父组件

</YouWillLearn>

## 如何移除不必要的 Effect | How to remove unnecessary Effects {/*how-to-remove-unnecessary-effects*/}

There are two common cases in which you don't need Effects:
有两种不必使用 Effect 的常见情况：

* **You don't need Effects to transform data for rendering.** For example, let's say you want to filter a list before displaying it. You might feel tempted to write an Effect that updates a state variable when the list changes. However, this is inefficient. When you update the state, React will first call your component functions to calculate what should be on the screen. Then React will ["commit"](/learn/render-and-commit) these changes to the DOM, updating the screen. Then React will run your Effects. If your Effect *also* immediately updates the state, this restarts the whole process from scratch! To avoid the unnecessary render passes, transform all the data at the top level of your components. That code will automatically re-run whenever your props or state change.
* **你不必使用 Effect 来转换渲染所需的数据**。例如，你想在展示一个列表前先做筛选。你的直觉可能是写一个当列表变化时更新 state 变量的 Effect。然而，这是低效的。当你更新这个 state 时，React 首先会调用你的组件函数来计算应该显示在屏幕上的内容。然后 React 会把这些变化“[提交](/learn/render-and-commit)”到 DOM 中来更新屏幕。然后 React 会执行你的 Effect。如果你的 Effect 也立即更新了这个 state，就会重新执行整个流程。为了避免不必要的渲染流程，应在你的组件顶层转换数据。这些代码会在你的 props 或 state 变化时自动重新执行。
* **You don't need Effects to handle user events.** For example, let's say you want to send an `/api/buy` POST request and show a notification when the user buys a product. In the Buy button click event handler, you know exactly what happened. By the time an Effect runs, you don't know *what* the user did (for example, which button was clicked). This is why you'll usually handle user events in the corresponding event handlers.
* **你不必使用 Effect 来处理用户事件**。例如，你想在用户购买一个产品时发送一个 `/api/buy` 的 POST 请求并展示一个提示。在这个购买按钮的点击事件处理函数中，你确切地知道会发生什么。但是当一个 Effect 运行时，你却不知道用户做了什么（例如，点击了哪个按钮）。这就是为什么你通常应该在相应的事件处理函数中处理用户事件。

You *do* need Effects to [synchronize](/learn/synchronizing-with-effects#what-are-effects-and-how-are-they-different-from-events) with external systems. For example, you can write an Effect that keeps a jQuery widget synchronized with the React state. You can also fetch data with Effects: for example, you can synchronize the search results with the current search query. Keep in mind that modern [frameworks](/learn/start-a-new-react-project#production-grade-react-frameworks) provide more efficient built-in data fetching mechanisms than writing Effects directly in your components.
你 **的确** 可以使用 Effect 来和外部系统 [同步](/learn/synchronizing-with-effects#what-are-effects-and-how-are-they-different-from-events) 。例如，你可以写一个 Effect 来保持一个 jQuery 的组件和 React state 之间的同步。你也可以使用 Effect 来获取数据：例如，你可以同步当前的查询搜索和查询结果。请记住，比起直接在你的组件中写 Effect，现代 [框架](/learn/start-a-new-react-project#production-grade-react-frameworks) 提供了更加高效的，内置的数据获取机制。

To help you gain the right intuition, let's look at some common concrete examples!
为了帮助你获得正确的直觉，让我们来看一些常见的实例吧！

### 根据 props 或 state 来更新 state | Updating state based on props or state {/*updating-state-based-on-props-or-state*/}

Suppose you have a component with two state variables: `firstName` and `lastName`. You want to calculate a `fullName` from them by concatenating them. Moreover, you'd like `fullName` to update whenever `firstName` or `lastName` change. Your first instinct might be to add a `fullName` state variable and update it in an Effect:
假设你有一个包含了两个 state 变量的组件：`firstName` 和 `lastName`。你想通过把它们联结起来计算出 `fullName`。此外，每当 `firstName` 和 `lastName` 变化时，你希望 `fullName` 都能更新。你的第一直觉可能是添加一个 state 变量：`fullName`，并在一个 Effect 中更新它：

```js {5-9}
function Form() {
  const [firstName, setFirstName] = useState('Taylor');
  const [lastName, setLastName] = useState('Swift');

  // 🔴 Avoid: redundant state and unnecessary Effect
  // 🔴 避免：多余的 state 和不必要的 Effect
  const [fullName, setFullName] = useState('');
  useEffect(() => {
    setFullName(firstName + ' ' + lastName);
  }, [firstName, lastName]);
  // ...
}
```

This is more complicated than necessary. It is inefficient too: it does an entire render pass with a stale value for `fullName`, then immediately re-renders with the updated value. Remove the state variable and the Effect:
大可不必这么复杂。而且这样效率也不高：它先是用 `fullName` 的旧值执行了整个渲染流程，然后立即使用更新后的值又重新渲染了一遍。让我们移除 state 变量和 Effect：

```js {4-5}
function Form() {
  const [firstName, setFirstName] = useState('Taylor');
  const [lastName, setLastName] = useState('Swift');
  // ✅ Good: calculated during rendering
  // ✅ 非常好：在渲染期间进行计算
  const fullName = firstName + ' ' + lastName;
  // ...
}
```

**When something can be calculated from the existing props or state, [don't put it in state.](/learn/choosing-the-state-structure#avoid-redundant-state) Instead, calculate it during rendering.** This makes your code faster (you avoid the extra "cascading" updates), simpler (you remove some code), and less error-prone (you avoid bugs caused by different state variables getting out of sync with each other). If this approach feels new to you, [Thinking in React](/learn/thinking-in-react#step-3-find-the-minimal-but-complete-representation-of-ui-state) explains what should go into state.
**如果一个值可以基于现有的 props 或 state 计算得出，[不要把它作为一个 state](/learn/choosing-the-state-structure#avoid-redundant-state)，而是在渲染期间直接计算这个值**。这将使你的代码更快（避免了多余的 “级联” 更新）、更简洁（移除了一些代码）以及更少出错（避免了一些因为不同的 state 变量之间没有正确同步而导致的问题）。如果这个观点对你来说很新奇，[React 哲学](/learn/thinking-in-react#step-3-find-the-minimal-but-complete-representation-of-ui-state) 中解释了什么值应该作为 state。

### 缓存昂贵的计算 | Caching expensive calculations {/*caching-expensive-calculations*/}

This component computes `visibleTodos` by taking the `todos` it receives by props and filtering them according to the `filter` prop. You might feel tempted to store the result in state and update it from an Effect:
这个组件使用它接收到的 props 中的 `filter` 对另一个 prop `todos` 进行筛选，计算得出 `visibleTodos`。你的直觉可能是把结果存到一个 state 中，并在 Effect 中更新它：

```js {4-8}
function TodoList({ todos, filter }) {
  const [newTodo, setNewTodo] = useState('');

  // 🔴 Avoid: redundant state and unnecessary Effect
  // 🔴 避免：多余的 state 和不必要的 Effect
  const [visibleTodos, setVisibleTodos] = useState([]);
  useEffect(() => {
    setVisibleTodos(getFilteredTodos(todos, filter));
  }, [todos, filter]);

  // ...
}
```

Like in the earlier example, this is both unnecessary and inefficient. First, remove the state and the Effect:
就像之前的例子一样，这既没有必要，也很低效。首先，移除 state 和 Effect：

```js {3-4}
function TodoList({ todos, filter }) {
  const [newTodo, setNewTodo] = useState('');
  // ✅ This is fine if getFilteredTodos() is not slow.
  // ✅ 如果 getFilteredTodos() 的耗时不长，这样写就可以了。
  const visibleTodos = getFilteredTodos(todos, filter);
  // ...
}
```

Usually, this code is fine! But maybe `getFilteredTodos()` is slow or you have a lot of `todos`. In that case you don't want to recalculate `getFilteredTodos()` if some unrelated state variable like `newTodo` has changed.
一般来说，这段代码没有问题！但是，`getFilteredTodos()` 的耗时可能会很长，或者你有很多 `todos`。这些情况下，当 `newTodo` 这样不相关的 state 变量变化时，你并不想重新执行 `getFilteredTodos()`。

You can cache (or ["memoize"](https://en.wikipedia.org/wiki/Memoization)) an expensive calculation by wrapping it in a [`useMemo`](/reference/react/useMemo) Hook:
你可以使用 [`useMemo`](/reference/react/useMemo) Hook 缓存（或者说 [记忆（memoize）](https://en.wikipedia.org/wiki/Memoization)）一个昂贵的计算。

```js {5-8}
import { useMemo, useState } from 'react';

function TodoList({ todos, filter }) {
  const [newTodo, setNewTodo] = useState('');
  const visibleTodos = useMemo(() => {
    // ✅ Does not re-run unless todos or filter change
    // ✅ 除非 todos 或 filter 发生变化，否则不会重新执行
    return getFilteredTodos(todos, filter);
  }, [todos, filter]);
  // ...
}
```

Or, written as a single line:
或者写成一行：

```js {5-6}
import { useMemo, useState } from 'react';

function TodoList({ todos, filter }) {
  const [newTodo, setNewTodo] = useState('');
  // ✅ Does not re-run getFilteredTodos() unless todos or filter change
  // ✅ 除非 todos 或 filter 发生变化，否则不会重新执行 getFilteredTodos()
  const visibleTodos = useMemo(() => getFilteredTodos(todos, filter), [todos, filter]);
  // ...
}
```

**This tells React that you don't want the inner function to re-run unless either `todos` or `filter` have changed.** React will remember the return value of `getFilteredTodos()` during the initial render. During the next renders, it will check if `todos` or `filter` are different. If they're the same as last time, `useMemo` will return the last result it has stored. But if they are different, React will call the inner function again (and store its result).
**这会告诉 React，除非 `todos` 或 `filter` 发生变化，否则不要重新执行传入的函数**。React 会在初次渲染的时候记住 `getFilteredTodos()` 的返回值。在下一次渲染中，它会检查 `todos` 或 `filter` 是否发生了变化。如果它们跟上次渲染时一样，`useMemo` 会直接返回它最后保存的结果。如果不一样，React 将再次调用传入的函数（并保存它的结果）。

The function you wrap in [`useMemo`](/reference/react/useMemo) runs during rendering, so this only works for [pure calculations.](/learn/keeping-components-pure)
你传入 [`useMemo`](/reference/react/useMemo) 的函数会在渲染期间执行，所以它仅适用于 [纯函数](/learn/keeping-components-pure) 场景。

<DeepDive>

#### 如何判断计算是昂贵的？| How to tell if a calculation is expensive? {/*how-to-tell-if-a-calculation-is-expensive*/}

In general, unless you're creating or looping over thousands of objects, it's probably not expensive. If you want to get more confidence, you can add a console log to measure the time spent in a piece of code:
一般来说只有你创建或循环遍历了成千上万个对象时才会很耗费时间。如果你想确认一下，可以添加控制台输出来测量某一段代码的执行时间：

```js {1,3}
console.time('筛选数组');
const visibleTodos = getFilteredTodos(todos, filter);
console.timeEnd('筛选数组');
```

Perform the interaction you're measuring (for example, typing into the input). You will then see logs like `filter array: 0.15ms` in your console. If the overall logged time adds up to a significant amount (say, `1ms` or more), it might make sense to memoize that calculation. As an experiment, you can then wrap the calculation in `useMemo` to verify whether the total logged time has decreased for that interaction or not:
触发要测量的交互（例如，在输入框中输入）。你会在控制台中看到类似 `筛选数组：0.15ms` 这样的输出日志。如果总耗时达到了一定量级（比方说 `1ms` 或更多），那么把计算结果记忆（memoize）起来可能是有意义的。做一个实验，你可以把计算传入 `useMemo` 中来验证该交互导致的总耗时是减少了还是没什么变化：

```js
console.time('筛选数组');
const visibleTodos = useMemo(() => {
  return getFilteredTodos(todos, filter); // 如果 todos 或 filter 没有发生变化将跳过执行 // Skipped if todos and filter haven't changed
}, [todos, filter]);
console.timeEnd('筛选数组');
```

`useMemo` won't make the *first* render faster. It only helps you skip unnecessary work on updates.
`useMemo` 不会让 **第一次** 渲染变快。它只是帮助你跳过不必要的更新。

Keep in mind that your machine is probably faster than your users' so it's a good idea to test the performance with an artificial slowdown. For example, Chrome offers a [CPU Throttling](https://developer.chrome.com/blog/new-in-devtools-61/#throttling) option for this.
请注意，你的设备性能可能比用户的更好，因此最好通过人工限制工具来测试性能。例如，Chrome 提供了 [CPU 节流](https://developer.chrome.com/blog/new-in-devtools-61/#throttling) 工具。

Also note that measuring performance in development will not give you the most accurate results. (For example, when [Strict Mode](/reference/react/StrictMode) is on, you will see each component render twice rather than once.) To get the most accurate timings, build your app for production and test it on a device like your users have.
同时需要注意的是，在开发环境测试性能并不能得到最准确的结果（例如，当开启 [严格模式](/reference/react/StrictMode) 时，你会看到每个组件渲染了两次，而不是一次）。所以为了得到最准确的时间，需要将你的应用构建为生产模式，同时使用与你的用户性能相当的设备进行测试。

</DeepDive>

### 当 props 变化时重置所有 state | Resetting all state when a prop changes {/*resetting-all-state-when-a-prop-changes*/}

This `ProfilePage` component receives a `userId` prop. The page contains a comment input, and you use a `comment` state variable to hold its value. One day, you notice a problem: when you navigate from one profile to another, the `comment` state does not get reset. As a result, it's easy to accidentally post a comment on a wrong user's profile. To fix the issue, you want to clear out the `comment` state variable whenever the `userId` changes:
`ProfilePage` 组件接收一个 prop：`userId`。页面上有一个评论输入框，你用了一个 state：`comment` 来保存它的值。有一天，你发现了一个问题：当你从一个人的个人资料导航到另一个时，`comment` 没有被重置。这导致很容易不小心把评论发送到不正确的个人资料。为了解决这个问题，你想在 `userId` 变化时，清除 `comment` 变量：

```js {4-7}
export default function ProfilePage({ userId }) {
  const [comment, setComment] = useState('');

  // 🔴 Avoid: Resetting state on prop change in an Effect
  // 🔴 避免：当 prop 变化时，在 Effect 中重置 state
  useEffect(() => {
    setComment('');
  }, [userId]);
  // ...
}
```

This is inefficient because `ProfilePage` and its children will first render with the stale value, and then render again. It is also complicated because you'd need to do this in *every* component that has some state inside `ProfilePage`. For example, if the comment UI is nested, you'd want to clear out nested comment state too.
但这是低效的，因为 `ProfilePage` 和它的子组件首先会用旧值渲染，然后再用新值重新渲染。并且这样做也很复杂，因为你需要在 `ProfilePage` 里面 **所有** 具有 state 的组件中都写这样的代码。例如，如果评论区的 UI 是嵌套的，你可能也想清除嵌套的 comment state。

Instead, you can tell React that each user's profile is conceptually a _different_ profile by giving it an explicit key. Split your component in two and pass a `key` attribute from the outer component to the inner one:
取而代之的是，你可以通过为每个用户的个人资料组件提供一个明确的键来告诉 React 它们原则上是 **不同** 的个人资料组件。将你的组件拆分为两个组件，并从外部的组件传递一个 `key` 属性给内部的组件：

```js {5,11-12}
export default function ProfilePage({ userId }) {
  return (
    <Profile
      userId={userId}
      key={userId}
    />
  );
}

function Profile({ userId }) {
  // ✅ This and any other state below will reset on key change automatically
  // ✅ 当 key 变化时，该组件内的 comment 或其他 state 会自动被重置
  const [comment, setComment] = useState('');
  // ...
}
```

Normally, React preserves the state when the same component is rendered in the same spot. **By passing `userId` as a `key` to the `Profile` component, you're asking React to treat two `Profile` components with different `userId` as two different components that should not share any state.** Whenever the key (which you've set to `userId`) changes, React will recreate the DOM and [reset the state](/learn/preserving-and-resetting-state#option-2-resetting-state-with-a-key) of the `Profile` component and all of its children. Now the `comment` field will clear out automatically when navigating between profiles.
通常，当在相同的位置渲染相同的组件时，React 会保留状态。**通过将 `userId` 作为 `key` 传递给 `Profile` 组件，使  React 将具有不同 `userId` 的两个 `Profile` 组件视为两个不应共享任何状态的不同组件**。每当 key（这里是 `userId`）变化时，React 将重新创建 DOM，并 [重置](/learn/preserving-and-resetting-state#option-2-resetting-state-with-a-key) `Profile` 组件和它的所有子组件的 state。现在，当在不同的个人资料之间导航时，`comment` 区域将自动被清空。

Note that in this example, only the outer `ProfilePage` component is exported and visible to other files in the project. Components rendering `ProfilePage` don't need to pass the key to it: they pass `userId` as a regular prop. The fact `ProfilePage` passes it as a `key` to the inner `Profile` component is an implementation detail.
请注意，在这个例子中，只有外部的 `ProfilePage` 组件被导出并在项目中对其他文件可见。渲染 `ProfilePage` 的那些组件不用传递 `key` 给它：它们只需把 `userId` 作为常规 prop 传入即可。而 `ProfilePage` 将其作为 `key` 传递给内部的 `Profile` 组件是它的实现细节而已。

### 当 prop 变化时调整部分 state | Adjusting some state when a prop changes {/*adjusting-some-state-when-a-prop-changes*/}

Sometimes, you might want to reset or adjust a part of the state on a prop change, but not all of it.
有时候，当 prop 变化时，你可能只想重置或调整部分 state ，而不是所有 state。

This `List` component receives a list of `items` as a prop, and maintains the selected item in the `selection` state variable. You want to reset the `selection` to `null` whenever the `items` prop receives a different array:
`List` 组件接收一个 `items` 列表作为 prop，然后用 state 变量 `selection` 来保持已选中的项。当 `items` 接收到一个不同的数组时，你想将 `selection` 重置为 `null`：

```js {5-8}
function List({ items }) {
  const [isReverse, setIsReverse] = useState(false);
  const [selection, setSelection] = useState(null);

  // 🔴 Avoid: Adjusting state on prop change in an Effect
  // 🔴 避免：当 prop 变化时，在 Effect 中调整 state
  useEffect(() => {
    setSelection(null);
  }, [items]);
  // ...
}
```

This, too, is not ideal. Every time the `items` change, the `List` and its child components will render with a stale `selection` value at first. Then React will update the DOM and run the Effects. Finally, the `setSelection(null)` call will cause another re-render of the `List` and its child components, restarting this whole process again.
这不太理想。每当 `items` 变化时，`List` 及其子组件会先使用旧的 `selection` 值渲染。然后 React 会更新 DOM 并执行 Effect。最后，调用 `setSelection(null)` 将导致 `List` 及其子组件重新渲染，重新启动整个流程。

Start by deleting the Effect. Instead, adjust the state directly during rendering:
让我们从删除 Effect 开始。取而代之的是在渲染期间直接调整 state：

```js {5-11}
function List({ items }) {
  const [isReverse, setIsReverse] = useState(false);
  const [selection, setSelection] = useState(null);

  // Better: Adjust the state while rendering
  // 好一些：在渲染期间调整 state
  const [prevItems, setPrevItems] = useState(items);
  if (items !== prevItems) {
    setPrevItems(items);
    setSelection(null);
  }
  // ...
}
```

[Storing information from previous renders](/reference/react/useState#storing-information-from-previous-renders) like this can be hard to understand, but it’s better than updating the same state in an Effect. In the above example, `setSelection` is called directly during a render. React will re-render the `List` *immediately* after it exits with a `return` statement. React has not rendered the `List` children or updated the DOM yet, so this lets the `List` children skip rendering the stale `selection` value.
像这样 [存储前序渲染的信息](/reference/react/useState#storing-information-from-previous-renders) 可能很难理解，但它比在 Effect 中更新这个 state 要好。上面的例子中，在渲染过程中直接调用了 `setSelection`。当它执行到 `return` 语句退出后，React 将 **立即** 重新渲染 `List`。此时 React 还没有渲染 `List` 的子组件或更新 DOM，这使得 `List` 的子组件可以跳过渲染旧的 `selection` 值。

When you update a component during rendering, React throws away the returned JSX and immediately retries rendering. To avoid very slow cascading retries, React only lets you update the *same* component's state during a render. If you update another component's state during a render, you'll see an error. A condition like `items !== prevItems` is necessary to avoid loops. You may adjust state like this, but any other side effects (like changing the DOM or setting timeouts) should stay in event handlers or Effects to [keep components pure.](/learn/keeping-components-pure)
在渲染期间更新组件时，React 会丢弃已经返回的 JSX 并立即尝试重新渲染。为了避免非常缓慢的级联重试，React 只允许在渲染期间更新 **同一** 组件的状态。如果你在渲染期间更新另一个组件的状态，你会看到一条报错信息。条件判断 `items !== prevItems` 是必要的，它可以避免无限循环。你可以像这样调整 state，但任何其他副作用（比如变化 DOM 或设置的延时）应该留在事件处理函数或 Effect 中，以 [保持组件纯粹](/learn/keeping-components-pure)。

**Although this pattern is more efficient than an Effect, most components shouldn't need it either.** No matter how you do it, adjusting state based on props or other state makes your data flow more difficult to understand and debug. Always check whether you can [reset all state with a key](#resetting-all-state-when-a-prop-changes) or [calculate everything during rendering](#updating-state-based-on-props-or-state) instead. For example, instead of storing (and resetting) the selected *item*, you can store the selected *item ID:*
**虽然这种方式比 Effect 更高效，但大多数组件也不需要它**。无论你怎么做，根据 props 或其他 state 来调整 state 都会使数据流更难理解和调试。总是检查是否可以通过添加 [key 来重置所有 state](#resetting-all-state-when-a-prop-changes)，或者 [在渲染期间计算所需内容](#updating-state-based-on-props-or-state)。例如，你可以存储已选中的 **item ID** 而不是存储（并重置）已选中的 **item**：

```js {3-5}
function List({ items }) {
  const [isReverse, setIsReverse] = useState(false);
  const [selectedId, setSelectedId] = useState(null);
  // ✅ Best: Calculate everything during rendering
  // ✅ 非常好：在渲染期间计算所需内容
  const selection = items.find(item => item.id === selectedId) ?? null;
  // ...
}
```

Now there is no need to "adjust" the state at all. If the item with the selected ID is in the list, it remains selected. If it's not, the `selection` calculated during rendering will be `null` because no matching item was found. This behavior is different, but arguably better because most changes to `items` preserve the selection.
现在完全不需要 “调整” state 了。如果包含已选中 ID 的项出现在列表中，则仍然保持选中状态。如果没有找到匹配的项，则在渲染期间计算的 `selection` 将会是 `null`。行为不同了，但可以说更好了，因为大多数对 `items` 的更改仍可以保持选中状态。

### 在事件处理函数中共享逻辑 | Sharing logic between event handlers {/*sharing-logic-between-event-handlers*/}

Let's say you have a product page with two buttons (Buy and Checkout) that both let you buy that product. You want to show a notification whenever the user puts the product in the cart. Calling `showNotification()` in both buttons' click handlers feels repetitive so you might be tempted to place this logic in an Effect:
假设你有一个产品页面，上面有两个按钮（购买和付款），都可以让你购买该产品。当用户将产品添加进购物车时，你想显示一个通知。在两个按钮的 click 事件处理函数中都调用 `showNotification()` 感觉有点重复，所以你可能想把这个逻辑放在一个 Effect 中：

```js {2-7}
function ProductPage({ product, addToCart }) {
  // 🔴 Avoid: Event-specific logic inside an Effect
  // 🔴 避免：在 Effect 中处理属于事件特定的逻辑
  useEffect(() => {
    if (product.isInCart) {
      showNotification(`已添加 ${product.name} 进购物车！`);
    }
  }, [product]);

  function handleBuyClick() {
    addToCart(product);
  }

  function handleCheckoutClick() {
    addToCart(product);
    navigateTo('/checkout');
  }
  // ...
}
```

This Effect is unnecessary. It will also most likely cause bugs. For example, let's say that your app "remembers" the shopping cart between the page reloads. If you add a product to the cart once and refresh the page, the notification will appear again. It will keep appearing every time you refresh that product's page. This is because `product.isInCart` will already be `true` on the page load, so the Effect above will call `showNotification()`.
这个 Effect 是多余的。而且很可能会导致问题。例如，假设你的应用在页面重新加载之前 “记住” 了购物车中的产品。如果你把一个产品添加到购物车中并刷新页面，通知将再次出现。每次刷新该产品页面时，它都会出现。这是因为 `product.isInCart` 在页面加载时已经是 `true` 了，所以上面的 Effect 每次都会调用 `showNotification()`。

**When you're not sure whether some code should be in an Effect or in an event handler, ask yourself *why* this code needs to run. Use Effects only for code that should run *because* the component was displayed to the user.** In this example, the notification should appear because the user *pressed the button*, not because the page was displayed! Delete the Effect and put the shared logic into a function called from both event handlers:
**当你不确定某些代码应该放在 Effect 中还是事件处理函数中时，先自问 为什么 要执行这些代码。Effect 只用来执行那些显示给用户时组件 需要执行 的代码**。在这个例子中，通知应该在用户 **按下按钮** 后出现，而不是因为页面显示出来时！删除 Effect 并将共享的逻辑放入一个被两个事件处理程序调用的函数中：

```js {2-6,9,13}
function ProductPage({ product, addToCart }) {
  // ✅ Good: Event-specific logic is called from event handlers
  // ✅ 非常好：事件特定的逻辑在事件处理函数中处理
  function buyProduct() {
    addToCart(product);
    showNotification(`已添加 ${product.name} 进购物车！`);
  }

  function handleBuyClick() {
    buyProduct();
  }

  function handleCheckoutClick() {
    buyProduct();
    navigateTo('/checkout');
  }
  // ...
}
```

This both removes the unnecessary Effect and fixes the bug.
这既移除了不必要的 Effect，又修复了问题。

### 发送 POST 请求 | Sending a POST request {/*sending-a-post-request*/}

This `Form` component sends two kinds of POST requests. It sends an analytics event when it mounts. When you fill in the form and click the Submit button, it will send a POST request to the `/api/register` endpoint:
这个 `Form` 组件会发送两种 POST 请求。它在页面加载之际会发送一个分析请求。当你填写表格并点击提交按钮时，它会向 `/api/register` 接口发送一个 POST 请求：

```js {5-8,10-16}
function Form() {
  const [firstName, setFirstName] = useState('');
  const [lastName, setLastName] = useState('');

  // ✅ Good: This logic should run because the component was displayed
  // ✅ 非常好：这个逻辑应该在组件显示时执行
  useEffect(() => {
    post('/analytics/event', { eventName: 'visit_form' });
  }, []);

  // 🔴 Avoid: Event-specific logic inside an Effect
  // 🔴 避免：在 Effect 中处理属于事件特定的逻辑
  const [jsonToSubmit, setJsonToSubmit] = useState(null);
  useEffect(() => {
    if (jsonToSubmit !== null) {
      post('/api/register', jsonToSubmit);
    }
  }, [jsonToSubmit]);

  function handleSubmit(e) {
    e.preventDefault();
    setJsonToSubmit({ firstName, lastName });
  }
  // ...
}
```

Let's apply the same criteria as in the example before.
让我们应用与之前示例相同的准则。

The analytics POST request should remain in an Effect. This is because the _reason_ to send the analytics event is that the form was displayed. (It would fire twice in development, but [see here](/learn/synchronizing-with-effects#sending-analytics) for how to deal with that.)
分析请求应该保留在 Effect 中。这是 **因为** 发送分析请求是表单显示时就需要执行的（在开发环境中它会发送两次，请 [参考这里](/learn/synchronizing-with-effects#sending-analytics) 了解如何处理）。

However, the `/api/register` POST request is not caused by the form being _displayed_. You only want to send the request at one specific moment in time: when the user presses the button. It should only ever happen _on that particular interaction_. Delete the second Effect and move that POST request into the event handler:
然而，发送到 `/api/register` 的 POST 请求并不是由表单 **显示** 时引起的。你只想在一个特定的时间点发送请求：当用户按下按钮时。它应该只在这个 **特定的交互** 中发生。删除第二个 Effect，将该 POST 请求移入事件处理函数中：

```js {12-13}
function Form() {
  const [firstName, setFirstName] = useState('');
  const [lastName, setLastName] = useState('');

  // ✅ Good: This logic runs because the component was displayed
  // ✅ 非常好：这个逻辑应该在组件显示时执行
  useEffect(() => {
    post('/analytics/event', { eventName: 'visit_form' });
  }, []);

  function handleSubmit(e) {
    e.preventDefault();
    // ✅ Good: Event-specific logic is in the event handler
    // ✅ 非常好：事件特定的逻辑在事件处理函数中处理
    post('/api/register', { firstName, lastName });
  }
  // ...
}
```

When you choose whether to put some logic into an event handler or an Effect, the main question you need to answer is _what kind of logic_ it is from the user's perspective. If this logic is caused by a particular interaction, keep it in the event handler. If it's caused by the user _seeing_ the component on the screen, keep it in the Effect.
当你决定将某些逻辑放入事件处理函数还是 Effect 中时，你需要回答的主要问题是：从用户的角度来看它是 **怎样的逻辑**。如果这个逻辑是由某个特定的交互引起的，请将它保留在相应的事件处理函数中。如果是由用户在屏幕上 **看到** 组件时引起的，请将它保留在 Effect 中。

### 链式计算 | Chains of computations {/*chains-of-computations*/}

Sometimes you might feel tempted to chain Effects that each adjust a piece of state based on other state:
有时候你可能想链接多个 Effect，每个 Effect 都基于某些 state 来调整其他的 state：

```js {7-29}
function Game() {
  const [card, setCard] = useState(null);
  const [goldCardCount, setGoldCardCount] = useState(0);
  const [round, setRound] = useState(1);
  const [isGameOver, setIsGameOver] = useState(false);

  // 🔴 Avoid: Chains of Effects that adjust the state solely to trigger each other
  // 🔴 避免：链接多个 Effect 仅仅为了相互触发调整 state
  useEffect(() => {
    if (card !== null && card.gold) {
      setGoldCardCount(c => c + 1);
    }
  }, [card]);

  useEffect(() => {
    if (goldCardCount > 3) {
      setRound(r => r + 1)
      setGoldCardCount(0);
    }
  }, [goldCardCount]);

  useEffect(() => {
    if (round > 5) {
      setIsGameOver(true);
    }
  }, [round]);

  useEffect(() => {
    alert('游戏结束！');
  }, [isGameOver]);

  function handlePlaceCard(nextCard) {
    if (isGameOver) {
      throw Error('游戏已经结束了。');
    } else {
      setCard(nextCard);
    }
  }

  // ...
```

There are two problems with this code.
这段代码里有两个问题。

The first problem is that it is very inefficient: the component (and its children) have to re-render between each `set` call in the chain. In the example above, in the worst case (`setCard` → render → `setGoldCardCount` → render → `setRound` → render → `setIsGameOver` → render) there are three unnecessary re-renders of the tree below.
第一个问题是它非常低效：在链式的每个 `set` 调用之间，组件（及其子组件）都不得不重新渲染。在上面的例子中，在最坏的情况下（`setCard` → 渲染 → `setGoldCardCount` → 渲染 → `setRound` → 渲染 → `setIsGameOver` → 渲染）有三次不必要的重新渲染。

The second problem is that even if it weren't slow, as your code evolves, you will run into cases where the "chain" you wrote doesn't fit the new requirements. Imagine you are adding a way to step through the history of the game moves. You'd do it by updating each state variable to a value from the past. However, setting the `card` state to a value from the past would trigger the Effect chain again and change the data you're showing. Such code is often rigid and fragile.
第二个问题是，即使不考虑渲染效率问题，随着代码不断扩展，你会遇到这条 “链式” 调用不符合新需求的情况。试想一下，你现在需要添加一种方法来回溯游戏的历史记录，可以通过更新每个 state 变量到之前的值来实现。然而，将 `card` 设置为之前的的某个值会再次触发 Effect 链并更改你正在显示的数据。这样的代码往往是僵硬而脆弱的。

In this case, it's better to calculate what you can during rendering, and adjust the state in the event handler:
在这个例子中，更好的做法是：尽可能在渲染期间进行计算，以及在事件处理函数中调整 state：

```js {6-7,14-26}
function Game() {
  const [card, setCard] = useState(null);
  const [goldCardCount, setGoldCardCount] = useState(0);
  const [round, setRound] = useState(1);

  // ✅ Calculate what you can during rendering
  // ✅ 尽可能在渲染期间进行计算
  const isGameOver = round > 5;

  function handlePlaceCard(nextCard) {
    if (isGameOver) {
      throw Error('游戏已经结束了。');
    }

    // ✅ Calculate all the next state in the event handler
    // ✅ 在事件处理函数中计算剩下的所有 state
    setCard(nextCard);
    if (nextCard.gold) {
      if (goldCardCount <= 3) {
        setGoldCardCount(goldCardCount + 1);
      } else {
        setGoldCardCount(0);
        setRound(round + 1);
        if (round === 5) {
          alert('游戏结束！');
        }
      }
    }
  }

  // ...
```

This is a lot more efficient. Also, if you implement a way to view game history, now you will be able to set each state variable to a move from the past without triggering the Effect chain that adjusts every other value. If you need to reuse logic between several event handlers, you can [extract a function](#sharing-logic-between-event-handlers) and call it from those handlers.
这高效得多。此外，如果你实现了一个回溯游戏历史的方法，现在你可以将每个 state 变量设置为之前的任何的一个值，而不会触发每个调整其他值的 Effect 链。如果你需要在多个事件处理函数之间复用逻辑，可以 [提取成一个函数](#sharing-logic-between-event-handlers) 并在这些事件处理函数中调用它。

Remember that inside event handlers, [state behaves like a snapshot.](/learn/state-as-a-snapshot) For example, even after you call `setRound(round + 1)`, the `round` variable will reflect the value at the time the user clicked the button. If you need to use the next value for calculations, define it manually like `const nextRound = round + 1`.
请记住，在事件处理函数内部，[state 的行为类似快照](/learn/state-as-a-snapshot)。例如，即使你调用了 `setRound(round + 1)`，`round` 变量仍然是用户点击按钮时的值。如果你需要使用下一个值进行计算，则需要像这样手动定义它：`const nextRound = round + 1`。

In some cases, you *can't* calculate the next state directly in the event handler. For example, imagine a form with multiple dropdowns where the options of the next dropdown depend on the selected value of the previous dropdown. Then, a chain of Effects is appropriate because you are synchronizing with network.
在某些情况下，你 **无法** 在事件处理函数中直接计算出下一个 state。例如，试想一个具有多个下拉菜单的表单，如果下一个下拉菜单的选项取决于前一个下拉菜单选择的值。这时，Effect 链是合适的，因为你需要与网络进行同步。

### 初始化应用 | Initializing the application {/*initializing-the-application*/}

Some logic should only run once when the app loads.
有些逻辑只需要在应用加载时执行一次。

You might be tempted to place it in an Effect in the top-level component:
你可能想把它放在一个顶层组件的 Effect 中：

```js {2-6}
function App() {
  // 🔴 Avoid: Effects with logic that should only ever run once
  // 🔴 避免：把只需要执行一次的逻辑放在 Effect 中
  useEffect(() => {
    loadDataFromLocalStorage();
    checkAuthToken();
  }, []);
  // ...
}
```

However, you'll quickly discover that it [runs twice in development.](/learn/synchronizing-with-effects#how-to-handle-the-effect-firing-twice-in-development) This can cause issues--for example, maybe it invalidates the authentication token because the function wasn't designed to be called twice. In general, your components should be resilient to being remounted. This includes your top-level `App` component.
然后，你很快就会发现它在 [开发环境会执行两次](/learn/synchronizing-with-effects#how-to-handle-the-effect-firing-twice-in-development)。这会导致一些问题——例如，它可能使身份验证 token 无效，因为该函数不是为被调用两次而设计的。一般来说，当组件重新挂载时应该具有一致性。包括你的顶层 `App` 组件。

Although it may not ever get remounted in practice in production, following the same constraints in all components makes it easier to move and reuse code. If some logic must run *once per app load* rather than *once per component mount*, add a top-level variable to track whether it has already executed:
尽管在实际的生产环境中它可能永远不会被重新挂载，但在所有组件中遵循相同的约束条件可以更容易地移动和复用代码。如果某些逻辑必须在 **每次应用加载时执行一次**，而不是在 **每次组件挂载时执行一次**，可以添加一个顶层变量来记录它是否已经执行过了：

```js {1,5-6,10}
let didInit = false;

function App() {
  useEffect(() => {
    if (!didInit) {
      didInit = true;
      // ✅ Only runs once per app load
      // ✅ 只在每次应用加载时执行一次
      loadDataFromLocalStorage();
      checkAuthToken();
    }
  }, []);
  // ...
}
```

You can also run it during module initialization and before the app renders:
你也可以在模块初始化和应用渲染之前执行它：

```js {1,5}
if (typeof window !== 'undefined') { // 检测我们是否在浏览器环境 // Check if we're running in the browser.
   // ✅ 只在每次应用加载时执行一次 // ✅ Only runs once per app load
  checkAuthToken();
  loadDataFromLocalStorage();
}

function App() {
  // ...
}
```

Code at the top level runs once when your component is imported--even if it doesn't end up being rendered. To avoid slowdown or surprising behavior when importing arbitrary components, don't overuse this pattern. Keep app-wide initialization logic to root component modules like `App.js` or in your application's entry point.
顶层代码会在组件被导入时执行一次——即使它最终并没有被渲染。为了避免在导入任意组件时降低性能或产生意外行为，请不要过度使用这种方法。将应用级别的初始化逻辑保留在像 `App.js` 这样的根组件模块或你的应用入口中。

### 通知父组件有关 state 变化的信息 | Notifying parent components about state changes {/*notifying-parent-components-about-state-changes*/}

Let's say you're writing a `Toggle` component with an internal `isOn` state which can be either `true` or `false`. There are a few different ways to toggle it (by clicking or dragging). You want to notify the parent component whenever the `Toggle` internal state changes, so you expose an `onChange` event and call it from an Effect:
假设你正在编写一个有具有内部 state `isOn` 的 `Toggle` 组件，该 state 可以是 `true` 或 `false`。有几种不同的方式来进行切换（通过点击或拖动）。你希望在 `Toggle` 的 state 变化时通知父组件，因此你暴露了一个 `onChange` 事件并在 `Effect` 中调用它：

```js {4-7}
function Toggle({ onChange }) {
  const [isOn, setIsOn] = useState(false);

  // 🔴 Avoid: The onChange handler runs too late
  // 🔴 避免：onChange 处理函数执行的时间太晚了
  useEffect(() => {
    onChange(isOn);
  }, [isOn, onChange])

  function handleClick() {
    setIsOn(!isOn);
  }

  function handleDragEnd(e) {
    if (isCloserToRightEdge(e)) {
      setIsOn(true);
    } else {
      setIsOn(false);
    }
  }

  // ...
}
```

Like earlier, this is not ideal. The `Toggle` updates its state first, and React updates the screen. Then React runs the Effect, which calls the `onChange` function passed from a parent component. Now the parent component will update its own state, starting another render pass. It would be better to do everything in a single pass.
和之前一样，这不太理想。`Toggle` 首先更新它的 state，然后 React 会更新屏幕。然后 React 执行 Effect 中的代码，调用从父组件传入的 `onChange` 函数。现在父组件开始更新它自己的 state，开启另一个渲染流程。更好的方式是在单个流程中完成所有操作。

Delete the Effect and instead update the state of *both* components within the same event handler:
删除 Effect，并在同一个事件处理函数中更新 **两个** 组件的 state：

```js {5-7,11,16,18}
function Toggle({ onChange }) {
  const [isOn, setIsOn] = useState(false);

  function updateToggle(nextIsOn) {
    // ✅ Good: Perform all updates during the event that caused them
    // ✅ 非常好：在触发它们的事件中执行所有更新
    setIsOn(nextIsOn);
    onChange(nextIsOn);
  }

  function handleClick() {
    updateToggle(!isOn);
  }

  function handleDragEnd(e) {
    if (isCloserToRightEdge(e)) {
      updateToggle(true);
    } else {
      updateToggle(false);
    }
  }

  // ...
}
```

With this approach, both the `Toggle` component and its parent component update their state during the event. React [batches updates](/learn/queueing-a-series-of-state-updates) from different components together, so there will only be one render pass.
通过这种方式，`Toggle` 组件及其父组件都在事件处理期间更新了各自的 state。React 会 [批量](/learn/queueing-a-series-of-state-updates) 处理来自不同组件的更新，所以只会有一个渲染流程。

You might also be able to remove the state altogether, and instead receive `isOn` from the parent component:
你也可以完全移除该 state，并从父组件中接收 `isOn`：

```js {1,2}
// ✅ Also good: the component is fully controlled by its parent
// ✅ 也很好：该组件完全由它的父组件控制
function Toggle({ isOn, onChange }) {
  function handleClick() {
    onChange(!isOn);
  }

  function handleDragEnd(e) {
    if (isCloserToRightEdge(e)) {
      onChange(true);
    } else {
      onChange(false);
    }
  }

  // ...
}
```

["Lifting state up"](/learn/sharing-state-between-components) lets the parent component fully control the `Toggle` by toggling the parent's own state. This means the parent component will have to contain more logic, but there will be less state overall to worry about. Whenever you try to keep two different state variables synchronized, try lifting state up instead!
“[状态提升](/learn/sharing-state-between-components)” 允许父组件通过切换自身的 state 来完全控制 `Toggle` 组件。这意味着父组件会包含更多的逻辑，但整体上需要关心的状态变少了。每当你尝试保持两个不同的 state 变量之间的同步时，试试状态提升！

### 将数据传递给父组件 | Passing data to the parent {/*passing-data-to-the-parent*/}

This `Child` component fetches some data and then passes it to the `Parent` component in an Effect:
`Child` 组件获取了一些数据并在 Effect 中传递给 `Parent` 组件：

```js {9-14}
function Parent() {
  const [data, setData] = useState(null);
  // ...
  return <Child onFetched={setData} />;
}

function Child({ onFetched }) {
  const data = useSomeAPI();
  // 🔴 Avoid: Passing data to the parent in an Effect
  // 🔴 避免：在 Effect 中传递数据给父组件
  useEffect(() => {
    if (data) {
      onFetched(data);
    }
  }, [onFetched, data]);
  // ...
}
```

In React, data flows from the parent components to their children. When you see something wrong on the screen, you can trace where the information comes from by going up the component chain until you find which component passes the wrong prop or has the wrong state. When child components update the state of their parent components in Effects, the data flow becomes very difficult to trace. Since both the child and the parent need the same data, let the parent component fetch that data, and *pass it down* to the child instead:
在 React 中，数据从父组件流向子组件。当你在屏幕上看到了一些错误时，你可以通过一路追踪组件树来寻找错误信息是从哪个组件传递下来的，从而找到传递了错误的 prop 或具有错误的 state 的组件。当子组件在 Effect 中更新其父组件的 state 时，数据流变得非常难以追踪。既然子组件和父组件都需要相同的数据，那么可以让父组件获取那些数据，并将其 **向下传递** 给子组件：

```js {4-5}
function Parent() {
  const data = useSomeAPI();
  // ...
  // ✅ Good: Passing data down to the child
  // ✅ 非常好：向子组件传递数据
  return <Child data={data} />;
}

function Child({ data }) {
  // ...
}
```

This is simpler and keeps the data flow predictable: the data flows down from the parent to the child.
这更简单，并且可以保持数据流的可预测性：数据从父组件流向子组件。

### 订阅外部 store | Subscribing to an external store {/*subscribing-to-an-external-store*/}

Sometimes, your components may need to subscribe to some data outside of the React state. This data could be from a third-party library or a built-in browser API. Since this data can change without React's knowledge, you need to manually subscribe your components to it. This is often done with an Effect, for example:
有时候，你的组件可能需要订阅 React state 之外的一些数据。这些数据可能来自第三方库或内置浏览器 API。由于这些数据可能在 React 无法感知的情况下发变化，你需要在你的组件中手动订阅它们。这经常使用 Effect 来实现，例如：

```js {2-17}
function useOnlineStatus() {
  // Not ideal: Manual store subscription in an Effect
  // 不理想：在 Effect 中手动订阅 store
  const [isOnline, setIsOnline] = useState(true);
  useEffect(() => {
    function updateState() {
      setIsOnline(navigator.onLine);
    }

    updateState();

    window.addEventListener('online', updateState);
    window.addEventListener('offline', updateState);
    return () => {
      window.removeEventListener('online', updateState);
      window.removeEventListener('offline', updateState);
    };
  }, []);
  return isOnline;
}

function ChatIndicator() {
  const isOnline = useOnlineStatus();
  // ...
}
```

Here, the component subscribes to an external data store (in this case, the browser `navigator.onLine` API). Since this API does not exist on the server (so it can't be used for the initial HTML), initially the state is set to `true`. Whenever the value of that data store changes in the browser, the component updates its state.
这个组件订阅了一个外部的 store 数据（在这里，是浏览器的 `navigator.onLine` API）。由于这个 API 在服务端不存在（因此不能用于初始的 HTML），因此 state 最初被设置为 `true`。每当浏览器 store 中的值发生变化时，组件都会更新它的 state。

Although it's common to use Effects for this, React has a purpose-built Hook for subscribing to an external store that is preferred instead. Delete the Effect and replace it with a call to [`useSyncExternalStore`](/reference/react/useSyncExternalStore):
尽管通常可以使用 Effect 来实现此功能，但 React 为此针对性地提供了一个 Hook 用于订阅外部 store。删除 Effect 并将其替换为调用 [`useSyncExternalStore`](/reference/react/useSyncExternalStore)：

```js {11-16}
function subscribe(callback) {
  window.addEventListener('online', callback);
  window.addEventListener('offline', callback);
  return () => {
    window.removeEventListener('online', callback);
    window.removeEventListener('offline', callback);
  };
}

function useOnlineStatus() {
  // ✅ Good: Subscribing to an external store with a built-in Hook
  // ✅ 非常好：用内置的 Hook 订阅外部 store
  return useSyncExternalStore(
    subscribe, // 只要传递的是同一个函数，React 不会重新订阅 // React won't resubscribe for as long as you pass the same function
    () => navigator.onLine, // 如何在客户端获取值 // How to get the value on the client
    () => true // 如何在服务端获取值 // How to get the value on the server
  );
}

function ChatIndicator() {
  const isOnline = useOnlineStatus();
  // ...
}
```

This approach is less error-prone than manually syncing mutable data to React state with an Effect. Typically, you'll write a custom Hook like `useOnlineStatus()` above so that you don't need to repeat this code in the individual components. [Read more about subscribing to external stores from React components.](/reference/react/useSyncExternalStore)
与手动使用 Effect 将可变数据同步到 React state 相比，这种方法能减少错误。通常，你可以写一个像上面的 `useOnlineStatus()` 这样的自定义 Hook，这样你就不需要在各个组件中重复写这些代码。[阅读更多关于在 React 组件中订阅外部数据 store 的信息](/reference/react/useSyncExternalStore)。

### 获取数据 | Fetching data {/*fetching-data*/}

Many apps use Effects to kick off data fetching. It is quite common to write a data fetching Effect like this:
许多应用使用 Effect 来发起数据获取请求。像这样在 Effect 中写一个数据获取请求是相当常见的：

```js {5-10}
function SearchResults({ query }) {
  const [results, setResults] = useState([]);
  const [page, setPage] = useState(1);

  useEffect(() => {
    // 🔴 Avoid: Fetching without cleanup logic
    // 🔴 避免：没有清除逻辑的获取数据
    fetchResults(query, page).then(json => {
      setResults(json);
    });
  }, [query, page]);

  function handleNextPageClick() {
    setPage(page + 1);
  }
  // ...
}
```

You *don't* need to move this fetch to an event handler.
你 **不需要** 把这个数据获取逻辑迁移到一个事件处理函数中。

This might seem like a contradiction with the earlier examples where you needed to put the logic into the event handlers! However, consider that it's not *the typing event* that's the main reason to fetch. Search inputs are often prepopulated from the URL, and the user might navigate Back and Forward without touching the input.
这可能看起来与之前需要将逻辑放入事件处理函数中的示例相矛盾！但是，考虑到这并不是 **键入事件**，这是在这里获取数据的主要原因。搜索输入框的值经常从 URL 中预填充，用户可以在不关心输入框的情况下导航到后退和前进页面。

It doesn't matter where `page` and `query` come from. While this component is visible, you want to keep `results` [synchronized](/learn/synchronizing-with-effects) with data from the network for the current `page` and `query`. This is why it's an Effect.
`page` 和 `query` 的来源其实并不重要。只要该组件可见，你就需要通过当前 `page` 和 `query` 的值，保持 `results` 和网络数据的 [同步](/learn/synchronizing-with-effects)。这就是这里是一个 Effect 的原因。

However, the code above has a bug. Imagine you type `"hello"` fast. Then the `query` will change from `"h"`, to `"he"`, `"hel"`, `"hell"`, and `"hello"`. This will kick off separate fetches, but there is no guarantee about which order the responses will arrive in. For example, the `"hell"` response may arrive *after* the `"hello"` response. Since it will call `setResults()` last, you will be displaying the wrong search results. This is called a ["race condition"](https://en.wikipedia.org/wiki/Race_condition): two different requests "raced" against each other and came in a different order than you expected.
然而，上面的代码有一个问题。假设你快速地输入 `“hello”`。那么 `query` 会从 `“h”` 变成 `“he”`，`“hel”`，`“hell”` 最后是 `“hello”`。这会触发一连串不同的数据获取请求，但无法保证对应的返回顺序。例如，`“hell”` 的响应可能在 `“hello”` 的响应 **之后** 返回。由于它的 `setResults()` 是在最后被调用的，你将会显示错误的搜索结果。这种情况被称为 “[竞态条件](https://en.wikipedia.org/wiki/Race_condition)”：两个不同的请求 “相互竞争”，并以与你预期不符的顺序返回。

**To fix the race condition, you need to [add a cleanup function](/learn/synchronizing-with-effects#fetching-data) to ignore stale responses:**
**为了修复这个问题，你需要添加一个 [清理函数](/learn/synchronizing-with-effects#fetching-data) 来忽略较早的返回结果：**

```js {5,7,9,11-13}
function SearchResults({ query }) {
  const [results, setResults] = useState([]);
  const [page, setPage] = useState(1);
  useEffect(() => {
    let ignore = false;
    fetchResults(query, page).then(json => {
      if (!ignore) {
        setResults(json);
      }
    });
    return () => {
      ignore = true;
    };
  }, [query, page]);

  function handleNextPageClick() {
    setPage(page + 1);
  }
  // ...
}
```

This ensures that when your Effect fetches data, all responses except the last requested one will be ignored.
这确保了当你在 Effect 中获取数据时，除了最后一次请求的所有返回结果都将被忽略。

Handling race conditions is not the only difficulty with implementing data fetching. You might also want to think about caching responses (so that the user can click Back and see the previous screen instantly), how to fetch data on the server (so that the initial server-rendered HTML contains the fetched content instead of a spinner), and how to avoid network waterfalls (so that a child can fetch data without waiting for every parent).
处理竞态条件并不是实现数据获取的唯一难点。你可能还需要考虑缓存响应结果（使用户点击后退按钮时可以立即看到先前的屏幕内容），如何在服务端获取数据（使服务端初始渲染的 HTML 中包含获取到的内容而不是加载动画），以及如何避免网络瀑布（使子组件不必等待每个父组件的数据获取完毕后才开始获取数据）。

**These issues apply to any UI library, not just React. Solving them is not trivial, which is why modern [frameworks](/learn/start-a-new-react-project#production-grade-react-frameworks) provide more efficient built-in data fetching mechanisms than fetching data in Effects.**
**这些问题适用于任何 UI 库，而不仅仅是 React。解决这些问题并不容易，这也是现代 [框架](/learn/start-a-new-react-project#production-grade-react-frameworks) 提供了比在 Effect 中获取数据更有效的内置数据获取机制的原因。**

If you don't use a framework (and don't want to build your own) but would like to make data fetching from Effects more ergonomic, consider extracting your fetching logic into a custom Hook like in this example:
如果你不使用框架（也不想开发自己的框架），但希望使从 Effect 中获取数据更符合人类直觉，请考虑像这个例子一样，将获取逻辑提取到一个自定义 Hook 中：

```js {4}
function SearchResults({ query }) {
  const [page, setPage] = useState(1);
  const params = new URLSearchParams({ query, page });
  const results = useData(`/api/search?${params}`);

  function handleNextPageClick() {
    setPage(page + 1);
  }
  // ...
}

function useData(url) {
  const [data, setData] = useState(null);
  useEffect(() => {
    let ignore = false;
    fetch(url)
      .then(response => response.json())
      .then(json => {
        if (!ignore) {
          setData(json);
        }
      });
    return () => {
      ignore = true;
    };
  }, [url]);
  return data;
}
```

You'll likely also want to add some logic for error handling and to track whether the content is loading. You can build a Hook like this yourself or use one of the many solutions already available in the React ecosystem. **Although this alone won't be as efficient as using a framework's built-in data fetching mechanism, moving the data fetching logic into a custom Hook will make it easier to adopt an efficient data fetching strategy later.**
你可能还想添加一些错误处理逻辑以及跟踪内容是否处于加载中。你可以自己编写这样的 Hook，也可以使用 React 生态中已经存在的许多解决方案。**虽然仅仅使用自定义 Hook 不如使用框架内置的数据获取机制高效，但将数据获取逻辑移动到自定义 Hook 中将使后续采用高效的数据获取策略更加容易。**

In general, whenever you have to resort to writing Effects, keep an eye out for when you can extract a piece of functionality into a custom Hook with a more declarative and purpose-built API like `useData` above. The fewer raw `useEffect` calls you have in your components, the easier you will find to maintain your application.
一般来说，当你不得不编写 Effect 时，请留意是否可以将某段功能提取到专门的内置 API 或一个更具声明性的自定义 Hook 中，比如上面的 `useData`。你会发现组件中的原始 `useEffect` 调用越少，维护应用将变得更加容易。

<Recap>

- If you can calculate something during render, you don't need an Effect.
- 如果你可以在渲染期间计算某些内容，则不需要使用 Effect。
- To cache expensive calculations, add `useMemo` instead of `useEffect`.
- 想要缓存昂贵的计算，请使用 `useMemo` 而不是 `useEffect`。
- To reset the state of an entire component tree, pass a different `key` to it.
- 想要重置整个组件树的 state，请传入不同的 `key`。
- To reset a particular bit of state in response to a prop change, set it during rendering.
- 想要在 prop 变化时重置某些特定的 state，请在渲染期间处理。
- Code that runs because a component was *displayed* should be in Effects, the rest should be in events.
- 组件 **显示** 时就需要执行的代码应该放在 Effect 中，否则应该放在事件处理函数中。
- If you need to update the state of several components, it's better to do it during a single event.
- 如果你需要更新多个组件的 state，最好在单个事件处理函数中处理。
- Whenever you try to synchronize state variables in different components, consider lifting state up.
- 当你尝试在不同组件中同步 state 变量时，请考虑状态提升。
- You can fetch data with Effects, but you need to implement cleanup to avoid race conditions.
- 你可以使用 Effect 获取数据，但你需要实现清除逻辑以避免竞态条件。

</Recap>

<Challenges>

#### 不用 Effect 转换数据 | Transform data without Effects {/*transform-data-without-effects*/}

The `TodoList` below displays a list of todos. When the "Show only active todos" checkbox is ticked, completed todos are not displayed in the list. Regardless of which todos are visible, the footer displays the count of todos that are not yet completed.
下面的 `TodoList` 显示了一个待办事项列表。当 “只显示未完成的事项” 复选框被勾选时，已完成的待办事项不会显示在列表中。无论哪些待办事项可见，页脚始终显示尚未完成的待办事项数量。

Simplify this component by removing all the unnecessary state and Effects.
通过移除不必要的 state 和 Effect 来简化这个组件。

<Sandpack>

```js
import { useState, useEffect } from 'react';
import { initialTodos, createTodo } from './todos.js';

export default function TodoList() {
  const [todos, setTodos] = useState(initialTodos);
  const [showActive, setShowActive] = useState(false);
  const [activeTodos, setActiveTodos] = useState([]);
  const [visibleTodos, setVisibleTodos] = useState([]);
  const [footer, setFooter] = useState(null);

  useEffect(() => {
    setActiveTodos(todos.filter(todo => !todo.completed));
  }, [todos]);

  useEffect(() => {
    setVisibleTodos(showActive ? activeTodos : todos);
  }, [showActive, todos, activeTodos]);

  useEffect(() => {
    setFooter(
      <footer>
        {activeTodos.length} 项待办
      </footer>
    );
  }, [activeTodos]);

  return (
    <>
      <label>
        <input
          type="checkbox"
          checked={showActive}
          onChange={e => setShowActive(e.target.checked)}
        />
        只显示未完成的事项
      </label>
      <NewTodo onAdd={newTodo => setTodos([...todos, newTodo])} />
      <ul>
        {visibleTodos.map(todo => (
          <li key={todo.id}>
            {todo.completed ? <s>{todo.text}</s> : todo.text}
          </li>
        ))}
      </ul>
      {footer}
    </>
  );
}

function NewTodo({ onAdd }) {
  const [text, setText] = useState('');

  function handleAddClick() {
    setText('');
    onAdd(createTodo(text));
  }

  return (
    <>
      <input value={text} onChange={e => setText(e.target.value)} />
      <button onClick={handleAddClick}>
        添加
      </button>
    </>
  );
}
```

```js src/todos.js
let nextId = 0;

export function createTodo(text, completed = false) {
  return {
    id: nextId++,
    text,
    completed
  };
}

export const initialTodos = [
  createTodo('买苹果', true),
  createTodo('买橘子', true),
  createTodo('买胡萝卜'),
];
```

```css
label { display: block; }
input { margin-top: 10px; }
```

</Sandpack>

<Hint>

If you can calculate something during rendering, you don't need state or an Effect that updates it.
如果你可以在渲染期间计算出某些值，那么就不需要使用 state 或 Effect 来更新它。

</Hint>

<Solution>

There are only two essential pieces of state in this example: the list of `todos` and the `showActive` state variable which represents whether the checkbox is ticked. All of the other state variables are [redundant](/learn/choosing-the-state-structure#avoid-redundant-state) and can be calculated during rendering instead. This includes the `footer` which you can move directly into the surrounding JSX.
这个例子中只有两个必要的 state 变量：`todos` 列表和代表复选框是否勾选的 `showActive`。其他所有的 state 变量都是 [多余的](/learn/choosing-the-state-structure#avoid-redundant-state)，可以在渲染期间计算得出。包括 `footer`，可以直接移到包含它的 JSX 中。

Your result should end up looking like this:
你的最终答案应该是这样：

<Sandpack>

```js
import { useState } from 'react';
import { initialTodos, createTodo } from './todos.js';

export default function TodoList() {
  const [todos, setTodos] = useState(initialTodos);
  const [showActive, setShowActive] = useState(false);
  const activeTodos = todos.filter(todo => !todo.completed);
  const visibleTodos = showActive ? activeTodos : todos;

  return (
    <>
      <label>
        <input
          type="checkbox"
          checked={showActive}
          onChange={e => setShowActive(e.target.checked)}
        />
        只显示未完成的事项
      </label>
      <NewTodo onAdd={newTodo => setTodos([...todos, newTodo])} />
      <ul>
        {visibleTodos.map(todo => (
          <li key={todo.id}>
            {todo.completed ? <s>{todo.text}</s> : todo.text}
          </li>
        ))}
      </ul>
      <footer>
        {activeTodos.length} 项待办
      </footer>
    </>
  );
}

function NewTodo({ onAdd }) {
  const [text, setText] = useState('');

  function handleAddClick() {
    setText('');
    onAdd(createTodo(text));
  }

  return (
    <>
      <input value={text} onChange={e => setText(e.target.value)} />
      <button onClick={handleAddClick}>
        添加
      </button>
    </>
  );
}
```

```js src/todos.js
let nextId = 0;

export function createTodo(text, completed = false) {
  return {
    id: nextId++,
    text,
    completed
  };
}

export const initialTodos = [
  createTodo('买苹果', true),
  createTodo('买橘子', true),
  createTodo('买胡萝卜'),
];
```

```css
label { display: block; }
input { margin-top: 10px; }
```

</Sandpack>

</Solution>

#### 不用 Effect 缓存计算结果 | Cache a calculation without Effects {/*cache-a-calculation-without-effects*/}

In this example, filtering the todos was extracted into a separate function called `getVisibleTodos()`. This function contains a `console.log()` call inside of it which helps you notice when it's being called. Toggle "Show only active todos" and notice that it causes `getVisibleTodos()` to re-run. This is expected because visible todos change when you toggle which ones to display.
在这个例子中，筛选 todos 的逻辑被提取到了一个叫做 `getVisibleTodos()` 的函数中，该函数内部包含一个 `console.log()`，它可以帮你了解函数的调用情况。切换 “只显示未完成的事项” 会导致 `getVisibleTodos()` 重新执行。这符合预期，因为切换未完成的待办事项会导致可见的待办事项发生变化。

Your task is to remove the Effect that recomputes the `visibleTodos` list in the `TodoList` component. However, you need to make sure that `getVisibleTodos()` does *not* re-run (and so does not print any logs) when you type into the input.
你的任务是移除 `TodoList` 组件中重复计算 `visibleTodos` 列表的 Effect。但是，你需要确保在输入框中输入时，`getVisibleTodos()` **不会** 重新执行（因此不会打印任何日志）。

<Hint>

One solution is to add a `useMemo` call to cache the visible todos. There is also another, less obvious solution.
一个解决方案是通过添加一个 `useMemo` 来缓存可见的 todos。还有另一种不太明显的解决方案。

</Hint>

<Sandpack>

```js
import { useState, useEffect } from 'react';
import { initialTodos, createTodo, getVisibleTodos } from './todos.js';

export default function TodoList() {
  const [todos, setTodos] = useState(initialTodos);
  const [showActive, setShowActive] = useState(false);
  const [text, setText] = useState('');
  const [visibleTodos, setVisibleTodos] = useState([]);

  useEffect(() => {
    setVisibleTodos(getVisibleTodos(todos, showActive));
  }, [todos, showActive]);

  function handleAddClick() {
    setText('');
    setTodos([...todos, createTodo(text)]);
  }

  return (
    <>
      <label>
        <input
          type="checkbox"
          checked={showActive}
          onChange={e => setShowActive(e.target.checked)}
        />
        只显示未完成的事项
      </label>
      <input value={text} onChange={e => setText(e.target.value)} />
      <button onClick={handleAddClick}>
        添加
      </button>
      <ul>
        {visibleTodos.map(todo => (
          <li key={todo.id}>
            {todo.completed ? <s>{todo.text}</s> : todo.text}
          </li>
        ))}
      </ul>
    </>
  );
}
```

```js src/todos.js
let nextId = 0;
let calls = 0;

export function getVisibleTodos(todos, showActive) {
  console.log(`getVisibleTodos() 被调用了 ${++calls} 次`);
  const activeTodos = todos.filter(todo => !todo.completed);
  const visibleTodos = showActive ? activeTodos : todos;
  return visibleTodos;
}

export function createTodo(text, completed = false) {
  return {
    id: nextId++,
    text,
    completed
  };
}

export const initialTodos = [
  createTodo('买苹果', true),
  createTodo('买橘子', true),
  createTodo('买胡萝卜'),
];
```

```css
label { display: block; }
input { margin-top: 10px; }
```

</Sandpack>

<Solution>

Remove the state variable and the Effect, and instead add a `useMemo` call to cache the result of calling `getVisibleTodos()`:
移除 state 变量和 Effect，取而代之的是添加一个 `useMemo` 来以缓存调用 `getVisibleTodos()` 的结果：

<Sandpack>

```js
import { useState, useMemo } from 'react';
import { initialTodos, createTodo, getVisibleTodos } from './todos.js';

export default function TodoList() {
  const [todos, setTodos] = useState(initialTodos);
  const [showActive, setShowActive] = useState(false);
  const [text, setText] = useState('');
  const visibleTodos = useMemo(
    () => getVisibleTodos(todos, showActive),
    [todos, showActive]
  );

  function handleAddClick() {
    setText('');
    setTodos([...todos, createTodo(text)]);
  }

  return (
    <>
      <label>
        <input
          type="checkbox"
          checked={showActive}
          onChange={e => setShowActive(e.target.checked)}
        />
        只显示未完成的事项
      </label>
      <input value={text} onChange={e => setText(e.target.value)} />
      <button onClick={handleAddClick}>
        添加
      </button>
      <ul>
        {visibleTodos.map(todo => (
          <li key={todo.id}>
            {todo.completed ? <s>{todo.text}</s> : todo.text}
          </li>
        ))}
      </ul>
    </>
  );
}
```

```js src/todos.js
let nextId = 0;
let calls = 0;

export function getVisibleTodos(todos, showActive) {
  console.log(`getVisibleTodos() 被调用了 ${++calls} 次`);
  const activeTodos = todos.filter(todo => !todo.completed);
  const visibleTodos = showActive ? activeTodos : todos;
  return visibleTodos;
}

export function createTodo(text, completed = false) {
  return {
    id: nextId++,
    text,
    completed
  };
}

export const initialTodos = [
  createTodo('买苹果', true),
  createTodo('买橘子', true),
  createTodo('买胡萝卜'),
];
```

```css
label { display: block; }
input { margin-top: 10px; }
```

</Sandpack>

With this change, `getVisibleTodos()` will be called only if `todos` or `showActive` change. Typing into the input only changes the `text` state variable, so it does not trigger a call to `getVisibleTodos()`.
通过这些改动，`getVisibleTodos()` 只有在 `todos` 或 `showActive` 变化时才会被调用。在输入框中输入只会更改 `text` 变量的值，并不会触发 `getVisibleTodos()`的调用。

There is also another solution which does not need `useMemo`. Since the `text` state variable can't possibly affect the list of todos, you can extract the `NewTodo` form into a separate component, and move the `text` state variable inside of it:
还有一个不使用 `useMemo` 的解决方案。由于 `text` 变量不可能影响待办事项列表，你可以将 `NewTodo` 表单提取到一个单独的组件中，并将 text 变量移动到其中：

<Sandpack>

```js
import { useState, useMemo } from 'react';
import { initialTodos, createTodo, getVisibleTodos } from './todos.js';

export default function TodoList() {
  const [todos, setTodos] = useState(initialTodos);
  const [showActive, setShowActive] = useState(false);
  const visibleTodos = getVisibleTodos(todos, showActive);

  return (
    <>
      <label>
        <input
          type="checkbox"
          checked={showActive}
          onChange={e => setShowActive(e.target.checked)}
        />
        只显示未完成的事项
      </label>
      <NewTodo onAdd={newTodo => setTodos([...todos, newTodo])} />
      <ul>
        {visibleTodos.map(todo => (
          <li key={todo.id}>
            {todo.completed ? <s>{todo.text}</s> : todo.text}
          </li>
        ))}
      </ul>
    </>
  );
}

function NewTodo({ onAdd }) {
  const [text, setText] = useState('');

  function handleAddClick() {
    setText('');
    onAdd(createTodo(text));
  }

  return (
    <>
      <input value={text} onChange={e => setText(e.target.value)} />
      <button onClick={handleAddClick}>
        添加
      </button>
    </>
  );
}
```

```js src/todos.js
let nextId = 0;
let calls = 0;

export function getVisibleTodos(todos, showActive) {
  console.log(`getVisibleTodos() 被调用了 ${++calls} 次`);
  const activeTodos = todos.filter(todo => !todo.completed);
  const visibleTodos = showActive ? activeTodos : todos;
  return visibleTodos;
}

export function createTodo(text, completed = false) {
  return {
    id: nextId++,
    text,
    completed
  };
}

export const initialTodos = [
  createTodo('买苹果', true),
  createTodo('买橘子', true),
  createTodo('买胡萝卜'),
];
```

```css
label { display: block; }
input { margin-top: 10px; }
```

</Sandpack>

This approach satisfies the requirements too. When you type into the input, only the `text` state variable updates. Since the `text` state variable is in the child `NewTodo` component, the parent `TodoList` component won't get re-rendered. This is why `getVisibleTodos()` doesn't get called when you type. (It would still be called if the `TodoList` re-renders for another reason.)
这种方法也满足要求。当你在输入框中输入时，只有 `text` 变量会更新。由于 `text` 变量在子组件 `NewTodo` 中，父组件 `TodoList` 不会重新渲染。这就是当你输入时 `getVisibleTodos()` 不会被调用的原因（如果 `TodoList` 由于其他原因被重新渲染了，它仍然会被调用）。

</Solution>

#### 不用 Effect 重置 state | Reset state without Effects {/*reset-state-without-effects*/}

This `EditContact` component receives a contact object shaped like `{ id, name, email }` as the `savedContact` prop. Try editing the name and email input fields. When you press Save, the contact's button above the form updates to the edited name. When you press Reset, any pending changes in the form are discarded. Play around with this UI to get a feel for it.
`EditContact` 组件的 prop `savedContact` 接收一个类似于 `{ id, name, email }` 这样的联系人对象。尝试编辑名称和邮箱输入框。当你点击保存按钮时，表单上方的联系人按钮会更新为编辑后的名称。当你点击重置按钮时，表单中的任何改动都会被丢弃。试着玩一玩儿这个用户界面感受一下。

When you select a contact with the buttons at the top, the form resets to reflect that contact's details. This is done with an Effect inside `EditContact.js`. Remove this Effect. Find another way to reset the form when `savedContact.id` changes.
当你用顶部按钮选择一个联系人时，该表单会重置并展示该联系人的详细信息。这是在 `EditContact.js` 内部使用 Effect 实现的。移除该 Effect，找到另一种方式在 `savedContact.id` 变化时重置表单。

<Sandpack>

```js src/App.js hidden
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
        savedContact={selectedContact}
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

```js src/ContactList.js hidden
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

```js src/EditContact.js active
import { useState, useEffect } from 'react';

export default function EditContact({ savedContact, onSave }) {
  const [name, setName] = useState(savedContact.name);
  const [email, setEmail] = useState(savedContact.email);

  useEffect(() => {
    setName(savedContact.name);
    setEmail(savedContact.email);
  }, [savedContact]);

  return (
    <section>
      <label>
        姓名：{' '}
        <input
          type="text"
          value={name}
          onChange={e => setName(e.target.value)}
        />
      </label>
      <label>
        邮箱：{' '}
        <input
          type="email"
          value={email}
          onChange={e => setEmail(e.target.value)}
        />
      </label>
      <button onClick={() => {
        const updatedData = {
          id: savedContact.id,
          name: name,
          email: email
        };
        onSave(updatedData);
      }}>
        保存
      </button>
      <button onClick={() => {
        setName(savedContact.name);
        setEmail(savedContact.email);
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

<Hint>

It would be nice if there was a way to tell React that when `savedContact.id` is different, the `EditContact` form is conceptually a _different contact's form_ and should not preserve state. Do you recall any such way?
如果有一种方式可以告诉 React，当 `savedContact.id` 变化时，`EditContact` 表单本质上是一个 **不同的联系人表单**，不应该保留状态，那将非常棒。你还记得这样的方式吗？

</Hint>

<Solution>

Split the `EditContact` component in two. Move all the form state into the inner `EditForm` component. Export the outer `EditContact` component, and make it pass `savedContact.id` as the `key` to the inner `EditForm` component. As a result, the inner `EditForm` component resets all of the form state and recreates the DOM whenever you select a different contact.
将 `EditContact` 组件拆分为两个组件。将所有表单 state 移动到内部的 `EditForm` 组件中。导出外部的 `EditContact` 组件，并将 `savedContact.id` 作为 `key` 传入内部的 `EditForm` 组件。结果是，每当你选择不同的联系人时，内部的 `EditForm` 组件会重置所有表单状态并重新创建 DOM。

<Sandpack>

```js src/App.js hidden
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
        savedContact={selectedContact}
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

```js src/ContactList.js hidden
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

```js src/EditContact.js active
import { useState } from 'react';

export default function EditContact(props) {
  return (
    <EditForm
      {...props}
      key={props.savedContact.id}
    />
  );
}

function EditForm({ savedContact, onSave }) {
  const [name, setName] = useState(savedContact.name);
  const [email, setEmail] = useState(savedContact.email);

  return (
    <section>
      <label>
        姓名：{' '}
        <input
          type="text"
          value={name}
          onChange={e => setName(e.target.value)}
        />
      </label>
      <label>
        邮箱：{' '}
        <input
          type="email"
          value={email}
          onChange={e => setEmail(e.target.value)}
        />
      </label>
      <button onClick={() => {
        const updatedData = {
          id: savedContact.id,
          name: name,
          email: email
        };
        onSave(updatedData);
      }}>
        保存
      </button>
      <button onClick={() => {
        setName(savedContact.name);
        setEmail(savedContact.email);
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

#### 不用 Effect 提交表单 | Submit a form without Effects {/*submit-a-form-without-effects*/}

This `Form` component lets you send a message to a friend. When you submit the form, the `showForm` state variable is set to `false`. This triggers an Effect calling `sendMessage(message)`, which sends the message (you can see it in the console). After the message is sent, you see a "Thank you" dialog with an "Open chat" button that lets you get back to the form.
`Form` 组件可以让你向朋友发送消息。当你提交表单时，state 变量 `showForm` 会被设置为 `false`。这会触发一个 Effect 调用 `sendMessage(message)` 发送消息（你可以在控制台中看到）。消息发送后，你会看到一个 “谢谢” 的提示语，里面有一个 “打开聊天” 按钮，让你回到表单。

Your app's users are sending way too many messages. To make chatting a little bit more difficult, you've decided to show the "Thank you" dialog *first* rather than the form. Change the `showForm` state variable to initialize to `false` instead of `true`. As soon as you make that change, the console will show that an empty message was sent. Something in this logic is wrong!
你的应用的用户发送的消息太多了。为了让聊天变得稍微困难一些，你决定 **先** 展示 “谢谢” 提示语，而不是表单。将 state 变量 `showForm` 的初始值改为 `false`，而不是 `true`。一旦你做了这些修改，控制台将发送一条空消息。这里的逻辑有问题！

What's the root cause of this problem? And how can you fix it?
导致这个问题的根本原因是什么？你能修复它吗？

<Hint>

Should the message be sent _because_ the user saw the "Thank you" dialog? Or is it the other way around?
是 **因为** 用户看到了 “谢谢” 提示语，才应该发送消息吗？还是其他什么原因？

</Hint>

<Sandpack>

```js
import { useState, useEffect } from 'react';

export default function Form() {
  const [showForm, setShowForm] = useState(true);
  const [message, setMessage] = useState('');

  useEffect(() => {
    if (!showForm) {
      sendMessage(message);
    }
  }, [showForm, message]);

  function handleSubmit(e) {
    e.preventDefault();
    setShowForm(false);
  }

  if (!showForm) {
    return (
      <>
        <h1>谢谢使用我们的服务！</h1>
        <button onClick={() => {
          setMessage('');
          setShowForm(true);
        }}>
          打开聊天
        </button>
      </>
    );
  }

  return (
    <form onSubmit={handleSubmit}>
      <textarea
        placeholder="消息"
        value={message}
        onChange={e => setMessage(e.target.value)}
      />
      <button type="submit" disabled={message === ''}>
        发送
      </button>
    </form>
  );
}

function sendMessage(message) {
  console.log('发送的消息：' + message);
}
```

```css
label, textarea { margin-bottom: 10px; display: block; }
```

</Sandpack>

<Solution>

The `showForm` state variable determines whether to show the form or the "Thank you" dialog. However, you aren't sending the message because the "Thank you" dialog was _displayed_. You want to send the message because the user has _submitted the form._ Delete the misleading Effect and move the `sendMessage` call inside the `handleSubmit` event handler:
state 变量 `showForm` 决定了显示表单还是 “谢谢” 提示语。然而，你并不是因为 “谢谢” 提示语被 **显示** 才发送消息的。你希望发送消息是因为用户 **提交了表单** 。删除误导性的 Effect，并将 `sendMessage` 调用移到 `handleSubmit` 事件处理函数中：

<Sandpack>

```js
import { useState, useEffect } from 'react';

export default function Form() {
  const [showForm, setShowForm] = useState(true);
  const [message, setMessage] = useState('');

  function handleSubmit(e) {
    e.preventDefault();
    setShowForm(false);
    sendMessage(message);
  }

  if (!showForm) {
    return (
      <>
        <h1>谢谢使用我们的服务！</h1>
        <button onClick={() => {
          setMessage('');
          setShowForm(true);
        }}>
          打开聊天
        </button>
      </>
    );
  }

  return (
    <form onSubmit={handleSubmit}>
      <textarea
        placeholder="消息"
        value={message}
        onChange={e => setMessage(e.target.value)}
      />
      <button type="submit" disabled={message === ''}>
        发送
      </button>
    </form>
  );
}

function sendMessage(message) {
  console.log('发送的消息： ' + message);
}
```

```css
label, textarea { margin-bottom: 10px; display: block; }
```

</Sandpack>

Notice how in this version, only _submitting the form_ (which is an event) causes the message to be sent. It works equally well regardless of whether `showForm` is initially set to `true` or `false`. (Set it to `false` and notice no extra console messages.)
注意在这个版本中，只有 **提交表单**（这是一个事件）才会导致消息被发送。采用这种方案，无论 `showForm` 最初被设置为 `true` 还是 `false` 都同样有效（将其设置为 `false`，注意没有额外的控制台消息）。

</Solution>

</Challenges>
