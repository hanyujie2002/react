---
title: '使用 ref 引用值 | Referencing Values with Refs'
translators: 
  - SylviaZ89
  - Neo42
---

<Intro>

When you want a component to "remember" some information, but you don't want that information to [trigger new renders](/learn/render-and-commit), you can use a *ref*.
当你希望组件“记住”某些信息，但又不想让这些信息 [触发新的渲染](/learn/render-and-commit) 时，你可以使用 **ref** 。

</Intro>

<YouWillLearn>

- 如何向组件添加 ref | How to add a ref to your component
- 如何更新 ref 的值 | How to update a ref's value
- ref 与 state 有何不同 | How refs are different from state
- 如何安全地使用 ref | How to use refs safely

</YouWillLearn>

## 给你的组件添加 ref | Adding a ref to your component {/*adding-a-ref-to-your-component*/}

You can add a ref to your component by importing the `useRef` Hook from React:
你可以通过从 React 导入 `useRef` Hook 来为你的组件添加一个 ref：

```js
import { useRef } from 'react';
```

Inside your component, call the `useRef` Hook and pass the initial value that you want to reference as the only argument. For example, here is a ref to the value `0`:
在你的组件内，调用 `useRef` Hook 并传入你想要引用的初始值作为唯一参数。例如，这里的 ref 引用的值是“0”：

```js
const ref = useRef(0);
```

`useRef` returns an object like this:
`useRef` 返回一个这样的对象:

```js
{ 
  current: 0 // 你向 useRef 传入的值 // The value you passed to useRef
}
```

<Illustration src="/images/docs/illustrations/i_ref.png" alt="An arrow with 'current' written on it stuffed into a pocket with 'ref' written on it." />

You can access the current value of that ref through the `ref.current` property. This value is intentionally mutable, meaning you can both read and write to it. It's like a secret pocket of your component that React doesn't track. (This is what makes it an "escape hatch" from React's one-way data flow--more on that below!)
你可以用 `ref.current` 属性访问该 ref 的当前值。这个值是有意被设置为可变的，意味着你既可以读取它也可以写入它。就像一个 React 追踪不到的、用来存储组件信息的秘密“口袋”。（这就是让它成为 React 单向数据流的“脱围机制”的原因 —— 详见下文！）

Here, a button will increment `ref.current` on every click:
这里，每次点击按钮时会使 `ref.current` 递增：

<Sandpack>

```js
import { useRef } from 'react';

export default function Counter() {
  let ref = useRef(0);

  function handleClick() {
    ref.current = ref.current + 1;
    alert('你点击了 ' + ref.current + ' 次！');
  }

  return (
    <button onClick={handleClick}>
      点击我！
    </button>
  );
}
```

</Sandpack>

The ref points to a number, but, like [state](/learn/state-a-components-memory), you could point to anything: a string, an object, or even a function. Unlike state, ref is a plain JavaScript object with the `current` property that you can read and modify.
这里的 ref 指向一个数字，但是，像 [state](/learn/state-a-components-memory) 一样，你可以让它指向任何东西：字符串、对象，甚至是函数。与 state 不同的是，ref 是一个普通的 JavaScript 对象，具有可以被读取和修改的 `current` 属性。

Note that **the component doesn't re-render with every increment.** Like state, refs are retained by React between re-renders. However, setting state re-renders a component. Changing a ref does not!
请注意，**组件不会在每次递增时重新渲染。** 与 state 一样，React 会在每次重新渲染之间保留 ref。但是，设置 state 会重新渲染组件，更改 ref 不会！

## 示例：制作秒表 | Example: building a stopwatch {/*example-building-a-stopwatch*/}

You can combine refs and state in a single component. For example, let's make a stopwatch that the user can start or stop by pressing a button. In order to display how much time has passed since the user pressed "Start", you will need to keep track of when the Start button was pressed and what the current time is. **This information is used for rendering, so you'll keep it in state:**
你可以在单个组件中把 ref 和 state 结合起来使用。例如，让我们制作一个秒表，用户可以通过按按钮来使其启动或停止。为了显示从用户按下“开始”以来经过的时间长度，你需要追踪按下“开始”按钮的时间和当前时间。**此信息用于渲染，所以你会把它保存在 state 中：**

```js
const [startTime, setStartTime] = useState(null);
const [now, setNow] = useState(null);
```

When the user presses "Start", you'll use [`setInterval`](https://developer.mozilla.org/docs/Web/API/setInterval) in order to update the time every 10 milliseconds:
当用户按下“开始”时，你将用 [`setInterval`](https://developer.mozilla.org/docs/Web/API/setInterval) 每 10 毫秒更新一次时间：

<Sandpack>

```js
import { useState } from 'react';

export default function Stopwatch() {
  const [startTime, setStartTime] = useState(null);
  const [now, setNow] = useState(null);

  function handleStart() {
    // Start counting.
    // 开始计时。
    setStartTime(Date.now());
    setNow(Date.now());

    setInterval(() => {
      // Update the current time every 10ms.
      // 每 10ms 更新一次当前时间。
      setNow(Date.now());
    }, 10);
  }

  let secondsPassed = 0;
  if (startTime != null && now != null) {
    secondsPassed = (now - startTime) / 1000;
  }

  return (
    <>
      <h1>时间过去了： {secondsPassed.toFixed(3)}</h1>
      <button onClick={handleStart}>
        开始
      </button>
    </>
  );
}
```

</Sandpack>

When the "Stop" button is pressed, you need to cancel the existing interval so that it stops updating the `now` state variable. You can do this by calling [`clearInterval`](https://developer.mozilla.org/en-US/docs/Web/API/clearInterval), but you need to give it the interval ID that was previously returned by the `setInterval` call when the user pressed Start. You need to keep the interval ID somewhere. **Since the interval ID is not used for rendering, you can keep it in a ref:**
当按下“停止”按钮时，你需要取消现有的 interval，以便让它停止更新 `now` state 变量。你可以通过调用 [`clearInterval`](https://developer.mozilla.org/zh-CN/docs/Web/API/clearInterval) 来完成此操作。但你需要为其提供 interval ID，此 ID 是之前用户按下 Start、调用 `setInterval` 时返回的。你需要将 interval ID 保留在某处。 **由于 interval ID 不用于渲染，你可以将其保存在 ref 中：**

<Sandpack>

```js
import { useState, useRef } from 'react';

export default function Stopwatch() {
  const [startTime, setStartTime] = useState(null);
  const [now, setNow] = useState(null);
  const intervalRef = useRef(null);

  function handleStart() {
    setStartTime(Date.now());
    setNow(Date.now());

    clearInterval(intervalRef.current);
    intervalRef.current = setInterval(() => {
      setNow(Date.now());
    }, 10);
  }

  function handleStop() {
    clearInterval(intervalRef.current);
  }

  let secondsPassed = 0;
  if (startTime != null && now != null) {
    secondsPassed = (now - startTime) / 1000;
  }

  return (
    <>
      <h1>时间过去了： {secondsPassed.toFixed(3)}</h1>
      <button onClick={handleStart}>
        开始
      </button>
      <button onClick={handleStop}>
        停止
      </button>
    </>
  );
}
```

</Sandpack>

When a piece of information is used for rendering, keep it in state. When a piece of information is only needed by event handlers and changing it doesn't require a re-render, using a ref may be more efficient.
当一条信息用于渲染时，将它保存在 state 中。当一条信息仅被事件处理器需要，并且更改它不需要重新渲染时，使用 ref 可能会更高效。 

## ref 和 state 的不同之处 | Differences between refs and state {/*differences-between-refs-and-state*/}

Perhaps you're thinking refs seem less "strict" than state—you can mutate them instead of always having to use a state setting function, for instance. But in most cases, you'll want to use state. Refs are an "escape hatch" you won't need often. Here's how state and refs compare:
也许你觉得 ref 似乎没有 state 那样“严格” —— 例如，你可以改变它们而非总是必须使用 state 设置函数。但在大多数情况下，我们建议你使用 state。ref 是一种“脱围机制”，你并不会经常用到它。 以下是 state 和 ref 的对比：

| ref                                                                                  | state                                                                                                                     |
| ------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------- |
| `useRef(initialValue)` returns `{ current: initialValue }`<br/>`useRef(initialValue)`返回 `{ current: initialValue }`  | `useState(initialValue)` returns the current value of a state variable and a state setter function ( `[value, setValue]`)<br/>`useState(initialValue)` 返回 state 变量的当前值和一个 state 设置函数 ( `[value, setValue]`) |
|  Doesn't trigger re-render when you change it.<br/>更改时不会触发重新渲染   | Triggers re-render when you change it.<br/>更改时触发重新渲染。 |
| Mutable—you can modify and update `current`'s value outside of the rendering process.<br/>可变 —— 你可以在渲染过程之外修改和更新 `current` 的值。 | "Immutable"—you must use the state setting function to modify state variables to queue a re-render.<br/>“不可变” —— 你必须使用 state 设置函数来修改 state 变量，从而排队重新渲染。 |
| You shouldn't read (or write) the `current` value during rendering.<br/>你不应在渲染期间读取（或写入） `current` 值。 | You can read state at any time. However, each render has its own [snapshot](/learn/state-as-a-snapshot) of state which does not change.<br/>你可以随时读取 state。但是，每次渲染都有自己不变的 state [快照](/learn/state-as-a-snapshot)。

Here is a counter button that's implemented with state:
这是一个使用 state 实现的计数器按钮：

<Sandpack>

```js
import { useState } from 'react';

export default function Counter() {
  const [count, setCount] = useState(0);

  function handleClick() {
    setCount(count + 1);
  }

  return (
    <button onClick={handleClick}>
      你点击了 {count} 次
    </button>
  );
}
```

</Sandpack>

Because the `count` value is displayed, it makes sense to use a state value for it. When the counter's value is set with `setCount()`, React re-renders the component and the screen updates to reflect the new count.
因为 `count` 的值将会被显示，所以为其使用 state 是合理的。当使用 setCount() 设置计数器的值时，React 会重新渲染组件，并且屏幕会更新以展示新的计数。

If you tried to implement this with a ref, React would never re-render the component, so you'd never see the count change! See how clicking this button **does not update its text**:
如果你试图用 ref 来实现它，React 永远不会重新渲染组件，所以你永远不会看到计数变化！看看点击这个按钮如何 **不更新它的文本**：

<Sandpack>

```js
import { useRef } from 'react';

export default function Counter() {
  let countRef = useRef(0);

  function handleClick() {
    // This doesn't re-render the component!
    // 这样并未重新渲染组件！
    countRef.current = countRef.current + 1;
  }

  return (
    <button onClick={handleClick}>
      你点击了 {countRef.current} 次
    </button>
  );
}
```

</Sandpack>

This is why reading `ref.current` during render leads to unreliable code. If you need that, use state instead.
这就是在渲染期间读取 `ref.current` 会导致代码不可靠的原因。如果需要，请改用 state。 

<DeepDive>

#### useRef 内部是如何运行的？| How does useRef work inside? {/*how-does-use-ref-work-inside*/}

Although both `useState` and `useRef` are provided by React, in principle `useRef` could be implemented _on top of_ `useState`. You can imagine that inside of React, `useRef` is implemented like this:
尽管 `useState` 和 `useRef` 都是由 React 提供的，原则上 `useRef` 可以在 `useState` **的基础上** 实现。 你可以想象在 React 内部，`useRef` 是这样实现的：

```js
// Inside of React
// React 内部
function useRef(initialValue) {
  const [ref, unused] = useState({ current: initialValue });
  return ref;
}
```

During the first render, `useRef` returns `{ current: initialValue }`. This object is stored by React, so during the next render the same object will be returned. Note how the state setter is unused in this example. It is unnecessary because `useRef` always needs to return the same object!
第一次渲染期间，`useRef` 返回 `{ current: initialValue }`。 该对象由 React 存储，因此在下一次渲染期间将返回相同的对象。 请注意，在这个示例中，state 设置函数没有被用到。它是不必要的，因为 `useRef` 总是需要返回相同的对象！

React provides a built-in version of `useRef` because it is common enough in practice. But you can think of it as a regular state variable without a setter. If you're familiar with object-oriented programming, refs might remind you of instance fields--but instead of `this.something` you write `somethingRef.current`.
React 提供了一个内置版本的 `useRef`，因为它在实践中很常见。 但是你可以将其视为没有设置函数的常规 state 变量。 如果你熟悉面向对象编程，ref 可能会让你想起实例字段 —— 但是你写的不是 `this.something`，而是 `somethingRef.current`。

</DeepDive>

## 何时使用 ref | When to use refs {/*when-to-use-refs*/}

Typically, you will use a ref when your component needs to "step outside" React and communicate with external APIs—often a browser API that won't impact the appearance of the component. Here are a few of these rare situations:
通常，当你的组件需要“跳出” React 并与外部 API 通信时，你会用到 ref —— 通常是不会影响组件外观的浏览器 API。以下是这些罕见情况中的几个：

- Storing [timeout IDs](https://developer.mozilla.org/docs/Web/API/setTimeout)
- 存储 [timeout ID](https://developer.mozilla.org/docs/Web/API/setTimeout)
- Storing and manipulating [DOM elements](https://developer.mozilla.org/docs/Web/API/Element), which we cover on [the next page](/learn/manipulating-the-dom-with-refs)
- 存储和操作 [DOM 元素](https://developer.mozilla.org/docs/Web/API/Element)，我们将在 [下一页](/learn/manipulating-the-dom-with-refs) 中介绍 
- Storing other objects that aren't necessary to calculate the JSX.
- 存储不需要被用来计算 JSX 的其他对象。

If your component needs to store some value, but it doesn't impact the rendering logic, choose refs.
如果你的组件需要存储一些值，但不影响渲染逻辑，请选择 ref。

## ref 的最佳实践 | Best practices for refs {/*best-practices-for-refs*/}

Following these principles will make your components more predictable:
遵循这些原则将使你的组件更具可预测性：

- **Treat refs as an escape hatch.** Refs are useful when you work with external systems or browser APIs. If much of your application logic and data flow relies on refs, you might want to rethink your approach.
- **将 ref 视为脱围机制**。当你使用外部系统或浏览器 API 时，ref 很有用。如果你很大一部分应用程序逻辑和数据流都依赖于 ref，你可能需要重新考虑你的方法。
- **Don't read or write `ref.current` during rendering.** If some information is needed during rendering, use [state](/learn/state-a-components-memory) instead. Since React doesn't know when `ref.current` changes, even reading it while rendering makes your component's behavior difficult to predict. (The only exception to this is code like `if (!ref.current) ref.current = new Thing()` which only sets the ref once during the first render.)
- **不要在渲染过程中读取或写入 `ref.current`。** 如果渲染过程中需要某些信息，请使用 [state](/learn/state-a-components-memory) 代替。由于 React 不知道 `ref.current` 何时发生变化，即使在渲染时读取它也会使组件的行为难以预测。（唯一的例外是像 `if (!ref.current) ref.current = new Thing()` 这样的代码，它只在第一次渲染期间设置一次 ref。）

Limitations of React state don't apply to refs. For example, state acts like a [snapshot for every render](/learn/state-as-a-snapshot) and [doesn't update synchronously.](/learn/queueing-a-series-of-state-updates) But when you mutate the current value of a ref, it changes immediately:
React state 的限制不适用于 ref。例如，state 就像 [每次渲染的快照](/learn/state-as-a-snapshot)，并且 [不会同步更新](/learn/queueing-a-series-of-state-updates)。但是当你改变 ref 的 current 值时，它会立即改变：

```js
ref.current = 5;
console.log(ref.current); // 5
```

This is because **the ref itself is a regular JavaScript object,** and so it behaves like one.
这是因为 **ref 本身是一个普通的 JavaScript 对象，** 所以它的行为就像对象那样。

You also don't need to worry about [avoiding mutation](/learn/updating-objects-in-state) when you work with a ref. As long as the object you're mutating isn't used for rendering, React doesn't care what you do with the ref or its contents.
当你使用 ref 时，也无需担心 [避免变更](/learn/updating-objects-in-state)。只要你改变的对象不用于渲染，React 就不会关心你对 ref 或其内容做了什么。

## ref 和 DOM | Refs and the DOM {/*refs-and-the-dom*/}

You can point a ref to any value. However, the most common use case for a ref is to access a DOM element. For example, this is handy if you want to focus an input programmatically. When you pass a ref to a `ref` attribute in JSX, like `<div ref={myRef}>`, React will put the corresponding DOM element into `myRef.current`. Once the element is removed from the DOM, React will update `myRef.current` to be `null`. You can read more about this in [Manipulating the DOM with Refs.](/learn/manipulating-the-dom-with-refs)
你可以将 ref 指向任何值。但是，ref 最常见的用法是访问 DOM 元素。例如，如果你想以编程方式聚焦一个输入框，这种用法就会派上用场。当你将 ref 传递给 JSX 中的 `ref` 属性时，比如 `<div ref={myRef}>`，React 会将相应的 DOM 元素放入 `myRef.current` 中。当元素从 DOM 中删除时，React 会将 `myRef.current` 更新为 `null`。你可以在 [使用 ref 操作 DOM](/learn/manipulating-the-dom-with-refs) 中阅读更多相关信息。

<Recap>

- Refs are an escape hatch to hold onto values that aren't used for rendering. You won't need them often.
- ref 是一种脱围机制，用于保留不用于渲染的值。 你不会经常需要它们。
- A ref is a plain JavaScript object with a single property called `current`, which you can read or set.
- ref 是一个普通的 JavaScript 对象，具有一个名为 `current` 的属性，你可以对其进行读取或设置。
- You can ask React to give you a ref by calling the `useRef` Hook.
- 你可以通过调用 `useRef` Hook 来让 React 给你一个 ref。
- Like state, refs let you retain information between re-renders of a component.
- 与 state 一样，ref 允许你在组件的重新渲染之间保留信息。
- Unlike state, setting the ref's `current` value does not trigger a re-render.
- 与 state 不同，设置 ref 的 `current` 值不会触发重新渲染。
- Don't read or write `ref.current` during rendering. This makes your component hard to predict.
- 不要在渲染过程中读取或写入 `ref.current`。这使你的组件难以预测。

</Recap>



<Challenges>

#### 修复坏掉的聊天输入框 | Fix a broken chat input {/*fix-a-broken-chat-input*/}

Type a message and click "Send". You will notice there is a three second delay before you see the "Sent!" alert. During this delay, you can see an "Undo" button. Click it. This "Undo" button is supposed to stop the "Sent!" message from appearing. It does this by calling [`clearTimeout`](https://developer.mozilla.org/en-US/docs/Web/API/clearTimeout) for the timeout ID saved during `handleSend`. However, even after "Undo" is clicked, the "Sent!" message still appears. Find why it doesn't work, and fix it.
输入消息并单击“发送”。你会注意到，在看到“已发送！”提示框之前有 3 秒的延迟。在此延迟期间，你可以看到一个“撤消”按钮。点击它。这个“撤消”按钮应该阻止“发送！”消息弹出。它通过调用 [`clearTimeout`](https://developer.mozilla.org/zh-CN/docs/Web/API/clearTimeout) 来做到这点，这一步骤需要使用在 `handleSend` 时保存的 timeout ID。但是，即使在单击“撤消”后，“已发送！”消息仍然出现。找出它不起作用的原因，然后修复它。

<Hint>

Regular variables like `let timeoutID` don't "survive" between re-renders because every render runs your component (and initializes its variables) from scratch. Should you keep the timeout ID somewhere else?
像`let timeoutID` 这样的常规变量不会在重新渲染之间“存活”，因为每次渲染都从头开始运行你的组件（并初始化其变量）。你应该将 timeout ID 保存在其他地方吗？

</Hint>

<Sandpack>

```js
import { useState } from 'react';

export default function Chat() {
  const [text, setText] = useState('');
  const [isSending, setIsSending] = useState(false);
  let timeoutID = null;

  function handleSend() {
    setIsSending(true);
    timeoutID = setTimeout(() => {
      alert('已发送！');
      setIsSending(false);
    }, 3000);
  }

  function handleUndo() {
    setIsSending(false);
    clearTimeout(timeoutID);
  }

  return (
    <>
      <input
        disabled={isSending}
        value={text}
        onChange={e => setText(e.target.value)}
      />
      <button
        disabled={isSending}
        onClick={handleSend}>
        {isSending ? '发送中……' : '发送'}
      </button>
      {isSending &&
        <button onClick={handleUndo}>
          撤销
        </button>
      }
    </>
  );
}
```

</Sandpack>

<Solution>

Whenever your component re-renders (such as when you set state), all local variables get initialized from scratch. This is why you can't save the timeout ID in a local variable like `timeoutID` and then expect another event handler to "see" it in the future. Instead, store it in a ref, which React will preserve between renders.
每当你的组件重新渲染时（例如当你设置 state 时），所有局部变量都会从头开始初始化。这就是为什么你不能将 timeout ID 保存在像 `timeoutID` 这样的局部变量中，然后期望未来另一个事件处理器“看到”它。相反，将它存储在一个 ref 中，React 将在渲染之间保留它。

<Sandpack>

```js
import { useState, useRef } from 'react';

export default function Chat() {
  const [text, setText] = useState('');
  const [isSending, setIsSending] = useState(false);
  const timeoutRef = useRef(null);

  function handleSend() {
    setIsSending(true);
    timeoutRef.current = setTimeout(() => {
      alert('已发送!');
      setIsSending(false);
    }, 3000);
  }

  function handleUndo() {
    setIsSending(false);
    clearTimeout(timeoutRef.current);
  }

  return (
    <>
      <input
        disabled={isSending}
        value={text}
        onChange={e => setText(e.target.value)}
      />
      <button
        disabled={isSending}
        onClick={handleSend}>
        {isSending ? '发送中……' : '发送'}
      </button>
      {isSending &&
        <button onClick={handleUndo}>
          撤销
        </button>
      }
    </>
  );
}
```

</Sandpack>

</Solution>


#### 修复无法重新渲染的组件 | Fix a component failing to re-render {/*fix-a-component-failing-to-re-render*/}

This button is supposed to toggle between showing "On" and "Off". However, it always shows "Off". What is wrong with this code? Fix it.
这个按钮本该在显示“开”和“关”之间切换。但是，它始终显示“关”。这段代码有什么问题？修复它。

<Sandpack>

```js
import { useRef } from 'react';

export default function Toggle() {
  const isOnRef = useRef(false);

  return (
    <button onClick={() => {
      isOnRef.current = !isOnRef.current;
    }}>
      {isOnRef.current ? '开' : '关'}
    </button>
  );
}
```

</Sandpack>

<Solution>

In this example, the current value of a ref is used to calculate the rendering output: `{isOnRef.current ? 'On' : 'Off'}`. This is a sign that this information should not be in a ref, and should have instead been put in state. To fix it, remove the ref and use state instead:
在这个例子中，ref 的 current 值被用于计算渲染输出：`{isOnRef.current ? '开'：'关'}`。这表明此信息本来不应该在 ref 中，而应该放在 state 里。要修复它，请删除 ref ，使用 state 代替：

<Sandpack>

```js
import { useState } from 'react';

export default function Toggle() {
  const [isOn, setIsOn] = useState(false);

  return (
    <button onClick={() => {
      setIsOn(!isOn);
    }}>
      {isOn ? '开' : '关'}
    </button>
  );
}
```

</Sandpack>

</Solution>

#### 修复防抖 | Fix debouncing {/*fix-debouncing*/}

In this example, all button click handlers are ["debounced".](https://redd.one/blog/debounce-vs-throttle) To see what this means, press one of the buttons. Notice how the message appears a second later. If you press the button while waiting for the message, the timer will reset. So if you keep clicking the same button fast many times, the message won't appear until a second *after* you stop clicking. Debouncing lets you delay some action until the user "stops doing things".
在这个例子中，所有按钮点击处理器都是 ["防抖的"](https://redd.one/blog/debounce-vs-throttle)。 要了解这意味着什么，请按下其中一个按钮。注意消息在一秒后显示。如果你在等待消息时按下按钮，计时器将重置。因此如果你多次快速单击同一个按钮，则直到你停止单击 **之后** 1 秒钟，该消息才会显示。防抖可以让你将一些动作推迟到用户“停止动作”之后。

This example works, but not quite as intended. The buttons are not independent. To see the problem, click one of the buttons, and then immediately click another button. You'd expect that after a delay, you would see both button's messages. But only the last button's message shows up. The first button's message gets lost.
这个例子可以正常运行，但并不完全符合预期。按钮不是独立的。要查看问题，请单击其中一个按钮，然后立即单击另一个按钮。你本来期望在延迟之后，你会看到两个按钮的消息。但只有最后一个按钮的消息出现了。第一个按钮的消息丢失了。

Why are the buttons interfering with each other? Find and fix the issue.
为什么按钮会相互干扰呢？查找并修复问题。

<Hint>

The last timeout ID variable is shared between all `DebouncedButton` components. This is why clicking one button resets another button's timeout. Can you store a separate timeout ID for each button?
最后一个 timeout ID 变量是被所有 `DebouncedButton` 组件所共享的。这就是为什么单击一个按钮会重置另一个按钮的 timeout 时间。你能为每个按钮存储一个单独的 timeout ID 吗？

</Hint>

<Sandpack>

```js
let timeoutID;

function DebouncedButton({ onClick, children }) {
  return (
    <button onClick={() => {
      clearTimeout(timeoutID);
      timeoutID = setTimeout(() => {
        onClick();
      }, 1000);
    }}>
      {children}
    </button>
  );
}

export default function Dashboard() {
  return (
    <>
      <DebouncedButton
        onClick={() => alert('宇宙飞船已发射！')}
      >
        发射宇宙飞船
      </DebouncedButton>
      <DebouncedButton
        onClick={() => alert('汤煮好了！')}
      >
        煮点儿汤
      </DebouncedButton>
      <DebouncedButton
        onClick={() => alert('摇篮曲唱完了！')}
      >
        唱首摇篮曲
      </DebouncedButton>
    </>
  )
}
```

```css
button { display: block; margin: 10px; }
```

</Sandpack>

<Solution>

A variable like `timeoutID` is shared between all components. This is why clicking on the second button resets the first button's pending timeout. To fix this, you can keep timeout in a ref. Each button will get its own ref, so they won't conflict with each other. Notice how clicking two buttons fast will show both messages.
像 `timeoutID` 这样的变量是被所有组件共享的。这就是单击第二个按钮会重置第一个按钮未完成的 timeout 的原因。要解决此问题，你可以把 timeout 保存在 ref 中。每个按钮都有自己的 ref，因此它们不会相互冲突。请注意快速单击两个按钮如何显示两个消息。

<Sandpack>

```js
import { useRef } from 'react';

function DebouncedButton({ onClick, children }) {
  const timeoutRef = useRef(null);
  return (
    <button onClick={() => {
      clearTimeout(timeoutRef.current);
      timeoutRef.current = setTimeout(() => {
        onClick();
      }, 1000);
    }}>
      {children}
    </button>
  );
}

export default function Dashboard() {
  return (
    <>
      <DebouncedButton
        onClick={() => alert('宇宙飞船已发射！')}
      >
        发射宇宙飞船
      </DebouncedButton>
      <DebouncedButton
        onClick={() => alert('汤煮好了！')}
      >
        煮点儿汤
      </DebouncedButton>
      <DebouncedButton
        onClick={() => alert('摇篮曲唱完了！')}
      >
        唱首摇篮曲
      </DebouncedButton>
    </>
  )
}
```

```css
button { display: block; margin: 10px; }
```

</Sandpack>

</Solution>

#### 读取最新的 state | Read the latest state {/*read-the-latest-state*/}

In this example, after you press "Send", there is a small delay before the message is shown. Type "hello", press Send, and then quickly edit the input again. Despite your edits, the alert would still show "hello" (which was the value of state [at the time](/learn/state-as-a-snapshot#state-over-time) the button was clicked).
在此示例中，当你按下“发送”后，在显示消息之前会有一小段延迟。输入“你好”，按下发送，然后再次快速编辑输入。尽管你进行了编辑，提示框仍会显示“你好”（这是按钮被点击 [那一刻](/learn/state-as-a-snapshot#state-over-time) state 的值）。

Usually, this behavior is what you want in an app. However, there may be occasional cases where you want some asynchronous code to read the *latest* version of some state. Can you think of a way to make the alert show the *current* input text rather than what it was at the time of the click?
通常，这种行为是你在应用程序中想要的。但是，有时可能需要一些异步代码来读取某些 state 的 **最新** 版本。你能想出一种方法，让提示框显示 **当前** 输入文本而不是点击时的内容吗？

<Sandpack>

```js
import { useState, useRef } from 'react';

export default function Chat() {
  const [text, setText] = useState('');

  function handleSend() {
    setTimeout(() => {
      alert('正在发送：' + text);
    }, 3000);
  }

  return (
    <>
      <input
        value={text}
        onChange={e => setText(e.target.value)}
      />
      <button
        onClick={handleSend}>
        发送
      </button>
    </>
  );
}
```

</Sandpack>

<Solution>

State works [like a snapshot](/learn/state-as-a-snapshot), so you can't read the latest state from an asynchronous operation like a timeout. However, you can keep the latest input text in a ref. A ref is mutable, so you can read the `current` property at any time. Since the current text is also used for rendering, in this example, you will need *both* a state variable (for rendering), *and* a ref (to read it in the timeout). You will need to update the current ref value manually.
state 运作起来 [就像快照](/learn/state-as-a-snapshot)，因此你无法从 timeout 等异步操作中读取最新的 state。但是，你可以在 ref 中保存最新的输入文本。ref 是可变的，因此你可以随时读取 `current` 属性。由于当前文本也用于渲染，在这个例子中，你需要 **同时** 使用一个 state 变量（用于渲染）**和** 一个 ref（在 timeout 时读取它）。你需要手动更新当前的 ref 值。

<Sandpack>

```js
import { useState, useRef } from 'react';

export default function Chat() {
  const [text, setText] = useState('');
  const textRef = useRef(text);

  function handleChange(e) {
    setText(e.target.value);
    textRef.current = e.target.value;
  }

  function handleSend() {
    setTimeout(() => {
      alert('正在发送：' + textRef.current);
    }, 3000);
  }

  return (
    <>
      <input
        value={text}
        onChange={handleChange}
      />
      <button
        onClick={handleSend}>
        发送
      </button>
    </>
  );
}
```

</Sandpack>

</Solution>

</Challenges>
