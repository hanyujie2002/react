---
title: '使用 Effect 进行同步 | Synchronizing with Effects'
---

<Intro>

Some components need to synchronize with external systems. For example, you might want to control a non-React component based on the React state, set up a server connection, or send an analytics log when a component appears on the screen. *Effects* let you run some code after rendering so that you can synchronize your component with some system outside of React.
有些组件需要与外部系统同步。例如，你可能希望根据 React state 控制非 React 组件、建立服务器连接或当组件在页面显示时发送分析日志。Effect 允许你在渲染结束后执行一些代码，以便将组件与 React 外部的某个系统相同步。

</Intro>

<YouWillLearn>

- What Effects are
- 什么是 Effect
- How Effects are different from events
- Effect 与事件（event）有何不同
- How to declare an Effect in your component
- 如何在组件中声明 Effect
- How to skip re-running an Effect unnecessarily
- 如何避免不必要地重新运行 Effect
- Why Effects run twice in development and how to fix them
- 为什么 Effect 在开发环境中会运行两次以及如何解决这个问题

</YouWillLearn>

## 什么是 Effect，它与事件（event）有何不同？| What are Effects and how are they different from events? {/*what-are-effects-and-how-are-they-different-from-events*/}

Before getting to Effects, you need to be familiar with two types of logic inside React components:
在接触 Effect 之前，你需要熟悉 React 组件中的两种逻辑类型：

- **Rendering code** (introduced in [Describing the UI](/learn/describing-the-ui)) lives at the top level of your component. This is where you take the props and state, transform them, and return the JSX you want to see on the screen. [Rendering code must be pure.](/learn/keeping-components-pure) Like a math formula, it should only _calculate_ the result, but not do anything else.
- **渲染代码**（在 [描述 UI](/learn/describing-the-ui) 中有介绍）位于组件的顶层。你在这里处理 props 和 state，对它们进行转换，并返回希望在页面上显示的 JSX。[渲染代码必须是纯粹的](/learn/keeping-components-pure)——就像数学公式一样，它只应该“计算”结果，而不做其他任何事情。

- **Event handlers** (introduced in [Adding Interactivity](/learn/adding-interactivity)) are nested functions inside your components that *do* things rather than just calculate them. An event handler might update an input field, submit an HTTP POST request to buy a product, or navigate the user to another screen. Event handlers contain ["side effects"](https://en.wikipedia.org/wiki/Side_effect_(computer_science)) (they change the program's state) caused by a specific user action (for example, a button click or typing).
- **事件处理程序**（在 [添加交互性](/learn/adding-interactivity) 中有介绍）是组件内部的嵌套函数，它们不光进行计算, 还会执行一些操作。事件处理程序可能会更新输入字段、提交 HTTP POST 请求来购买产品，或者将用户导航到另一个页面。事件处理程序包含由特定用户操作（例如按钮点击或输入）引起的“副作用”（它们改变了程序的状态）。

Sometimes this isn't enough. Consider a `ChatRoom` component that must connect to the chat server whenever it's visible on the screen. Connecting to a server is not a pure calculation (it's a side effect) so it can't happen during rendering. However, there is no single particular event like a click that causes `ChatRoom` to be displayed.
有时这还不够。考虑一个 `ChatRoom` 组件，它在页面上显示时必须连接到聊天服务器。连接到服务器并不是纯粹的计算（它是一个副作用），因此它不能在渲染期间发生。然而，并没有一个特定的事件（比如点击）能让 `ChatRoom` 被显示。

***Effects* let you specify side effects that are caused by rendering itself, rather than by a particular event.** Sending a message in the chat is an *event* because it is directly caused by the user clicking a specific button. However, setting up a server connection is an *Effect* because it should happen no matter which interaction caused the component to appear. Effects run at the end of a [commit](/learn/render-and-commit) after the screen updates. This is a good time to synchronize the React components with some external system (like network or a third-party library).
**Effect 允许你指定由渲染自身，而不是特定事件引起的副作用**。在聊天中发送消息是一个“事件”，因为它直接由用户点击特定按钮引起。然而，建立服务器连接是一个 Effect，因为无论哪种交互致使组件出现，它都应该发生。Effect 在 [提交](/learn/render-and-commit) 结束后、页面更新后运行。此时是将 React 组件与外部系统（如网络或第三方库）同步的最佳时机。

<Note>

Here and later in this text, capitalized "Effect" refers to the React-specific definition above, i.e. a side effect caused by rendering. To refer to the broader programming concept, we'll say "side effect".
在本文此处和后续文本中，大写的 `Effect` 是 React 中的专有定义——由渲染引起的副作用。至于更广泛的编程概念(任何改变程序状态或外部系统的行为)，我们则使用“副作用（side effect）” 来指代。

</Note>


## 你可能不需要 Effect | You might not need an Effect {/*you-might-not-need-an-effect*/}

**Don't rush to add Effects to your components.** Keep in mind that Effects are typically used to "step out" of your React code and synchronize with some *external* system. This includes browser APIs, third-party widgets, network, and so on. If your Effect only adjusts some state based on other state, [you might not need an Effect.](/learn/you-might-not-need-an-effect)
**不要急着在你的组件中使用 Effect**。记住，Effect 通常用于暂时“跳出” React 并与一些 **外部** 系统进行同步。这包括浏览器 API、第三方小部件，以及网络等等。如果你的 Effect 只是根据其他状态来调整某些状态，那么 [你可能并不需要一个 Effect](/learn/you-might-not-need-an-effect)。

## 如何编写 Effect | How to write an Effect {/*how-to-write-an-effect*/}

To write an Effect, follow these three steps:
要编写一个 Effect, 请遵循以下三个步骤：

1. **Declare an Effect.** By default, your Effect will run after every [commit](/learn/render-and-commit).
- **声明 Effect**。通常 Effect 会在每次 [提交](/learn/render-and-commit) 后运行。
2. **Specify the Effect dependencies.** Most Effects should only re-run *when needed* rather than after every render. For example, a fade-in animation should only trigger when a component appears. Connecting and disconnecting to a chat room should only happen when the component appears and disappears, or when the chat room changes. You will learn how to control this by specifying *dependencies.*
- **指定 Effect 依赖**。大多数 Effect 应该按需运行，而不是在每次渲染后都运行。例如，淡入动画应该只在组件出现时触发。连接和断开服务器的操作只应在组件出现和消失时，或者切换聊天室时执行。你将通过指定 **依赖项** 来学习如何控制这一点。
3. **Add cleanup if needed.** Some Effects need to specify how to stop, undo, or clean up whatever they were doing. For example, "connect" needs "disconnect", "subscribe" needs "unsubscribe", and "fetch" needs either "cancel" or "ignore". You will learn how to do this by returning a *cleanup function*.
- **必要时添加清理操作**。一些 Effect 需要指定如何停止、撤销，或者清除它们所执行的操作。例如，“连接”需要“断开”，“订阅”需要“退订”，而“获取数据”需要“取消”或者“忽略”。你将学习如何通过返回一个 **清理函数** 来实现这些。

Let's look at each of these steps in detail.
让我们详细看看每一步。

### 第一步：声明 Effect | Step 1: Declare an Effect {/*step-1-declare-an-effect*/}

To declare an Effect in your component, import the [`useEffect` Hook](/reference/react/useEffect) from React:
先从 React 中导入 [`useEffect` Hook](/reference/react/useEffect)：

```js
import { useEffect } from 'react';
```

Then, call it at the top level of your component and put some code inside your Effect:
再在组件顶部调用, 并在其中加入一些代码：

```js {2-4}
function MyComponent() {
  useEffect(() => {
    // Code here will run after *every* render
    // 每次渲染后都会执行此处的代码
  });
  return <div />;
}
```

Every time your component renders, React will update the screen *and then* run the code inside `useEffect`. In other words, **`useEffect` "delays" a piece of code from running until that render is reflected on the screen.**
每当你的组件渲染时，React 会先更新页面，然后再运行 `useEffect` 中的代码。换句话说，**`useEffect` 会“延迟”一段代码的运行，直到渲染结果反映在页面上**。

Let's see how you can use an Effect to synchronize with an external system. Consider a `<VideoPlayer>` React component. It would be nice to control whether it's playing or paused by passing an `isPlaying` prop to it:
接下来，让我们看看如何使用 Effect 来与外部系统同步。考虑一个 `<VideoPlayer>` React 组件。我们想要通过传递一个 `isPlaying` prop 来控制它播放或者暂停：

```js
<VideoPlayer isPlaying={isPlaying} />;
```

Your custom `VideoPlayer` component renders the built-in browser [`<video>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/video) tag:
这个 `VideoPlayer` 组件渲染了浏览器内置的 [`<video>`](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Element/video) 标签：

```js
function VideoPlayer({ src, isPlaying }) {
  // TODO: do something with isPlaying
  // TODO：使用 isPlaying 做一些事情
  return <video src={src} />;
}
```

However, the browser `<video>` tag does not have an `isPlaying` prop. The only way to control it is to manually call the [`play()`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLMediaElement/play) and [`pause()`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLMediaElement/pause) methods on the DOM element. **You need to synchronize the value of `isPlaying` prop, which tells whether the video _should_ currently be playing, with calls like `play()` and `pause()`.**
但是，浏览器的 `<video>` 标签没有 `isPlaying` 属性。控制它的唯一方式是在 DOM 元素上调用 [`play()`](https://developer.mozilla.org/zh-CN/docs/Web/API/HTMLMediaElement/play) 和 [`pause()`](https://developer.mozilla.org/zh-CN/docs/Web/API/HTMLMediaElement/pause) 方法。因此，**你需要将 `isPlaying` prop 的值（表示视频当前是否应该播放）与 `play()` 和 `pause()` 等函数的调用进行同步**。

We'll need to first [get a ref](/learn/manipulating-the-dom-with-refs) to the `<video>` DOM node.
我们首先需要获取 `<video>` DOM 节点的 [对象引用](/learn/manipulating-the-dom-with-refs)。

You might be tempted to try to call `play()` or `pause()` during rendering, but that isn't correct:
你可能会尝试在渲染期间调用 `play()` 或 `pause()`，但这样做是不对的：

<Sandpack>

```js
import { useState, useRef, useEffect } from 'react';

function VideoPlayer({ src, isPlaying }) {
  const ref = useRef(null);

  if (isPlaying) {
    ref.current.play();  // 渲染期间不能调用 `play()`。 // Calling these while rendering isn't allowed.
  } else {
    ref.current.pause(); // 同样，调用 `pause()` 也不行。 // Also, this crashes.
  }

  return <video ref={ref} src={src} loop playsInline />;
}

export default function App() {
  const [isPlaying, setIsPlaying] = useState(false);
  return (
    <>
      <button onClick={() => setIsPlaying(!isPlaying)}>
        {isPlaying ? '暂停' : '播放'}
      </button>
      <VideoPlayer
        isPlaying={isPlaying}
        src="https://interactive-examples.mdn.mozilla.net/media/cc0-videos/flower.mp4"
      />
    </>
  );
}
```

```css
button { display: block; margin-bottom: 20px; }
video { width: 250px; }
```

</Sandpack>

The reason this code isn't correct is that it tries to do something with the DOM node during rendering. In React, [rendering should be a pure calculation](/learn/keeping-components-pure) of JSX and should not contain side effects like modifying the DOM.
这段代码之所以不对，是因为它试图在渲染期间对 DOM 节点进行操作。在 React 中，[渲染应该是纯粹的计算](/learn/keeping-components-pure) JSX，不应该包含任何像修改 DOM 这样的副作用。

Moreover, when `VideoPlayer` is called for the first time, its DOM does not exist yet! There isn't a DOM node yet to call `play()` or `pause()` on, because React doesn't know what DOM to create until you return the JSX.
而且，当第一次调用 `VideoPlayer` 时，对应的 DOM 节点还不存在！因为 React 在你返回 JSX 之前不知道要创建什么样的 DOM，所以没有 DOM 节点可以调用 `play()` 或 `pause()` 方法。

The solution here is to **wrap the side effect with `useEffect` to move it out of the rendering calculation:**
解决办法是 **使用 `useEffect` 包裹副作用，把它分离到渲染逻辑的计算过程之外**：

```js {6,12}
import { useEffect, useRef } from 'react';

function VideoPlayer({ src, isPlaying }) {
  const ref = useRef(null);

  useEffect(() => {
    if (isPlaying) {
      ref.current.play();
    } else {
      ref.current.pause();
    }
  });

  return <video ref={ref} src={src} loop playsInline />;
}
```

By wrapping the DOM update in an Effect, you let React update the screen first. Then your Effect runs.
通过将 DOM 更新封装在 Effect 中，你可以让 React 先更新页面，然后再运行 Effect。

When your `VideoPlayer` component renders (either the first time or if it re-renders), a few things will happen. First, React will update the screen, ensuring the `<video>` tag is in the DOM with the right props. Then React will run your Effect. Finally, your Effect will call `play()` or `pause()` depending on the value of `isPlaying`.
当 `VideoPlayer` 组件渲染时（无论是否为首次渲染），会发生以下几件事：首先 React 会更新页面，确保 `<video>` 标签带着正确的 props 出现在 DOM 中；接着 React 将运行 Effect；最后 Effect 将根据 `isPlaying` 的值调用 `play()` 或 `pause()`。

Press Play/Pause multiple times and see how the video player stays synchronized to the `isPlaying` value:
试试点击几次播放和暂停按钮，观察视频播放器的行为是如何与 `isPlaying` 的值相同步的：

<Sandpack>

```js
import { useState, useRef, useEffect } from 'react';

function VideoPlayer({ src, isPlaying }) {
  const ref = useRef(null);

  useEffect(() => {
    if (isPlaying) {
      ref.current.play();
    } else {
      ref.current.pause();
    }
  });

  return <video ref={ref} src={src} loop playsInline />;
}

export default function App() {
  const [isPlaying, setIsPlaying] = useState(false);
  return (
    <>
      <button onClick={() => setIsPlaying(!isPlaying)}>
        {isPlaying ? '暂停' : '播放'}
      </button>
      <VideoPlayer
        isPlaying={isPlaying}
        src="https://interactive-examples.mdn.mozilla.net/media/cc0-videos/flower.mp4"
      />
    </>
  );
}
```

```css
button { display: block; margin-bottom: 20px; }
video { width: 250px; }
```

</Sandpack>

In this example, the "external system" you synchronized to React state was the browser media API. You can use a similar approach to wrap legacy non-React code (like jQuery plugins) into declarative React components.
在这个示例中，你同步到 React state 的“外部系统”是浏览器媒体 API。你也可以使用类似的方法将传统的非 React 代码（如 jQuery 插件）封装成声明式的 React 组件。

Note that controlling a video player is much more complex in practice. Calling `play()` may fail, the user might play or pause using the built-in browser controls, and so on. This example is very simplified and incomplete.
需要注意的是，控制视频播放器在实际应用中要复杂得多：比如调用 `play()` 可能会失败、用户可能会使用内置的浏览器控件来进行播放或暂停等操作。本例子是一个非常简化且不完整的示例。

<Pitfall>

By default, Effects run after *every* render. This is why code like this will **produce an infinite loop:**
默认情况下，Effect 会在 **每次** 渲染后运行。**正因如此，以下代码会陷入死循环**：

```js
const [count, setCount] = useState(0);
useEffect(() => {
  setCount(count + 1);
});
```

Effects run as a *result* of rendering. Setting state *triggers* rendering. Setting state immediately in an Effect is like plugging a power outlet into itself. The Effect runs, it sets the state, which causes a re-render, which causes the Effect to run, it sets the state again, this causes another re-render, and so on.
Effect 在渲染结束后运行。更新 state 会触发重新渲染。在 Effect 中直接更新 state 就像是把电源插座的插头插回自身：Effect 运行、更新 state、触发重新渲染、于是又触发 Effect 运行、再次更新 state，继而再次触发重新渲染。如此反复，从而陷入死循环。

Effects should usually synchronize your components with an *external* system. If there's no external system and you only want to adjust some state based on other state, [you might not need an Effect.](/learn/you-might-not-need-an-effect)
Effect 应该用于将你的组件与一个 **外部** 的系统保持同步。如果没有外部系统，你只是想根据其他状态调整一些状态，那么 [你也许不需要 Effect](/learn/you-might-not-need-an-effect)。

</Pitfall>

### 第二步：指定 Effect 的依赖项 | Step 2: Specify the Effect dependencies {/*step-2-specify-the-effect-dependencies*/}

By default, Effects run after *every* render. Often, this is **not what you want:**
默认情况下，Effect 会在 **每次** 渲染后运行。但往往 **这并不是你想要的**：

- Sometimes, it's slow. Synchronizing with an external system is not always instant, so you might want to skip doing it unless it's necessary. For example, you don't want to reconnect to the chat server on every keystroke.
- 有时，它可能会很慢。与外部系统的同步并不总是即时的，所以你可能希望在不必要时跳过它。例如，你不会想在每次打字时都得重新连接聊天服务器。
- Sometimes, it's wrong. For example, you don't want to trigger a component fade-in animation on every keystroke. The animation should only play once when the component appears for the first time.
- 有时，它可能会出错。例如，你不会想在每次按键时都触发组件的淡入动画。动画应该只在组件首次出现时播放。

To demonstrate the issue, here is the previous example with a few `console.log` calls and a text input that updates the parent component's state. Notice how typing causes the Effect to re-run:
为了演示这个问题，以下是在之前的示例中加入了一些 `console.log` 调用和一个更新父组件 state 的文本输入框。注意在输入时是如何触发 Effect 重新运行的：

<Sandpack>

```js
import { useState, useRef, useEffect } from 'react';

function VideoPlayer({ src, isPlaying }) {
  const ref = useRef(null);

  useEffect(() => {
    if (isPlaying) {
      console.log('调用 video.play()');
      ref.current.play();
    } else {
      console.log('调用 video.pause()');
      ref.current.pause();
    }
  });

  return <video ref={ref} src={src} loop playsInline />;
}

export default function App() {
  const [isPlaying, setIsPlaying] = useState(false);
  const [text, setText] = useState('');
  return (
    <>
      <input value={text} onChange={e => setText(e.target.value)} />
      <button onClick={() => setIsPlaying(!isPlaying)}>
        {isPlaying ? '暂停' : '播放'}
      </button>
      <VideoPlayer
        isPlaying={isPlaying}
        src="https://interactive-examples.mdn.mozilla.net/media/cc0-videos/flower.mp4"
      />
    </>
  );
}
```

```css
input, button { display: block; margin-bottom: 20px; }
video { width: 250px; }
```

</Sandpack>

You can tell React to **skip unnecessarily re-running the Effect** by specifying an array of *dependencies* as the second argument to the `useEffect` call. Start by adding an empty `[]` array to the above example on line 14:
通过在调用 `useEffect` 时指定一个 **依赖数组** 作为第二个参数，你可以让 React **跳过不必要地重新运行 Effect**。首先，在上面示例的第 14 行中传入一个空数组 `[]`：

```js {3}
  useEffect(() => {
    // ...
  }, []);
```

You should see an error saying `React Hook useEffect has a missing dependency: 'isPlaying'`:
你会看到一个错误提示：`React Hook useEffect has a missing dependency: 'isPlaying'`：

<Sandpack>

```js
import { useState, useRef, useEffect } from 'react';

function VideoPlayer({ src, isPlaying }) {
  const ref = useRef(null);

  useEffect(() => {
    if (isPlaying) {
      console.log('调用 video.play()');
      ref.current.play();
    } else {
      console.log('调用 video.pause()');
      ref.current.pause();
    }
  }, []); // 这将产生错误 // This causes an error

  return <video ref={ref} src={src} loop playsInline />;
}

export default function App() {
  const [isPlaying, setIsPlaying] = useState(false);
  const [text, setText] = useState('');
  return (
    <>
      <input value={text} onChange={e => setText(e.target.value)} />
      <button onClick={() => setIsPlaying(!isPlaying)}>
        {isPlaying ? '暂停' : '播放'}
      </button>
      <VideoPlayer
        isPlaying={isPlaying}
        src="https://interactive-examples.mdn.mozilla.net/media/cc0-videos/flower.mp4"
      />
    </>
  );
}
```

```css
input, button { display: block; margin-bottom: 20px; }
video { width: 250px; }
```

</Sandpack>

The problem is that the code inside of your Effect *depends on* the `isPlaying` prop to decide what to do, but this dependency was not explicitly declared. To fix this issue, add `isPlaying` to the dependency array:
原因在于，你的 Effect 内部代码依赖于 `isPlaying` prop 来决定该做什么，但你并没有显式声明这个依赖关系。为了解决这个问题，将 `isPlaying` 添加至依赖数组中：

```js {2,7}
  useEffect(() => {
    if (isPlaying) { // isPlaying 在此处使用…… // It's used here...
      // ...
    } else {
      // ...
    }
  }, [isPlaying]); // ……所以它必须在此处声明！ // ...so it must be declared here!
```

Now all dependencies are declared, so there is no error. Specifying `[isPlaying]` as the dependency array tells React that it should skip re-running your Effect if `isPlaying` is the same as it was during the previous render. With this change, typing into the input doesn't cause the Effect to re-run, but pressing Play/Pause does:
现在所有的依赖都已经声明，所以没有错误了。指定 `[isPlaying]` 作为依赖数组会告诉 React：如果 `isPlaying` 与上次渲染时相同，就跳过重新运行 Effect。这样一来，输入框的输入不会触发 Effect 重新运行，只有按下播放/暂停按钮会触发。

<Sandpack>

```js
import { useState, useRef, useEffect } from 'react';

function VideoPlayer({ src, isPlaying }) {
  const ref = useRef(null);

  useEffect(() => {
    if (isPlaying) {
      console.log('调用 video.play()');
      ref.current.play();
    } else {
      console.log('调用 video.pause()');
      ref.current.pause();
    }
  }, [isPlaying]);

  return <video ref={ref} src={src} loop playsInline />;
}

export default function App() {
  const [isPlaying, setIsPlaying] = useState(false);
  const [text, setText] = useState('');
  return (
    <>
      <input value={text} onChange={e => setText(e.target.value)} />
      <button onClick={() => setIsPlaying(!isPlaying)}>
        {isPlaying ? '暂停' : '播放'}
      </button>
      <VideoPlayer
        isPlaying={isPlaying}
        src="https://interactive-examples.mdn.mozilla.net/media/cc0-videos/flower.mp4"
      />
    </>
  );
}
```

```css
input, button { display: block; margin-bottom: 20px; }
video { width: 250px; }
```

</Sandpack>

The dependency array can contain multiple dependencies. React will only skip re-running the Effect if *all* of the dependencies you specify have exactly the same values as they had during the previous render. React compares the dependency values using the [`Object.is`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/is) comparison. See the [`useEffect` reference](/reference/react/useEffect#reference) for details.
依赖数组可以包含多个依赖项。只有当你指定的 **所有** 依赖项的值都与上一次渲染时完全相同，React 才会跳过重新运行该 Effect。React 使用 [`Object.is`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/is) 来比较依赖项的值。有关详细信息，请参阅 [`useEffect` 参考文档](/reference/react/useEffect#reference)。

**Notice that you can't "choose" your dependencies.** You will get a lint error if the dependencies you specified don't match what React expects based on the code inside your Effect. This helps catch many bugs in your code. If you don't want some code to re-run, [*edit the Effect code itself* to not "need" that dependency.](/learn/lifecycle-of-reactive-effects#what-to-do-when-you-dont-want-to-re-synchronize)
**请注意，你不能随意“选择”依赖项**。如果你指定的依赖项与 React 根据 Effect 内部代码所推断出的依赖项不匹配，你将收到来自 lint 的错误提示。这有助于捕捉代码中的许多 bug。如果你不希望某些代码重新运行，[那么你应当 **修改 Effect 代码本身**，使其不再“需要”该依赖项](/learn/lifecycle-of-reactive-effects#what-to-do-when-you-dont-want-to-re-synchronize)。

<Pitfall>

没有依赖数组和使用空数组 `[]` 作为依赖数组，行为是不同的：

```js {3,7,11}
useEffect(() => {
  // This runs after every render
  // 这里的代码会在每次渲染后运行
});

useEffect(() => {
  // This runs only on mount (when the component appears)
  // 这里的代码只会在组件挂载（首次出现）时运行
}, []);

useEffect(() => {
  // This runs on mount *and also* if either a or b have changed since the last render
  // 这里的代码不但会在组件挂载时运行，而且当 a 或 b 的值自上次渲染后发生变化后也会运行
}, [a, b]);
```

We'll take a close look at what "mount" means in the next step.
我们会在下一步详细了解什么是 **挂载（mount）**。

</Pitfall>

<DeepDive>

#### 为什么依赖数组中可以省略 ref? | Why was the ref omitted from the dependency array? {/*why-was-the-ref-omitted-from-the-dependency-array*/}

This Effect uses _both_ `ref` and `isPlaying`, but only `isPlaying` is declared as a dependency:
下面的 Effect 同时使用了 `ref` 与 `isPlaying` prop，但是只有 `isPlaying` 被声明为依赖项：

```js {9}
function VideoPlayer({ src, isPlaying }) {
  const ref = useRef(null);
  useEffect(() => {
    if (isPlaying) {
      ref.current.play();
    } else {
      ref.current.pause();
    }
  }, [isPlaying]);
```

This is because the `ref` object has a *stable identity:* React guarantees [you'll always get the same object](/reference/react/useRef#returns) from the same `useRef` call on every render. It never changes, so it will never by itself cause the Effect to re-run. Therefore, it does not matter whether you include it or not. Including it is fine too:
这是因为 `ref` 具有 **稳定** 的标识：React 确保你在 [每轮渲染中调用同一个 `useRef` 时，总能获得相同的对象](/reference/react/useRef#returns)。ref 不会改变，所以它不会导致重新运行 Effect。因此，在依赖数组中它可有可无。把它加进去也可以：

```js {9}
function VideoPlayer({ src, isPlaying }) {
  const ref = useRef(null);
  useEffect(() => {
    if (isPlaying) {
      ref.current.play();
    } else {
      ref.current.pause();
    }
  }, [isPlaying, ref]);
```

The [`set` functions](/reference/react/useState#setstate) returned by `useState` also have stable identity, so you will often see them omitted from the dependencies too. If the linter lets you omit a dependency without errors, it is safe to do.
`useState` 返回的 [`set` 函数](/reference/react/useState#setstate) 也具有稳定的标识，因此它们通常也会被省略。如果在省略某个依赖项时 linter 不会报错，那么这么做就是安全的。

Omitting always-stable dependencies only works when the linter can "see" that the object is stable. For example, if `ref` was passed from a parent component, you would have to specify it in the dependency array. However, this is good because you can't know whether the parent component always passes the same ref, or passes one of several refs conditionally. So your Effect _would_ depend on which ref is passed.
省略始终稳定的依赖项仅在 linter 能“看到”对象是稳定的时候才有效。例如，如果 `ref` 是从父组件传递过来的，则必须在依赖数组中指定它。这很有必要，因为你无法确定父组件是一直传递相同的 ref，还是根据条件传递不同的 ref。所以，你的 Effect 会依赖于被传递的是哪个 ref。

</DeepDive>

### 第三步：按需添加清理（cleanup）函数 | Step 3: Add cleanup if needed {/*step-3-add-cleanup-if-needed*/}

Consider a different example. You're writing a `ChatRoom` component that needs to connect to the chat server when it appears. You are given a `createConnection()` API that returns an object with `connect()` and `disconnect()` methods. How do you keep the component connected while it is displayed to the user?
考虑一个不同的例子。假如你正在编写一个 `ChatRoom` 组件，该组件在显示时需要连接到聊天服务器。现在为你提供了 `createConnection()` API，该 API 返回一个包含 `connect()` 与 `disconnection()` 方法的对象。如何确保组件在显示时始终保持连接？

Start by writing the Effect logic:
从编写 Effect 的逻辑开始：

```js
useEffect(() => {
  const connection = createConnection();
  connection.connect();
});
```

It would be slow to connect to the chat after every re-render, so you add the dependency array:
如果每次重新渲染后都得进行连接，这会很慢，所以你需要添加依赖数组：

```js {4}
useEffect(() => {
  const connection = createConnection();
  connection.connect();
}, []);
```

**The code inside the Effect does not use any props or state, so your dependency array is `[]` (empty). This tells React to only run this code when the component "mounts", i.e. appears on the screen for the first time.**
**由于 Effect 中的代码没有使用任何 props 或 state，所以依赖数组为空数组 `[]`。这告诉 React 仅在组件“挂载”（即首次显示在页面上）时运行此代码**。

Let's try running this code:
试试运行下面的代码：

<Sandpack>

```js
import { useEffect } from 'react';
import { createConnection } from './chat.js';

export default function ChatRoom() {
  useEffect(() => {
    const connection = createConnection();
    connection.connect();
  }, []);
  return <h1>欢迎来到聊天室！</h1>;
}
```

```js src/chat.js
export function createConnection() {
  // 真正的实现实际上会连接到服务器
  return {
    connect() {
      console.log('✅ 连接中……');
    },
    disconnect() {
      console.log('❌ 连接断开。');
    }
  };
}
```

```css
input { display: block; margin-bottom: 20px; }
```

</Sandpack>

This Effect only runs on mount, so you might expect `"✅ Connecting..."` to be printed once in the console. **However, if you check the console, `"✅ Connecting..."` gets printed twice. Why does it happen?**
这里的 Effect 仅在组件挂载时运行，所以你可能以为 `"✅ 连接中……"` 只会在控制台中被打印一次。**然而实际情况是 `"✅ 连接中……"` 被打印了两次！为什么会这样**？

Imagine the `ChatRoom` component is a part of a larger app with many different screens. The user starts their journey on the `ChatRoom` page. The component mounts and calls `connection.connect()`. Then imagine the user navigates to another screen--for example, to the Settings page. The `ChatRoom` component unmounts. Finally, the user clicks Back and `ChatRoom` mounts again. This would set up a second connection--but the first connection was never destroyed! As the user navigates across the app, the connections would keep piling up.
假设 `ChatRoom` 组件是一个大型多页面应用中的一部分。用户最初在 `ChatRoom` 页面上。组件挂载并调用 `connection.connect()` 。接着用户可能会导航到另一个页面，比如切换到“设置”页面，于是 `ChatRoom` 组件被卸载。最后，当用户点击“返回”时，`ChatRoom` 组件再次挂载。这将建立第二个连接——但第一个连接从未被销毁！随着用户在应用中来回切换，连接将会不断累积。

Bugs like this are easy to miss without extensive manual testing. To help you spot them quickly, in development React remounts every component once immediately after its initial mount.
这类 bug 在没有大量手动测试的情况下很容易被忽略。为了帮助你快速发现它们，在开发环境中，React 会在组件首次挂载后立即重新挂载一次。

Seeing the `"✅ Connecting..."` log twice helps you notice the real issue: your code doesn't close the connection when the component unmounts.
两次出现 `"✅ 连接中……"` 能够帮助你注意到真正的问题：在代码中，组件被卸载时没有关闭连接。

To fix the issue, return a *cleanup function* from your Effect:
为了解决这个问题，可以在 Effect 中返回一个 **清理（cleanup）函数** 。

```js {4-6}
  useEffect(() => {
    const connection = createConnection();
    connection.connect();
    return () => {
      connection.disconnect();
    };
  }, []);
```

React will call your cleanup function each time before the Effect runs again, and one final time when the component unmounts (gets removed). Let's see what happens when the cleanup function is implemented:
React 会在每次 Effect 重新运行之前调用清理函数，并在组件卸载（被移除）时最后一次调用清理函数。让我们看看实现清理函数后会发生什么：

<Sandpack>

```js
import { useState, useEffect } from 'react';
import { createConnection } from './chat.js';

export default function ChatRoom() {
  useEffect(() => {
    const connection = createConnection();
    connection.connect();
    return () => connection.disconnect();
  }, []);
  return <h1>欢迎来到聊天室！</h1>;
}
```

```js src/chat.js
export function createConnection() {
  // 真正的实现实际上会连接到服务器
  return {
    connect() {
      console.log('✅ 连接中……');
    },
    disconnect() {
      console.log('❌ 连接断开。');
    }
  };
}
```

```css
input { display: block; margin-bottom: 20px; }
```

</Sandpack>

Now you get three console logs in development:
现在在开发环境下，你会看到三条控制台日志：

1. `"✅ 连接中……"`
2. `"❌ 连接断开。"`
3. `"✅ 连接中……"`

**This is the correct behavior in development.** By remounting your component, React verifies that navigating away and back would not break your code. Disconnecting and then connecting again is exactly what should happen! When you implement the cleanup well, there should be no user-visible difference between running the Effect once vs running it, cleaning it up, and running it again. There's an extra connect/disconnect call pair because React is probing your code for bugs in development. This is normal--don't try to make it go away!
**在开发环境下，这是正确的行为**。通过重新挂载你的组件，React 验证了离开页面再返回不会导致代码出错。因为本就应该先断开然后再重新连接！如果你很好地实现了清理函数，那么无论是只执行一次 Effect ，还是执行、清理、再执行，都应该没有用户可见的区别。之所以会有额外的一次 connect/disconnect 调用，是因为在开发环境下 React 在检测你代码中的 bug。因此这是正常现象，不要去试图消除它！

**In production, you would only see `"✅ Connecting..."` printed once.** Remounting components only happens in development to help you find Effects that need cleanup. You can turn off [Strict Mode](/reference/react/StrictMode) to opt out of the development behavior, but we recommend keeping it on. This lets you find many bugs like the one above.
**在生产环境下，你只会看到 `"✅ 连接中……"` 打印一次**。这是因为重新挂载组件只会在开发环境下发生，以此帮助你找到需要清理的 Effect。你可以通过关闭 [严格模式](/reference/react/StrictMode) 来禁用这个行为，但我们建议保留它。它可以帮助你发现许多类似上述的 bug。

## 如何处理在开发环境下 Effect 运行了两次？| How to handle the Effect firing twice in development? {/*how-to-handle-the-effect-firing-twice-in-development*/}

React intentionally remounts your components in development to find bugs like in the last example. **The right question isn't "how to run an Effect once", but "how to fix my Effect so that it works after remounting".**
React 有意在开发环境下重新挂载你的组件，来找到类似上例中的 bug。**你需要思考的不是“如何只运行一次 Effect”，而是“如何修复我的 Effect 来让它在重新挂载后正常运行”**。

Usually, the answer is to implement the cleanup function.  The cleanup function should stop or undo whatever the Effect was doing. The rule of thumb is that the user shouldn't be able to distinguish between the Effect running once (as in production) and a _setup → cleanup → setup_ sequence (as you'd see in development).
通常，答案是实现清理函数。清理函数应该停止或撤销 Effect 所做的一切。原则是用户不应该感受到 Effect 只执行一次（在生产环境中）和连续执行“挂载 → 清理 → 挂载”（在开发环境中）之间的区别。

Most of the Effects you'll write will fit into one of the common patterns below.
你将编写的大多数 Effect 都会符合下列的常见模式之一。

<Pitfall>

#### 不要使用 ref 来防止触发 Effect | Don't use refs to prevent Effects from firing {/*dont-use-refs-to-prevent-effects-from-firing*/}

A common pitfall for preventing Effects firing twice in development is to use a `ref` to prevent the Effect from running more than once. For example, you could "fix" the above bug with a `useRef`:
为了防止 Effect 在开发环境中触发两次，一个常见错误是使用 `ref` 来让 Effect 只运行一次。例如，你可能会用 `useRef` “修复”上述的 bug：

```js {1,3-4}
  const connectionRef = useRef(null);
  useEffect(() => {
    // 🚩 This wont fix the bug!!!
    // 🚩 这并不能修复这个 bug！！！
    if (!connectionRef.current) {
      connectionRef.current = createConnection();
      connectionRef.current.connect();
    }
  }, []);
```

This makes it so you only see `"✅ Connecting..."` once in development, but it doesn't fix the bug.
它虽然使你在开发环境下只看到一次 `“✅ 正在连接...”`，但并没有修复这个 bug。

When the user navigates away, the connection still isn't closed and when they navigate back, a new connection is created. As the user navigates across the app, the connections would keep piling up, the same as it would before the "fix". 
当用户离开时，连接没有被关闭，当用户返回时，又会创建一个新的连接。随着用户浏览应用，连接会不断累积，就像“修复”之前一样。

To fix the bug, it is not enough to just make the Effect run once. The effect needs to work after re-mounting, which means the connection needs to be cleaned up like in the solution above.
要修复这个 bug，仅仅让 Effect 只运行一次是不够的。想要 Effect 在重新挂载后正常运行，就得按照之前的方法清除连接。

See the examples below for how to handle common patterns.
请看下面的示例，了解如何处理常见模式。

</Pitfall>

### 管理非 React 小部件 | Controlling non-React widgets {/*controlling-non-react-widgets*/}

Sometimes you need to add UI widgets that aren't written in React. For example, let's say you're adding a map component to your page. It has a `setZoomLevel()` method, and you'd like to keep the zoom level in sync with a `zoomLevel` state variable in your React code. Your Effect would look similar to this:
有时你需要添加不是用 React 实现的 UI 小部件。比如说你想在你的页面添加一个地图组件。它有一个 `setZoomLevel()` 方法，然后你希望地图的缩放比例和代码中的 `zoomLevel` state 保持同步。你的 Effect 应该类似于：

```js
useEffect(() => {
  const map = mapRef.current;
  map.setZoomLevel(zoomLevel);
}, [zoomLevel]);
```

Note that there is no cleanup needed in this case. In development, React will call the Effect twice, but this is not a problem because calling `setZoomLevel` twice with the same value does not do anything. It may be slightly slower, but this doesn't matter because it won't remount needlessly in production.
请注意，这种情况下不需要清理操作。在开发环境中，虽然 React 会调用 Effect 两次，但这没关系，因为用相同的值调用 `setZoomLevel` 两次不会造成任何影响。虽然在开发环境下它可能会稍微慢一些，但问题不大，因为在生产环境下它不会多余地重新挂载。

Some APIs may not allow you to call them twice in a row. For example, the [`showModal`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLDialogElement/showModal) method of the built-in [`<dialog>`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLDialogElement) element throws if you call it twice. Implement the cleanup function and make it close the dialog:
有些 API 可能不允许你连续调用两次。例如，内置的 [`<dialog>`](https://developer.mozilla.org/zh-CN/docs/Web/API/HTMLDialogElement) 元素的 [`showModal`](https://developer.mozilla.org/zh-CN/docs/Web/API/HTMLDialogElement/showModal) 方法在连续被调用两次时会抛出异常。此时可以通过实现清理函数来使其关闭对话框：

```js {4}
useEffect(() => {
  const dialog = dialogRef.current;
  dialog.showModal();
  return () => dialog.close();
}, []);
```

In development, your Effect will call `showModal()`, then immediately `close()`, and then `showModal()` again. This has the same user-visible behavior as calling `showModal()` once, as you would see in production.
在开发环境中，你的 Effect 会先调用 `showModal()`，然后立即调用 `close()`，之后再次调用 `showModal()`。这与在生产环境中只调用一次 `showModal()` 的用户可见行为是相同的。

### 订阅事件 | Subscribing to events {/*subscribing-to-events*/}

If your Effect subscribes to something, the cleanup function should unsubscribe:
如果你的 Effect 订阅了某些事件，清理函数应退订这些事件：

```js {6}
useEffect(() => {
  function handleScroll(e) {
    console.log(window.scrollX, window.scrollY);
  }
  window.addEventListener('scroll', handleScroll);
  return () => window.removeEventListener('scroll', handleScroll);
}, []);
```

In development, your Effect will call `addEventListener()`, then immediately `removeEventListener()`, and then `addEventListener()` again with the same handler. So there would be only one active subscription at a time. This has the same user-visible behavior as calling `addEventListener()` once, as in production.
在开发环境中，你的 Effect 会先调用 `addEventListener()`，然后立即调用 `removeEventListener()`，接着再次使用相同的处理函数调用 `addEventListener()`。因此，每次只会有一个有效订阅。这与在生产环境中只调用一次 `addEventListener()` 所产生的用户可见行为是相同的。

### 触发动画 | Triggering animations {/*triggering-animations*/}

If your Effect animates something in, the cleanup function should reset the animation to the initial values:
如果你的 Effect 触发了一些动画，清理函数应将动画重置为初始状态：

```js {4-6}
useEffect(() => {
  const node = ref.current;
  node.style.opacity = 1; // 触发动画 // Trigger the animation
  return () => {
    node.style.opacity = 0; // 重置为初始值 // Reset to the initial value
  };
}, []);
```

In development, opacity will be set to `1`, then to `0`, and then to `1` again. This should have the same user-visible behavior as setting it to `1` directly, which is what would happen in production. If you use a third-party animation library with support for tweening, your cleanup function should reset the timeline to its initial state.
在开发环境中，透明度由 `1` 变为 `0`，再变为 `1`。这与在生产环境中，直接将其设置为 `1` 具有相同的用户可见行为。如果你使用了支持补间动画的第三方动画库，你的清理函数应将时间轴重置为初始状态。

### 获取数据 | Fetching data {/*fetching-data*/}

If your Effect fetches something, the cleanup function should either [abort the fetch](https://developer.mozilla.org/en-US/docs/Web/API/AbortController) or ignore its result:
如果你的 Effect 需要获取数据，清理函数应 [中止请求](https://developer.mozilla.org/zh-CN/docs/Web/API/AbortController) 或忽略其结果：

```js {2,6,13-15}
useEffect(() => {
  let ignore = false;

  async function startFetching() {
    const json = await fetchTodos(userId);
    if (!ignore) {
      setTodos(json);
    }
  }

  startFetching();

  return () => {
    ignore = true;
  };
}, [userId]);
```

You can't "undo" a network request that already happened, but your cleanup function should ensure that the fetch that's _not relevant anymore_ does not keep affecting your application. If the `userId` changes from `'Alice'` to `'Bob'`, cleanup ensures that the `'Alice'` response is ignored even if it arrives after `'Bob'`.
你无法“撤销”已经发生的网络请求，但是你的清理函数应当确保那些不再相关的请求不会继续影响你的应用。如果 `userId` 从 `'Alice'` 变为 `'Bob'`，那么请确保 `'Alice'` 的响应数据被忽略，即使它在 `'Bob'` 之后到达。

**In development, you will see two fetches in the Network tab.** There is nothing wrong with that. With the approach above, the first Effect will immediately get cleaned up so its copy of the `ignore` variable will be set to `true`. So even though there is an extra request, it won't affect the state thanks to the `if (!ignore)` check.
**在开发环境中，你会在浏览器调试工具的“网络”选项卡中看到两条请求**。这是正常的。使用上述方法，第一个 Effect 将立即被清理，所以它的 `ignore` 变量会被设置为 `true`。因此，即使有额外的请求，由于有 `if (!ignore)` 的检查，也不会影响 state。

**In production, there will only be one request.** If the second request in development is bothering you, the best approach is to use a solution that deduplicates requests and caches their responses between components:
**在生产环境中，只会有一条请求**。如果开发环境中的第二次请求给你造成了困扰，最好的办法是使用一个能够对请求去重并缓存响应的方案：

```js
function TodoList() {
  const todos = useSomeDataLibrary(`/api/user/${userId}/todos`);
  // ...
```

This will not only improve the development experience, but also make your application feel faster. For example, the user pressing the Back button won't have to wait for some data to load again because it will be cached. You can either build such a cache yourself or use one of the many alternatives to manual fetching in Effects.
这不仅可以提高开发体验，还可以让你的应用程序响应更快。例如，当用户点击返回按钮时，不用再等待数据重新加载，因为它已经被缓存。你可以自己构建这样的缓存机制，也可以使用很多在 Effect 中手动获取数据的替代方法。

<DeepDive>

#### 在 Effect 中进行数据请求的替代方案 | What are good alternatives to data fetching in Effects? {/*what-are-good-alternatives-to-data-fetching-in-effects*/}

Writing `fetch` calls inside Effects is a [popular way to fetch data](https://www.robinwieruch.de/react-hooks-fetch-data/), especially in fully client-side apps. This is, however, a very manual approach and it has significant downsides:
在 Effect 中直接编写 `fetch` 请求 [是一种常见的数据获取方式](https://www.robinwieruch.de/react-hooks-fetch-data/)，特别是在完全客户端渲染的应用中。然而，这种方法非常手动化，并且有明显的弊端：

- **Effects don't run on the server.** This means that the initial server-rendered HTML will only include a loading state with no data. The client computer will have to download all JavaScript and render your app only to discover that now it needs to load the data. This is not very efficient.
- **Effect 不会在服务端运行**。这意味着最初由服务器渲染的 HTML 只会包含加载状态，而没有实际数据。客户端必须先下载所有的 JavaScript 并渲染应用，才会发现它需要加载数据——这并不高效。
- **Fetching directly in Effects makes it easy to create "network waterfalls".** You render the parent component, it fetches some data, renders the child components, and then they start fetching their data. If the network is not very fast, this is significantly slower than fetching all data in parallel.
- **直接在 Effect 中进行数据请求，容易产生“网络瀑布（network waterfall）”**。首先父组件渲染时请求一些数据，随后渲染子组件，接着子组件开始请求它们的数据。如果网络速度不快，这种方式会比并行获取所有数据慢得多。
- **Fetching directly in Effects usually means you don't preload or cache data.** For example, if the component unmounts and then mounts again, it would have to fetch the data again.
- **直接在 Effect 中进行数据请求往往无法预加载或缓存数据**。例如，如果组件卸载后重新挂载，它必须重新获取数据。
- **It's not very ergonomic.** There's quite a bit of boilerplate code involved when writing `fetch` calls in a way that doesn't suffer from bugs like [race conditions.](https://maxrozen.com/race-conditions-fetching-data-react-with-useeffect)
- **不够简洁**。编写 fecth 请求时为了避免 [竞态条件（race condition）](https://maxrozen.com/race-conditions-fetching-data-react-with-useeffect) 等问题，会需要很多样板代码。

This list of downsides is not specific to React. It applies to fetching data on mount with any library. Like with routing, data fetching is not trivial to do well, so we recommend the following approaches:
这些弊端并不仅限于 React。任何库在组件挂载时进行数据获取都会遇到这些问题。与路由处理一样，要做好数据获取并非易事，因此我们推荐以下方法：

- **If you use a [framework](/learn/start-a-new-react-project#production-grade-react-frameworks), use its built-in data fetching mechanism.** Modern React frameworks have integrated data fetching mechanisms that are efficient and don't suffer from the above pitfalls.
- **如果你正在使用 [框架](/learn/start-a-new-react-project#production-grade-react-frameworks) ，请使用其内置的数据获取机制**。现代 React 框架集成了高效的数据获取机制，不会出现上述问题。
- **Otherwise, consider using or building a client-side cache.** Popular open source solutions include [React Query](https://tanstack.com/query/latest), [useSWR](https://swr.vercel.app/), and [React Router 6.4+.](https://beta.reactrouter.com/en/main/start/overview) You can build your own solution too, in which case you would use Effects under the hood, but add logic for deduplicating requests, caching responses, and avoiding network waterfalls (by preloading data or hoisting data requirements to routes).
- **否则，请考虑使用或构建客户端缓存**。流行的开源解决方案包括 [React Query](https://tanstack.com/query/latest)、[useSWR](https://swr.vercel.app/) 和 [React Router v6.4+](https://beta.reactrouter.com/en/main/start/overview)。你也可以自己构建解决方案：在底层使用 Effect，但添加对请求的去重、缓存响应以及避免网络瀑布（通过预加载数据或将数据请求提升到路由层次）的逻辑。

You can continue fetching data directly in Effects if neither of these approaches suit you.
如果这些方法都不适合你，你可以继续直接在 Effect 中获取数据。

</DeepDive>

### 发送分析报告 | Sending analytics {/*sending-analytics*/}

Consider this code that sends an analytics event on the page visit:
考虑以下代码，它在页面访问时发送一个分析事件：

```js
useEffect(() => {
  logVisit(url); // 发送 POST 请求 // Sends a POST request
}, [url]);
```

In development, `logVisit` will be called twice for every URL, so you might be tempted to try to fix that. **We recommend keeping this code as is.** Like with earlier examples, there is no *user-visible* behavior difference between running it once and running it twice. From a practical point of view, `logVisit` should not do anything in development because you don't want the logs from the development machines to skew the production metrics. Your component remounts every time you save its file, so it logs extra visits in development anyway.
在开发环境中，对于每个 URL，`logVisit` 都会被调用两次，因此你可能会尝试修复这个问题。**我们建议保持不动**。与之前示例类似，运行一次还是运行两次，在用户可见的行为上没有区别。从实际角度来看，`logVisit` 不应该在开发环境中执行任何操作，因为你不会想让开发设备的日志影响生产环境的统计数据。每次保存文件时组件都会重新挂载，因此在开发环境中会记录额外的访问日志。

**In production, there will be no duplicate visit logs.**
**在生产环境中，不会有重复的访问日志**。

To debug the analytics events you're sending, you can deploy your app to a staging environment (which runs in production mode) or temporarily opt out of [Strict Mode](/reference/react/StrictMode) and its development-only remounting checks. You may also send analytics from the route change event handlers instead of Effects. For more precise analytics, [intersection observers](https://developer.mozilla.org/en-US/docs/Web/API/Intersection_Observer_API) can help track which components are in the viewport and how long they remain visible.
为了调试发送的分析事件，你可以将应用部署到一个运行在生产模式下的暂存环境，或者暂时禁用 [严格模式](/reference/react/StrictMode) 及其仅在开发环境中的重新挂载检查。你还可以在路由更改的事件处理程序中发送分析数据，而不是在 Effect 中发送。对于更精确的分析，可以使用[交叉观察器](https://developer.mozilla.org/zh-CN/docs/Web/API/Intersection_Observer_API) 来跟踪哪些组件位于视口中以及它们保持可见的时间。

### 不适用于 Effect：初始化应用 | Not an Effect: Initializing the application {/*not-an-effect-initializing-the-application*/}

Some logic should only run once when the application starts. You can put it outside your components:
某些逻辑应该只在应用启动时运行一次。你可以将它放在组件外部：

```js {2-3}
if (typeof window !== 'undefined') { // 检查是否在浏览器中运行 // Check if we're running in the browser.
  checkAuthToken();
  loadDataFromLocalStorage();
}

function App() {
  // ……
}
```

This guarantees that such logic only runs once after the browser loads the page.
这可以确保此类逻辑只在浏览器加载页面后运行一次。

### 不适用于 Effect：购买商品 | Not an Effect: Buying a product {/*not-an-effect-buying-a-product*/}

Sometimes, even if you write a cleanup function, there's no way to prevent user-visible consequences of running the Effect twice. For example, maybe your Effect sends a POST request like buying a product:
有时，即使你编写了清理函数，也无法避免用户观察到 Effect 运行了两次。比如你的 Effect 发送了一个像购买商品这样的 POST 请求：

```js {2-3}
useEffect(() => {
  // 🔴 Wrong: This Effect fires twice in development, exposing a problem in the code.
  // 🔴 错误：此处的 Effect 在开发环境中会触发两次，暴露出代码中的问题。
  fetch('/api/buy', { method: 'POST' });
}, []);
```

You wouldn't want to buy the product twice. However, this is also why you shouldn't put this logic in an Effect. What if the user goes to another page and then presses Back? Your Effect would run again. You don't want to buy the product when the user *visits* a page; you want to buy it when the user *clicks* the Buy button.
你肯定不希望购买两次商品。这也是为什么你不应该把这种逻辑放在 Effect 中。如果用户跳转到另一个页面，然后按下“返回”按钮，你的 Effect 就会再次运行。你不希望用户在访问页面时就购买产品，而是在他们点击“购买”按钮时才购买。

Buying is not caused by rendering; it's caused by a specific interaction. It should run only when the user presses the button. **Delete the Effect and move your `/api/buy` request into the Buy button event handler:**
购买操作并不是由渲染引起的，而是由特定的交互引起的。它应该只在用户按下按钮时执行。因此，**它不应该写在 Effect 中，应当把 `/api/buy` 请求移动到“购买”按钮的事件处理程序中**：

```js {2-3}
  function handleClick() {
    // ✅ Buying is an event because it is caused by a particular interaction.
    // ✅ 购买行为是一个事件，因为它是由特定的交互引起的。
    fetch('/api/buy', { method: 'POST' });
  }
```

**This illustrates that if remounting breaks the logic of your application, this usually uncovers existing bugs.** From a user's perspective, visiting a page shouldn't be different from visiting it, clicking a link, then pressing Back to view the page again. React verifies that your components abide by this principle by remounting them once in development.
**这说明了如果重新挂载破坏了应用的逻辑，通常便暴露了存在的 bug**。对用户而言，访问一个页面不应该与访问页面后点击链接、再按下“返回”按钮查看页面有区别。React 通过在开发环境中重新挂载组件来验证你的组件是否遵守这一原则。

## 综合以上内容 | Putting it all together {/*putting-it-all-together*/}

This playground can help you "get a feel" for how Effects work in practice.
这个演练场可以帮助你“感受” Effect 在实际中的工作方式。

This example uses [`setTimeout`](https://developer.mozilla.org/en-US/docs/Web/API/setTimeout) to schedule a console log with the input text to appear three seconds after the Effect runs. The cleanup function cancels the pending timeout. Start by pressing "Mount the component":
这个例子使用 [`setTimeout`](https://developer.mozilla.org/zh-CN/docs/Web/API/setTimeout) 调度一个日志记录，日志会在 Effect 运行三秒后显示输入的文本。清理函数会取消挂起的延时器。从按下“挂载组件”开始：

<Sandpack>

```js
import { useState, useEffect } from 'react';

function Playground() {
  const [text, setText] = useState('a');

  useEffect(() => {
    function onTimeout() {
      console.log('⏰ ' + text);
    }

    console.log('🔵 调度 "' + text + '" 日志');
    const timeoutId = setTimeout(onTimeout, 3000);

    return () => {
      console.log('🟡 取消 "' + text + '" 日志');
      clearTimeout(timeoutId);
    };
  }, [text]);

  return (
    <>
      <label>
        日志内容：{' '}
        <input
          value={text}
          onChange={e => setText(e.target.value)}
        />
      </label>
      <h1>{text}</h1>
    </>
  );
}

export default function App() {
  const [show, setShow] = useState(false);
  return (
    <>
      <button onClick={() => setShow(!show)}>
        {show ? '卸载' : '挂载'}组件
      </button>
      {show && <hr />}
      {show && <Playground />}
    </>
  );
}
```

</Sandpack>

You will see three logs at first: `Schedule "a" log`, `Cancel "a" log`, and `Schedule "a" log` again. Three second later there will also be a log saying `a`. As you learned earlier, the extra schedule/cancel pair is because React remounts the component once in development to verify that you've implemented cleanup well.
你首先会看到三条日志：`调度 "a" 日志`，`取消 "a" 日志`，再一条 `调度 "a" 日志`。三秒后，你还会看到一条日志：`a`。正如你先前所学，额外的调度/取消日志是因为 React 在开发环境中会重新挂载组件一次，以验证你是否正确实现了清理操作。

Now edit the input to say `abc`. If you do it fast enough, you'll see `Schedule "ab" log` immediately followed by `Cancel "ab" log` and `Schedule "abc" log`. **React always cleans up the previous render's Effect before the next render's Effect.** This is why even if you type into the input fast, there is at most one timeout scheduled at a time. Edit the input a few times and watch the console to get a feel for how Effects get cleaned up.
现在编辑输入框，输入 `abc`。如果输入速度足够快，你会看到 `调度 "ab" 日志`，紧接着 `取消 "ab" 日志` 和 `调度 "abc" 日志`。**React 总是在执行下一轮渲染的 Effect 之前清理上一轮渲染的 Effect**。这就是为什么即使你快速输入，最多也只有一个延时器被调度。试试多次编辑输入框，并观察控制台以了解 Effect 是如何被清理的。

Type something into the input and then immediately press "Unmount the component". Notice how unmounting cleans up the last render's Effect. Here, it clears the last timeout before it has a chance to fire.
在输入框中输入一些内容，然后立即按下“卸载组件”。注意卸载组件时是如何清理最后一轮渲染的 Effect 的。在这里，它会在最后一个延迟器要触发之前取消它。

Finally, edit the component above and comment out the cleanup function so that the timeouts don't get cancelled. Try typing `abcde` fast. What do you expect to happen in three seconds? Will `console.log(text)` inside the timeout print the *latest* `text` and produce five `abcde` logs? Give it a try to check your intuition!
最后，在上面的代码中注释掉清理函数，这样延时器就不会被取消。尝试快速输入 `abcde`。你觉得三秒后会发生什么？延时器中的 `console.log(text)` 会打印 **最新** 的 `text` 值并生成五条 `abcde` 日志吗？试试看吧，验证一下你的直觉！

Three seconds later, you should see a sequence of logs (`a`, `ab`, `abc`, `abcd`, and `abcde`) rather than five `abcde` logs. **Each Effect "captures" the `text` value from its corresponding render.**  It doesn't matter that the `text` state changed: an Effect from the render with `text = 'ab'` will always see `'ab'`. In other words, Effects from each render are isolated from each other. If you're curious how this works, you can read about [closures](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Closures).
三秒后，你应该会看到一系列的日志：`a`、`ab`、`abc`、`abcd` 与 `abcde`，而不是五条 `abcde` 日志。这是因为 **每个 Effect 都会“捕获”它对应渲染时的 `text` 值**。即使 `text` 的值发生了变化，渲染时 `text = 'ab'` 的 Effect 总是会得到 `'ab'`。换句话说，每个渲染的 Effect 都是相互独立的。如果你对这种机制感兴趣，可以阅读有关 [闭包](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Closures) 的内容。

<DeepDive>

#### 每一轮渲染都有其自己的 Effect | Each render has its own Effects {/*each-render-has-its-own-effects*/}

You can think of `useEffect` as "attaching" a piece of behavior to the render output. Consider this Effect:
你可以将 `useEffect` 理解为“附加”一段行为到渲染输出中。考虑下面这个 Effect：

```js
export default function ChatRoom({ roomId }) {
  useEffect(() => {
    const connection = createConnection(roomId);
    connection.connect();
    return () => connection.disconnect();
  }, [roomId]);

  return <h1>欢迎来到 {roomId}！</h1>;
}
```

Let's see what exactly happens as the user navigates around the app.
让我们看看当用户在应用程序中切换页面时到底发生了什么。

#### 初次渲染 | Initial render {/*initial-render*/}

The user visits `<ChatRoom roomId="general" />`. Let's [mentally substitute](/learn/state-as-a-snapshot#rendering-takes-a-snapshot-in-time) `roomId` with `'general'`:
用户访问 `<ChatRoom roomId="general" />`。我们 [假设](/learn/state-as-a-snapshot#rendering-takes-a-snapshot-in-time) `roomId` 的值为 `'general'` ：

```js
  // JSX for the first render (roomId = "general")
  // 首次渲染时的 JSX（roomId 为 "general"）
  return <h1>欢迎来到 general！</h1>;
```

**The Effect is *also* a part of the rendering output.** The first render's Effect becomes:
**Effect 也是渲染输出的一部分**。首次渲染的 Effect 变为：

```js
  // Effect for the first render (roomId = "general")
  //首先渲染时的 Effect（roomId 为 "general"）
  () => {
    const connection = createConnection('general');
    connection.connect();
    return () => connection.disconnect();
  },
  // Dependencies for the first render (roomId = "general")
  // 首次渲染时的依赖项（roomId 为 "general"）
  ['general']
```

React runs this Effect, which connects to the `'general'` chat room.
React 运行这个 Effect 来连接到 `'general'` 聊天室。

#### 依赖项相同时的重新渲染 | Re-render with same dependencies {/*re-render-with-same-dependencies*/}

Let's say `<ChatRoom roomId="general" />` re-renders. The JSX output is the same:
假设 `<ChatRoom roomId="general" />` 重新渲染。输出的 JSX 不变：

```js
  // JSX for the second render (roomId = "general")
  // 第二次渲染时的 JSX（roomId 为 "general"）
  return <h1>Welcome to general!</h1>;
```

React sees that the rendering output has not changed, so it doesn't update the DOM.
React 看到渲染输出没有改变，所以不更新 DOM 。

The Effect from the second render looks like this:
第二次渲染的 Effect 如下所示：

```js
  // Effect for the second render (roomId = "general")
  // 第二次渲染时的 Effect（roomId 为 "general"）
  () => {
    const connection = createConnection('general');
    connection.connect();
    return () => connection.disconnect();
  },
  // Dependencies for the second render (roomId = "general")
  // 第二次渲染时的依赖项（roomId 为 "general"）
  ['general']
```

React compares `['general']` from the second render with `['general']` from the first render. **Because all dependencies are the same, React *ignores* the Effect from the second render.** It never gets called.
React 将第二次渲染时的 `['general']` 与第一次渲染时的 `['general']` 进行比较。**因为所有的依赖项都相同，React 忽略第二次渲染时的 Effect**。Effect 不会被调用。

#### 依赖项不同时的重新渲染 | Re-render with different dependencies {/*re-render-with-different-dependencies*/}

Then, the user visits `<ChatRoom roomId="travel" />`. This time, the component returns different JSX:
接着，用户访问了 `<ChatRoom roomId="travel" />`。这一次，组件返回了不同的 JSX：

```js
  // JSX for the third render (roomId = "travel")
  // 第三次渲染时的 JSX（roomId 为 "travel"）
  return <h1>欢迎来到 travel！</h1>;
```

React updates the DOM to change `"Welcome to general"` into `"Welcome to travel"`.
React 更新 DOM ，将 `"欢迎来到 general"` 改为 `"欢迎来到 travel"`。

The Effect from the third render looks like this:
第三次渲染的 Effect 如下所示：

```js
  // Effect for the third render (roomId = "travel")
  // 第三次渲染时的 Effect（roomId 为 "travel"）
  () => {
    const connection = createConnection('travel');
    connection.connect();
    return () => connection.disconnect();
  },
  // Dependencies for the third render (roomId = "travel")
  // 第三次渲染时的依赖项（roomId 为 "travel"）
  ['travel']
```

React compares `['travel']` from the third render with `['general']` from the second render. One dependency is different: `Object.is('travel', 'general')` is `false`. The Effect can't be skipped.
React 将第三次渲染时的 `['travel']` 与第二次渲染时的 `['general']` 相比较。发现依赖项不同：`Object.is('travel', 'general')` 为 `false`。因此 Effect 不能被跳过。

**Before React can apply the Effect from the third render, it needs to clean up the last Effect that _did_ run.** The second render's Effect was skipped, so React needs to clean up the first render's Effect. If you scroll up to the first render, you'll see that its cleanup calls `disconnect()` on the connection that was created with `createConnection('general')`. This disconnects the app from the `'general'` chat room.
**在 React 执行第三次渲染的 Effect 之前，它需要清理上一个运行的 Effect**。第二次渲染的 Effect 被跳过了。所以 React 需要清理第一次渲染时的 Effect。如果你回看第一次渲染，你会发现第一次渲染时的清理函数所做的事，是在 `createConnection('general')` 所创建的连接上调用 `disconnect()`。也就是从 `'general'` 聊天室断开连接。

After that, React runs the third render's Effect. It connects to the `'travel'` chat room.
之后，React 执行第三次渲染的 Effect。它连接到 `'travel'` 聊天室。

#### 组件卸载 | Unmount {/*unmount*/}

Finally, let's say the user navigates away, and the `ChatRoom` component unmounts. React runs the last Effect's cleanup function. The last Effect was from the third render. The third render's cleanup destroys the `createConnection('travel')` connection. So the app disconnects from the `'travel'` room.
最后，假设用户离开了当前页面，`ChatRoom` 组件被卸载。React 执行上一个运行的 Effect 的清理函数，也就是第三次渲染时的 Effect。这个清理函数会销毁 `createConnection('travel')` 所创建的连接。这样，应用与 `travel` 房间断开了连接。

#### 仅开发环境下的行为 | Development-only behaviors {/*development-only-behaviors*/}

When [Strict Mode](/reference/react/StrictMode) is on, React remounts every component once after mount (state and DOM are preserved). This [helps you find Effects that need cleanup](#step-3-add-cleanup-if-needed) and exposes bugs like race conditions early. Additionally, React will remount the Effects whenever you save a file in development. Both of these behaviors are development-only.
开启 [严格模式](/reference/react/StrictMode) 时，React 在每次挂载组件后都会重新挂载组件（组件的 state 与 创建的 DOM 都会被保留）。[它可以帮助你找出需要添加清理函数的 Effect](#step-3-add-cleanup-if-needed)，并在早期暴露类似竞态条件这样的 bug。此外，每当你在开发环境中保存文件时，React 也会重新挂载 Effect。这些行为都仅限于开发环境。

</DeepDive>

<Recap>

- Unlike events, Effects are caused by rendering itself rather than a particular interaction.
- 与事件不同，Effect 由渲染本身引起，而非特定的交互。
- Effects let you synchronize a component with some external system (third-party API, network, etc).
- Effect 允许你将组件与某些外部系统（第三方 API、网络等）同步。
- By default, Effects run after every render (including the initial one).
- 默认情况下，Effect 在每次渲染（包括初始渲染）后运行。
- React will skip the Effect if all of its dependencies have the same values as during the last render.
- 如果所有依赖项都与上一次渲染时相同，React 会跳过本次 Effect。
- You can't "choose" your dependencies. They are determined by the code inside the Effect.
- 你不能“选择”依赖项，它们是由 Effect 内部的代码所决定的。
- Empty dependency array (`[]`) corresponds to the component "mounting", i.e. being added to the screen.
- 空的依赖数组（`[]`）对应于组件的“挂载”，即组件被添加到页面上时。
- In Strict Mode, React mounts components twice (in development only!) to stress-test your Effects.
- 仅在严格模式下的开发环境中，React 会挂载两次组件，以对 Effect 进行压力测试。
- If your Effect breaks because of remounting, you need to implement a cleanup function.
- 如果你的 Effect 因为重新挂载而出现问题，那么你需要实现一个清理函数。
- React will call your cleanup function before the Effect runs next time, and during the unmount.
- React 会在 Effect 再次运行之前和在组件卸载时调用你的清理函数。

</Recap>

<Challenges>

#### 挂载后聚焦于表单字段 | Focus a field on mount {/*focus-a-field-on-mount*/}

In this example, the form renders a `<MyInput />` component.
在下面的例子中，表单中渲染了一个 `<MyInput />` 组件。

Use the input's [`focus()`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLElement/focus) method to make `MyInput` automatically focus when it appears on the screen. There is already a commented out implementation, but it doesn't quite work. Figure out why it doesn't work, and fix it. (If you're familiar with the `autoFocus` attribute, pretend that it does not exist: we are reimplementing the same functionality from scratch.)
使用输入框的 [`focus()`](https://developer.mozilla.org/zh-CN/docs/Web/API/HTMLElement/focus) 方法，让 `MyInput` 在页面上出现时自动聚焦。已经有一个被注释掉的实现，但它并不能正常工作。找出它为什么不起作用，并修复它。如果你熟悉 `autoFocus` 属性，请假装它不存在：我们正在从头开始实现相同的功能。

<Sandpack>

```js src/MyInput.js active
import { useEffect, useRef } from 'react';

export default function MyInput({ value, onChange }) {
  const ref = useRef(null);

  // TODO: This doesn't quite work. Fix it.
  // ref.current.focus()    
  // TODO：下面的这种做法不会生效，请修复。
  // ref.current.focus()    

  return (
    <input
      ref={ref}
      value={value}
      onChange={onChange}
    />
  );
}
```

```js src/App.js hidden
import { useState } from 'react';
import MyInput from './MyInput.js';

export default function Form() {
  const [show, setShow] = useState(false);
  const [name, setName] = useState('Taylor');
  const [upper, setUpper] = useState(false);
  return (
    <>
      <button onClick={() => setShow(s => !s)}>{show ? '隐藏' : '展示'}表单</button>
      <br />
      <hr />
      {show && (
        <>
          <label>
            输入你的姓名：
            <MyInput
              value={name}
              onChange={e => setName(e.target.value)}
            />
          </label>
          <label>
            <input
              type="checkbox"
              checked={upper}
              onChange={e => setUpper(e.target.checked)}
            />
            大写
          </label>
          <p>你好， <b>{upper ? name.toUpperCase() : name}</b></p>
        </>
      )}
    </>
  );
}
```

```css
label {
  display: block;
  margin-top: 20px;
  margin-bottom: 20px;
}

body {
  min-height: 150px;
}
```

</Sandpack>


To verify that your solution works, press "Show form" and verify that the input receives focus (becomes highlighted and the cursor is placed inside). Press "Hide form" and "Show form" again. Verify the input is highlighted again.
要验证你的方法是否奏效，请点击“展示表单”，并确认输入框获得焦点（高亮显示并且光标位于内部）；再次点击“隐藏表单”和“展示表单”，确认输入框是否再次被高亮显示。

`MyInput` should only focus _on mount_ rather than after every render. To verify that the behavior is right, press "Show form" and then repeatedly press the "Make it uppercase" checkbox. Clicking the checkbox should _not_ focus the input above it.
`MyInput` 组件应该只在 **挂载** 时获得焦点，而非每次渲染后。为了验证这一行为是否正确，点击“展示表单”，然后反复点击“大写”的复选框。点击复选框时，上方的输入框不应该获得焦点。

<Solution>

Calling `ref.current.focus()` during render is wrong because it is a *side effect*. Side effects should either be placed inside an event handler or be declared with `useEffect`. In this case, the side effect is _caused_ by the component appearing rather than by any specific interaction, so it makes sense to put it in an Effect.
在渲染期间调用 `ref.current.focus()` 是错误的。因为它是一个“副作用”。副作用应该放在事件处理程序中或通过 `useEffect` 来声明。在这里，这个副作用是由组件渲染引起的，而不是任何特定的交互引起的，因此应该将它放在 Effect 中。

To fix the mistake, wrap the `ref.current.focus()` call into an Effect declaration. Then, to ensure that this Effect runs only on mount rather than after every render, add the empty `[]` dependencies to it.
要修复这个错误，需要将 `ref.current.focus()` 调用包裹在 Effect 中。然后，为了确保这个 Effect 只在组件挂载时运行，而不是每一轮渲染后都运行，再添加一个空的依赖数组 `[]`。

<Sandpack>

```js src/MyInput.js active
import { useEffect, useRef } from 'react';

export default function MyInput({ value, onChange }) {
  const ref = useRef(null);

  useEffect(() => {
    ref.current.focus();
  }, []);

  return (
    <input
      ref={ref}
      value={value}
      onChange={onChange}
    />
  );
}
```

```js src/App.js hidden
import { useState } from 'react';
import MyInput from './MyInput.js';

export default function Form() {
  const [show, setShow] = useState(false);
  const [name, setName] = useState('Taylor');
  const [upper, setUpper] = useState(false);
  return (
    <>
      <button onClick={() => setShow(s => !s)}>{show ? '隐藏' : '展示'}表单</button>
      <br />
      <hr />
      {show && (
        <>
          <label>
            输入你的姓名：
            <MyInput
              value={name}
              onChange={e => setName(e.target.value)}
            />
          </label>
          <label>
            <input
              type="checkbox"
              checked={upper}
              onChange={e => setUpper(e.target.checked)}
            />
            大写
          </label>
          <p>你好，<b>{upper ? name.toUpperCase() : name}</b></p>
        </>
      )}
    </>
  );
}
```

```css
label {
  display: block;
  margin-top: 20px;
  margin-bottom: 20px;
}

body {
  min-height: 150px;
}
```

</Sandpack>

</Solution>

#### 有条件地聚焦于表单字段 | Focus a field conditionally {/*focus-a-field-conditionally*/}

This form renders two `<MyInput />` components.
下面的表单渲染两个 `<MyInput />` 组件。

Press "Show form" and notice that the second field automatically gets focused. This is because both of the `<MyInput />` components try to focus the field inside. When you call `focus()` for two input fields in a row, the last one always "wins".
点击“展示表单”后，注意第二个输入框会自动获取焦点。这是因为两个 `<MyInput />` 组件在内部抢占焦点。当你连续为两个输入框调用 `focus()` 时，最后一个总会“获胜”。

Let's say you want to focus the first field. The first `MyInput` component now receives a boolean `shouldFocus` prop set to `true`. Change the logic so that `focus()` is only called if the `shouldFocus` prop received by `MyInput` is `true`.
假设你希望聚焦于第一个输入框。现在，第一个 `MyInput` 组件接收一个布尔类型的 `shouldFocus` 属性，且值设置为 `true`。请修改程序逻辑，使得仅当 `MyInput` 接收到的 `shouldFocus` 属性为 `true` 时才调用 `focus()` 。

<Sandpack>

```js src/MyInput.js active
import { useEffect, useRef } from 'react';

export default function MyInput({ shouldFocus, value, onChange }) {
  const ref = useRef(null);

  // TODO: call focus() only if shouldFocus is true.
  // TODO：只在 shouldFocus 为 true 时才调用 focus()
  useEffect(() => {
    ref.current.focus();
  }, []);

  return (
    <input
      ref={ref}
      value={value}
      onChange={onChange}
    />
  );
}
```

```js src/App.js hidden
import { useState } from 'react';
import MyInput from './MyInput.js';

export default function Form() {
  const [show, setShow] = useState(false);
  const [firstName, setFirstName] = useState('Taylor');
  const [lastName, setLastName] = useState('Swift');
  const [upper, setUpper] = useState(false);
  const name = firstName + ' ' + lastName;
  return (
    <>
      <button onClick={() => setShow(s => !s)}>{show ? '隐藏' : '展示'}表单</button>
      <br />
      <hr />
      {show && (
        <>
          <label>
            输入你的名：
            <MyInput
              value={firstName}
              onChange={e => setFirstName(e.target.value)}
              shouldFocus={true}
            />
          </label>
          <label>
            输入你的姓：
            <MyInput
              value={lastName}
              onChange={e => setLastName(e.target.value)}
              shouldFocus={false}
            />
          </label>
          <p>你好，<b>{upper ? name.toUpperCase() : name}</b></p>
        </>
      )}
    </>
  );
}
```

```css
label {
  display: block;
  margin-top: 20px;
  margin-bottom: 20px;
}

body {
  min-height: 150px;
}
```

</Sandpack>

To verify your solution, press "Show form" and "Hide form" repeatedly. When the form appears, only the *first* input should get focused. This is because the parent component renders the first input with `shouldFocus={true}` and the second input with `shouldFocus={false}`. Also check that both inputs still work and you can type into both of them.
要验证你的方法是否奏效，请重复点击“展示表单”和“隐藏表单”。当表单显示时，只有第一个输入框该获得焦点。因为父组件在渲染第一个输入框时传入了 `shouldFocus={true}`，而渲染第二个输入框时传入了 `shouldFocus={false}`。同时，请确认两个输入框都能正常工作并且你都能在其中输入。

<Hint>

You can't declare an Effect conditionally, but your Effect can include conditional logic.
你不能有条件地声明 Effect，但你的 Effect 中可以包含条件逻辑。

</Hint>

<Solution>

Put the conditional logic inside the Effect. You will need to specify `shouldFocus` as a dependency because you are using it inside the Effect. (This means that if some input's `shouldFocus` changes from `false` to `true`, it will focus after mount.)
在 Effect 中加入条件逻辑。由于你在 Effect 中使用了 `shouldFocus`，因此需要将它指定为依赖项。这意味着如果某个输入框的 `shouldFocus` 由 `false` 变为 `true`，它将在挂载后获得焦点。

<Sandpack>

```js src/MyInput.js active
import { useEffect, useRef } from 'react';

export default function MyInput({ shouldFocus, value, onChange }) {
  const ref = useRef(null);

  useEffect(() => {
    if (shouldFocus) {
      ref.current.focus();
    }
  }, [shouldFocus]);

  return (
    <input
      ref={ref}
      value={value}
      onChange={onChange}
    />
  );
}
```

```js src/App.js hidden
import { useState } from 'react';
import MyInput from './MyInput.js';

export default function Form() {
  const [show, setShow] = useState(false);
  const [firstName, setFirstName] = useState('Taylor');
  const [lastName, setLastName] = useState('Swift');
  const [upper, setUpper] = useState(false);
  const name = firstName + ' ' + lastName;
  return (
    <>
      <button onClick={() => setShow(s => !s)}>{show ? '隐藏' : '展示'}表单</button>
      <br />
      <hr />
      {show && (
        <>
          <label>
            输入你的名：
            <MyInput
              value={firstName}
              onChange={e => setFirstName(e.target.value)}
              shouldFocus={true}
            />
          </label>
          <label>
            输入你的姓：
            <MyInput
              value={lastName}
              onChange={e => setLastName(e.target.value)}
              shouldFocus={false}
            />
          </label>
          <p>你好，<b>{upper ? name.toUpperCase() : name}</b></p>
        </>
      )}
    </>
  );
}
```

```css
label {
  display: block;
  margin-top: 20px;
  margin-bottom: 20px;
}

body {
  min-height: 150px;
}
```

</Sandpack>

</Solution>

#### 修复会触发两次的定时器 | Fix an interval that fires twice {/*fix-an-interval-that-fires-twice*/}

This `Counter` component displays a counter that should increment every second. On mount, it calls [`setInterval`.](https://developer.mozilla.org/en-US/docs/Web/API/setInterval) This causes `onTick` to run every second. The `onTick` function increments the counter.
这个 `Counter` 组件展示了一个每秒递增的计数器。在组件挂载时，它调用了 [`setInterval`](https://developer.mozilla.org/zh-CN/docs/Web/API/setInterval)。这使得 `onTick` 每秒运行一次。`onTick` 函数会递增计数器。

However, instead of incrementing once per second, it increments twice. Why is that? Find the cause of the bug and fix it.
然而，计数器不是每秒递增一次，而是两次。这是为什么呢？找出 bug 的原因并修复它。

<Hint>

Keep in mind that `setInterval` returns an interval ID, which you can pass to [`clearInterval`](https://developer.mozilla.org/en-US/docs/Web/API/clearInterval) to stop the interval.
请记住，`setInterval` 返回一个 interval ID，你可以将它传递给 [`clearInterval`](https://developer.mozilla.org/zh-CN/docs/Web/API/clearInterval) 来停止这个定时器。

</Hint>

<Sandpack>

```js src/Counter.js active
import { useState, useEffect } from 'react';

export default function Counter() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    function onTick() {
      setCount(c => c + 1);
    }

    setInterval(onTick, 1000);
  }, []);

  return <h1>{count}</h1>;
}
```

```js src/App.js hidden
import { useState } from 'react';
import Counter from './Counter.js';

export default function Form() {
  const [show, setShow] = useState(false);
  return (
    <>
      <button onClick={() => setShow(s => !s)}>{show ? '隐藏' : '展示'}计数器</button>
      <br />
      <hr />
      {show && <Counter />}
    </>
  );
}
```

```css
label {
  display: block;
  margin-top: 20px;
  margin-bottom: 20px;
}

body {
  min-height: 150px;
}
```

</Sandpack>

<Solution>

When [Strict Mode](/reference/react/StrictMode) is on (like in the sandboxes on this site), React remounts each component once in development. This causes the interval to be set up twice, and this is why each second the counter increments twice.
当开启 [严格模式](/reference/react/StrictMode) 时（例如在本站的示例沙盒（sandbox）中），React 在开发环境中会将每个组件重新挂载一次。这使得计数器组件被挂载了两次，于是定时器也被设置了两次，因此计数器会每秒增加两次。

However, React's behavior is not the *cause* of the bug: the bug already exists in the code. React's behavior makes the bug more noticeable. The real cause is that this Effect starts a process but doesn't provide a way to clean it up.
然而，React 的行为并不是导致这个 bug 的根本原因：代码中本就存在这个 bug。React 的行为只是让这个 bug 更加明显。真正的原因是这个 Effect 开启了一个进程但没有提供清理它的方式。

To fix this code, save the interval ID returned by `setInterval`, and implement a cleanup function with [`clearInterval`](https://developer.mozilla.org/en-US/docs/Web/API/clearInterval):
要修复这段代码，保存 `setInterval` 返回的 interval ID，并使用 [`clearInterval`](https://developer.mozilla.org/zh-CN/docs/Web/API/clearInterval) 实现一个清理函数：

<Sandpack>

```js src/Counter.js active
import { useState, useEffect } from 'react';

export default function Counter() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    function onTick() {
      setCount(c => c + 1);
    }

    const intervalId = setInterval(onTick, 1000);
    return () => clearInterval(intervalId);
  }, []);

  return <h1>{count}</h1>;
}
```

```js src/App.js hidden
import { useState } from 'react';
import Counter from './Counter.js';

export default function App() {
  const [show, setShow] = useState(false);
  return (
    <>
      <button onClick={() => setShow(s => !s)}>{show ? '隐藏' : '展示'}计数器</button>
      <br />
      <hr />
      {show && <Counter />}
    </>
  );
}
```

```css
label {
  display: block;
  margin-top: 20px;
  margin-bottom: 20px;
}

body {
  min-height: 150px;
}
```

</Sandpack>

In development, React will still remount your component once to verify that you've implemented cleanup well. So there will be a `setInterval` call, immediately followed by `clearInterval`, and `setInterval` again. In production, there will be only one `setInterval` call. The user-visible behavior in both cases is the same: the counter increments once per second.
在开发环境中，React 仍然会重新挂载一次你的组件，以确保你已经正确地实现了清理函数。因此，在调用 `setInterval` 后会紧接着调用 `clearInterval`，然后再调用 `setInterval`。在生产环境中，则只调用一次 `setInterval`。两种情况下用户可见的行为都是相同的：计数器每秒递增一次。

</Solution>

#### 解决在 Effect 中获取数据的问题 | Fix fetching inside an Effect {/*fix-fetching-inside-an-effect*/}

This component shows the biography for the selected person. It loads the biography by calling an asynchronous function `fetchBio(person)` on mount and whenever `person` changes. That asynchronous function returns a [Promise](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise) which eventually resolves to a string. When fetching is done, it calls `setBio` to display that string under the select box.
下面这个组件显示所选人物的传记。它在挂载时和每当 `person` 改变时，通过调用一个异步函数 `fetchBio(person)` 来加载传记。该异步函数返回一个 [Promise](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Promise)，最终解析为一个字符串。当请求结束时，它调用 `setBio` 以将该字符串显示在选择框下方。

<Sandpack>

```js src/App.js
import { useState, useEffect } from 'react';
import { fetchBio } from './api.js';

export default function Page() {
  const [person, setPerson] = useState('Alice');
  const [bio, setBio] = useState(null);

  useEffect(() => {
    setBio(null);
    fetchBio(person).then(result => {
      setBio(result);
    });
  }, [person]);

  return (
    <>
      <select value={person} onChange={e => {
        setPerson(e.target.value);
      }}>
        <option value="Alice">Alice</option>
        <option value="Bob">Bob</option>
        <option value="Taylor">Taylor</option>
      </select>
      <hr />
      <p><i>{bio ?? '加载中……'}</i></p>
    </>
  );
}
```

```js src/api.js hidden
export async function fetchBio(person) {
  const delay = person === 'Bob' ? 2000 : 200;
  return new Promise(resolve => {
    setTimeout(() => {
      resolve('这是' + person + '的传记。');
    }, delay);
  })
}

```

</Sandpack>


There is a bug in this code. Start by selecting "Alice". Then select "Bob" and then immediately after that select "Taylor". If you do this fast enough, you will notice that bug: Taylor is selected, but the paragraph below says "This is Bob's bio."
这段代码中有一个 bug。试试先选择 `Alice`，再选择 `Bob`，接着立即选择 `Taylor`。如果操作得足够快，你会观察到这个 bug：虽然 Taylor 被选中了，但下面的一段却说：“这是 Bob 的传记。”

Why does this happen? Fix the bug inside this Effect.
为什么会出现这种情况？试试修复这个 Effect 中的 bug。

<Hint>

If an Effect fetches something asynchronously, it usually needs cleanup.
如果 Effect 需要异步获取某些数据，它往往需要清理函数。

</Hint>

<Solution>

To trigger the bug, things need to happen in this order:
要触发这个 bug，事情需要按以下顺序发生：

- Selecting `'Bob'` triggers `fetchBio('Bob')`
- 选中 `'Bob'` 触发 `fetchBio('Bob')`
- Selecting `'Taylor'` triggers `fetchBio('Taylor')`
- 选中 `'Taylor'` 触发 `fetchBio('Taylor')`
- **Fetching `'Taylor'` completes *before* fetching `'Bob'`**
- **`fetchBio('Taylor')` 在 `fetchBio('Bob')` 之前完成。**
- The Effect from the `'Taylor'` render calls `setBio('This is Taylor’s bio')`
- 渲染 `'Taylor'` 时的 Effect 调用 `setBio('这是Taylor的传记')`
- Fetching `'Bob'` completes
- `fetchBio('Bob')` 请求完成
- The Effect from the `'Bob'` render calls `setBio('This is Bob’s bio')`
- 渲染 `'Bob'` 时的 Effect 调用 `setBio('这是Bob的传记')`

This is why you see Bob's bio even though Taylor is selected. Bugs like this are called [race conditions](https://en.wikipedia.org/wiki/Race_condition) because two asynchronous operations are "racing" with each other, and they might arrive in an unexpected order.
这就是为什么即使选择了 Taylor，但显示的仍然是 Bob 的传记。像这样的 bug 被称为 [竞态条件](https://en.wikipedia.org/wiki/Race_condition)，因为两个异步操作在“竞速”，并且可能会以意外的顺序完成。

To fix this race condition, add a cleanup function:
要修复这个 bug，添加一个清理函数：

<Sandpack>

```js src/App.js
import { useState, useEffect } from 'react';
import { fetchBio } from './api.js';

export default function Page() {
  const [person, setPerson] = useState('Alice');
  const [bio, setBio] = useState(null);
  useEffect(() => {
    let ignore = false;
    setBio(null);
    fetchBio(person).then(result => {
      if (!ignore) {
        setBio(result);
      }
    });
    return () => {
      ignore = true;
    }
  }, [person]);

  return (
    <>
      <select value={person} onChange={e => {
        setPerson(e.target.value);
      }}>
        <option value="Alice">Alice</option>
        <option value="Bob">Bob</option>
        <option value="Taylor">Taylor</option>
      </select>
      <hr />
      <p><i>{bio ?? '加载中……'}</i></p>
    </>
  );
}
```

```js src/api.js hidden
export async function fetchBio(person) {
  const delay = person === 'Bob' ? 2000 : 200;
  return new Promise(resolve => {
    setTimeout(() => {
      resolve('这是' + person + '的传记。');
    }, delay);
  })
}

```

</Sandpack>

Each render's Effect has its own `ignore` variable. Initially, the `ignore` variable is set to `false`. However, if an Effect gets cleaned up (such as when you select a different person), its `ignore` variable becomes `true`. So now it doesn't matter in which order the requests complete. Only the last person's Effect will have `ignore` set to `false`, so it will call `setBio(result)`. Past Effects have been cleaned up, so the `if (!ignore)` check will prevent them from calling `setBio`:
每轮渲染的 Effect 都有其独立的 `ignore` 变量。最初，`ignore` 变量被设置为 `false`。但如果一个 Effect 被清理（例如，当你选择不同的人时），它的 `ignore` 变量会变为 `true`。因此，请求完成的顺序已经不再重要。只有最后选中的人的 Effect 的 `ignore` 变量会是 `false`，因此它将会调用 `setBio(result)`。而之前的 Effect 已经被清理，所以 `if (!ignore)` 的检查会阻止它们调用 `setBio`：

- Selecting `'Bob'` triggers `fetchBio('Bob')`
- 选中 `'Bob'` 触发 `fetchBio('Bob')`
- Selecting `'Taylor'` triggers `fetchBio('Taylor')` **and cleans up the previous (Bob's) Effect**
- 选中 `'Taylor'` 触发 `fetchBio('Taylor')`，**并清理之前的（Bob 的） Effect**
- Fetching `'Taylor'` completes *before* fetching `'Bob'`
- `fetchBio('Taylor')` 在 `fetchBio('Bob')` **之前** 完成。
- The Effect from the `'Taylor'` render calls `setBio('This is Taylor’s bio')`
- 渲染 `'Taylor'` 时的 Effect 调用 `setBio('这是Taylor的传记')`
- Fetching `'Bob'` completes
- `fetchBio('Bob')` 请求完成
- The Effect from the `'Bob'` render **does not do anything because its `ignore` flag was set to `true`**
- 渲染 `'Bob'` 时的 Effect 不会做任何事情，因为它的 `ignore` 变量被设为了 `true`。

In addition to ignoring the result of an outdated API call, you can also use [`AbortController`](https://developer.mozilla.org/en-US/docs/Web/API/AbortController) to cancel the requests that are no longer needed. However, by itself this is not enough to protect against race conditions. More asynchronous steps could be chained after the fetch, so using an explicit flag like `ignore` is the most reliable way to fix this type of problem.
除了忽略过时 API 调用的结果外，你还可以使用 [`AbortController`](https://developer.mozilla.org/zh-CN/docs/Web/API/AbortController) 来取消不再需要的请求。然而，仅凭这一点还不足以防止竞态条件。因为可能在 fetch 之后还有更多的异步操作，因此使用像 `ignore` 这样的显式标志是解决这类问题最可靠的方法。

</Solution>

</Challenges>

