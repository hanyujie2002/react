---
title: 在组件间共享状态 | Sharing State Between Components
translators:
  - qinhua
---

<Intro>

Sometimes, you want the state of two components to always change together. To do it, remove state from both of them, move it to their closest common parent, and then pass it down to them via props. This is known as *lifting state up,* and it's one of the most common things you will do writing React code.
有时候，你希望两个组件的状态始终同步更改。要实现这一点，可以将相关 state 从这两个组件上移除，并把 state 放到它们的公共父级，再通过 props 将 state 传递给这两个组件。这被称为“状态提升”，这是编写 React 代码时常做的事。

</Intro>

<YouWillLearn>

- How to share state between components by lifting it up
- 如何使用状态提升在组件之间共享状态
- What are controlled and uncontrolled components
- 什么是受控组件和非受控组件

</YouWillLearn>

## 举例说明一下状态提升 | Lifting state up by example {/*lifting-state-up-by-example*/}

In this example, a parent `Accordion` component renders two separate `Panel`s:
在这个例子中，父组件 `Accordion` 渲染了 2 个独立的 `Panel` 组件。

* `Accordion`
  - `Panel`
  - `Panel`

Each `Panel` component has a boolean `isActive` state that determines whether its content is visible.
每个 `Panel` 组件都有一个布尔值 `isActive`，用于控制其内容是否可见。

Press the Show button for both panels:
请点击 2 个面板中的显示按钮：

<Sandpack>

```js
import { useState } from 'react';

function Panel({ title, children }) {
  const [isActive, setIsActive] = useState(false);
  return (
    <section className="panel">
      <h3>{title}</h3>
      {isActive ? (
        <p>{children}</p>
      ) : (
        <button onClick={() => setIsActive(true)}>
          显示
        </button>
      )}
    </section>
  );
}

export default function Accordion() {
  return (
    <>
      <h2>哈萨克斯坦，阿拉木图</h2>
      <Panel title="关于">
        阿拉木图人口约200万，是哈萨克斯坦最大的城市。它在 1929 年到 1997 年间都是首都。
      </Panel>
      <Panel title="词源">
        这个名字来自于 <span lang="kk-KZ">алма</span>，哈萨克语中“苹果”的意思，经常被翻译成“苹果之乡”。事实上，阿拉木图的周边地区被认为是苹果的发源地，<i lang="la">Malus sieversii</i> 被认为是现今苹果的祖先。
      </Panel>
    </>
  );
}
```

```css
h3, p { margin: 5px 0px; }
.panel {
  padding: 10px;
  border: 1px solid #aaa;
}
```

</Sandpack>

Notice how pressing one panel's button does not affect the other panel--they are independent.
我们发现点击其中一个面板中的按钮并不会影响另外一个，他们是独立的。


<DiagramGroup>

<Diagram name="sharing_state_child" height={367} width={477} alt="这个图表展示了一个包含三个组件的树形结构图，其中父组件是 Accordion，子组件是 Panel。这 2 个 Panel 组件内部都有 isActive 属性，值为 false。 | Diagram showing a tree of three components, one parent labeled Accordion and two children labeled Panel. Both Panel components contain isActive with value false.">

Initially, each `Panel`'s `isActive` state is `false`, so they both appear collapsed
由于一开始所有 `Panel` 内部的 `isActive` 都是 `false`，因此它们的内容都不会显示。

</Diagram>

<Diagram name="sharing_state_child_clicked" height={367} width={480} alt="这个图表与前面的是一样的，只是这里高亮显示了第一个子 Panel 组件的 isActive 属性，表示鼠标点击后将 isActive 的值设置为了 true。而第二个 Panel 组件的 isActive 值仍然还是 false。| The same diagram as the previous, with the isActive of the first child Panel component highlighted indicating a click with the isActive value set to true. The second Panel component still contains value false." >

Clicking either `Panel`'s button will only update that `Panel`'s `isActive` state alone
此时，点击任意一个 `Panel` 组件中的按钮都只会更新当前 `Panel` 组件内的 `isActive` 值

</Diagram>

</DiagramGroup>

**But now let's say you want to change it so that only one panel is expanded at any given time.** With that design, expanding the second panel should collapse the first one. How would you do that?
**假设现在你想改变这种行为，以便在任何时候只展开一个面板**。在这种设计下，展开第 2 个面板应会折叠第 1 个面板。你该如何做到这一点呢？"

To coordinate these two panels, you need to "lift their state up" to a parent component in three steps:
要协调好这两个面板，我们需要分 3 步将状态“提升”到他们的父组件中。

1. **Remove** state from the child components.
- 从子组件中 **移除** state 。
2. **Pass** hardcoded data from the common parent.
- 从父组件 **传递** 硬编码数据。 
3. **Add** state to the common parent and pass it down together with the event handlers.
- 为共同的父组件添加 state ，并将其与事件处理函数一起向下传递。

This will allow the `Accordion` component to coordinate both `Panel`s and only expand one at a time.
这样，`Accordion` 组件就可以控制 2 个 `Panel` 组件，保证同一时间只能展开一个。

### 第 1 步: 从子组件中移除状态 | Step 1: Remove state from the child components {/*step-1-remove-state-from-the-child-components*/}

You will give control of the `Panel`'s `isActive` to its parent component. This means that the parent component will pass `isActive` to `Panel` as a prop instead. Start by **removing this line** from the `Panel` component:
你将把 `Panel` 组件对 `isActive` 的控制权交给他们的父组件。这意味着，父组件会将 `isActive` 作为 `prop` 传给子组件 `Panel`。我们先从 `Panel` 组件中 **删除下面这一行**：

```js
const [isActive, setIsActive] = useState(false);
```

And instead, add `isActive` to the `Panel`'s list of props:
然后，把 `isActive` 加入 `Panel` 组件的 `props` 中：

```js
function Panel({ title, children, isActive }) {
```

Now the `Panel`'s parent component can *control* `isActive` by [passing it down as a prop.](/learn/passing-props-to-a-component) Conversely, the `Panel` component now has *no control* over the value of `isActive`--it's now up to the parent component!
现在 `Panel` 的父组件就可以通过 [向下传递 prop](/learn/passing-props-to-a-component) 来 **控制** `isActive`。但相反地，`Panel` 组件对 `isActive` 的值 **没有控制权** —— 现在完全由父组件决定！

### 第 2 步: 从公共父组件传递硬编码数据 | Step 2: Pass hardcoded data from the common parent {/*step-2-pass-hardcoded-data-from-the-common-parent*/}

To lift state up, you must locate the closest common parent component of *both* of the child components that you want to coordinate:
为了实现状态提升，必须定位到你想协调的 **两个** 子组件最近的公共父组件：

* `Accordion` **(最近的公共父组件)** | *(closest common parent)* 
  - `Panel`
  - `Panel`

In this example, it's the `Accordion` component. Since it's above both panels and can control their props, it will become the "source of truth" for which panel is currently active. Make the `Accordion` component pass a hardcoded value of `isActive` (for example, `true`) to both panels:
在这个例子中，公共父组件是 `Accordion`。因为它位于两个面板之上，可以控制它们的 props，所以它将成为当前激活面板的“控制之源”。通过 `Accordion` 组件将硬编码值 `isActive`（例如 `true` ）传递给两个面板：

<Sandpack>

```js
import { useState } from 'react';

export default function Accordion() {
  return (
    <>
      <h2>哈萨克斯坦，阿拉木图</h2>
      <Panel title="关于" isActive={true}>
        阿拉木图人口约200万，是哈萨克斯坦最大的城市。它在 1929 年到 1997 年间都是首都。
      </Panel>
      <Panel title="词源" isActive={true}>
        这个名字来自于 <span lang="kk-KZ">алма</span>，哈萨克语中“苹果”的意思，经常被翻译成“苹果之乡”。事实上，阿拉木图的周边地区被认为是苹果的发源地，<i lang="la">Malus sieversii</i> 被认为是现今苹果的祖先。
      </Panel>
    </>
  );
}

function Panel({ title, children, isActive }) {
  return (
    <section className="panel">
      <h3>{title}</h3>
      {isActive ? (
        <p>{children}</p>
      ) : (
        <button onClick={() => setIsActive(true)}>
          显示
        </button>
      )}
    </section>
  );
}
```

```css
h3, p { margin: 5px 0px; }
.panel {
  padding: 10px;
  border: 1px solid #aaa;
}
```

</Sandpack>

Try editing the hardcoded `isActive` values in the `Accordion` component and see the result on the screen.
你可以尝试修改 `Accordion` 组件中 `isActive` 的值，并在屏幕上查看结果。

### 第 3 步: 为公共父组件添加状态 | Step 3: Add state to the common parent {/*step-3-add-state-to-the-common-parent*/}

Lifting state up often changes the nature of what you're storing as state.
状态提升通常会改变原状态的数据存储类型。

In this case, only one panel should be active at a time. This means that the `Accordion` common parent component needs to keep track of *which* panel is the active one. Instead of a `boolean` value, it could use a number as the index of the active `Panel` for the state variable:
在这个例子中，一次只能激活一个面板。这意味着 `Accordion` 这个父组件需要记录 **哪个** 面板是被激活的面板。我们可以用数字作为当前被激活 `Panel` 的索引，而不是 `boolean` 值：

```js
const [activeIndex, setActiveIndex] = useState(0);
```

When the `activeIndex` is `0`, the first panel is active, and when it's `1`, it's the second one.
当 `activeIndex` 为 `0` 时，激活第一个面板，为 `1` 时，激活第二个面板。

Clicking the "Show" button in either `Panel` needs to change the active index in `Accordion`. A `Panel` can't set the `activeIndex` state directly because it's defined inside the `Accordion`. The `Accordion` component needs to *explicitly allow* the `Panel` component to change its state by [passing an event handler down as a prop](/learn/responding-to-events#passing-event-handlers-as-props):
在任意一个 `Panel` 中点击“显示”按钮都需要更改 `Accordion` 中的激活索引值。 `Panel` 中无法直接设置状态 `activeIndex` 的值，因为该状态是在 `Accordion` 组件内部定义的。 `Accordion` 组件需要 **显式允许** `Panel` 组件通过 [将事件处理程序作为 prop 向下传递](/learn/responding-to-events#passing-event-handlers-as-props) 来更改其状态：

```js
<>
  <Panel
    isActive={activeIndex === 0}
    onShow={() => setActiveIndex(0)}
  >
    ...
  </Panel>
  <Panel
    isActive={activeIndex === 1}
    onShow={() => setActiveIndex(1)}
  >
    ...
  </Panel>
</>
```

The `<button>` inside the `Panel` will now use the `onShow` prop as its click event handler:
现在 `Panel` 组件中的 `<button>` 将使用 `onShow` 这个属性作为其点击事件的处理程序：

<Sandpack>

```js
import { useState } from 'react';

export default function Accordion() {
  const [activeIndex, setActiveIndex] = useState(0);
  return (
    <>
      <h2>哈萨克斯坦，阿拉木图</h2>
      <Panel
        title="关于"
        isActive={activeIndex === 0}
        onShow={() => setActiveIndex(0)}
      >
        阿拉木图人口约200万，是哈萨克斯坦最大的城市。它在 1929 年到 1997 年间都是首都。
      </Panel>
      <Panel
        title="词源"
        isActive={activeIndex === 1}
        onShow={() => setActiveIndex(1)}
      >
        这个名字来自于 <span lang="kk-KZ">алма</span>，哈萨克语中“苹果”的意思，经常被翻译成“苹果之乡”。事实上，阿拉木图的周边地区被认为是苹果的发源地，<i lang="la">Malus sieversii</i> 被认为是现今苹果的祖先。
      </Panel>
    </>
  );
}

function Panel({
  title,
  children,
  isActive,
  onShow
}) {
  return (
    <section className="panel">
      <h3>{title}</h3>
      {isActive ? (
        <p>{children}</p>
      ) : (
        <button onClick={onShow}>
          显示
        </button>
      )}
    </section>
  );
}
```

```css
h3, p { margin: 5px 0px; }
.panel {
  padding: 10px;
  border: 1px solid #aaa;
}
```

</Sandpack>

This completes lifting state up! Moving state into the common parent component allowed you to coordinate the two panels. Using the active index instead of two "is shown" flags ensured that only one panel is active at a given time. And passing down the event handler to the child allowed the child to change the parent's state.
这样，我们就完成了对状态的提升！将状态移至公共父组件中可以让你更好的管理这两个面板。使用激活索引值代替之前的 `是否显示` 标识确保了一次只能激活一个面板。而通过向下传递事件处理函数可以让子组件修改父组件的状态。

<DiagramGroup>

<Diagram name="sharing_state_parent" height={385} width={487} alt="这个图表展示了一个包含三个组件的树形结构图，其中父组件是 Accordion，两个子组件是 Panel。Accordion 包含一个值为 0 的 activeIndex 属性，该属性会传递给第一个Panel 组件，让其内部的 isActive 值变为 true，同时会传递给第二个 Panel 组件，让其内部的 isActive 值变为 false。| Diagram showing a tree of three components, one parent labeled Accordion and two children labeled Panel. Accordion contains an activeIndex value of zero which turns into isActive value of true passed to the first Panel, and isActive value of false passed to the second Panel." >

Initially, `Accordion`'s `activeIndex` is `0`, so the first `Panel` receives `isActive = true`
一开始, `Accordion` 组件中的 `activeIndex` 是 `0`，此时第一个 `Panel` 组件接受到的属性为 `isActive = true`

</Diagram>

<Diagram name="sharing_state_parent_clicked" height={385} width={521} alt="这个图表与前面的是一样的，只是突出显示了父 Accordion 组件的 activeIndex值，表示单击后该值已更改为 1。同时，强调了两个子 Panel 组件的流程，并将传递给每个子组件的 isActive 值设置为相反的值：第一个 Panel 的值为 false，第二个 Panel 的值为 true。| The same diagram as the previous, with the activeIndex value of the parent Accordion component highlighted indicating a click with the value changed to one. The flow to both of the children Panel components is also highlighted, and the isActive value passed to each child is set to the opposite: false for the first Panel and true for the second one." >

When `Accordion`'s `activeIndex` state changes to `1`, the second `Panel` receives `isActive = true` instead
当 `Accordion` 组件中的 `activeIndex` 变为 `1` 时，第二个 `Panel` 组件接受到的属性变为 `isActive = true`，第一个 `Panel` 组件接受到的属性变为 `isActive = false`

</Diagram>

</DiagramGroup>

<DeepDive>

#### 受控组件和非受控组件 | Controlled and uncontrolled components {/*controlled-and-uncontrolled-components*/}

It is common to call a component with some local state "uncontrolled". For example, the original `Panel` component with an `isActive` state variable is uncontrolled because its parent cannot influence whether the panel is active or not.
通常我们把包含“不受控制”状态的组件称为“非受控组件”。例如，最开始带有 `isActive` 状态变量的 `Panel` 组件就是不受控制的，因为其父组件无法控制面板的激活状态。

In contrast, you might say a component is "controlled" when the important information in it is driven by props rather than its own local state. This lets the parent component fully specify its behavior. The final `Panel` component with the `isActive` prop is controlled by the `Accordion` component.
相反，当组件中的重要信息是由 `props` 而不是其自身状态驱动时，就可以认为该组件是“受控组件”。这就允许父组件完全指定其行为。最后带有 `isActive` 属性的 `Panel` 组件是由 `Accordion` 组件控制的。

Uncontrolled components are easier to use within their parents because they require less configuration. But they're less flexible when you want to coordinate them together. Controlled components are maximally flexible, but they require the parent components to fully configure them with props.
非受控组件通常很简单，因为它们不需要太多配置。但是当你想把它们组合在一起使用时，就不那么灵活了。受控组件具有最大的灵活性，但它们需要父组件使用 `props` 对其进行配置。

In practice, "controlled" and "uncontrolled" aren't strict technical terms--each component usually has some mix of both local state and props. However, this is a useful way to talk about how components are designed and what capabilities they offer.
在实践中，“受控”和“非受控”并不是严格的技术术语——通常每个组件都同时拥有内部状态和 `props`。然而，这对于组件该如何设计和提供什么样功能的讨论是有帮助的。

When writing a component, consider which information in it should be controlled (via props), and which information should be uncontrolled (via state). But you can always change your mind and refactor later.
当编写一个组件时，你应该考虑哪些信息应该受控制（通过 `props`），哪些信息不应该受控制（通过 `state`）。当然，你可以随时改变主意并重构代码。

</DeepDive>

## 每个状态都对应唯一的数据源 | A single source of truth for each state {/*a-single-source-of-truth-for-each-state*/}

In a React application, many components will have their own state. Some state may "live" close to the leaf components (components at the bottom of the tree) like inputs. Other state may "live" closer to the top of the app. For example, even client-side routing libraries are usually implemented by storing the current route in the React state, and passing it down by props!
在 `React` 应用中，很多组件都有自己的状态。一些状态可能“活跃”在叶子组件（树形结构最底层的组件）附近，例如输入框。另一些状态可能在应用程序顶部“活动”。例如，客户端路由库也是通过将当前路由存储在 `React` 状态中，利用 `props` 将状态层层传递下去来实现的！

**For each unique piece of state, you will choose the component that "owns" it.** This principle is also known as having a ["single source of truth".](https://en.wikipedia.org/wiki/Single_source_of_truth) It doesn't mean that all state lives in one place--but that for _each_ piece of state, there is a _specific_ component that holds that piece of information. Instead of duplicating shared state between components, *lift it up* to their common shared parent, and *pass it down* to the children that need it.
**对于每个独特的状态，都应该存在且只存在于一个指定的组件中作为 state**。这一原则也被称为拥有 [“可信单一数据源”](https://en.wikipedia.org/wiki/Single_source_of_truth)。它并不意味着所有状态都存在一个地方——对每个状态来说，都需要一个特定的组件来保存这些状态信息。你应该 **将状态提升** 到公共父级，或 **将状态传递** 到需要它的子级中，而不是在组件之间复制共享的状态。

Your app will change as you work on it. It is common that you will move state down or back up while you're still figuring out where each piece of the state "lives". This is all part of the process!
你的应用会随着你的操作而变化。当你将状态上下移动时，你依然会想要确定每个状态在哪里“活跃”。这都是过程的一部分！

To see what this feels like in practice with a few more components, read [Thinking in React.](/learn/thinking-in-react)
想了解在更多组件中的实践，请阅读 [React 思维](/learn/thinking-in-react).

<Recap>

* When you want to coordinate two components, move their state to their common parent.
* 当你想要整合两个组件时，将它们的 state 移动到共同的父组件中。
* Then pass the information down through props from their common parent.
* 然后在父组件中通过 `props` 把信息传递下去。
* Finally, pass the event handlers down so that the children can change the parent's state.
* 最后，向下传递事件处理程序，以便子组件可以改变父组件的 state 。
* It's useful to consider components as "controlled" (driven by props) or "uncontrolled" (driven by state).
* 考虑该将组件视为“受控”（由 prop 驱动）或是“不受控”（由 state 驱动）是十分有益的。

</Recap>

<Challenges>

#### 同步输入状态 | Synced inputs {/*synced-inputs*/}

These two inputs are independent. Make them stay in sync: editing one input should update the other input with the same text, and vice versa. 
现在有两个独立的输入框。为了让它们保持同步：即编辑一个输入框时，另一个输入框也会更新相同的文本，反之亦然。

<Hint>

You'll need to lift their state up into the parent component.
你需要将它们的状态移动到父组件中。

</Hint>

<Sandpack>

```js
import { useState } from 'react';

export default function SyncedInputs() {
  return (
    <>
      <Input label="第一个输入框" />
      <Input label="第二个输入框" />
    </>
  );
}

function Input({ label }) {
  const [text, setText] = useState('');

  function handleChange(e) {
    setText(e.target.value);
  }

  return (
    <label>
      {label}
      {' '}
      <input
        value={text}
        onChange={handleChange}
      />
    </label>
  );
}
```

```css
input { margin: 5px; }
label { display: block; }
```

</Sandpack>

<Solution>

Move the `text` state variable into the parent component along with the `handleChange` handler. Then pass them down as props to both of the `Input` components. This will keep them in sync.
将 `text` 状态变量与 `handleChange` 处理程序一起移动到父组件中。然后将它们作为 `props` 传递给两个 `Input` 组件。这样它们就能保持同步了。

<Sandpack>

```js
import { useState } from 'react';

export default function SyncedInputs() {
  const [text, setText] = useState('');

  function handleChange(e) {
    setText(e.target.value);
  }

  return (
    <>
      <Input
        label="第一个输入框"
        value={text}
        onChange={handleChange}
      />
      <Input
        label="第二个输入框"
        value={text}
        onChange={handleChange}
      />
    </>
  );
}

function Input({ label, value, onChange }) {
  return (
    <label>
      {label}
      {' '}
      <input
        value={value}
        onChange={onChange}
      />
    </label>
  );
}
```

```css
input { margin: 5px; }
label { display: block; }
```

</Sandpack>

</Solution>

#### 列表过滤 {/*filtering-a-list*/}

In this example, the `SearchBar` has its own `query` state that controls the text input. Its parent `FilterableList` component displays a `List` of items, but it doesn't take the search query into account.
在这个例子中，`SearchBar` 组件拥有一个用来控制输入框的 `query` 状态，它的父组件中展示了一个 `List` 组件，但是没有考虑搜索条件。

Use the `filterItems(foods, query)` function to filter the list according to the search query. To test your changes, verify that typing "s" into the input filters down the list to "Sushi", "Shish kebab", and "Dim sum".
使用 `filterItems(foods, query)` 方法来通过搜索条件过滤列表项。为了测试修改是否正确，请尝试在输入框中输入 “寿司” ，“烤肉串” 或 “点心”。

Note that `filterItems` is already implemented and imported so you don't need to write it yourself!
可以看到 `filterItems` 已经自动引入了，所以不需要我们自己再引入了。

<Hint>

You will want to remove the `query` state and the `handleChange` handler from the `SearchBar`, and move them to the `FilterableList`. Then pass them down to `SearchBar` as `query` and `onChange` props.
你需要从 `SearchBar` 组件中移除 `query` 状态和 `handleChange` 处理程序，并把它们移到 `FilterableList` 组件中。然后将 `query` 和 `onChange` 当做 props 向下传递给 `SearchBar` 组件。

</Hint>

<Sandpack>

```js
import { useState } from 'react';
import { foods, filterItems } from './data.js';

export default function FilterableList() {
  return (
    <>
      <SearchBar />
      <hr />
      <List items={foods} />
    </>
  );
}

function SearchBar() {
  const [query, setQuery] = useState('');

  function handleChange(e) {
    setQuery(e.target.value);
  }

  return (
    <label>
      搜索：{' '}
      <input
        value={query}
        onChange={handleChange}
      />
    </label>
  );
}

function List({ items }) {
  return (
    <table>
      <tbody>
        {items.map(food => (
          <tr key={food.id}>
            <td>{food.name}</td>
            <td>{food.description}</td>
          </tr>
        ))}
      </tbody>
    </table>
  );
}
```

```js src/data.js
export function filterItems(items, query) {
  query = query.toLowerCase();
  return items.filter(item =>
    item.name.split(' ').some(word =>
      word.toLowerCase().startsWith(query)
    )
  );
}

export const foods = [{
  id: 0,
  name: '寿司',
  description: '寿司是一道传统的日本菜，是用醋米饭做成的'
}, {
  id: 1,
  name: '木豆',
  description: '制作木豆最常见的方法是在汤中加入洋葱、西红柿和各种香料'
}, {
  id: 2,
  name: '饺子',
  description: '饺子是用未发酵的面团包裹咸的或甜的馅料，然后在沸水中煮制而成的'
}, {
  id: 3,
  name: '烤肉串',
  description: '烤肉串是一种很受欢迎的食物，是用肉串和肉块做成。'
}, {
  id: 4,
  name: '点心',
  description: '点心是广东人的传统喜好，是在餐馆吃早餐和午餐时喜欢吃的一系列小菜'
}];
```

</Sandpack>

<Solution>

Lift the `query` state up into the `FilterableList` component. Call `filterItems(foods, query)` to get the filtered list and pass it down to the `List`. Now changing the query input is reflected in the list:
将 `query` 状态提升到 `FilterableList` 组件中。调用 `filterItems(foods, query)` 方法获取过滤后的列表并将其传递给 `List` 组件。现在修改查询条件就会反映在列表中：

<Sandpack>

```js
import { useState } from 'react';
import { foods, filterItems } from './data.js';

export default function FilterableList() {
  const [query, setQuery] = useState('');
  const results = filterItems(foods, query);

  function handleChange(e) {
    setQuery(e.target.value);
  }

  return (
    <>
      <SearchBar
        query={query}
        onChange={handleChange}
      />
      <hr />
      <List items={results} />
    </>
  );
}

function SearchBar({ query, onChange }) {
  return (
    <label>
      搜索：{' '}
      <input
        value={query}
        onChange={onChange}
      />
    </label>
  );
}

function List({ items }) {
  return (
    <table>
      <tbody> 
        {items.map(food => (
          <tr key={food.id}>
            <td>{food.name}</td>
            <td>{food.description}</td>
          </tr>
        ))}
      </tbody>
    </table>
  );
}
```

```js src/data.js
export function filterItems(items, query) {
  query = query.toLowerCase();
  return items.filter(item =>
    item.name.split(' ').some(word =>
      word.toLowerCase().startsWith(query)
    )
  );
}

export const foods = [{
  id: 0,
  name: '寿司',
  description: '寿司是一道传统的日本菜，是用醋米饭做成的'
}, {
  id: 1,
  name: '木豆',
  description: '制作木豆最常见的方法是在汤中加入洋葱、西红柿和各种香料'
}, {
  id: 2,
  name: '饺子',
  description: '饺子是用未发酵的面团包裹咸的或甜的馅料，然后在沸水中煮制而成的'
}, {
  id: 3,
  name: '烤肉串',
  description: '烤肉串是一种很受欢迎的食物，是用肉串和肉块做成。'
}, {
  id: 4,
  name: '点心',
  description: '点心是广东人的传统喜好，是在餐馆吃早餐和午餐时喜欢吃的一系列小菜'
}];
```

</Sandpack>

</Solution>

</Challenges>
