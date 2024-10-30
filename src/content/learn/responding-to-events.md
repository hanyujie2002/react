---
title: 响应事件 | Responding to Events
translators:
  - Jiacheng787
  - QC-L
  - Neo42
  - Zhou Chenyang
---

<Intro>

React lets you add *event handlers* to your JSX. Event handlers are your own functions that will be triggered in response to interactions like clicking, hovering, focusing form inputs, and so on.
使用 React 可以在 JSX 中添加 **事件处理函数**。其中事件处理函数为自定义函数，它将在响应交互（如点击、悬停、表单输入框获得焦点等）时触发。

</Intro>

<YouWillLearn>

* Different ways to write an event handler
* 编写事件处理函数的不同方法
* How to pass event handling logic from a parent component
* 如何从父组件传递事件处理逻辑
* How events propagate and how to stop them
* 事件如何传播以及如何停止它们

</YouWillLearn>

## 添加事件处理函数 | Adding event handlers {/*adding-event-handlers*/}

To add an event handler, you will first define a function and then [pass it as a prop](/learn/passing-props-to-a-component) to the appropriate JSX tag. For example, here is a button that doesn't do anything yet:
如需添加一个事件处理函数，你需要先定义一个函数，然后 [将其作为 prop 传入](/learn/passing-props-to-a-component) 合适的 JSX 标签。例如，这里有一个没绑定任何事件的按钮：

<Sandpack>

```js
export default function Button() {
  return (
    <button>
      未绑定任何事件
    </button>
  );
}
```

</Sandpack>

You can make it show a message when a user clicks by following these three steps:
按照如下三个步骤，即可让它在用户点击时显示消息：

1. Declare a function called `handleClick` *inside* your `Button` component.
- 在 `Button` 组件 **内部** 声明一个名为 `handleClick` 的函数。
2. Implement the logic inside that function (use `alert` to show the message).
- 实现函数内部的逻辑（使用 `alert` 来显示消息）。
3. Add `onClick={handleClick}` to the `<button>` JSX.
- 添加 `onClick={handleClick}` 到 `<button>` JSX 中。

<Sandpack>

```js
export default function Button() {
  function handleClick() {
    alert('你点击了我！');
  }

  return (
    <button onClick={handleClick}>
      点我
    </button>
  );
}
```

```css
button { margin-right: 10px; }
```

</Sandpack>

You defined the `handleClick` function and then [passed it as a prop](/learn/passing-props-to-a-component) to `<button>`.  `handleClick` is an **event handler.** Event handler functions:
你可以定义 `handleClick` 函数然后 [将其作为 prop 传入](/learn/passing-props-to-a-component) `<button>`。其中 `handleClick` 是一个 **事件处理函数** 。事件处理函数有如下特点:

* Are usually defined *inside* your components.
* 通常在你的组件 **内部** 定义。
* Have names that start with `handle`, followed by the name of the event.
* 名称以 `handle` 开头，后跟事件名称。

By convention, it is common to name event handlers as `handle` followed by the event name. You'll often see `onClick={handleClick}`, `onMouseEnter={handleMouseEnter}`, and so on.
按照惯例，通常将事件处理程序命名为 `handle`，后接事件名。你会经常看到 `onClick={handleClick}`，`onMouseEnter={handleMouseEnter}` 等。

Alternatively, you can define an event handler inline in the JSX:
或者，你也可以在 JSX 中定义一个内联的事件处理函数：

```jsx
<button onClick={function handleClick() {
  alert('你点击了我！');
}}>
```

Or, more concisely, using an arrow function:
或者，直接使用更为简洁箭头函数：

```jsx
<button onClick={() => {
  alert('你点击了我！');
}}>
```

All of these styles are equivalent. Inline event handlers are convenient for short functions.
以上所有方式都是等效的。当函数体较短时，内联事件处理函数会很方便。

<Pitfall>

Functions passed to event handlers must be passed, not called. For example:
传递给事件处理函数的函数应直接传递，而非调用。例如：

| 传递一个函数（正确）/ passing a function (correct)     | 调用一个函数（错误）/ calling a function (incorrect)    |
| -------------------------------- | ---------------------------------- |
| `<button onClick={handleClick}>` | `<button onClick={handleClick()}>` |

The difference is subtle. In the first example, the `handleClick` function is passed as an `onClick` event handler. This tells React to remember it and only call your function when the user clicks the button.
区别很微妙。在第一个示例中，`handleClick` 函数作为 `onClick` 事件处理函数传递。这会让 React 记住它，并且只在用户点击按钮时调用你的函数。

In the second example, the `()` at the end of `handleClick()` fires the function *immediately* during [rendering](/learn/render-and-commit), without any clicks. This is because JavaScript inside the [JSX `{` and `}`](/learn/javascript-in-jsx-with-curly-braces) executes right away.
在第二个示例中，`handleClick()` 中最后的 `()` 会在 [渲染](/learn/render-and-commit) 过程中 **立即** 触发函数，即使没有任何点击。这是因为位于 [JSX `{}`](/learn/javascript-in-jsx-with-curly-braces) 之间的 JavaScript 会立即执行。

When you write code inline, the same pitfall presents itself in a different way:
当你编写内联代码时，同样的陷阱可能会以不同的方式出现：

| 传递一个函数（正确）/ passing a function (correct)    | 调用一个函数（错误）/ calling a function (incorrect)     |
| --------------------------------------- | --------------------------------- |
| `<button onClick={() => alert('...')}>` | `<button onClick={alert('...')}>` |


Passing inline code like this won't fire on click—it fires every time the component renders:
如果按如下方式传递内联代码，并不会在点击时触发，而是会在每次组件渲染时触发：

```jsx
// This alert fires when the component renders, not when clicked!
// 这个 alert 在组件渲染时触发，而不是点击时触发！
<button onClick={alert('你点击了我！')}>
```

If you want to define your event handler inline, wrap it in an anonymous function like so:
如果你想要定义内联事件处理函数，请将其包装在匿名函数中，如下所示：

```jsx
<button onClick={() => alert('你点击了我！')}>
```

Rather than executing the code inside with every render, this creates a function to be called later.
这里创建了一个稍后调用的函数，而不会在每次渲染时执行其内部代码。

In both cases, what you want to pass is a function:
在这两种情况下，你都应该传递一个函数：

* `<button onClick={handleClick}>` passes the `handleClick` function.
* `<button onClick={handleClick}>` 传递了 `handleClick` 函数。
* `<button onClick={() => alert('...')}>` passes the `() => alert('...')` function.
* `<button onClick={() => alert('...')}>` 传递了 `() => alert('...')` 函数。

[Read more about arrow functions.](https://javascript.info/arrow-functions-basics)
[了解更多箭头函数的信息](https://zh.javascript.info/arrow-functions-basics)。

</Pitfall>

### 在事件处理函数中读取 props | Reading props in event handlers {/*reading-props-in-event-handlers*/}

Because event handlers are declared inside of a component, they have access to the component's props. Here is a button that, when clicked, shows an alert with its `message` prop:
由于事件处理函数声明于组件内部，因此它们可以直接访问组件的 props。示例中的按钮，当点击时会弹出带有 `message` prop 的 alert：

<Sandpack>

```js
function AlertButton({ message, children }) {
  return (
    <button onClick={() => alert(message)}>
      {children}
    </button>
  );
}

export default function Toolbar() {
  return (
    <div>
      <AlertButton message="正在播放！">
        播放电影
      </AlertButton>
      <AlertButton message="正在上传！">
        上传图片
      </AlertButton>
    </div>
  );
}
```

```css
button { margin-right: 10px; }
```

</Sandpack>

This lets these two buttons show different messages. Try changing the messages passed to them.
此处有两个按钮，会展示不同的消息。你可以尝试更改传递给它们的消息。

### 将事件处理函数作为 props 传递 | Passing event handlers as props {/*passing-event-handlers-as-props*/}

Often you'll want the parent component to specify a child's event handler. Consider buttons: depending on where you're using a `Button` component, you might want to execute a different function—perhaps one plays a movie and another uploads an image. 
通常，我们会在父组件中定义子组件的事件处理函数。比如：置于不同位置的 `Button` 组件，可能最终执行的功能也不同 —— 也许是播放电影，也许是上传图片。

To do this, pass a prop the component receives from its parent as the event handler like so:
为此，将组件从父组件接收的 prop 作为事件处理函数传递，如下所示：

<Sandpack>

```js
function Button({ onClick, children }) {
  return (
    <button onClick={onClick}>
      {children}
    </button>
  );
}

function PlayButton({ movieName }) {
  function handlePlayClick() {
    alert(`正在播放 ${movieName}！`);
  }

  return (
    <Button onClick={handlePlayClick}>
      播放 "{movieName}"
    </Button>
  );
}

function UploadButton() {
  return (
    <Button onClick={() => alert('正在上传！')}>
      上传图片
    </Button>
  );
}

export default function Toolbar() {
  return (
    <div>
      <PlayButton movieName="魔女宅急便" />
      <UploadButton />
    </div>
  );
}
```

```css
button { margin-right: 10px; }
```

</Sandpack>

Here, the `Toolbar` component renders a `PlayButton` and an `UploadButton`:
示例中，`Toolbar` 组件渲染了一个 `PlayButton` 组件和 `UploadButton` 组件：

- `PlayButton` passes `handlePlayClick` as the `onClick` prop to the `Button` inside.
- `PlayButton` 将 `handlePlayClick` 作为 `onClick` prop 传入 `Button` 组件内部。
- `UploadButton` passes `() => alert('Uploading!')` as the `onClick` prop to the `Button` inside.
- `UploadButton` 将 `() => alert('正在上传！')` 作为 `onClick` prop 传入 `Button` 组件内部。

Finally, your `Button` component accepts a prop called `onClick`. It passes that prop directly to the built-in browser `<button>` with `onClick={onClick}`. This tells React to call the passed function on click.
最后，你的 `Button` 组件接收一个名为 `onClick` 的 prop。它直接将这个 prop 以 `onClick={onClick}` 方式传递给浏览器内置的 `<button>`。当点击按钮时，React 会调用传入的函数。

If you use a [design system](https://uxdesign.cc/everything-you-need-to-know-about-design-systems-54b109851969), it's common for components like buttons to contain styling but not specify behavior. Instead, components like `PlayButton` and `UploadButton` will pass event handlers down.
如果你遵循某个 [设计系统](https://uxdesign.cc/everything-you-need-to-know-about-design-systems-54b109851969) 时，按钮之类的组件通常会包含样式，但不会指定行为。而 `PlayButton` 和 `UploadButton` 之类的组件则会向下传递事件处理函数。

### 命名事件处理函数 prop | Naming event handler props {/*naming-event-handler-props*/}

Built-in components like `<button>` and `<div>` only support [browser event names](/reference/react-dom/components/common#common-props) like `onClick`. However, when you're building your own components, you can name their event handler props any way that you like.
内置组件（`<button>` 和 `<div>`）仅支持 [浏览器事件名称](/reference/react-dom/components/common#common-props)，例如 `onClick`。但是，当你构建自己的组件时，你可以按你个人喜好命名事件处理函数的 prop。

By convention, event handler props should start with `on`, followed by a capital letter.
按照惯例，事件处理函数 props 应该以 `on` 开头，后跟一个大写字母。

For example, the `Button` component's `onClick` prop could have been called `onSmash`:
例如，`Button` 组件的 `onClick` prop 本来也可以被命名为 `onSmash`：

<Sandpack>

```js
function Button({ onSmash, children }) {
  return (
    <button onClick={onSmash}>
      {children}
    </button>
  );
}

export default function App() {
  return (
    <div>
      <Button onSmash={() => alert('正在播放！')}>
        播放电影
      </Button>
      <Button onSmash={() => alert('正在上传！')}>
        上传图片
      </Button>
    </div>
  );
}
```

```css
button { margin-right: 10px; }
```

</Sandpack>

In this example, `<button onClick={onSmash}>` shows that the browser `<button>` (lowercase) still needs a prop called `onClick`, but the prop name received by your custom `Button` component is up to you!
上述示例中，`<button onClick={onSmash}>` 代表浏览器内置的 `<button>`（小写）仍然需要使用 `onClick` prop，而自定义的 `Button` 组件接收到的 prop 名称可由你决定！

When your component supports multiple interactions, you might name event handler props for app-specific concepts. For example, this `Toolbar` component receives `onPlayMovie` and `onUploadImage` event handlers:
当你的组件支持多种交互时，你可以根据不同的应用程序命名事件处理函数 prop。例如，一个 `Toolbar` 组件接收 `onPlayMovie` 和 `onUploadImage` 两个事件处理函数：

<Sandpack>

```js
export default function App() {
  return (
    <Toolbar
      onPlayMovie={() => alert('正在播放！')}
      onUploadImage={() => alert('正在上传！')}
    />
  );
}

function Toolbar({ onPlayMovie, onUploadImage }) {
  return (
    <div>
      <Button onClick={onPlayMovie}>
        播放电影
      </Button>
      <Button onClick={onUploadImage}>
        上传图片
      </Button>
    </div>
  );
}

function Button({ onClick, children }) {
  return (
    <button onClick={onClick}>
      {children}
    </button>
  );
}
```

```css
button { margin-right: 10px; }
```

</Sandpack>

Notice how the `App` component does not need to know *what* `Toolbar` will do with `onPlayMovie` or `onUploadImage`. That's an implementation detail of the `Toolbar`. Here, `Toolbar` passes them down as `onClick` handlers to its `Button`s, but it could later also trigger them on a keyboard shortcut. Naming props after app-specific interactions like `onPlayMovie` gives you the flexibility to change how they're used later.
请注意，`App` 组件并不需要知道 `Toolbar` 将会对 `onPlayMovie` 和 `onUploadImage` 做 **什么** 。上述示例是 `Toolbar` 的实现细节。其中，`Toolbar` 将它们作为 `onClick` 处理函数传递给了 `Button` 组件，其实还可以通过键盘快捷键来触发它们。根据应用程序特定的交互方式（如 `onPlayMovie`）来命名 prop ，可以让你灵活地更改以后使用它们的方式。

<Note>

Make sure that you use the appropriate HTML tags for your event handlers. For example, to handle clicks, use [`<button onClick={handleClick}>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/button) instead of `<div onClick={handleClick}>`. Using a real browser `<button>` enables built-in browser behaviors like keyboard navigation. If you don't like the default browser styling of a button and want to make it look more like a link or a different UI element, you can achieve it with CSS. [Learn more about writing accessible markup.](https://developer.mozilla.org/en-US/docs/Learn/Accessibility/HTML)
确保为事件处理程序使用适当的 HTML 标签。例如，要处理点击事件，请使用 [`<button onClick={handleClick}>`](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Element/button) 而不是 `<div onClick={handleClick}>`。使用真正的浏览器 `<button>` 启用内置的浏览器行为，如键盘导航。如果你不喜欢按钮的默认浏览器样式，并且想让它看起来更像一个链接或不同的 UI 元素，你可以使用 CSS 来实现。[了解有关编写无障碍标签的更多信息](https://developer.mozilla.org/zh-CN/docs/Learn/Accessibility/HTML)。

</Note>

## 事件传播 | Event propagation {/*event-propagation*/}

Event handlers will also catch events from any children your component might have. We say that an event "bubbles" or "propagates" up the tree: it starts with where the event happened, and then goes up the tree.
事件处理函数还将捕获任何来自子组件的事件。通常，我们会说事件会沿着树向上“冒泡”或“传播”：它从事件发生的地方开始，然后沿着树向上传播。

This `<div>` contains two buttons. Both the `<div>` *and* each button have their own `onClick` handlers. Which handlers do you think will fire when you click a button?
下面这个 `<div>` 包含两个按钮。`<div>` 和每个按钮都有自己的 `onClick` 处理函数。你认为点击按钮时会触发哪些处理函数？

<Sandpack>

```js
export default function Toolbar() {
  return (
    <div className="Toolbar" onClick={() => {
      alert('你点击了 toolbar ！');
    }}>
      <button onClick={() => alert('正在播放！')}>
        播放电影
      </button>
      <button onClick={() => alert('正在上传！')}>
        上传图片
      </button>
    </div>
  );
}
```

```css
.Toolbar {
  background: #aaa;
  padding: 5px;
}
button { margin: 5px; }
```

</Sandpack>

If you click on either button, its `onClick` will run first, followed by the parent `<div>`'s `onClick`. So two messages will appear. If you click the toolbar itself, only the parent `<div>`'s `onClick` will run.
如果你点击任一按钮，它自身的 `onClick` 将首先执行，然后父级 `<div>` 的 `onClick` 会接着执行。因此会出现两条消息。如果你点击 toolbar 本身，将只有父级 `<div>` 的 `onClick` 会执行。

<Pitfall>

All events propagate in React except `onScroll`, which only works on the JSX tag you attach it to.
在 React 中所有事件都会传播，除了 `onScroll`，它仅适用于你附加到的 JSX 标签。

</Pitfall>

### 阻止传播 | Stopping propagation {/*stopping-propagation*/}

Event handlers receive an **event object** as their only argument. By convention, it's usually called `e`, which stands for "event". You can use this object to read information about the event.
事件处理函数接收一个 **事件对象** 作为唯一的参数。按照惯例，它通常被称为 `e` ，代表 "event"（事件）。你可以使用此对象来读取有关事件的信息。

That event object also lets you stop the propagation. If you want to prevent an event from reaching parent components, you need to call `e.stopPropagation()` like this `Button` component does:
这个事件对象还允许你阻止传播。如果你想阻止一个事件到达父组件，你需要像下面 `Button` 组件那样调用 `e.stopPropagation()` ：

<Sandpack>

```js
function Button({ onClick, children }) {
  return (
    <button onClick={e => {
      e.stopPropagation();
      onClick();
    }}>
      {children}
    </button>
  );
}

export default function Toolbar() {
  return (
    <div className="Toolbar" onClick={() => {
      alert('你点击了 toolbar ！');
    }}>
      <Button onClick={() => alert('正在播放！')}>
        播放电影
      </Button>
      <Button onClick={() => alert('正在上传！')}>
        上传图片
      </Button>
    </div>
  );
}
```

```css
.Toolbar {
  background: #aaa;
  padding: 5px;
}
button { margin: 5px; }
```

</Sandpack>

When you click on a button:
当你点击按钮时：

1. React calls the `onClick` handler passed to `<button>`. 
- React 调用了传递给 `<button>` 的 `onClick` 处理函数。
2. That handler, defined in `Button`, does the following:
- 定义在 `Button` 中的处理函数执行了如下操作：
   * Calls `e.stopPropagation()`, preventing the event from bubbling further.
   * 调用 `e.stopPropagation()`，阻止事件进一步冒泡。
   * Calls the `onClick` function, which is a prop passed from the `Toolbar` component.
   * 调用 `onClick` 函数，它是从 `Toolbar` 组件传递过来的 prop。
3. That function, defined in the `Toolbar` component, displays the button's own alert.
- 在 `Toolbar` 组件中定义的函数，显示按钮对应的 alert。
4. Since the propagation was stopped, the parent `<div>`'s `onClick` handler does *not* run.
- 由于传播被阻止，父级 `<div>` 的 `onClick` 处理函数不会执行。

As a result of `e.stopPropagation()`, clicking on the buttons now only shows a single alert (from the `<button>`) rather than the two of them (from the `<button>` and the parent toolbar `<div>`). Clicking a button is not the same thing as clicking the surrounding toolbar, so stopping the propagation makes sense for this UI.
由于调用了 `e.stopPropagation()`，点击按钮现在将只显示一个 alert（来自 `<button>`），而并非两个（分别来自 `<button>` 和父级 toolbar `<div>`）。点击按钮与点击周围的 toolbar 不同，因此阻止传播对这个 UI 是有意义的。

<DeepDive>

#### 捕获阶段事件 | Capture phase events {/*capture-phase-events*/}

In rare cases, you might need to catch all events on child elements, *even if they stopped propagation*. For example, maybe you want to log every click to analytics, regardless of the propagation logic. You can do this by adding `Capture` at the end of the event name:
极少数情况下，你可能需要捕获子元素上的所有事件，**即便它们阻止了传播**。例如，你可能想对每次点击进行埋点记录，传播逻辑暂且不论。那么你可以通过在事件名称末尾添加 `Capture` 来实现这一点：

```js
<div onClickCapture={() => { /* 这会首先执行 | this runs first */ }}>
  <button onClick={e => e.stopPropagation()} />
  <button onClick={e => e.stopPropagation()} />
</div>
```

Each event propagates in three phases: 
每个事件分三个阶段传播：

1. It travels down, calling all `onClickCapture` handlers.
- 它向下传播，调用所有的 `onClickCapture` 处理函数。
2. It runs the clicked element's `onClick` handler. 
- 它执行被点击元素的 `onClick` 处理函数。 
3. It travels upwards, calling all `onClick` handlers.
- 它向上传播，调用所有的 `onClick` 处理函数。

Capture events are useful for code like routers or analytics, but you probably won't use them in app code.
捕获事件对于路由或数据分析之类的代码很有用，但你可能不会在应用程序代码中使用它们。

</DeepDive>

### 传递处理函数作为事件传播的替代方案 | Passing handlers as alternative to propagation {/*passing-handlers-as-alternative-to-propagation*/}

Notice how this click handler runs a line of code _and then_ calls the `onClick` prop passed by the parent:
注意，此处的点击事件处理函数先执行了一行代码，**然后**调用了父组件传递的 `onClick` prop：

```js {4,5}
function Button({ onClick, children }) {
  return (
    <button onClick={e => {
      e.stopPropagation();
      onClick();
    }}>
      {children}
    </button>
  );
}
```

You could add more code to this handler before calling the parent `onClick` event handler, too. This pattern provides an *alternative* to propagation. It lets the child component handle the event, while also letting the parent component specify some additional behavior. Unlike propagation, it's not automatic. But the benefit of this pattern is that you can clearly follow the whole chain of code that executes as a result of some event.
你也可以在调用父元素 `onClick` 函数之前，向这个处理函数添加更多代码。此模式是事件传播的另一种 **替代方案** 。它让子组件处理事件，同时也让父组件指定一些额外的行为。与事件传播不同，它并非自动。但使用这种模式的好处是你可以清楚地追踪因某个事件的触发而执行的整条代码链。

If you rely on propagation and it's difficult to trace which handlers execute and why, try this approach instead.
如果你依赖于事件传播，而且很难追踪哪些处理程序在执行，及其执行的原因，可以尝试这种方法。

### 阻止默认行为 | Preventing default behavior {/*preventing-default-behavior*/}

Some browser events have default behavior associated with them. For example, a `<form>` submit event, which happens when a button inside of it is clicked, will reload the whole page by default:
某些浏览器事件具有与事件相关联的默认行为。例如，点击 `<form>` 表单内部的按钮会触发表单提交事件，默认情况下将重新加载整个页面：

<Sandpack>

```js
export default function Signup() {
  return (
    <form onSubmit={() => alert('提交表单！')}>
      <input />
      <button>发送</button>
    </form>
  );
}
```

```css
button { margin-left: 5px; }
```

</Sandpack>

You can call `e.preventDefault()` on the event object to stop this from happening:
你可以调用事件对象中的 `e.preventDefault()` 来阻止这种情况发生：

<Sandpack>

```js
export default function Signup() {
  return (
    <form onSubmit={e => {
      e.preventDefault();
      alert('提交表单！');
    }}>
      <input />
      <button>发送</button>
    </form>
  );
}
```

```css
button { margin-left: 5px; }
```

</Sandpack>

Don't confuse `e.stopPropagation()` and `e.preventDefault()`. They are both useful, but are unrelated:
不要混淆 `e.stopPropagation()` 和 `e.preventDefault()`。它们都很有用，但二者并不相关：

* [`e.stopPropagation()`](https://developer.mozilla.org/docs/Web/API/Event/stopPropagation) stops the event handlers attached to the tags above from firing.
* [`e.stopPropagation()`](https://developer.mozilla.org/docs/Web/API/Event/stopPropagation) 阻止触发绑定在外层标签上的事件处理函数。
* [`e.preventDefault()` ](https://developer.mozilla.org/docs/Web/API/Event/preventDefault) prevents the default browser behavior for the few events that have it.
* [`e.preventDefault()`](https://developer.mozilla.org/docs/Web/API/Event/preventDefault) 阻止少数事件的默认浏览器行为。

## 事件处理函数可以包含副作用吗？| Can event handlers have side effects? {/*can-event-handlers-have-side-effects*/}

Absolutely! Event handlers are the best place for side effects.
当然可以！事件处理函数是执行副作用的最佳位置。

Unlike rendering functions, event handlers don't need to be [pure](/learn/keeping-components-pure), so it's a great place to *change* something—for example, change an input's value in response to typing, or change a list in response to a button press. However, in order to change some information, you first need some way to store it. In React, this is done by using [state, a component's memory.](/learn/state-a-components-memory) You will learn all about it on the next page.
与渲染函数不同，事件处理函数不需要是 [纯函数](/learn/keeping-components-pure)，因此它是用来 *更改* 某些值的绝佳位置。例如，更改输入框的值以响应键入，或者更改列表以响应按钮的触发。但是，为了更改某些信息，你首先需要某种方式存储它。在 React 中，这是通过 [state（组件的记忆）](/learn/state-a-components-memory) 来完成的。你将在下一章节了解所有相关信息。

<Recap>

* You can handle events by passing a function as a prop to an element like `<button>`.
* 你可以通过将函数作为 prop 传递给元素如 `<button>` 来处理事件。
* Event handlers must be passed, **not called!** `onClick={handleClick}`, not `onClick={handleClick()}`.
* 必须传递事件处理函数，**而非函数调用！** `onClick={handleClick}` ，不是 `onClick={handleClick()}`。
* You can define an event handler function separately or inline.
* 你可以单独或者内联定义事件处理函数。
* Event handlers are defined inside a component, so they can access props.
* 事件处理函数在组件内部定义，所以它们可以访问 props。
* You can declare an event handler in a parent and pass it as a prop to a child.
* 你可以在父组件中定义一个事件处理函数，并将其作为 prop 传递给子组件。
* You can define your own event handler props with application-specific names.
* 你可以根据特定于应用程序的名称定义事件处理函数的 prop。
* Events propagate upwards. Call `e.stopPropagation()` on the first argument to prevent that.
* 事件会向上传播。通过事件的第一个参数调用 `e.stopPropagation()` 来防止这种情况。
* Events may have unwanted default browser behavior. Call `e.preventDefault()` to prevent that.
* 事件可能具有不需要的浏览器默认行为。调用 `e.preventDefault()` 来阻止这种情况。
* Explicitly calling an event handler prop from a child handler is a good alternative to propagation.
* 从子组件显式调用事件处理函数 prop 是事件传播的另一种优秀替代方案。

</Recap>



<Challenges>

#### 修复事件处理函数 | Fix an event handler {/*fix-an-event-handler*/}

Clicking this button is supposed to switch the page background between white and black. However, nothing happens when you click it. Fix the problem. (Don't worry about the logic inside `handleClick`—that part is fine.)
点击此按钮理论上应该在黑白主题之间切换页面背景。然而，当你点击它时，什么也没有发生。解决这个问题。（无需担心 `handleClick` 的内部逻辑。）

<Sandpack>

```js
export default function LightSwitch() {
  function handleClick() {
    let bodyStyle = document.body.style;
    if (bodyStyle.backgroundColor === 'black') {
      bodyStyle.backgroundColor = 'white';
    } else {
      bodyStyle.backgroundColor = 'black';
    }
  }

  return (
    <button onClick={handleClick()}>
      切换背景
    </button>
  );
}
```

</Sandpack>

<Solution>

The problem is that `<button onClick={handleClick()}>` _calls_ the `handleClick` function while rendering instead of _passing_ it. Removing the `()` call so that it's `<button onClick={handleClick}>` fixes the issue:
这是由于 `<button onClick={handleClick()}>` 在渲染过程中 _调用_ 了 `handleClick` 函数，而没有将其进行 _传递_。移除 `()` 调用改为 `<button onClick={handleClick}>` 进而修复问题：

<Sandpack>

```js
export default function LightSwitch() {
  function handleClick() {
    let bodyStyle = document.body.style;
    if (bodyStyle.backgroundColor === 'black') {
      bodyStyle.backgroundColor = 'white';
    } else {
      bodyStyle.backgroundColor = 'black';
    }
  }

  return (
    <button onClick={handleClick}>
      切换背景
    </button>
  );
}
```

</Sandpack>

Alternatively, you could wrap the call into another function, like `<button onClick={() => handleClick()}>`:
或者，你可以把函数调用包裹在另一个函数内，例如 `<button onClick={() => handleClick()}>` ：

<Sandpack>

```js
export default function LightSwitch() {
  function handleClick() {
    let bodyStyle = document.body.style;
    if (bodyStyle.backgroundColor === 'black') {
      bodyStyle.backgroundColor = 'white';
    } else {
      bodyStyle.backgroundColor = 'black';
    }
  }

  return (
    <button onClick={() => handleClick()}>
      切换背景
    </button>
  );
}
```

</Sandpack>

</Solution>

#### 关联事件 | Wire up the events {/*wire-up-the-events*/}

This `ColorSwitch` component renders a button. It's supposed to change the page color. Wire it up to the `onChangeColor` event handler prop it receives from the parent so that clicking the button changes the color.
`ColorSwitch` 组件渲染了一个按钮。它应该改变页面颜色。将它与从父组件接收的 `onChangeColor` 事件处理函数关联，以便在点击按钮时改变颜色。

After you do this, notice that clicking the button also increments the page click counter. Your colleague who wrote the parent component insists that `onChangeColor` does not increment any counters. What else might be happening? Fix it so that clicking the button *only* changes the color, and does _not_ increment the counter.
如此操作后，你会发现点击按钮时，也会增加页面点击计数器的值。而编写父组件的同事坚持认为 `onChangeColor` 不应该使得计数器的值递增。应该如何处理？修改问题使得点击按钮 **只** 改变颜色，并且 **不** 增加计数器。

<Sandpack>

```js src/ColorSwitch.js active
export default function ColorSwitch({
  onChangeColor
}) {
  return (
    <button>
      改变颜色
    </button>
  );
}
```

```js src/App.js hidden
import { useState } from 'react';
import ColorSwitch from './ColorSwitch.js';

export default function App() {
  const [clicks, setClicks] = useState(0);

  function handleClickOutside() {
    setClicks(c => c + 1);
  }

  function getRandomLightColor() {
    let r = 150 + Math.round(100 * Math.random());
    let g = 150 + Math.round(100 * Math.random());
    let b = 150 + Math.round(100 * Math.random());
    return `rgb(${r}, ${g}, ${b})`;
  }

  function handleChangeColor() {
    let bodyStyle = document.body.style;
    bodyStyle.backgroundColor = getRandomLightColor();
  }

  return (
    <div style={{ width: '100%', height: '100%' }} onClick={handleClickOutside}>
      <ColorSwitch onChangeColor={handleChangeColor} />
      <br />
      <br />
      <h2>页面点击次数：{clicks}</h2>
    </div>
  );
}
```

</Sandpack>

<Solution>

First, you need to add the event handler, like `<button onClick={onChangeColor}>`.
首先，你需要添加事件处理函数，例如 `<button onClick={onChangeColor}>`。

However, this introduces the problem of the incrementing counter. If `onChangeColor` does not do this, as your colleague insists, then the problem is that this event propagates up, and some handler above does it. To solve this problem, you need to stop the propagation. But don't forget that you should still call `onChangeColor`.
然而，同时又引入了计数器递增的问题。正如你的同事所坚持的那样，`onChangeColor` 并不符合预期，这主要是因为事件发生向上传播，并且上层的某些事件处理函数执行了递增操作。为了解决这个问题，你需要阻止事件冒泡。但是不要忘记调用 `onChangeColor`。

<Sandpack>

```js src/ColorSwitch.js active
export default function ColorSwitch({
  onChangeColor
}) {
  return (
    <button onClick={e => {
      e.stopPropagation();
      onChangeColor();
    }}>
      改变颜色
    </button>
  );
}
```

```js src/App.js hidden
import { useState } from 'react';
import ColorSwitch from './ColorSwitch.js';

export default function App() {
  const [clicks, setClicks] = useState(0);

  function handleClickOutside() {
    setClicks(c => c + 1);
  }

  function getRandomLightColor() {
    let r = 150 + Math.round(100 * Math.random());
    let g = 150 + Math.round(100 * Math.random());
    let b = 150 + Math.round(100 * Math.random());
    return `rgb(${r}, ${g}, ${b})`;
  }

  function handleChangeColor() {
    let bodyStyle = document.body.style;
    bodyStyle.backgroundColor = getRandomLightColor();
  }

  return (
    <div style={{ width: '100%', height: '100%' }} onClick={handleClickOutside}>
      <ColorSwitch onChangeColor={handleChangeColor} />
      <br />
      <br />
      <h2>页面点击次数：{clicks}</h2>
    </div>
  );
}
```

</Sandpack>

</Solution>

</Challenges>
