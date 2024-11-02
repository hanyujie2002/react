---
title: 选择 State 结构 | Choosing the State Structure
translators:
  - Davont
---

<Intro>

Structuring state well can make a difference between a component that is pleasant to modify and debug, and one that is a constant source of bugs. Here are some tips you should consider when structuring state.
构建良好的 state 可以让组件变得易于修改和调试，而不会经常出错。以下是你在构建 state 时应该考虑的一些建议。

</Intro>

<YouWillLearn>

* When to use a single vs multiple state variables
* 使用单个 state 变量还是多个 state 变量
* What to avoid when organizing state
* 组织 state 时应避免的内容
* How to fix common issues with the state structure
* 如何解决 state 结构中的常见问题

</YouWillLearn>

## 构建 state 的原则 | Principles for structuring state {/*principles-for-structuring-state*/}

When you write a component that holds some state, you'll have to make choices about how many state variables to use and what the shape of their data should be. While it's possible to write correct programs even with a suboptimal state structure, there are a few principles that can guide you to make better choices:
当你编写一个存有 state 的组件时，你需要选择使用多少个 state 变量以及它们都是怎样的数据格式。尽管选择次优的 state 结构下也可以编写正确的程序，但有几个原则可以指导你做出更好的决策：

1. **Group related state.** If you always update two or more state variables at the same time, consider merging them into a single state variable.
- **合并关联的 state**。如果你总是同时更新两个或更多的 state 变量，请考虑将它们合并为一个单独的 state 变量。
2. **Avoid contradictions in state.** When the state is structured in a way that several pieces of state may contradict and "disagree" with each other, you leave room for mistakes. Try to avoid this.
- **避免互相矛盾的 state**。当 state 结构中存在多个相互矛盾或“不一致”的 state 时，你就可能为此会留下隐患。应尽量避免这种情况。
3. **Avoid redundant state.** If you can calculate some information from the component's props or its existing state variables during rendering, you should not put that information into that component's state.
- **避免冗余的 state**。如果你能在渲染期间从组件的 props 或其现有的 state 变量中计算出一些信息，则不应将这些信息放入该组件的 state 中。
4. **Avoid duplication in state.** When the same data is duplicated between multiple state variables, or within nested objects, it is difficult to keep them in sync. Reduce duplication when you can.
- **避免重复的 state**。当同一数据在多个 state 变量之间或在多个嵌套对象中重复时，这会很难保持它们同步。应尽可能减少重复。
5. **Avoid deeply nested state.** Deeply hierarchical state is not very convenient to update. When possible, prefer to structure state in a flat way.
- **避免深度嵌套的 state**。深度分层的 state 更新起来不是很方便。如果可能的话，最好以扁平化方式构建 state。

The goal behind these principles is to *make state easy to update without introducing mistakes*. Removing redundant and duplicate data from state helps ensure that all its pieces stay in sync. This is similar to how a database engineer might want to ["normalize" the database structure](https://docs.microsoft.com/en-us/office/troubleshoot/access/database-normalization-description) to reduce the chance of bugs. To paraphrase Albert Einstein, **"Make your state as simple as it can be--but no simpler."**
这些原则背后的目标是 **使 state 易于更新而不引入错误**。从 state 中删除冗余和重复数据有助于确保所有部分保持同步。这类似于数据库工程师想要 [“规范化”数据库结构](https://docs.microsoft.com/zh-CN/office/troubleshoot/access/database-normalization-description)，以减少出现错误的机会。用爱因斯坦的话说，**“让你的状态尽可能简单，但不要过于简单。”**

现在让我们来看看这些原则在实际中是如何应用的。

## 合并关联的 state | Group related state {/*group-related-state*/}

You might sometimes be unsure between using a single or multiple state variables.
有时候你可能会不确定是使用单个 state 变量还是多个 state 变量。

Should you do this?
你会像下面这样做吗？

```js
const [x, setX] = useState(0);
const [y, setY] = useState(0);
```

或这样？

```js
const [position, setPosition] = useState({ x: 0, y: 0 });
```

Technically, you can use either of these approaches. But **if some two state variables always change together, it might be a good idea to unify them into a single state variable.** Then you won't forget to always keep them in sync, like in this example where moving the cursor updates both coordinates of the red dot:
从技术上讲，你可以使用其中任何一种方法。但是，**如果某两个 state 变量总是一起变化，则将它们统一成一个 state 变量可能更好**。这样你就不会忘记让它们始终保持同步，就像下面这个例子中，移动光标会同时更新红点的两个坐标：

<Sandpack>

```js
import { useState } from 'react';

export default function MovingDot() {
  const [position, setPosition] = useState({
    x: 0,
    y: 0
  });
  return (
    <div
      onPointerMove={e => {
        setPosition({
          x: e.clientX,
          y: e.clientY
        });
      }}
      style={{
        position: 'relative',
        width: '100vw',
        height: '100vh',
      }}>
      <div style={{
        position: 'absolute',
        backgroundColor: 'red',
        borderRadius: '50%',
        transform: `translate(${position.x}px, ${position.y}px)`,
        left: -10,
        top: -10,
        width: 20,
        height: 20,
      }} />
    </div>
  )
}
```

```css
body { margin: 0; padding: 0; height: 250px; }
```

</Sandpack>

Another case where you'll group data into an object or an array is when you don't know how many pieces of state you'll need. For example, it's helpful when you have a form where the user can add custom fields.
另一种你需要将数据整合到一个对象或一个数组的情况是，你不知道未来需要多少个 state 片段。例如，有一个用户可以添加自定义字段的表单时，这将会很有帮助。

<Pitfall>

If your state variable is an object, remember that [you can't update only one field in it](/learn/updating-objects-in-state) without explicitly copying the other fields. For example, you can't do `setPosition({ x: 100 })` in the above example because it would not have the `y` property at all! Instead, if you wanted to set `x` alone, you would either do `setPosition({ ...position, x: 100 })`, or split them into two state variables and do `setX(100)`.
如果你的 state 变量是一个对象时，请记住，[你不能只更新其中的一个字段](/learn/updating-objects-in-state) 而不显式复制其他字段。例如，在上面的例子中，你不能写成 `setPosition({ x: 100 })`，因为它根本就没有 `y` 属性! 相反，如果你想要仅设置 `x`，则可执行 `setPosition({ ...position, x: 100 })`，或将它们分成两个 state 变量，并执行 `setX(100)`。

</Pitfall>

## 避免矛盾的 state | Avoid contradictions in state {/*avoid-contradictions-in-state*/}

Here is a hotel feedback form with `isSending` and `isSent` state variables:
下面是带有 `isSending` 和 `isSent` 两个 state 变量的酒店反馈表单：

<Sandpack>

```js
import { useState } from 'react';

export default function FeedbackForm() {
  const [text, setText] = useState('');
  const [isSending, setIsSending] = useState(false);
  const [isSent, setIsSent] = useState(false);

  async function handleSubmit(e) {
    e.preventDefault();
    setIsSending(true);
    await sendMessage(text);
    setIsSending(false);
    setIsSent(true);
  }

  if (isSent) {
    return <h1>Thanks for feedback!</h1>
  }

  return (
    <form onSubmit={handleSubmit}>
      <p>How was your stay at The Prancing Pony?</p>
      <textarea
        disabled={isSending}
        value={text}
        onChange={e => setText(e.target.value)}
      />
      <br />
      <button
        disabled={isSending}
        type="submit"
      >
        Send
      </button>
      {isSending && <p>Sending...</p>}
    </form>
  );
}

// Pretend to send a message.
// 假装发送一条消息。
function sendMessage(text) {
  return new Promise(resolve => {
    setTimeout(resolve, 2000);
  });
}
```

</Sandpack>

While this code works, it leaves the door open for "impossible" states. For example, if you forget to call `setIsSent` and `setIsSending` together, you may end up in a situation where both `isSending` and `isSent` are `true` at the same time. The more complex your component is, the harder it is to understand what happened.
尽管这段代码是有效的，但也会让一些 state “极难处理”。例如，如果你忘记同时调用 `setIsSent` 和 `setIsSending`，则可能会出现 `isSending` 和 `isSent` 同时为 `true` 的情况。你的组件越复杂，你就越难理解发生了什么。

**Since `isSending` and `isSent` should never be `true` at the same time, it is better to replace them with one `status` state variable that may take one of *three* valid states:** `'typing'` (initial), `'sending'`, and `'sent'`:
**因为 `isSending` 和 `isSent` 不应同时为 `true`，所以最好用一个 `status` 变量来代替它们，这个 state 变量可以采取三种有效状态其中之一**：`'typing'` (初始), `'sending'`, 和 `'sent'`:

<Sandpack>

```js
import { useState } from 'react';

export default function FeedbackForm() {
  const [text, setText] = useState('');
  const [status, setStatus] = useState('typing');

  async function handleSubmit(e) {
    e.preventDefault();
    setStatus('sending');
    await sendMessage(text);
    setStatus('sent');
  }

  const isSending = status === 'sending';
  const isSent = status === 'sent';

  if (isSent) {
    return <h1>Thanks for feedback!</h1>
  }

  return (
    <form onSubmit={handleSubmit}>
      <p>How was your stay at The Prancing Pony?</p>
      <textarea
        disabled={isSending}
        value={text}
        onChange={e => setText(e.target.value)}
      />
      <br />
      <button
        disabled={isSending}
        type="submit"
      >
        Send
      </button>
      {isSending && <p>Sending...</p>}
    </form>
  );
}

// Pretend to send a message.
// 假装发送一条消息。
function sendMessage(text) {
  return new Promise(resolve => {
    setTimeout(resolve, 2000);
  });
}
```

</Sandpack>

You can still declare some constants for readability:
你仍然可以声明一些常量，以提高可读性：

```js
const isSending = status === 'sending';
const isSent = status === 'sent';
```

But they're not state variables, so you don't need to worry about them getting out of sync with each other.
但它们不是 state 变量，所以你不必担心它们彼此失去同步。

## 避免冗余的 state | Avoid redundant state {/*avoid-redundant-state*/}

If you can calculate some information from the component's props or its existing state variables during rendering, you **should not** put that information into that component's state.
如果你能在渲染期间从组件的 props 或其现有的 state 变量中计算出一些信息，则不应该把这些信息放到该组件的 state 中。

For example, take this form. It works, but can you find any redundant state in it?
例如，以这个表单为例。它可以运行，但你能找到其中任何冗余的 state 吗？

<Sandpack>

```js
import { useState } from 'react';

export default function Form() {
  const [firstName, setFirstName] = useState('');
  const [lastName, setLastName] = useState('');
  const [fullName, setFullName] = useState('');

  function handleFirstNameChange(e) {
    setFirstName(e.target.value);
    setFullName(e.target.value + ' ' + lastName);
  }

  function handleLastNameChange(e) {
    setLastName(e.target.value);
    setFullName(firstName + ' ' + e.target.value);
  }

  return (
    <>
      <h2>Let’s check you in</h2>
      <label>
        First name:{' '}
        <input
          value={firstName}
          onChange={handleFirstNameChange}
        />
      </label>
      <label>
        Last name:{' '}
        <input
          value={lastName}
          onChange={handleLastNameChange}
        />
      </label>
      <p>
        Your ticket will be issued to: <b>{fullName}</b>
      </p>
    </>
  );
}
```

```css
label { display: block; margin-bottom: 5px; }
```

</Sandpack>

This form has three state variables: `firstName`, `lastName`, and `fullName`. However, `fullName` is redundant. **You can always calculate `fullName` from `firstName` and `lastName` during render, so remove it from state.**
这个表单有三个 state 变量：`firstName`、`lastName` 和 `fullName`。然而，`fullName` 是多余的。**在渲染期间，你始终可以从 `firstName` 和 `lastName` 中计算出 `fullName`，因此需要把它从 state 中删除。**

This is how you can do it:
你可以这样做：

<Sandpack>

```js
import { useState } from 'react';

export default function Form() {
  const [firstName, setFirstName] = useState('');
  const [lastName, setLastName] = useState('');

  const fullName = firstName + ' ' + lastName;

  function handleFirstNameChange(e) {
    setFirstName(e.target.value);
  }

  function handleLastNameChange(e) {
    setLastName(e.target.value);
  }

  return (
    <>
      <h2>Let’s check you in</h2>
      <label>
        First name:{' '}
        <input
          value={firstName}
          onChange={handleFirstNameChange}
        />
      </label>
      <label>
        Last name:{' '}
        <input
          value={lastName}
          onChange={handleLastNameChange}
        />
      </label>
      <p>
        Your ticket will be issued to: <b>{fullName}</b>
      </p>
    </>
  );
}
```

```css
label { display: block; margin-bottom: 5px; }
```

</Sandpack>

Here, `fullName` is *not* a state variable. Instead, it's calculated during render:
这里的 `fullName` **不是** 一个 state 变量。相反，它是在渲染期间中计算出的：

```js
const fullName = firstName + ' ' + lastName;
```

As a result, the change handlers don't need to do anything special to update it. When you call `setFirstName` or `setLastName`, you trigger a re-render, and then the next `fullName` will be calculated from the fresh data.
因此，更改处理程序不需要做任何特殊操作来更新它。当你调用 `setFirstName` 或 `setLastName` 时，你会触发一次重新渲染，然后下一个 `fullName` 将从新数据中计算出来。

<DeepDive>

#### 不要在 state 中镜像 props | Don't mirror props in state {/*don-t-mirror-props-in-state*/}

A common example of redundant state is code like this:
以下代码是体现 state 冗余的一个常见例子：

```js
function Message({ messageColor }) {
  const [color, setColor] = useState(messageColor);
```

Here, a `color` state variable is initialized to the `messageColor` prop. The problem is that **if the parent component passes a different value of `messageColor` later (for example, `'red'` instead of `'blue'`), the `color` *state variable* would not be updated!** The state is only initialized during the first render.
这里，一个 `color` state 变量被初始化为 `messageColor` 的 prop 值。这段代码的问题在于，**如果父组件稍后传递不同的 `messageColor` 值（例如，将其从 `'blue'` 更改为 `'red'`），则 `color`** state 变量**将不会更新！** state 仅在第一次渲染期间初始化。

This is why "mirroring" some prop in a state variable can lead to confusion. Instead, use the `messageColor` prop directly in your code. If you want to give it a shorter name, use a constant:
这就是为什么在 state 变量中，“镜像”一些 prop 属性会导致混淆的原因。相反，你要在代码中直接使用 `messageColor` 属性。如果你想给它起一个更短的名称，请使用常量：

```js
function Message({ messageColor }) {
  const color = messageColor;
```

This way it won't get out of sync with the prop passed from the parent component.
这种写法就不会与从父组件传递的属性失去同步。

"Mirroring" props into state only makes sense when you *want* to ignore all updates for a specific prop. By convention, start the prop name with `initial` or `default` to clarify that its new values are ignored:
只有当你 **想要** 忽略特定 props 属性的所有更新时，将 props “镜像”到 state 才有意义。按照惯例，prop 名称以 `initial` 或 `default` 开头，以阐明该 prop 的新值将被忽略：

```js
function Message({ initialColor }) {
  // The `color` state variable holds the *first* value of `initialColor`.
  // 这个 `color` state 变量用于保存 `initialColor` 的 **初始值**。
  // Further changes to the `initialColor` prop are ignored.
  // 对于 `initialColor` 属性的进一步更改将被忽略。
  const [color, setColor] = useState(initialColor);
```

</DeepDive>

## 避免重复的 state | Avoid duplication in state {/*avoid-duplication-in-state*/}

This menu list component lets you choose a single travel snack out of several:
下面这个菜单列表组件可以让你在多种旅行小吃中选择一个：

<Sandpack>

```js
import { useState } from 'react';

const initialItems = [
  { title: 'pretzels', id: 0 },
  { title: 'crispy seaweed', id: 1 },
  { title: 'granola bar', id: 2 },
];

export default function Menu() {
  const [items, setItems] = useState(initialItems);
  const [selectedItem, setSelectedItem] = useState(
    items[0]
  );

  return (
    <>
      <h2>What's your travel snack?</h2>
      <ul>
        {items.map(item => (
          <li key={item.id}>
            {item.title}
            {' '}
            <button onClick={() => {
              setSelectedItem(item);
            }}>Choose</button>
          </li>
        ))}
      </ul>
      <p>You picked {selectedItem.title}.</p>
    </>
  );
}
```

```css
button { margin-top: 10px; }
```

</Sandpack>

Currently, it stores the selected item as an object in the `selectedItem` state variable. However, this is not great: **the contents of the `selectedItem` is the same object as one of the items inside the `items` list.** This means that the information about the item itself is duplicated in two places.
当前，它将所选元素作为对象存储在 `selectedItem` state 变量中。然而，这并不好：**`selectedItem` 的内容与 `items` 列表中的某个项是同一个对象。** 这意味着关于该项本身的信息在两个地方产生了重复。

Why is this a problem? Let's make each item editable:
为什么这是个问题？让我们使每个项目都可以编辑：

<Sandpack>

```js
import { useState } from 'react';

const initialItems = [
  { title: 'pretzels', id: 0 },
  { title: 'crispy seaweed', id: 1 },
  { title: 'granola bar', id: 2 },
];

export default function Menu() {
  const [items, setItems] = useState(initialItems);
  const [selectedItem, setSelectedItem] = useState(
    items[0]
  );

  function handleItemChange(id, e) {
    setItems(items.map(item => {
      if (item.id === id) {
        return {
          ...item,
          title: e.target.value,
        };
      } else {
        return item;
      }
    }));
  }

  return (
    <>
      <h2>What's your travel snack?</h2> 
      <ul>
        {items.map((item, index) => (
          <li key={item.id}>
            <input
              value={item.title}
              onChange={e => {
                handleItemChange(item.id, e)
              }}
            />
            {' '}
            <button onClick={() => {
              setSelectedItem(item);
            }}>Choose</button>
          </li>
        ))}
      </ul>
      <p>You picked {selectedItem.title}.</p>
    </>
  );
}
```

```css
button { margin-top: 10px; }
```

</Sandpack>

Notice how if you first click "Choose" on an item and *then* edit it, **the input updates but the label at the bottom does not reflect the edits.** This is because you have duplicated state, and you forgot to update `selectedItem`.
请注意，如果你首先单击菜单上的“Choose” **然后** 编辑它，**输入会更新，但底部的标签不会反映编辑内容。** 这是因为你有重复的 state，并且你忘记更新了 `selectedItem`。

Although you could update `selectedItem` too, an easier fix is to remove duplication. In this example, instead of a `selectedItem` object (which creates a duplication with objects inside `items`), you hold the `selectedId` in state, and *then* get the `selectedItem` by searching the `items` array for an item with that ID:
尽管你也可以更新 `selectedItem`，但更简单的解决方法是消除重复项。在下面这个例子中，你将 `selectedId` 保存在 state 中，而不是在 `selectedItem` 对象中（它创建了一个与 `items` 内重复的对象），**然后** 通过搜索 `items` 数组中具有该 ID 的项，以此获取 `selectedItem`：

<Sandpack>

```js
import { useState } from 'react';

const initialItems = [
  { title: 'pretzels', id: 0 },
  { title: 'crispy seaweed', id: 1 },
  { title: 'granola bar', id: 2 },
];

export default function Menu() {
  const [items, setItems] = useState(initialItems);
  const [selectedId, setSelectedId] = useState(0);

  const selectedItem = items.find(item =>
    item.id === selectedId
  );

  function handleItemChange(id, e) {
    setItems(items.map(item => {
      if (item.id === id) {
        return {
          ...item,
          title: e.target.value,
        };
      } else {
        return item;
      }
    }));
  }

  return (
    <>
      <h2>What's your travel snack?</h2>
      <ul>
        {items.map((item, index) => (
          <li key={item.id}>
            <input
              value={item.title}
              onChange={e => {
                handleItemChange(item.id, e)
              }}
            />
            {' '}
            <button onClick={() => {
              setSelectedId(item.id);
            }}>Choose</button>
          </li>
        ))}
      </ul>
      <p>You picked {selectedItem.title}.</p>
    </>
  );
}
```

```css
button { margin-top: 10px; }
```

</Sandpack>

The state used to be duplicated like this:
state 过去常常是这样复制的：

* `items = [{ id: 0, title: 'pretzels'}, ...]`
* `selectedItem = {id: 0, title: 'pretzels'}`

But after the change it's like this:
改了之后是这样的：

* `items = [{ id: 0, title: 'pretzels'}, ...]`
* `selectedId = 0`

The duplication is gone, and you only keep the essential state!
重复的 state 没有了，你只保留了必要的 state！

Now if you edit the *selected* item, the message below will update immediately. This is because `setItems` triggers a re-render, and `items.find(...)` would find the item with the updated title. You didn't need to hold *the selected item* in state, because only the *selected ID* is essential. The rest could be calculated during render.
现在，如果你编辑 **selected** 元素，下面的消息将立即更新。这是因为 `setItems` 会触发重新渲染，而 `items.find(...)` 会找到带有更新文本的元素。你不需要在 state 中保存 **选定的元素**，因为只有 **选定的 ID** 是必要的。其余的可以在渲染期间计算。

## 避免深度嵌套的 state | Avoid deeply nested state {/*avoid-deeply-nested-state*/}

Imagine a travel plan consisting of planets, continents, and countries. You might be tempted to structure its state using nested objects and arrays, like in this example:
想象一下，一个由行星、大陆和国家组成的旅行计划。你可能会尝试使用嵌套对象和数组来构建它的 state，就像下面这个例子：

<Sandpack>

```js
import { useState } from 'react';
import { initialTravelPlan } from './places.js';

function PlaceTree({ place }) {
  const childPlaces = place.childPlaces;
  return (
    <li>
      {place.title}
      {childPlaces.length > 0 && (
        <ol>
          {childPlaces.map(place => (
            <PlaceTree key={place.id} place={place} />
          ))}
        </ol>
      )}
    </li>
  );
}

export default function TravelPlan() {
  const [plan, setPlan] = useState(initialTravelPlan);
  const planets = plan.childPlaces;
  return (
    <>
      <h2>Places to visit</h2>
      <ol>
        {planets.map(place => (
          <PlaceTree key={place.id} place={place} />
        ))}
      </ol>
    </>
  );
}
```

```js src/places.js active
export const initialTravelPlan = {
  id: 0,
  title: '(Root)',
  childPlaces: [{
    id: 1,
    title: 'Earth',
    childPlaces: [{
      id: 2,
      title: 'Africa',
      childPlaces: [{
        id: 3,
        title: 'Botswana',
        childPlaces: []
      }, {
        id: 4,
        title: 'Egypt',
        childPlaces: []
      }, {
        id: 5,
        title: 'Kenya',
        childPlaces: []
      }, {
        id: 6,
        title: 'Madagascar',
        childPlaces: []
      }, {
        id: 7,
        title: 'Morocco',
        childPlaces: []
      }, {
        id: 8,
        title: 'Nigeria',
        childPlaces: []
      }, {
        id: 9,
        title: 'South Africa',
        childPlaces: []
      }]
    }, {
      id: 10,
      title: 'Americas',
      childPlaces: [{
        id: 11,
        title: 'Argentina',
        childPlaces: []
      }, {
        id: 12,
        title: 'Brazil',
        childPlaces: []
      }, {
        id: 13,
        title: 'Barbados',
        childPlaces: []
      }, {
        id: 14,
        title: 'Canada',
        childPlaces: []
      }, {
        id: 15,
        title: 'Jamaica',
        childPlaces: []
      }, {
        id: 16,
        title: 'Mexico',
        childPlaces: []
      }, {
        id: 17,
        title: 'Trinidad and Tobago',
        childPlaces: []
      }, {
        id: 18,
        title: 'Venezuela',
        childPlaces: []
      }]
    }, {
      id: 19,
      title: 'Asia',
      childPlaces: [{
        id: 20,
        title: 'China',
        childPlaces: []
      }, {
        id: 21,
        title: 'India',
        childPlaces: []
      }, {
        id: 22,
        title: 'Singapore',
        childPlaces: []
      }, {
        id: 23,
        title: 'South Korea',
        childPlaces: []
      }, {
        id: 24,
        title: 'Thailand',
        childPlaces: []
      }, {
        id: 25,
        title: 'Vietnam',
        childPlaces: []
      }]
    }, {
      id: 26,
      title: 'Europe',
      childPlaces: [{
        id: 27,
        title: 'Croatia',
        childPlaces: [],
      }, {
        id: 28,
        title: 'France',
        childPlaces: [],
      }, {
        id: 29,
        title: 'Germany',
        childPlaces: [],
      }, {
        id: 30,
        title: 'Italy',
        childPlaces: [],
      }, {
        id: 31,
        title: 'Portugal',
        childPlaces: [],
      }, {
        id: 32,
        title: 'Spain',
        childPlaces: [],
      }, {
        id: 33,
        title: 'Turkey',
        childPlaces: [],
      }]
    }, {
      id: 34,
      title: 'Oceania',
      childPlaces: [{
        id: 35,
        title: 'Australia',
        childPlaces: [],
      }, {
        id: 36,
        title: 'Bora Bora (French Polynesia)',
        childPlaces: [],
      }, {
        id: 37,
        title: 'Easter Island (Chile)',
        childPlaces: [],
      }, {
        id: 38,
        title: 'Fiji',
        childPlaces: [],
      }, {
        id: 39,
        title: 'Hawaii (the USA)',
        childPlaces: [],
      }, {
        id: 40,
        title: 'New Zealand',
        childPlaces: [],
      }, {
        id: 41,
        title: 'Vanuatu',
        childPlaces: [],
      }]
    }]
  }, {
    id: 42,
    title: 'Moon',
    childPlaces: [{
      id: 43,
      title: 'Rheita',
      childPlaces: []
    }, {
      id: 44,
      title: 'Piccolomini',
      childPlaces: []
    }, {
      id: 45,
      title: 'Tycho',
      childPlaces: []
    }]
  }, {
    id: 46,
    title: 'Mars',
    childPlaces: [{
      id: 47,
      title: 'Corn Town',
      childPlaces: []
    }, {
      id: 48,
      title: 'Green Hill',
      childPlaces: []      
    }]
  }]
};
```

</Sandpack>

Now let's say you want to add a button to delete a place you've already visited. How would you go about it? [Updating nested state](/learn/updating-objects-in-state#updating-a-nested-object) involves making copies of objects all the way up from the part that changed. Deleting a deeply nested place would involve copying its entire parent place chain. Such code can be very verbose.
现在，假设你想添加一个按钮来删除一个你已经去过的地方。你会怎么做呢？[更新嵌套的 state](/learn/updating-objects-in-state#updating-a-nested-object) 需要从更改部分一直向上复制对象。删除一个深度嵌套的地点将涉及复制其整个父级地点链。这样的代码可能非常冗长。

**If the state is too nested to update easily, consider making it "flat".** Here is one way you can restructure this data. Instead of a tree-like structure where each `place` has an array of *its child places*, you can have each place hold an array of *its child place IDs*. Then store a mapping from each place ID to the corresponding place.
**如果 state 嵌套太深，难以轻松更新，可以考虑将其“扁平化”。** 这里有一个方法可以重构上面这个数据。不同于树状结构，每个节点的 `place` 都是一个包含 **其子节点** 的数组，你可以让每个节点的 `place` 作为数组保存 **其子节点的 ID**。然后存储一个节点 ID 与相应节点的映射关系。

This data restructuring might remind you of seeing a database table:
这个数据重组可能会让你想起看到一个数据库表：

<Sandpack>

```js
import { useState } from 'react';
import { initialTravelPlan } from './places.js';

function PlaceTree({ id, placesById }) {
  const place = placesById[id];
  const childIds = place.childIds;
  return (
    <li>
      {place.title}
      {childIds.length > 0 && (
        <ol>
          {childIds.map(childId => (
            <PlaceTree
              key={childId}
              id={childId}
              placesById={placesById}
            />
          ))}
        </ol>
      )}
    </li>
  );
}

export default function TravelPlan() {
  const [plan, setPlan] = useState(initialTravelPlan);
  const root = plan[0];
  const planetIds = root.childIds;
  return (
    <>
      <h2>Places to visit</h2>
      <ol>
        {planetIds.map(id => (
          <PlaceTree
            key={id}
            id={id}
            placesById={plan}
          />
        ))}
      </ol>
    </>
  );
}
```

```js src/places.js active
export const initialTravelPlan = {
  0: {
    id: 0,
    title: '(Root)',
    childIds: [1, 42, 46],
  },
  1: {
    id: 1,
    title: 'Earth',
    childIds: [2, 10, 19, 26, 34]
  },
  2: {
    id: 2,
    title: 'Africa',
    childIds: [3, 4, 5, 6 , 7, 8, 9]
  }, 
  3: {
    id: 3,
    title: 'Botswana',
    childIds: []
  },
  4: {
    id: 4,
    title: 'Egypt',
    childIds: []
  },
  5: {
    id: 5,
    title: 'Kenya',
    childIds: []
  },
  6: {
    id: 6,
    title: 'Madagascar',
    childIds: []
  }, 
  7: {
    id: 7,
    title: 'Morocco',
    childIds: []
  },
  8: {
    id: 8,
    title: 'Nigeria',
    childIds: []
  },
  9: {
    id: 9,
    title: 'South Africa',
    childIds: []
  },
  10: {
    id: 10,
    title: 'Americas',
    childIds: [11, 12, 13, 14, 15, 16, 17, 18],   
  },
  11: {
    id: 11,
    title: 'Argentina',
    childIds: []
  },
  12: {
    id: 12,
    title: 'Brazil',
    childIds: []
  },
  13: {
    id: 13,
    title: 'Barbados',
    childIds: []
  }, 
  14: {
    id: 14,
    title: 'Canada',
    childIds: []
  },
  15: {
    id: 15,
    title: 'Jamaica',
    childIds: []
  },
  16: {
    id: 16,
    title: 'Mexico',
    childIds: []
  },
  17: {
    id: 17,
    title: 'Trinidad and Tobago',
    childIds: []
  },
  18: {
    id: 18,
    title: 'Venezuela',
    childIds: []
  },
  19: {
    id: 19,
    title: 'Asia',
    childIds: [20, 21, 22, 23, 24, 25],   
  },
  20: {
    id: 20,
    title: 'China',
    childIds: []
  },
  21: {
    id: 21,
    title: 'India',
    childIds: []
  },
  22: {
    id: 22,
    title: 'Singapore',
    childIds: []
  },
  23: {
    id: 23,
    title: 'South Korea',
    childIds: []
  },
  24: {
    id: 24,
    title: 'Thailand',
    childIds: []
  },
  25: {
    id: 25,
    title: 'Vietnam',
    childIds: []
  },
  26: {
    id: 26,
    title: 'Europe',
    childIds: [27, 28, 29, 30, 31, 32, 33],   
  },
  27: {
    id: 27,
    title: 'Croatia',
    childIds: []
  },
  28: {
    id: 28,
    title: 'France',
    childIds: []
  },
  29: {
    id: 29,
    title: 'Germany',
    childIds: []
  },
  30: {
    id: 30,
    title: 'Italy',
    childIds: []
  },
  31: {
    id: 31,
    title: 'Portugal',
    childIds: []
  },
  32: {
    id: 32,
    title: 'Spain',
    childIds: []
  },
  33: {
    id: 33,
    title: 'Turkey',
    childIds: []
  },
  34: {
    id: 34,
    title: 'Oceania',
    childIds: [35, 36, 37, 38, 39, 40, 41],   
  },
  35: {
    id: 35,
    title: 'Australia',
    childIds: []
  },
  36: {
    id: 36,
    title: 'Bora Bora (French Polynesia)',
    childIds: []
  },
  37: {
    id: 37,
    title: 'Easter Island (Chile)',
    childIds: []
  },
  38: {
    id: 38,
    title: 'Fiji',
    childIds: []
  },
  39: {
    id: 40,
    title: 'Hawaii (the USA)',
    childIds: []
  },
  40: {
    id: 40,
    title: 'New Zealand',
    childIds: []
  },
  41: {
    id: 41,
    title: 'Vanuatu',
    childIds: []
  },
  42: {
    id: 42,
    title: 'Moon',
    childIds: [43, 44, 45]
  },
  43: {
    id: 43,
    title: 'Rheita',
    childIds: []
  },
  44: {
    id: 44,
    title: 'Piccolomini',
    childIds: []
  },
  45: {
    id: 45,
    title: 'Tycho',
    childIds: []
  },
  46: {
    id: 46,
    title: 'Mars',
    childIds: [47, 48]
  },
  47: {
    id: 47,
    title: 'Corn Town',
    childIds: []
  },
  48: {
    id: 48,
    title: 'Green Hill',
    childIds: []
  }
};
```

</Sandpack>

**Now that the state is "flat" (also known as "normalized"), updating nested items becomes easier.**
**现在 state 已经“扁平化”（也称为“规范化”），更新嵌套项会变得更加容易。**

现在要删除一个地点，您只需要更新两个 state 级别：

- The updated version of its *parent* place should exclude the removed ID from its `childIds` array.
- 其 **父级** 地点的更新版本应该从其 `childIds` 数组中排除已删除的 ID。
- The updated version of the root "table" object should include the updated version of the parent place.
- 其根级“表”对象的更新版本应包括父级地点的更新版本。

Here is an example of how you could go about it:
下面是展示如何处理它的一个示例：

<Sandpack>

```js
import { useState } from 'react';
import { initialTravelPlan } from './places.js';

export default function TravelPlan() {
  const [plan, setPlan] = useState(initialTravelPlan);

  function handleComplete(parentId, childId) {
    const parent = plan[parentId];
    // Create a new version of the parent place
    // that doesn't include this child ID.
    // 创建一个其父级地点的新版本
    // 但不包括子级 ID。
    const nextParent = {
      ...parent,
      childIds: parent.childIds
        .filter(id => id !== childId)
    };
    // Update the root state object...
    // 更新根 state 对象...
    setPlan({
      ...plan,
      // ...so that it has the updated parent.
      // ...以便它拥有更新的父级。
      [parentId]: nextParent
    });
  }

  const root = plan[0];
  const planetIds = root.childIds;
  return (
    <>
      <h2>Places to visit</h2>
      <ol>
        {planetIds.map(id => (
          <PlaceTree
            key={id}
            id={id}
            parentId={0}
            placesById={plan}
            onComplete={handleComplete}
          />
        ))}
      </ol>
    </>
  );
}

function PlaceTree({ id, parentId, placesById, onComplete }) {
  const place = placesById[id];
  const childIds = place.childIds;
  return (
    <li>
      {place.title}
      <button onClick={() => {
        onComplete(parentId, id);
      }}>
        Complete
      </button>
      {childIds.length > 0 &&
        <ol>
          {childIds.map(childId => (
            <PlaceTree
              key={childId}
              id={childId}
              parentId={id}
              placesById={placesById}
              onComplete={onComplete}
            />
          ))}
        </ol>
      }
    </li>
  );
}
```

```js src/places.js
export const initialTravelPlan = {
  0: {
    id: 0,
    title: '(Root)',
    childIds: [1, 42, 46],
  },
  1: {
    id: 1,
    title: 'Earth',
    childIds: [2, 10, 19, 26, 34]
  },
  2: {
    id: 2,
    title: 'Africa',
    childIds: [3, 4, 5, 6 , 7, 8, 9]
  }, 
  3: {
    id: 3,
    title: 'Botswana',
    childIds: []
  },
  4: {
    id: 4,
    title: 'Egypt',
    childIds: []
  },
  5: {
    id: 5,
    title: 'Kenya',
    childIds: []
  },
  6: {
    id: 6,
    title: 'Madagascar',
    childIds: []
  }, 
  7: {
    id: 7,
    title: 'Morocco',
    childIds: []
  },
  8: {
    id: 8,
    title: 'Nigeria',
    childIds: []
  },
  9: {
    id: 9,
    title: 'South Africa',
    childIds: []
  },
  10: {
    id: 10,
    title: 'Americas',
    childIds: [11, 12, 13, 14, 15, 16, 17, 18],   
  },
  11: {
    id: 11,
    title: 'Argentina',
    childIds: []
  },
  12: {
    id: 12,
    title: 'Brazil',
    childIds: []
  },
  13: {
    id: 13,
    title: 'Barbados',
    childIds: []
  }, 
  14: {
    id: 14,
    title: 'Canada',
    childIds: []
  },
  15: {
    id: 15,
    title: 'Jamaica',
    childIds: []
  },
  16: {
    id: 16,
    title: 'Mexico',
    childIds: []
  },
  17: {
    id: 17,
    title: 'Trinidad and Tobago',
    childIds: []
  },
  18: {
    id: 18,
    title: 'Venezuela',
    childIds: []
  },
  19: {
    id: 19,
    title: 'Asia',
    childIds: [20, 21, 22, 23, 24, 25],   
  },
  20: {
    id: 20,
    title: 'China',
    childIds: []
  },
  21: {
    id: 21,
    title: 'India',
    childIds: []
  },
  22: {
    id: 22,
    title: 'Singapore',
    childIds: []
  },
  23: {
    id: 23,
    title: 'South Korea',
    childIds: []
  },
  24: {
    id: 24,
    title: 'Thailand',
    childIds: []
  },
  25: {
    id: 25,
    title: 'Vietnam',
    childIds: []
  },
  26: {
    id: 26,
    title: 'Europe',
    childIds: [27, 28, 29, 30, 31, 32, 33],   
  },
  27: {
    id: 27,
    title: 'Croatia',
    childIds: []
  },
  28: {
    id: 28,
    title: 'France',
    childIds: []
  },
  29: {
    id: 29,
    title: 'Germany',
    childIds: []
  },
  30: {
    id: 30,
    title: 'Italy',
    childIds: []
  },
  31: {
    id: 31,
    title: 'Portugal',
    childIds: []
  },
  32: {
    id: 32,
    title: 'Spain',
    childIds: []
  },
  33: {
    id: 33,
    title: 'Turkey',
    childIds: []
  },
  34: {
    id: 34,
    title: 'Oceania',
    childIds: [35, 36, 37, 38, 39, 40, 41],   
  },
  35: {
    id: 35,
    title: 'Australia',
    childIds: []
  },
  36: {
    id: 36,
    title: 'Bora Bora (French Polynesia)',
    childIds: []
  },
  37: {
    id: 37,
    title: 'Easter Island (Chile)',
    childIds: []
  },
  38: {
    id: 38,
    title: 'Fiji',
    childIds: []
  },
  39: {
    id: 39,
    title: 'Hawaii (the USA)',
    childIds: []
  },
  40: {
    id: 40,
    title: 'New Zealand',
    childIds: []
  },
  41: {
    id: 41,
    title: 'Vanuatu',
    childIds: []
  },
  42: {
    id: 42,
    title: 'Moon',
    childIds: [43, 44, 45]
  },
  43: {
    id: 43,
    title: 'Rheita',
    childIds: []
  },
  44: {
    id: 44,
    title: 'Piccolomini',
    childIds: []
  },
  45: {
    id: 45,
    title: 'Tycho',
    childIds: []
  },
  46: {
    id: 46,
    title: 'Mars',
    childIds: [47, 48]
  },
  47: {
    id: 47,
    title: 'Corn Town',
    childIds: []
  },
  48: {
    id: 48,
    title: 'Green Hill',
    childIds: []
  }
};
```

```css
button { margin: 10px; }
```

</Sandpack>

You can nest state as much as you like, but making it "flat" can solve numerous problems. It makes state easier to update, and it helps ensure you don't have duplication in different parts of a nested object.
你确实可以随心所欲地嵌套 state，但是将其“扁平化”可以解决许多问题。这使得 state 更容易更新，并且有助于确保在嵌套对象的不同部分中没有重复。

<DeepDive>

#### 改善内存使用 | Improving memory usage {/*improving-memory-usage*/}

理想情况下，您还应该从“表”对象中删除已删除的项目（以及它们的子项！）以改善内存使用。还可以 [使用 Immer](/learn/updating-objects-in-state#write-concise-update-logic-with-immer) 使更新逻辑更加简洁。

<Sandpack>

```js
import { useImmer } from 'use-immer';
import { initialTravelPlan } from './places.js';

export default function TravelPlan() {
  const [plan, updatePlan] = useImmer(initialTravelPlan);

  function handleComplete(parentId, childId) {
    updatePlan(draft => {
      // 从父级地点的子 ID 中移除。
      const parent = draft[parentId];
      parent.childIds = parent.childIds
        .filter(id => id !== childId);

      // 删除这个地点和它的所有子目录。
      deleteAllChildren(childId);
      function deleteAllChildren(id) {
        const place = draft[id];
        place.childIds.forEach(deleteAllChildren);
        delete draft[id];
      }
    });
  }

  const root = plan[0];
  const planetIds = root.childIds;
  return (
    <>
      <h2>Places to visit</h2>
      <ol>
        {planetIds.map(id => (
          <PlaceTree
            key={id}
            id={id}
            parentId={0}
            placesById={plan}
            onComplete={handleComplete}
          />
        ))}
      </ol>
    </>
  );
}

function PlaceTree({ id, parentId, placesById, onComplete }) {
  const place = placesById[id];
  const childIds = place.childIds;
  return (
    <li>
      {place.title}
      <button onClick={() => {
        onComplete(parentId, id);
      }}>
        Complete
      </button>
      {childIds.length > 0 &&
        <ol>
          {childIds.map(childId => (
            <PlaceTree
              key={childId}
              id={childId}
              parentId={id}
              placesById={placesById}
              onComplete={onComplete}
            />
          ))}
        </ol>
      }
    </li>
  );
}
```

```js src/places.js
export const initialTravelPlan = {
  0: {
    id: 0,
    title: '(Root)',
    childIds: [1, 42, 46],
  },
  1: {
    id: 1,
    title: 'Earth',
    childIds: [2, 10, 19, 26, 34]
  },
  2: {
    id: 2,
    title: 'Africa',
    childIds: [3, 4, 5, 6 , 7, 8, 9]
  }, 
  3: {
    id: 3,
    title: 'Botswana',
    childIds: []
  },
  4: {
    id: 4,
    title: 'Egypt',
    childIds: []
  },
  5: {
    id: 5,
    title: 'Kenya',
    childIds: []
  },
  6: {
    id: 6,
    title: 'Madagascar',
    childIds: []
  }, 
  7: {
    id: 7,
    title: 'Morocco',
    childIds: []
  },
  8: {
    id: 8,
    title: 'Nigeria',
    childIds: []
  },
  9: {
    id: 9,
    title: 'South Africa',
    childIds: []
  },
  10: {
    id: 10,
    title: 'Americas',
    childIds: [11, 12, 13, 14, 15, 16, 17, 18],   
  },
  11: {
    id: 11,
    title: 'Argentina',
    childIds: []
  },
  12: {
    id: 12,
    title: 'Brazil',
    childIds: []
  },
  13: {
    id: 13,
    title: 'Barbados',
    childIds: []
  }, 
  14: {
    id: 14,
    title: 'Canada',
    childIds: []
  },
  15: {
    id: 15,
    title: 'Jamaica',
    childIds: []
  },
  16: {
    id: 16,
    title: 'Mexico',
    childIds: []
  },
  17: {
    id: 17,
    title: 'Trinidad and Tobago',
    childIds: []
  },
  18: {
    id: 18,
    title: 'Venezuela',
    childIds: []
  },
  19: {
    id: 19,
    title: 'Asia',
    childIds: [20, 21, 22, 23, 24, 25,],   
  },
  20: {
    id: 20,
    title: 'China',
    childIds: []
  },
  21: {
    id: 21,
    title: 'India',
    childIds: []
  },
  22: {
    id: 22,
    title: 'Singapore',
    childIds: []
  },
  23: {
    id: 23,
    title: 'South Korea',
    childIds: []
  },
  24: {
    id: 24,
    title: 'Thailand',
    childIds: []
  },
  25: {
    id: 25,
    title: 'Vietnam',
    childIds: []
  },
  26: {
    id: 26,
    title: 'Europe',
    childIds: [27, 28, 29, 30, 31, 32, 33],   
  },
  27: {
    id: 27,
    title: 'Croatia',
    childIds: []
  },
  28: {
    id: 28,
    title: 'France',
    childIds: []
  },
  29: {
    id: 29,
    title: 'Germany',
    childIds: []
  },
  30: {
    id: 30,
    title: 'Italy',
    childIds: []
  },
  31: {
    id: 31,
    title: 'Portugal',
    childIds: []
  },
  32: {
    id: 32,
    title: 'Spain',
    childIds: []
  },
  33: {
    id: 33,
    title: 'Turkey',
    childIds: []
  },
  34: {
    id: 34,
    title: 'Oceania',
    childIds: [35, 36, 37, 38, 39, 40,, 41],   
  },
  35: {
    id: 35,
    title: 'Australia',
    childIds: []
  },
  36: {
    id: 36,
    title: 'Bora Bora (French Polynesia)',
    childIds: []
  },
  37: {
    id: 37,
    title: 'Easter Island (Chile)',
    childIds: []
  },
  38: {
    id: 38,
    title: 'Fiji',
    childIds: []
  },
  39: {
    id: 39,
    title: 'Hawaii (the USA)',
    childIds: []
  },
  40: {
    id: 40,
    title: 'New Zealand',
    childIds: []
  },
  41: {
    id: 41,
    title: 'Vanuatu',
    childIds: []
  },
  42: {
    id: 42,
    title: 'Moon',
    childIds: [43, 44, 45]
  },
  43: {
    id: 43,
    title: 'Rheita',
    childIds: []
  },
  44: {
    id: 44,
    title: 'Piccolomini',
    childIds: []
  },
  45: {
    id: 45,
    title: 'Tycho',
    childIds: []
  },
  46: {
    id: 46,
    title: 'Mars',
    childIds: [47, 48]
  },
  47: {
    id: 47,
    title: 'Corn Town',
    childIds: []
  },
  48: {
    id: 48,
    title: 'Green Hill',
    childIds: []
  }
};
```

```css
button { margin: 10px; }
```

```json package.json
{
  "dependencies": {
    "immer": "1.7.3",
    "react": "latest",
    "react-dom": "latest",
    "react-scripts": "latest",
    "use-immer": "0.5.1"
  },
  "scripts": {
    "start": "react-scripts start",
    "build": "react-scripts build",
    "test": "react-scripts test --env=jsdom",
    "eject": "react-scripts eject"
  }
}
```

</Sandpack>

</DeepDive>

Sometimes, you can also reduce state nesting by moving some of the nested state into the child components. This works well for ephemeral UI state that doesn't need to be stored, like whether an item is hovered.
有时候，你也可以通过将一些嵌套 state 移动到子组件中来减少 state 的嵌套。这对于不需要保存的短暂 UI 状态非常有效，比如一个选项是否被悬停。

<Recap>

* If two state variables always update together, consider merging them into one. 
* 如果两个 state 变量总是一起更新，请考虑将它们合并为一个。
* Choose your state variables carefully to avoid creating "impossible" states.
* 仔细选择你的 state 变量，以避免创建“极难处理”的 state。
* Structure your state in a way that reduces the chances that you'll make a mistake updating it.
* 用一种减少出错更新的机会的方式来构建你的 state。
* 避免冗余和重复的 state，这样您就不需要保持同步。
* 除非您特别想防止更新，否则不要将 props **放入** state 中。 
* 对于选择类型的 UI 模式，请在 state 中保存 ID 或索引而不是对象本身。
* If updating deeply nested state is complicated, try flattening it.
* 如果深度嵌套 state 更新很复杂，请尝试将其展开扁平化。

</Recap>

<Challenges>

#### 修复一个未更新的组件 | Fix a component that's not updating {/*fix-a-component-thats-not-updating*/}

这个 `Clock` 组件接收两个属性：`color` 和 `time`。当您在选择框中选择不同的颜色时，`Clock` 组件将从其父组件接收到一个不同的 `color` 属性。然而，由于某种原因，显示的颜色没有更新。为什么？请修复这个问题。

<Sandpack>

```js src/Clock.js active
import { useState } from 'react';

export default function Clock(props) {
  const [color, setColor] = useState(props.color);
  return (
    <h1 style={{ color: color }}>
      {props.time}
    </h1>
  );
}
```

```js src/App.js hidden
import { useState, useEffect } from 'react';
import Clock from './Clock.js';

function useTime() {
  const [time, setTime] = useState(() => new Date());
  useEffect(() => {
    const id = setInterval(() => {
      setTime(new Date());
    }, 1000);
    return () => clearInterval(id);
  }, []);
  return time;
}

export default function App() {
  const time = useTime();
  const [color, setColor] = useState('lightcoral');
  return (
    <div>
      <p>
        Pick a color:{' '}
        <select value={color} onChange={e => setColor(e.target.value)}>
          <option value="lightcoral">lightcoral</option>
          <option value="midnightblue">midnightblue</option>
          <option value="rebeccapurple">rebeccapurple</option>
        </select>
      </p>
      <Clock color={color} time={time.toLocaleTimeString()} />
    </div>
  );
}
```

</Sandpack>

<Solution>

The issue is that this component has `color` state initialized with the initial value of the `color` prop. But when the `color` prop changes, this does not affect the state variable! So they get out of sync. To fix this issue, remove the state variable altogether, and use the `color` prop directly.
这个问题点在于此组件的 `color` state 是使用 `color` prop 的初始值进行初始化的。但是当 `color` prop 值发生更改时，这不会影响 state 变量！因此它们会失去同步。为了解决这个问题，完全删除 state 变量，并直接使用 `color` prop 即可。

<Sandpack>

```js src/Clock.js active
import { useState } from 'react';

export default function Clock(props) {
  return (
    <h1 style={{ color: props.color }}>
      {props.time}
    </h1>
  );
}
```

```js src/App.js hidden
import { useState, useEffect } from 'react';
import Clock from './Clock.js';

function useTime() {
  const [time, setTime] = useState(() => new Date());
  useEffect(() => {
    const id = setInterval(() => {
      setTime(new Date());
    }, 1000);
    return () => clearInterval(id);
  }, []);
  return time;
}

export default function App() {
  const time = useTime();
  const [color, setColor] = useState('lightcoral');
  return (
    <div>
      <p>
        Pick a color:{' '}
        <select value={color} onChange={e => setColor(e.target.value)}>
          <option value="lightcoral">lightcoral</option>
          <option value="midnightblue">midnightblue</option>
          <option value="rebeccapurple">rebeccapurple</option>
        </select>
      </p>
      <Clock color={color} time={time.toLocaleTimeString()} />
    </div>
  );
}
```

</Sandpack>

Or, using the destructuring syntax:
或者，使用解构语法：

<Sandpack>

```js src/Clock.js active
import { useState } from 'react';

export default function Clock({ color, time }) {
  return (
    <h1 style={{ color: color }}>
      {time}
    </h1>
  );
}
```

```js src/App.js hidden
import { useState, useEffect } from 'react';
import Clock from './Clock.js';

function useTime() {
  const [time, setTime] = useState(() => new Date());
  useEffect(() => {
    const id = setInterval(() => {
      setTime(new Date());
    }, 1000);
    return () => clearInterval(id);
  }, []);
  return time;
}

export default function App() {
  const time = useTime();
  const [color, setColor] = useState('lightcoral');
  return (
    <div>
      <p>
        Pick a color:{' '}
        <select value={color} onChange={e => setColor(e.target.value)}>
          <option value="lightcoral">lightcoral</option>
          <option value="midnightblue">midnightblue</option>
          <option value="rebeccapurple">rebeccapurple</option>
        </select>
      </p>
      <Clock color={color} time={time.toLocaleTimeString()} />
    </div>
  );
}
```

</Sandpack>

</Solution>

#### 修复一个损坏的打包清单 | Fix a broken packing list {/*fix-a-broken-packing-list*/}

This packing list has a footer that shows how many items are packed, and how many items there are overall. It seems to work at first, but it is buggy. For example, if you mark an item as packed and then delete it, the counter will not be updated correctly. Fix the counter so that it's always correct.
这个打包清单有一个页脚，显示了打包的物品数量和总共的物品数量。一开始看起来似乎很好用，但是它也存在漏洞。例如，如果你将一个物品标记为已打包然后删除它，计数器就不会正确更新。请修复计数器以使其始终正确。

<Hint>

Is any state in this example redundant?
在这个例子中，是否有 state 是多余的？

</Hint>

<Sandpack>

```js src/App.js
import { useState } from 'react';
import AddItem from './AddItem.js';
import PackingList from './PackingList.js';

let nextId = 3;
const initialItems = [
  { id: 0, title: 'Warm socks', packed: true },
  { id: 1, title: 'Travel journal', packed: false },
  { id: 2, title: 'Watercolors', packed: false },
];

export default function TravelPlan() {
  const [items, setItems] = useState(initialItems);
  const [total, setTotal] = useState(3);
  const [packed, setPacked] = useState(1);

  function handleAddItem(title) {
    setTotal(total + 1);
    setItems([
      ...items,
      {
        id: nextId++,
        title: title,
        packed: false
      }
    ]);
  }

  function handleChangeItem(nextItem) {
    if (nextItem.packed) {
      setPacked(packed + 1);
    } else {
      setPacked(packed - 1);
    }
    setItems(items.map(item => {
      if (item.id === nextItem.id) {
        return nextItem;
      } else {
        return item;
      }
    }));
  }

  function handleDeleteItem(itemId) {
    setTotal(total - 1);
    setItems(
      items.filter(item => item.id !== itemId)
    );
  }

  return (
    <>  
      <AddItem
        onAddItem={handleAddItem}
      />
      <PackingList
        items={items}
        onChangeItem={handleChangeItem}
        onDeleteItem={handleDeleteItem}
      />
      <hr />
      <b>{packed} out of {total} packed!</b>
    </>
  );
}
```

```js src/AddItem.js hidden
import { useState } from 'react';

export default function AddItem({ onAddItem }) {
  const [title, setTitle] = useState('');
  return (
    <>
      <input
        placeholder="Add item"
        value={title}
        onChange={e => setTitle(e.target.value)}
      />
      <button onClick={() => {
        setTitle('');
        onAddItem(title);
      }}>Add</button>
    </>
  )
}
```

```js src/PackingList.js hidden
import { useState } from 'react';

export default function PackingList({
  items,
  onChangeItem,
  onDeleteItem
}) {
  return (
    <ul>
      {items.map(item => (
        <li key={item.id}>
          <label>
            <input
              type="checkbox"
              checked={item.packed}
              onChange={e => {
                onChangeItem({
                  ...item,
                  packed: e.target.checked
                });
              }}
            />
            {' '}
            {item.title}
          </label>
          <button onClick={() => onDeleteItem(item.id)}>
            Delete
          </button>
        </li>
      ))}
    </ul>
  );
}
```

```css
button { margin: 5px; }
li { list-style-type: none; }
ul, li { margin: 0; padding: 0; }
```

</Sandpack>

<Solution>

Although you could carefully change each event handler to update the `total` and `packed` counters correctly, the root problem is that these state variables exist at all. They are redundant because you can always calculate the number of items (packed or total) from the `items` array itself. Remove the redundant state to fix the bug:
虽然你可以仔细更改每个事件处理程序来正确更新 `total` 和 `packed` 计数器，但根本问题在于这些 state 变量一直存在。它们是冗余的，因为你始终可以从 `item` 数组本身计算出物品（已打包或总共）的数量。因此需要删除冗余 state 以修复错误：

<Sandpack>

```js src/App.js
import { useState } from 'react';
import AddItem from './AddItem.js';
import PackingList from './PackingList.js';

let nextId = 3;
const initialItems = [
  { id: 0, title: 'Warm socks', packed: true },
  { id: 1, title: 'Travel journal', packed: false },
  { id: 2, title: 'Watercolors', packed: false },
];

export default function TravelPlan() {
  const [items, setItems] = useState(initialItems);

  const total = items.length;
  const packed = items
    .filter(item => item.packed)
    .length;

  function handleAddItem(title) {
    setItems([
      ...items,
      {
        id: nextId++,
        title: title,
        packed: false
      }
    ]);
  }

  function handleChangeItem(nextItem) {
    setItems(items.map(item => {
      if (item.id === nextItem.id) {
        return nextItem;
      } else {
        return item;
      }
    }));
  }

  function handleDeleteItem(itemId) {
    setItems(
      items.filter(item => item.id !== itemId)
    );
  }

  return (
    <>  
      <AddItem
        onAddItem={handleAddItem}
      />
      <PackingList
        items={items}
        onChangeItem={handleChangeItem}
        onDeleteItem={handleDeleteItem}
      />
      <hr />
      <b>{packed} out of {total} packed!</b>
    </>
  );
}
```

```js src/AddItem.js hidden
import { useState } from 'react';

export default function AddItem({ onAddItem }) {
  const [title, setTitle] = useState('');
  return (
    <>
      <input
        placeholder="Add item"
        value={title}
        onChange={e => setTitle(e.target.value)}
      />
      <button onClick={() => {
        setTitle('');
        onAddItem(title);
      }}>Add</button>
    </>
  )
}
```

```js src/PackingList.js hidden
import { useState } from 'react';

export default function PackingList({
  items,
  onChangeItem,
  onDeleteItem
}) {
  return (
    <ul>
      {items.map(item => (
        <li key={item.id}>
          <label>
            <input
              type="checkbox"
              checked={item.packed}
              onChange={e => {
                onChangeItem({
                  ...item,
                  packed: e.target.checked
                });
              }}
            />
            {' '}
            {item.title}
          </label>
          <button onClick={() => onDeleteItem(item.id)}>
            Delete
          </button>
        </li>
      ))}
    </ul>
  );
}
```

```css
button { margin: 5px; }
li { list-style-type: none; }
ul, li { margin: 0; padding: 0; }
```

</Sandpack>

Notice how the event handlers are only concerned with calling `setItems` after this change. The item counts are now calculated during the next render from `items`, so they are always up-to-date.
请注意，事件处理程序在这次更改后只关心调用 `setItems`。现在，项目计数是从 `items` 中在下一次渲染期间计算的，因此它们始终是最新的。

</Solution>

#### 修复消失的选项 | Fix the disappearing selection {/*fix-the-disappearing-selection*/}

有一个 `letters` 列表在 state 中。当你悬停或聚焦到特定的信件时，它会被突出显示。当前突出显示的信件存储在 `highlightedLetter` state 变量中。您可以“Star”和“Unstar”单个信件，这将更新 state 中的 `letters` 数组。

This code works, but there is a minor UI glitch. When you press "Star" or "Unstar", the highlighting disappears for a moment. However, it reappears as soon as you move your pointer or switch to another letter with keyboard. Why is this happening? Fix it so that the highlighting doesn't disappear after the button click.
虽然这段代码可以运行，但是有一个小的 UI 问题。当你点击“Star”或“Unstar”时，高亮会短暂消失。不过只要你移动鼠标指针或者用键盘切换到另一个信件，它就会重新出现。为什么会这样？请修复它，使得在按钮点击后高亮不会消失。

<Sandpack>

```js src/App.js
import { useState } from 'react';
import { initialLetters } from './data.js';
import Letter from './Letter.js';

export default function MailClient() {
  const [letters, setLetters] = useState(initialLetters);
  const [highlightedLetter, setHighlightedLetter] = useState(null);

  function handleHover(letter) {
    setHighlightedLetter(letter);
  }

  function handleStar(starred) {
    setLetters(letters.map(letter => {
      if (letter.id === starred.id) {
        return {
          ...letter,
          isStarred: !letter.isStarred
        };
      } else {
        return letter;
      }
    }));
  }

  return (
    <>
      <h2>Inbox</h2>
      <ul>
        {letters.map(letter => (
          <Letter
            key={letter.id}
            letter={letter}
            isHighlighted={
              letter === highlightedLetter
            }
            onHover={handleHover}
            onToggleStar={handleStar}
          />
        ))}
      </ul>
    </>
  );
}
```

```js src/Letter.js
export default function Letter({
  letter,
  isHighlighted,
  onHover,
  onToggleStar,
}) {
  return (
    <li
      className={
        isHighlighted ? 'highlighted' : ''
      }
      onFocus={() => {
        onHover(letter);        
      }}
      onPointerMove={() => {
        onHover(letter);
      }}
    >
      <button onClick={() => {
        onToggleStar(letter);
      }}>
        {letter.isStarred ? 'Unstar' : 'Star'}
      </button>
      {letter.subject}
    </li>
  )
}
```

```js src/data.js
export const initialLetters = [{
  id: 0,
  subject: 'Ready for adventure?',
  isStarred: true,
}, {
  id: 1,
  subject: 'Time to check in!',
  isStarred: false,
}, {
  id: 2,
  subject: 'Festival Begins in Just SEVEN Days!',
  isStarred: false,
}];
```

```css
button { margin: 5px; }
li { border-radius: 5px; }
.highlighted { background: #d2eaff; }
```

</Sandpack>

<Solution>

The problem is that you're holding the letter object in `highlightedLetter`. But you're also holding the same information in the `letters` array. So your state has duplication! When you update the `letters` array after the button click, you create a new letter object which is different from `highlightedLetter`. This is why `highlightedLetter === letter` check becomes `false`, and the highlight disappears. It reappears the next time you call `setHighlightedLetter` when the pointer moves.
这个问题点在于你将信件对象存储在 `highlightedLetter` 中。但是，你也将相同的信息存储在 `letters` 数组中。因此，你的 state 存在重复！当你在按钮点击后更新 `letters` 数组时，会创建一个新的信件对象，它与 `highlightedLetter` 不同。这就是 `highlightedLetter === letter` 执行变为 `false`，并且高亮消失的原因。当指针移动时下一次调用 `setHighlightedLetter` 时它会重新出现。

为了解决这个问题，请从 state 中删除重复项。不要在两个地方存储 **信件对象本身**，而是存储 `highlightedId`。然后，您可以使用 `letter.id === highlightedId` 检查每个带有 `isHighlighted` 属性的信件，即使 `letter` 对象在上次渲染后发生了变化，这也是可行的。

<Sandpack>

```js src/App.js
import { useState } from 'react';
import { initialLetters } from './data.js';
import Letter from './Letter.js';

export default function MailClient() {
  const [letters, setLetters] = useState(initialLetters);
  const [highlightedId, setHighlightedId ] = useState(null);

  function handleHover(letterId) {
    setHighlightedId(letterId);
  }

  function handleStar(starredId) {
    setLetters(letters.map(letter => {
      if (letter.id === starredId) {
        return {
          ...letter,
          isStarred: !letter.isStarred
        };
      } else {
        return letter;
      }
    }));
  }

  return (
    <>
      <h2>Inbox</h2>
      <ul>
        {letters.map(letter => (
          <Letter
            key={letter.id}
            letter={letter}
            isHighlighted={
              letter.id === highlightedId
            }
            onHover={handleHover}
            onToggleStar={handleStar}
          />
        ))}
      </ul>
    </>
  );
}
```

```js src/Letter.js
export default function Letter({
  letter,
  isHighlighted,
  onHover,
  onToggleStar,
}) {
  return (
    <li
      className={
        isHighlighted ? 'highlighted' : ''
      }
      onFocus={() => {
        onHover(letter.id);        
      }}
      onPointerMove={() => {
        onHover(letter.id);
      }}
    >
      <button onClick={() => {
        onToggleStar(letter.id);
      }}>
        {letter.isStarred ? 'Unstar' : 'Star'}
      </button>
      {letter.subject}
    </li>
  )
}
```

```js src/data.js
export const initialLetters = [{
  id: 0,
  subject: 'Ready for adventure?',
  isStarred: true,
}, {
  id: 1,
  subject: 'Time to check in!',
  isStarred: false,
}, {
  id: 2,
  subject: 'Festival Begins in Just SEVEN Days!',
  isStarred: false,
}];
```

```css
button { margin: 5px; }
li { border-radius: 5px; }
.highlighted { background: #d2eaff; }
```

</Sandpack>

</Solution>

#### 实现多选功能 | Implement multiple selection {/*implement-multiple-selection*/}

In this example, each `Letter` has an `isSelected` prop and an `onToggle` handler that marks it as selected. This works, but the state is stored as a `selectedId` (either `null` or an ID), so only one letter can get selected at any given time.
在这个例子中，每个 `Letter` 都有一个 `isSelected` prop 和一个 `onToggle` 处理程序来标记它为选定 state。这样做是有效的，但是 state 被存储为 `selectedId`（也可以是 `null` 或 `ID`），因此任何时候只能选择一个 letter。

Change the state structure to support multiple selection. (How would you structure it? Think about this before writing the code.) Each checkbox should become independent from the others. Clicking a selected letter should uncheck it. Finally, the footer should show the correct number of the selected items.
你需要将 state 结构更改为支持多选功能。（在编写代码之前，请考虑如何构建它。）每个复选框应该独立于其他复选框。单击已选择的项目应取消选择。最后，页脚应显示所选项目的正确数量。

<Hint>

Instead of a single selected ID, you might want to hold an array or a [Set](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Set) of selected IDs in state.
你可以在 state 中保存一个选定 ID 的数组或 [Set](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Set)，而不是单个选定的 ID。

</Hint>

<Sandpack>

```js src/App.js
import { useState } from 'react';
import { letters } from './data.js';
import Letter from './Letter.js';

export default function MailClient() {
  const [selectedId, setSelectedId] = useState(null);

  // TODO: allow multiple selection
  // TODO: 支持多选
  const selectedCount = 1;

  function handleToggle(toggledId) {
    // TODO: allow multiple selection
    // TODO: 支持多选
    setSelectedId(toggledId);
  }

  return (
    <>
      <h2>Inbox</h2>
      <ul>
        {letters.map(letter => (
          <Letter
            key={letter.id}
            letter={letter}
            isSelected={
              // TODO: allow multiple selection
              // TODO: 支持多选
              letter.id === selectedId
            }
            onToggle={handleToggle}
          />
        ))}
        <hr />
        <p>
          <b>
            You selected {selectedCount} letters
          </b>
        </p>
      </ul>
    </>
  );
}
```

```js src/Letter.js
export default function Letter({
  letter,
  onToggle,
  isSelected,
}) {
  return (
    <li className={
      isSelected ? 'selected' : ''
    }>
      <label>
        <input
          type="checkbox"
          checked={isSelected}
          onChange={() => {
            onToggle(letter.id);
          }}
        />
        {letter.subject}
      </label>
    </li>
  )
}
```

```js src/data.js
export const letters = [{
  id: 0,
  subject: 'Ready for adventure?',
  isStarred: true,
}, {
  id: 1,
  subject: 'Time to check in!',
  isStarred: false,
}, {
  id: 2,
  subject: 'Festival Begins in Just SEVEN Days!',
  isStarred: false,
}];
```

```css
input { margin: 5px; }
li { border-radius: 5px; }
label { width: 100%; padding: 5px; display: inline-block; }
.selected { background: #d2eaff; }
```

</Sandpack>

<Solution>

在 state 中保留一个 `selectedIds` **数组**，而不是单个的 `selectedId`。例如，如果您选择了第一个和最后一个信件，则它将包含 `[0, 2]`。当没有选定任何内容时，它将为空数组 `[]`：

<Sandpack>

```js src/App.js
import { useState } from 'react';
import { letters } from './data.js';
import Letter from './Letter.js';

export default function MailClient() {
  const [selectedIds, setSelectedIds] = useState([]);

  const selectedCount = selectedIds.length;

  function handleToggle(toggledId) {
    // Was it previously selected?
    // 它以前是被选中的吗？
    if (selectedIds.includes(toggledId)) {
      // Then remove this ID from the array.
      setSelectedIds(selectedIds.filter(id =>
        id !== toggledId
      ));
    } else {
      // Otherwise, add this ID to the array.
      // 否则，增加 ID 到数组中。
      setSelectedIds([
        ...selectedIds,
        toggledId
      ]);
    }
  }

  return (
    <>
      <h2>Inbox</h2>
      <ul>
        {letters.map(letter => (
          <Letter
            key={letter.id}
            letter={letter}
            isSelected={
              selectedIds.includes(letter.id)
            }
            onToggle={handleToggle}
          />
        ))}
        <hr />
        <p>
          <b>
            You selected {selectedCount} letters
          </b>
        </p>
      </ul>
    </>
  );
}
```

```js src/Letter.js
export default function Letter({
  letter,
  onToggle,
  isSelected,
}) {
  return (
    <li className={
      isSelected ? 'selected' : ''
    }>
      <label>
        <input
          type="checkbox"
          checked={isSelected}
          onChange={() => {
            onToggle(letter.id);
          }}
        />
        {letter.subject}
      </label>
    </li>
  )
}
```

```js src/data.js
export const letters = [{
  id: 0,
  subject: 'Ready for adventure?',
  isStarred: true,
}, {
  id: 1,
  subject: 'Time to check in!',
  isStarred: false,
}, {
  id: 2,
  subject: 'Festival Begins in Just SEVEN Days!',
  isStarred: false,
}];
```

```css
input { margin: 5px; }
li { border-radius: 5px; }
label { width: 100%; padding: 5px; display: inline-block; }
.selected { background: #d2eaff; }
```

</Sandpack>

One minor downside of using an array is that for each item, you're calling `selectedIds.includes(letter.id)` to check whether it's selected. If the array is very large, this can become a performance problem because array search with [`includes()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/includes) takes linear time, and you're doing this search for each individual item.
使用数组的一个小缺点是，对于每个项目，你都需要调用 `selectedIds.includes(letter.id)` 来检查它是否被选中。如果数组非常大，则这可能会成为性能问题，因为带有 [`includes()`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/includes) 的数组搜索需要线性时间，并且你正在为每个单独的项目执行此搜索。

To fix this, you can hold a [Set](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Set) in state instead, which provides a fast [`has()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Set/has) operation:
要解决这个问题，你可以在 state 中使用一个 [Set](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Set) 对象，它提供了快速的 [`has()`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Set/has) 操作：

<Sandpack>

```js src/App.js
import { useState } from 'react';
import { letters } from './data.js';
import Letter from './Letter.js';

export default function MailClient() {
  const [selectedIds, setSelectedIds] = useState(
    new Set()
  );

  const selectedCount = selectedIds.size;

  function handleToggle(toggledId) {
    // Create a copy (to avoid mutation).
    const nextIds = new Set(selectedIds);
    if (nextIds.has(toggledId)) {
      nextIds.delete(toggledId);
    } else {
      nextIds.add(toggledId);
    }
    setSelectedIds(nextIds);
  }

  return (
    <>
      <h2>Inbox</h2>
      <ul>
        {letters.map(letter => (
          <Letter
            key={letter.id}
            letter={letter}
            isSelected={
              selectedIds.has(letter.id)
            }
            onToggle={handleToggle}
          />
        ))}
        <hr />
        <p>
          <b>
            You selected {selectedCount} letters
          </b>
        </p>
      </ul>
    </>
  );
}
```

```js src/Letter.js
export default function Letter({
  letter,
  onToggle,
  isSelected,
}) {
  return (
    <li className={
      isSelected ? 'selected' : ''
    }>
      <label>
        <input
          type="checkbox"
          checked={isSelected}
          onChange={() => {
            onToggle(letter.id);
          }}
        />
        {letter.subject}
      </label>
    </li>
  )
}
```

```js src/data.js
export const letters = [{
  id: 0,
  subject: 'Ready for adventure?',
  isStarred: true,
}, {
  id: 1,
  subject: 'Time to check in!',
  isStarred: false,
}, {
  id: 2,
  subject: 'Festival Begins in Just SEVEN Days!',
  isStarred: false,
}];
```

```css
input { margin: 5px; }
li { border-radius: 5px; }
label { width: 100%; padding: 5px; display: inline-block; }
.selected { background: #d2eaff; }
```

</Sandpack>

Now each item does a `selectedIds.has(letter.id)` check, which is very fast.
现在每个项目都会进行 `selectedIds.has(letter.id)` 检查，这非常快。

Keep in mind that you [should not mutate objects in state](/learn/updating-objects-in-state), and that includes Sets, too. This is why the `handleToggle` function creates a *copy* of the Set first, and then updates that copy.
请记住，你[不应该在 state 中改变对象](/learn/updating-objects-in-state)，包括 Set 中。这就是 `handleToggle` 函数首先创建 Set 的 **副本**，然后更新该副本的原因。

</Solution>

</Challenges>
