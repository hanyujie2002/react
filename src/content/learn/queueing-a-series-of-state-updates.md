---
title: 把一系列 state 更新加入队列 | Queueing a Series of State Updates
translators: 
  - Jiacheng787
  - Neo42
---

<Intro>

Setting a state variable will queue another render. But sometimes you might want to perform multiple operations on the value before queueing the next render. To do this, it helps to understand how React batches state updates.

设置组件 state 会把一次重新渲染加入队列。但有时你可能会希望在下次渲染加入队列之前对 state 的值执行多次操作。为此，了解 React 如何批量更新 state 会很有帮助。

</Intro>

<YouWillLearn>

* What "batching" is and how React uses it to process multiple state updates
* 什么是“批处理”以及 React 如何使用它来处理多个 state 更新
* How to apply several updates to the same state variable in a row
* 如何连续多次对同一 state 变量进行更新

</YouWillLearn>

## React 会对 state 更新进行批处理 | React batches state updates {/*react-batches-state-updates*/}

You might expect that clicking the "+3" button will increment the counter three times because it calls `setNumber(number + 1)` three times:
在下面的示例中，你可能会认为点击 “+3” 按钮会使计数器递增三次，因为它调用了 `setNumber(number + 1)` 三次：

<Sandpack>

```js
import { useState } from 'react';

export default function Counter() {
  const [number, setNumber] = useState(0);

  return (
    <>
      <h1>{number}</h1>
      <button onClick={() => {
        setNumber(number + 1);
        setNumber(number + 1);
        setNumber(number + 1);
      }}>+3</button>
    </>
  )
}
```

```css
button { display: inline-block; margin: 10px; font-size: 20px; }
h1 { display: inline-block; margin: 10px; width: 30px; text-align: center; }
```

</Sandpack>

However, as you might recall from the previous section, [each render's state values are fixed](/learn/state-as-a-snapshot#rendering-takes-a-snapshot-in-time), so the value of `number` inside the first render's event handler is always `0`, no matter how many times you call `setNumber(1)`:
但是，你可能还记得上一节中的内容，[每一次渲染的 state 值都是固定的](/learn/state-as-a-snapshot#rendering-takes-a-snapshot-in-time)，因此无论你调用多少次 `setNumber(1)`，在第一次渲染的事件处理函数内部的 `number` 值总是 `0` ：

```js
setNumber(0 + 1);
setNumber(0 + 1);
setNumber(0 + 1);
```

But there is one other factor at play here. **React waits until *all* code in the event handlers has run before processing your state updates.** This is why the re-render only happens *after* all these `setNumber()` calls.
但是这里还有另外一个影响因素需要讨论。**React 会等到事件处理函数中的** 所有 **代码都运行完毕再处理你的 state 更新。** 这就是重新渲染只会发生在所有这些 `setNumber()` 调用 **之后** 的原因。

This might remind you of a waiter taking an order at the restaurant. A waiter doesn't run to the kitchen at the mention of your first dish! Instead, they let you finish your order, let you make changes to it, and even take orders from other people at the table.
这可能会让你想起餐厅里帮你点菜的服务员。服务员不会在你说第一道菜的时候就跑到厨房！相反，他们会让你把菜点完，让你修改菜品，甚至会帮桌上的其他人点菜。

<Illustration src="/images/docs/illustrations/i_react-batching.png"  alt="在餐厅里，优雅的光标跟扮演服务员角色的 React 多次点菜。在她多次调用 setState() 之后，服务员将她所点的最后一个作为她的最终订单记录下来。| An elegant cursor at a restaurant places and order multiple times with React, playing the part of the waiter. After she calls setState() multiple times, the waiter writes down the last one she requested as her final order." />

This lets you update multiple state variables--even from multiple components--without triggering too many [re-renders.](/learn/render-and-commit#re-renders-when-state-updates) But this also means that the UI won't be updated until _after_ your event handler, and any code in it, completes. This behavior, also known as **batching,** makes your React app run much faster. It also avoids dealing with confusing "half-finished" renders where only some of the variables have been updated.
这让你可以更新多个 state 变量——甚至来自多个组件的 state 变量——而不会触发太多的 [重新渲染](/learn/render-and-commit#re-renders-when-state-updates)。但这也意味着只有在你的事件处理函数及其中任何代码执行完成 **之后**，UI 才会更新。这种特性也就是 **批处理**，它会使你的 React 应用运行得更快。它还会帮你避免处理只​​更新了一部分 state 变量的令人困惑的“半成品”渲染。

**React does not batch across *multiple* intentional events like clicks**--each click is handled separately. Rest assured that React only does batching when it's generally safe to do. This ensures that, for example, if the first button click disables a form, the second click would not submit it again.
**React 不会跨 *多个* 需要刻意触发的事件（如点击）进行批处理**——每次点击都是单独处理的。请放心，React 只会在一般来说安全的情况下才进行批处理。这可以确保，例如，如果第一次点击按钮会禁用表单，那么第二次点击就不会再次提交它。

## 在下次渲染前多次更新同一个 state | Updating the same state multiple times before the next render {/*updating-the-same-state-multiple-times-before-the-next-render*/}

It is an uncommon use case, but if you would like to update the same state variable multiple times before the next render, instead of passing the *next state value* like `setNumber(number + 1)`, you can pass a *function* that calculates the next state based on the previous one in the queue, like `setNumber(n => n + 1)`. It is a way to tell React to "do something with the state value" instead of just replacing it.
这是一个不常见的用例，但是如果你想在下次渲染之前多次更新同一个 state，你可以像 `setNumber(n => n + 1)` 这样传入一个根据队列中的前一个 state 计算下一个 state 的 **函数**，而不是像 `setNumber(number + 1)` 这样传入 **下一个 state 值**。这是一种告诉 React “用 state 值做某事”而不是仅仅替换它的方法。

Try incrementing the counter now:
现在尝试递增计数器：

<Sandpack>

```js
import { useState } from 'react';

export default function Counter() {
  const [number, setNumber] = useState(0);

  return (
    <>
      <h1>{number}</h1>
      <button onClick={() => {
        setNumber(n => n + 1);
        setNumber(n => n + 1);
        setNumber(n => n + 1);
      }}>+3</button>
    </>
  )
}
```

```css
button { display: inline-block; margin: 10px; font-size: 20px; }
h1 { display: inline-block; margin: 10px; width: 30px; text-align: center; }
```

</Sandpack>

Here, `n => n + 1` is called an **updater function.** When you pass it to a state setter:
在这里，`n => n + 1` 被称为 **更新函数**。当你将它传递给一个 state 设置函数时：

1. React queues this function to be processed after all the other code in the event handler has run.
1. React 会将此函数加入队列，以便在事件处理函数中的所有其他代码运行后进行处理。
2. During the next render, React goes through the queue and gives you the final updated state.
2. 在下一次渲染期间，React 会遍历队列并给你更新之后的最终 state。

```js
setNumber(n => n + 1);
setNumber(n => n + 1);
setNumber(n => n + 1);
```

Here's how React works through these lines of code while executing the event handler:
下面是 React 在执行事件处理函数时处理这几行代码的过程：

1. `setNumber(n => n + 1)`: `n => n + 1` is a function. React adds it to a queue.
- `setNumber(n => n + 1)`：`n => n + 1` 是一个函数。React 将它加入队列。
2. `setNumber(n => n + 1)`: `n => n + 1` is a function. React adds it to a queue.
- `setNumber(n => n + 1)`：`n => n + 1` 是一个函数。React 将它加入队列。
3. `setNumber(n => n + 1)`: `n => n + 1` is a function. React adds it to a queue.
- `setNumber(n => n + 1)`：`n => n + 1` 是一个函数。React 将它加入队列。

When you call `useState` during the next render, React goes through the queue. The previous `number` state was `0`, so that's what React passes to the first updater function as the `n` argument. Then React takes the return value of your previous updater function and passes it to the next updater as `n`, and so on:
当你在下次渲染期间调用 `useState` 时，React 会遍历队列。之前的 `number` state 的值是 `0`，所以这就是 React 作为参数 `n` 传递给第一个更新函数的值。然后 React 会获取你上一个更新函数的返回值，并将其作为 `n` 传递给下一个更新函数，以此类推：

|  更新队列/queued update | `n` | 返回值/returns |
|--------------|---------|-----|
| `n => n + 1` | `0` | `0 + 1 = 1` |
| `n => n + 1` | `1` | `1 + 1 = 2` |
| `n => n + 1` | `2` | `2 + 1 = 3` |

React stores `3` as the final result and returns it from `useState`.
React 会保存 `3` 为最终结果并从 `useState` 中返回。

This is why clicking "+3" in the above example correctly increments the value by 3.
这就是为什么在上面的示例中点击“+3”正确地将值增加“+3”。
### 如果你在替换 state 后更新 state 会发生什么 | What happens if you update state after replacing it {/*what-happens-if-you-update-state-after-replacing-it*/}

What about this event handler? What do you think `number` will be in the next render?
这个事件处理函数会怎么样？你认为 `number` 在下一次渲染中的值是什么？

```js
<button onClick={() => {
  setNumber(number + 5);
  setNumber(n => n + 1);
}}>
```

<Sandpack>

```js
import { useState } from 'react';

export default function Counter() {
  const [number, setNumber] = useState(0);

  return (
    <>
      <h1>{number}</h1>
      <button onClick={() => {
        setNumber(number + 5);
        setNumber(n => n + 1);
      }}>增加数字</button>
    </>
  )
}
```

```css
button { display: inline-block; margin: 10px; font-size: 20px; }
h1 { display: inline-block; margin: 10px; width: 30px; text-align: center; }
```

</Sandpack>

Here's what this event handler tells React to do:
这是事件处理函数告诉 React 要做的事情：

1. `setNumber(number + 5)`: `number` is `0`, so `setNumber(0 + 5)`. React adds *"replace with `5`"* to its queue.
- `setNumber(number + 5)`：`number` 为 `0`，所以 `setNumber(0 + 5)`。React 将 *“替换为 `5`”* 添加到其队列中。
2. `setNumber(n => n + 1)`: `n => n + 1` is an updater function. React adds *that function* to its queue.
- `setNumber(n => n + 1)`：`n => n + 1` 是一个更新函数。 React 将 **该函数** 添加到其队列中。

During the next render, React goes through the state queue:
在下一次渲染期间，React 会遍历 state 队列：

|   更新队列/queued update      | `n` | 返回值/returns |
|--------------|---------|-----|
| "replace with `5`"<br/>“替换为 `5`”|  `0` (unused)<br/>`0`（未使用） | `5` |
| `n => n + 1` | `5` | `5 + 1 = 6` |

React stores `6` as the final result and returns it from `useState`. 
React 会保存 `6` 为最终结果并从 `useState` 中返回。

<Note>

You may have noticed that `setState(5)` actually works like `setState(n => 5)`, but `n` is unused!
你可能已经注意到，`setState(x)` 实际上会像 `setState(n => x)` 一样运行，只是没有使用 `n`！

</Note>

### 如果你在更新 state 后替换 state 会发生什么 | What happens if you replace state after updating it {/*what-happens-if-you-replace-state-after-updating-it*/}

Let's try one more example. What do you think `number` will be in the next render?
让我们再看一个例子。你认为 `number` 在下一次渲染中的值是什么？

```js
<button onClick={() => {
  setNumber(number + 5);
  setNumber(n => n + 1);
  setNumber(42);
}}>
```

<Sandpack>

```js
import { useState } from 'react';

export default function Counter() {
  const [number, setNumber] = useState(0);

  return (
    <>
      <h1>{number}</h1>
      <button onClick={() => {
        setNumber(number + 5);
        setNumber(n => n + 1);
        setNumber(42);
      }}>增加数字</button>
    </>
  )
}
```

```css
button { display: inline-block; margin: 10px; font-size: 20px; }
h1 { display: inline-block; margin: 10px; width: 30px; text-align: center; }
```

</Sandpack>

Here's how React works through these lines of code while executing this event handler:
以下是 React 在执行事件处理函数时处理这几行代码的过程：

1. `setNumber(number + 5)`: `number` is `0`, so `setNumber(0 + 5)`. React adds *"replace with `5`"* to its queue.
- `setNumber(number + 5)`：`number` 为 `0`，所以 `setNumber(0 + 5)`。React 将 *“替换为 `5`”* 添加到其队列中。
2. `setNumber(n => n + 1)`: `n => n + 1` is an updater function. React adds *that function* to its queue.
- `setNumber(n => n + 1)`：`n => n + 1` 是一个更新函数。React 将该函数添加到其队列中。
3. `setNumber(42)`: React adds *"replace with `42`"* to its queue.
- `setNumber(42)`：React 将 *“替换为 `42`”* 添加到其队列中。

During the next render, React goes through the state queue:
在下一次渲染期间，React 会遍历 state 队列：

|   queued update<br/>更新队列     | `n` | returns<br/>返回值 |
|--------------|---------|-----|
| "replace with `5`"<br/>“替换为 `5`” | `0` (unused)<br/>`0`（未使用） | `5` |
| `n => n + 1` | `5` | `5 + 1 = 6` |
| "replace with `42`"<br/>“替换为 `42`” | `6` (unused)<br/>`6`（未使用） | `42` |

Then React stores `42` as the final result and returns it from `useState`.
然后 React 会保存 `42` 为最终结果并从 `useState` 中返回。

To summarize, here's how you can think of what you're passing to the `setNumber` state setter:
总而言之，以下是你可以考虑传递给 `setNumber` state 设置函数的内容：

* **An updater function** (e.g. `n => n + 1`) gets added to the queue.
* **一个更新函数**（例如：`n => n + 1`）会被添加到队列中。
* **Any other value** (e.g. number `5`) adds "replace with `5`" to the queue, ignoring what's already queued.
* **任何其他的值**（例如：数字 `5`）会导致“替换为 `5`”被添加到队列中，已经在队列中的内容会被忽略。

After the event handler completes, React will trigger a re-render. During the re-render, React will process the queue. Updater functions run during rendering, so **updater functions must be [pure](/learn/keeping-components-pure)** and only *return* the result. Don't try to set state from inside of them or run other side effects. In Strict Mode, React will run each updater function twice (but discard the second result) to help you find mistakes.
事件处理函数执行完成后，React 将触发重新渲染。在重新渲染期间，React 将处理队列。更新函数会在渲染期间执行，因此 **更新函数必须是 [纯函数](/learn/keeping-components-pure)** 并且只 **返回** 结果。不要尝试从它们内部设置 state 或者执行其他副作用。在严格模式下，React 会执行每个更新函数两次（但是丢弃第二个结果）以便帮助你发现错误。

### 命名惯例 | Naming conventions {/*naming-conventions*/}

It's common to name the updater function argument by the first letters of the corresponding state variable:
通常可以通过相应 state 变量的第一个字母来命名更新函数的参数：

```js
setEnabled(e => !e);
setLastName(ln => ln.reverse());
setFriendCount(fc => fc * 2);
```

If you prefer more verbose code, another common convention is to repeat the full state variable name, like `setEnabled(enabled => !enabled)`, or to use a prefix like `setEnabled(prevEnabled => !prevEnabled)`.
如果你喜欢更冗长的代码，另一个常见的惯例是重复使用完整的 state 变量名称，如 `setEnabled(enabled => !enabled)`，或使用前缀，如 `setEnabled(prevEnabled => !prevEnabled)`。

<Recap>

* Setting state does not change the variable in the existing render, but it requests a new render.
* 设置 state 不会更改现有渲染中的变量，但会请求一次新的渲染。
* React processes state updates after event handlers have finished running. This is called batching.
* React 会在事件处理函数执行完成之后处理 state 更新。这被称为批处理。
* To update some state multiple times in one event, you can use `setNumber(n => n + 1)` updater function.
* 要在一个事件中多次更新某些 state，你可以使用 `setNumber(n => n + 1)` 更新函数。

</Recap>



<Challenges>

#### 修复请求计数器 | Fix a request counter {/*fix-a-request-counter*/}

You're working on an art marketplace app that lets the user submit multiple orders for an art item at the same time. Each time the user presses the "Buy" button, the "Pending" counter should increase by one. After three seconds, the "Pending" counter should decrease, and the "Completed" counter should increase.
你正在开发一个艺术市场应用，该应用允许一个用户为一个艺术品同时提交多个订单。每次用户按下“购买”按钮，“等待”计数器应该增加一。三秒后，“等待”计数器应该减少，“完成”计数器应该增加。

However, the "Pending" counter does not behave as intended. When you press "Buy", it decreases to `-1` (which should not be possible!). And if you click fast twice, both counters seem to behave unpredictably.
但是，“等待”计数器的行为并不符合预期。当你按下“购买”按钮时，它会减少到 `-1`（这本应该是不可能的）。如果你快速点击两次，两个计数器似乎都会出现无法预测的行为。

Why does this happen? Fix both counters.
为什么会发生这种情况？修复两个计数器。

<Sandpack>

```js
import { useState } from 'react';

export default function RequestTracker() {
  const [pending, setPending] = useState(0);
  const [completed, setCompleted] = useState(0);

  async function handleClick() {
    setPending(pending + 1);
    await delay(3000);
    setPending(pending - 1);
    setCompleted(completed + 1);
  }

  return (
    <>
      <h3>
        等待：{pending}
      </h3>
      <h3>
        完成：{completed}
      </h3>
      <button onClick={handleClick}>
        购买
      </button>
    </>
  );
}

function delay(ms) {
  return new Promise(resolve => {
    setTimeout(resolve, ms);
  });
}
```

</Sandpack>

<Solution>

Inside the `handleClick` event handler, the values of `pending` and `completed` correspond to what they were at the time of the click event. For the first render, `pending` was `0`, so `setPending(pending - 1)` becomes `setPending(-1)`, which is wrong. Since you want to *increment* or *decrement* the counters, rather than set them to a concrete value determined during the click, you can instead pass the updater functions:
在 `handleClick` 事件处理函数内部，`pending` 和 `completed` 的值与他们在点击事件发生时的值相对应。对于第一次渲染，`pending` 为 `0` ，因此 `setPending(pending - 1)` 变成了 `setPending(-1)`，而这是错误的。既然你想要 *增加* 或 *减少* 计数器，你可以改为传递更新函数，而不是将计数器设置为在点击期间确定的具体值：

<Sandpack>

```js
import { useState } from 'react';

export default function RequestTracker() {
  const [pending, setPending] = useState(0);
  const [completed, setCompleted] = useState(0);

  async function handleClick() {
    setPending(p => p + 1);
    await delay(3000);
    setPending(p => p - 1);
    setCompleted(c => c + 1);
  }

  return (
    <>
      <h3>
        等待：{pending}
      </h3>
      <h3>
        完成：{completed}
      </h3>
      <button onClick={handleClick}>
        购买
      </button>
    </>
  );
}

function delay(ms) {
  return new Promise(resolve => {
    setTimeout(resolve, ms);
  });
}
```

</Sandpack>

This ensures that when you increment or decrement a counter, you do it in relation to its *latest* state rather than what the state was at the time of the click.
这可以确保你在增加或减少计数器时是根据其 **最新** 的 state 而不是点击时的 state 来进行增减的。

</Solution>

#### 自己实现状态队列 | Implement the state queue yourself {/*implement-the-state-queue-yourself*/}

In this challenge, you will reimplement a tiny part of React from scratch! It's not as hard as it sounds.
在这个挑战中，你将从头开始重新实现 React 的一小部分！这并不像听起来那么难。

Scroll through the sandbox preview. Notice that it shows **four test cases.** They correspond to the examples you've seen earlier on this page. Your task is to implement the `getFinalState` function so that it returns the correct result for each of those cases. If you implement it correctly, all four tests should pass.
滚动 sandbox 进行预览。请注意，它显示了 **四个测试用例** 。它们与你之前在本页上看到过的示例相对应。你的任务是实现 `getFinalState` 函数，让它为每种情况返回正确的结果。如果你对它进行了正确的实现的话，那么所有四个测试用例都应该会通过。

You will receive two arguments: `baseState` is the initial state (like `0`), and the `queue` is an array which contains a mix of numbers (like `5`) and updater functions (like `n => n + 1`) in the order they were added.
你将收到两个参数：`baseState` 是初始状态（例如：`0`），`queue` 是一个既包含数字（例如：`5`）也包含更新函数（例如：`n => n + 1`）的数组，这些数字和数组会被按照它们被添加进来的顺序排列。

Your task is to return the final state, just like the tables on this page show!
你的任务是返回最终的 state，如同页面上的表格展示的那样！

<Hint>

If you're feeling stuck, start with this code structure:
如果你没有思路，可以从下面的代码结构开始:

```js
export function getFinalState(baseState, queue) {
  let finalState = baseState;

  for (let update of queue) {
    if (typeof update === 'function') {
      // TODO: apply the updater function
      // TODO: 调用更新函数
    } else {
      // TODO: replace the state
      // TODO: 替换 state
    }
  }

  return finalState;
}
```

Fill out the missing lines!
补全缺失的几行代码！

</Hint>

<Sandpack>

```js src/processQueue.js active
export function getFinalState(baseState, queue) {
  let finalState = baseState;

  // TODO: do something with the queue...
  // TODO: 对队列做些什么...

  return finalState;
}
```

```js src/App.js
import { getFinalState } from './processQueue.js';

function increment(n) {
  return n + 1;
}
increment.toString = () => 'n => n+1';

export default function App() {
  return (
    <>
      <TestCase
        baseState={0}
        queue={[1, 1, 1]}
        expected={1}
      />
      <hr />
      <TestCase
        baseState={0}
        queue={[
          increment,
          increment,
          increment
        ]}
        expected={3}
      />
      <hr />
      <TestCase
        baseState={0}
        queue={[
          5,
          increment,
        ]}
        expected={6}
      />
      <hr />
      <TestCase
        baseState={0}
        queue={[
          5,
          increment,
          42,
        ]}
        expected={42}
      />
    </>
  );
}

function TestCase({
  baseState,
  queue,
  expected
}) {
  const actual = getFinalState(baseState, queue);
  return (
    <>
      <p>初始 state：<b>{baseState}</b></p>
      <p>队列：<b>[{queue.join(', ')}]</b></p>
      <p>预期结果：<b>{expected}</b></p>
      <p style={{
        color: actual === expected ?
        'green' :
        'red'
      }}>
        你的结果：<b>{actual}</b>
        {' '}
        ({actual === expected ?
          '正确' :
          '错误'
        })
      </p>
    </>
  );
}
```

</Sandpack>

<Solution>

This is the exact algorithm described on this page that React uses to calculate the final state:
这其实就是这一页中所描述的 React 用来计算最终 state 的算法:

<Sandpack>

```js src/processQueue.js active
export function getFinalState(baseState, queue) {
  let finalState = baseState;

  for (let update of queue) {
    if (typeof update === 'function') {
      // Apply the updater function.
      // 调用更新函数
      finalState = update(finalState);
    } else {
      // Replace the next state.
      // 替换下一个 state
      finalState = update;
    }
  }

  return finalState;
}
```

```js src/App.js
import { getFinalState } from './processQueue.js';

function increment(n) {
  return n + 1;
}
increment.toString = () => 'n => n+1';

export default function App() {
  return (
    <>
      <TestCase
        baseState={0}
        queue={[1, 1, 1]}
        expected={1}
      />
      <hr />
      <TestCase
        baseState={0}
        queue={[
          increment,
          increment,
          increment
        ]}
        expected={3}
      />
      <hr />
      <TestCase
        baseState={0}
        queue={[
          5,
          increment,
        ]}
        expected={6}
      />
      <hr />
      <TestCase
        baseState={0}
        queue={[
          5,
          increment,
          42,
        ]}
        expected={42}
      />
    </>
  );
}

function TestCase({
  baseState,
  queue,
  expected
}) {
  const actual = getFinalState(baseState, queue);
  return (
    <>
      <p>初始 state：<b>{baseState}</b></p>
      <p>队列：<b>[{queue.join(', ')}]</b></p>
      <p>预期结果：<b>{expected}</b></p>
      <p style={{
        color: actual === expected ?
        'green' :
        'red'
      }}>
        你的结果：<b>{actual}</b>
        {' '}
        ({actual === expected ?
          '正确' :
          '错误'
        })
      </p>
    </>
  );
}
```

</Sandpack>

Now you know how this part of React works!
现在你知道 React 的这一部分是如何运行的了吧！

</Solution>

</Challenges>
