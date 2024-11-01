---
title: 添加交互 | Adding Interactivity
---

<Intro>

Some things on the screen update in response to user input. For example, clicking an image gallery switches the active image. In React, data that changes over time is called *state.* You can add state to any component, and update it as needed. In this chapter, you'll learn how to write components that handle interactions, update their state, and display different output over time.
界面上的控件会根据用户的输入而更新。例如，点击按钮切换轮播图的展示。在 React 中，随时间变化的数据被称为状态（state）。你可以向任何组件添加状态，并按需进行更新。在本章节中，你将学习如何编写处理交互的组件，更新它们的状态，并根据时间变化显示不同的效果。

</Intro>

<YouWillLearn isChapter={true}>

* [如何处理用户发起的事件 | How to handle user-initiated events](/learn/responding-to-events)
* [如何用状态使组件“记住”信息 | How to make components "remember" information with state](/learn/state-a-components-memory)
* [React 是如何分两个阶段更新 UI 的 | How React updates the UI in two phases](/learn/render-and-commit)
* [为什么状态在你改变后没有立即更新 | Why state doesn't update right after you change it](/learn/state-as-a-snapshot)
* [如何排队进行多个状态的更新 | How to queue multiple state updates](/learn/queueing-a-series-of-state-updates)
* [如何更新状态中的对象 | How to update an object in state](/learn/updating-objects-in-state)
* [如何更新状态中的数组 | How to update an array in state](/learn/updating-arrays-in-state)

</YouWillLearn>

## 响应事件 | Responding to events {/*responding-to-events*/}

React lets you add *event handlers* to your JSX. Event handlers are your own functions that will be triggered in response to user interactions like clicking, hovering, focusing on form inputs, and so on.
React 允许你向 JSX 中添加事件处理程序。事件处理程序是你自己的函数，它将在用户交互时被触发，如点击、悬停、焦点在表单输入框上等等。

Built-in components like `<button>` only support built-in browser events like `onClick`. However, you can also create your own components, and give their event handler props any application-specific names that you like.
`<button>` 等内置组件只支持内置浏览器事件，如 `onClick`。但是，你也可以创建你自己的组件，并给它们的事件处理程序 props 指定你喜欢的任何特定于应用的名称。

<Sandpack>

```js
export default function App() {
  return (
    <Toolbar
      onPlayMovie={() => alert('Playing!')}
      onUploadImage={() => alert('Uploading!')}
    />
  );
}

function Toolbar({ onPlayMovie, onUploadImage }) {
  return (
    <div>
      <Button onClick={onPlayMovie}>
        Play Movie
      </Button>
      <Button onClick={onUploadImage}>
        Upload Image
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

<LearnMore path="/learn/responding-to-events">

Read **[Responding to Events](/learn/responding-to-events)** to learn how to add event handlers.
阅读 **[响应事件](/learn/responding-to-events)** 了解如何添加事件处理程序。

</LearnMore>

## State: 组件的记忆 {/*state-a-components-memory*/}

Components often need to change what's on the screen as a result of an interaction. Typing into the form should update the input field, clicking "next" on an image carousel should change which image is displayed, clicking "buy" puts a product in the shopping cart. Components need to "remember" things: the current input value, the current image, the shopping cart. In React, this kind of component-specific memory is called *state.*
组件通常需要根据交互改变屏幕上的内容。在表单中键入更新输入栏，在轮播图上点击“下一个”改变显示的图片，点击“购买”将产品放入购物车。组件需要“记住”一些东西：当前的输入值、当前的图片、购物车。在 React 中，这种特定于组件的记忆被称为状态。

You can add state to a component with a [`useState`](/reference/react/useState) Hook. *Hooks* are special functions that let your components use React features (state is one of those features). The `useState` Hook lets you declare a state variable. It takes the initial state and returns a pair of values: the current state, and a state setter function that lets you update it.
你可以用 [`useState`](/reference/react/useState) Hook 为组件添加状态。*Hook* 是能让你的组件使用 React 功能的特殊函数（状态是这些功能之一）。`useState` Hook 让你声明一个状态变量。它接收初始状态并返回一对值：当前状态，以及一个让你更新状态的设置函数。

```js
const [index, setIndex] = useState(0);
const [showMore, setShowMore] = useState(false);
```

Here is how an image gallery uses and updates state on click:
下面是一个图库，演示如何使用并在点击时更新状态：

<Sandpack>

```js
import { useState } from 'react';
import { sculptureList } from './data.js';

export default function Gallery() {
  const [index, setIndex] = useState(0);
  const [showMore, setShowMore] = useState(false);
  const hasNext = index < sculptureList.length - 1;

  function handleNextClick() {
    if (hasNext) {
      setIndex(index + 1);
    } else {
      setIndex(0);
    }
  }

  function handleMoreClick() {
    setShowMore(!showMore);
  }

  let sculpture = sculptureList[index];
  return (
    <>
      <button onClick={handleNextClick}>
        Next
      </button>
      <h2>
        <i>{sculpture.name} </i>
        by {sculpture.artist}
      </h2>
      <h3>
        ({index + 1} of {sculptureList.length})
      </h3>
      <button onClick={handleMoreClick}>
        {showMore ? 'Hide' : 'Show'} details
      </button>
      {showMore && <p>{sculpture.description}</p>}
      <img
        src={sculpture.url}
        alt={sculpture.alt}
      />
    </>
  );
}
```

```js src/data.js
export const sculptureList = [{
  name: 'Homenaje a la Neurocirugía',
  artist: 'Marta Colvin Andrade',
  description: 'Although Colvin is predominantly known for abstract themes that allude to pre-Hispanic symbols, this gigantic sculpture, an homage to neurosurgery, is one of her most recognizable public art pieces.',
  url: 'https://i.imgur.com/Mx7dA2Y.jpg',
  alt: 'A bronze statue of two crossed hands delicately holding a human brain in their fingertips.'
}, {
  name: 'Floralis Genérica',
  artist: 'Eduardo Catalano',
  description: 'This enormous (75 ft. or 23m) silver flower is located in Buenos Aires. It is designed to move, closing its petals in the evening or when strong winds blow and opening them in the morning.',
  url: 'https://i.imgur.com/ZF6s192m.jpg',
  alt: 'A gigantic metallic flower sculpture with reflective mirror-like petals and strong stamens.'
}, {
  name: 'Eternal Presence',
  artist: 'John Woodrow Wilson',
  description: 'Wilson was known for his preoccupation with equality, social justice, as well as the essential and spiritual qualities of humankind. This massive (7ft. or 2,13m) bronze represents what he described as "a symbolic Black presence infused with a sense of universal humanity."',
  url: 'https://i.imgur.com/aTtVpES.jpg',
  alt: 'The sculpture depicting a human head seems ever-present and solemn. It radiates calm and serenity.'
}, {
  name: 'Moai',
  artist: 'Unknown Artist',
  description: 'Located on the Easter Island, there are 1,000 moai, or extant monumental statues, created by the early Rapa Nui people, which some believe represented deified ancestors.',
  url: 'https://i.imgur.com/RCwLEoQm.jpg',
  alt: 'Three monumental stone busts with the heads that are disproportionately large with somber faces.'
}, {
  name: 'Blue Nana',
  artist: 'Niki de Saint Phalle',
  description: 'The Nanas are triumphant creatures, symbols of femininity and maternity. Initially, Saint Phalle used fabric and found objects for the Nanas, and later on introduced polyester to achieve a more vibrant effect.',
  url: 'https://i.imgur.com/Sd1AgUOm.jpg',
  alt: 'A large mosaic sculpture of a whimsical dancing female figure in a colorful costume emanating joy.'
}, {
  name: 'Ultimate Form',
  artist: 'Barbara Hepworth',
  description: 'This abstract bronze sculpture is a part of The Family of Man series located at Yorkshire Sculpture Park. Hepworth chose not to create literal representations of the world but developed abstract forms inspired by people and landscapes.',
  url: 'https://i.imgur.com/2heNQDcm.jpg',
  alt: 'A tall sculpture made of three elements stacked on each other reminding of a human figure.'
}, {
  name: 'Cavaliere',
  artist: 'Lamidi Olonade Fakeye',
  description: "Descended from four generations of woodcarvers, Fakeye's work blended traditional and contemporary Yoruba themes.",
  url: 'https://i.imgur.com/wIdGuZwm.png',
  alt: 'An intricate wood sculpture of a warrior with a focused face on a horse adorned with patterns.'
}, {
  name: 'Big Bellies',
  artist: 'Alina Szapocznikow',
  description: "Szapocznikow is known for her sculptures of the fragmented body as a metaphor for the fragility and impermanence of youth and beauty. This sculpture depicts two very realistic large bellies stacked on top of each other, each around five feet (1,5m) tall.",
  url: 'https://i.imgur.com/AlHTAdDm.jpg',
  alt: 'The sculpture reminds a cascade of folds, quite different from bellies in classical sculptures.'
}, {
  name: 'Terracotta Army',
  artist: 'Unknown Artist',
  description: 'The Terracotta Army is a collection of terracotta sculptures depicting the armies of Qin Shi Huang, the first Emperor of China. The army consisted of more than 8,000 soldiers, 130 chariots with 520 horses, and 150 cavalry horses.',
  url: 'https://i.imgur.com/HMFmH6m.jpg',
  alt: '12 terracotta sculptures of solemn warriors, each with a unique facial expression and armor.'
}, {
  name: 'Lunar Landscape',
  artist: 'Louise Nevelson',
  description: 'Nevelson was known for scavenging objects from New York City debris, which she would later assemble into monumental constructions. In this one, she used disparate parts like a bedpost, juggling pin, and seat fragment, nailing and gluing them into boxes that reflect the influence of Cubism’s geometric abstraction of space and form.',
  url: 'https://i.imgur.com/rN7hY6om.jpg',
  alt: 'A black matte sculpture where the individual elements are initially indistinguishable.'
}, {
  name: 'Aureole',
  artist: 'Ranjani Shettar',
  description: 'Shettar merges the traditional and the modern, the natural and the industrial. Her art focuses on the relationship between man and nature. Her work was described as compelling both abstractly and figuratively, gravity defying, and a "fine synthesis of unlikely materials."',
  url: 'https://i.imgur.com/okTpbHhm.jpg',
  alt: 'A pale wire-like sculpture mounted on concrete wall and descending on the floor. It appears light.'
}, {
  name: 'Hippos',
  artist: 'Taipei Zoo',
  description: 'The Taipei Zoo commissioned a Hippo Square featuring submerged hippos at play.',
  url: 'https://i.imgur.com/6o5Vuyu.jpg',
  alt: 'A group of bronze hippo sculptures emerging from the sett sidewalk as if they were swimming.'
}];
```

```css
h2 { margin-top: 10px; margin-bottom: 0; }
h3 {
 margin-top: 5px;
 font-weight: normal;
 font-size: 100%;
}
img { width: 120px; height: 120px; }
button {
  display: block;
  margin-top: 10px;
  margin-bottom: 10px;
}
```

</Sandpack>

<LearnMore path="/learn/state-a-components-memory">

Read **[State: A Component's Memory](/learn/state-a-components-memory)** to learn how to remember a value and update it on interaction.
阅读 **[State: 组件的记忆](/learn/state-a-components-memory)** 了解如何记住一个值并在交互时更新它。

</LearnMore>

## 渲染和提交 | Render and commit {/*render-and-commit*/}

Before your components are displayed on the screen, they must be rendered by React. Understanding the steps in this process will help you think about how your code executes and explain its behavior.
在你的组件显示在屏幕上之前，它们必须由 React 进行渲染。理解这个过程中的步骤有助于你思考你的代码如何执行并解释其行为。

Imagine that your components are cooks in the kitchen, assembling tasty dishes from ingredients. In this scenario, React is the waiter who puts in requests from customers and brings them their orders. This process of requesting and serving UI has three steps:
想象一下，你的组件是厨房里的厨师，用食材制作出美味的菜肴。在这个场景中，React 是服务员，负责提出顾客的要求，并给顾客上菜。这个请求和服务 UI 的过程有三个步骤：

1. **Triggering** a render (delivering the diner's order to the kitchen)
- **触发**渲染（将食客的订单送到厨房）
2. **Rendering** the component (preparing the order in the kitchen)
- **渲染**组件（在厨房准备订单）
3. **Committing** to the DOM (placing the order on the table)
- **提交**到 DOM（将订单送到桌前）

<IllustrationBlock sequential>
  <Illustration caption="Trigger" alt="React 就像一个餐厅的服务员，从用户那里获取订单，并将它们送到组件厨房。| React as a server in a restaurant, fetching orders from the users and delivering them to the Component Kitchen." src="/images/docs/illustrations/i_render-and-commit1.png" />
  <Illustration caption="Render" alt="Card 厨师给 React 提供了一个新鲜的 Card 组件。| The Card Chef gives React a fresh Card component." src="/images/docs/illustrations/i_render-and-commit2.png" />
  <Illustration caption="Commit" alt="React 将 Card 送到用户桌前。| React delivers the Card to the user at their table." src="/images/docs/illustrations/i_render-and-commit3.png" />
</IllustrationBlock>

<LearnMore path="/learn/render-and-commit">

Read **[Render and Commit](/learn/render-and-commit)** to learn the lifecycle of a UI update.
阅读 **[渲染和提交](/learn/render-and-commit)** 了解 UI 更新的生命周期。

</LearnMore>

## state 如同一张快照 | State as a snapshot {/*state-as-a-snapshot*/}

Unlike regular JavaScript variables, React state behaves more like a snapshot. Setting it does not change the state variable you already have, but instead triggers a re-render. This can be surprising at first!
与普通 JavaScript 变量不同，React 状态的行为更像一个快照。设置它并不改变你已有的状态变量，而是触发一次重新渲染。这在一开始可能会让人感到惊讶！

```js
console.log(count);  // 0
setCount(count + 1); // 请求用 1 重新渲染 // Request a re-render with 1
console.log(count);  // 仍然是 0！// Still 0!
```

This behavior helps you avoid subtle bugs. Here is a little chat app. Try to guess what happens if you press "Send" first and *then* change the recipient to Bob. Whose name will appear in the `alert` five seconds later?
React 这样工作是为了帮助你避免微妙的 bug。这里有一个小的聊天应用程序。试着猜一猜，如果先按下“发送”，然后再把收件人改为 Bob，会发生什么？五秒钟后，谁的名字会出现在 `alert` 中？

<Sandpack>

```js
import { useState } from 'react';

export default function Form() {
  const [to, setTo] = useState('Alice');
  const [message, setMessage] = useState('Hello');

  function handleSubmit(e) {
    e.preventDefault();
    setTimeout(() => {
      alert(`You said ${message} to ${to}`);
    }, 5000);
  }

  return (
    <form onSubmit={handleSubmit}>
      <label>
        To:{' '}
        <select
          value={to}
          onChange={e => setTo(e.target.value)}>
          <option value="Alice">Alice</option>
          <option value="Bob">Bob</option>
        </select>
      </label>
      <textarea
        placeholder="Message"
        value={message}
        onChange={e => setMessage(e.target.value)}
      />
      <button type="submit">Send</button>
    </form>
  );
}
```

```css
label, textarea { margin-bottom: 10px; display: block; }
```

</Sandpack>


<LearnMore path="/learn/state-as-a-snapshot">

Read **[State as a Snapshot](/learn/state-as-a-snapshot)** to learn why state appears "fixed" and unchanging inside the event handlers.
阅读 **[state 如同一张快照](/learn/state-as-a-snapshot)** 了解为什么状态在事件处理程序中是“固定的”和不变的。

</LearnMore>

## 把一系列 state 更新加入队列 | Queueing a series of state updates {/*queueing-a-series-of-state-updates*/}

This component is buggy: clicking "+3" increments the score only once.
这个组件有问题：点击“+3”只能增加一次分数。

<Sandpack>

```js
import { useState } from 'react';

export default function Counter() {
  const [score, setScore] = useState(0);

  function increment() {
    setScore(score + 1);
  }

  return (
    <>
      <button onClick={() => increment()}>+1</button>
      <button onClick={() => {
        increment();
        increment();
        increment();
      }}>+3</button>
      <h1>Score: {score}</h1>
    </>
  )
}
```

```css
button { display: inline-block; margin: 10px; font-size: 20px; }
```

</Sandpack>

[State as a Snapshot](/learn/state-as-a-snapshot) explains why this is happening. Setting state requests a new re-render, but does not change it in the already running code. So `score` continues to be `0` right after you call `setScore(score + 1)`.
[state 如同一张快照](/learn/state-as-a-snapshot) 解释了为什么会出现这种情况。设置状态会请求一个新的重新渲染，但不会在已运行的代码中更改它。所以在你调用 `setScore(score + 1)` 后，`score` 仍然是 `0`。

```js
console.log(score);  // 0
setScore(score + 1); // setScore(0 + 1);
console.log(score);  // 0
setScore(score + 1); // setScore(0 + 1);
console.log(score);  // 0
setScore(score + 1); // setScore(0 + 1);
console.log(score);  // 0
```

You can fix this by passing an *updater function* when setting state. Notice how replacing `setScore(score + 1)` with `setScore(s => s + 1)` fixes the "+3" button. This lets you queue multiple state updates.
你可以通过在设置状态时传递一个 *更新器函数* 来解决这个问题。注意用 `setScore(s => s + 1)` 替换 `setScore(score + 1)` 是如何修复“+3”按钮的。如果你需要排队进行多次状态更新，那么这非常方便。

<Sandpack>

```js
import { useState } from 'react';

export default function Counter() {
  const [score, setScore] = useState(0);

  function increment() {
    setScore(s => s + 1);
  }

  return (
    <>
      <button onClick={() => increment()}>+1</button>
      <button onClick={() => {
        increment();
        increment();
        increment();
      }}>+3</button>
      <h1>Score: {score}</h1>
    </>
  )
}
```

```css
button { display: inline-block; margin: 10px; font-size: 20px; }
```

</Sandpack>

<LearnMore path="/learn/queueing-a-series-of-state-updates">

Read **[Queueing a Series of State Updates](/learn/queueing-a-series-of-state-updates)** to learn how to queue a sequence of state updates.
阅读 **[把一系列 state 更新加入队列](/learn/queueing-a-series-of-state-updates)** 了解如何在下一次渲染前排队多个更新。

</LearnMore>

## 更新 state 中的对象 | Updating objects in state {/*updating-objects-in-state*/}

State can hold any kind of JavaScript value, including objects. But you shouldn't change objects and arrays that you hold in the React state directly. Instead, when you want to update an object and array, you need to create a new one (or make a copy of an existing one), and then update the state to use that copy.
状态可以持有任何类型的 JavaScript 值，包括对象。但你不应该直接改变你在 React 状态中持有的对象和数组。相反，当你想更新一个对象和数组时，你需要创建一个新的对象（或复制现有的对象），然后用这个副本来更新状态。

Usually, you will use the `...` spread syntax to copy objects and arrays that you want to change. For example, updating a nested object could look like this:
通常情况下，你会使用 `...` 展开语法来复制你想改变的对象和数组。例如，更新一个嵌套对象可以是这样的：

<Sandpack>

```js
import { useState } from 'react';

export default function Form() {
  const [person, setPerson] = useState({
    name: 'Niki de Saint Phalle',
    artwork: {
      title: 'Blue Nana',
      city: 'Hamburg',
      image: 'https://i.imgur.com/Sd1AgUOm.jpg',
    }
  });

  function handleNameChange(e) {
    setPerson({
      ...person,
      name: e.target.value
    });
  }

  function handleTitleChange(e) {
    setPerson({
      ...person,
      artwork: {
        ...person.artwork,
        title: e.target.value
      }
    });
  }

  function handleCityChange(e) {
    setPerson({
      ...person,
      artwork: {
        ...person.artwork,
        city: e.target.value
      }
    });
  }

  function handleImageChange(e) {
    setPerson({
      ...person,
      artwork: {
        ...person.artwork,
        image: e.target.value
      }
    });
  }

  return (
    <>
      <label>
        Name:
        <input
          value={person.name}
          onChange={handleNameChange}
        />
      </label>
      <label>
        Title:
        <input
          value={person.artwork.title}
          onChange={handleTitleChange}
        />
      </label>
      <label>
        City:
        <input
          value={person.artwork.city}
          onChange={handleCityChange}
        />
      </label>
      <label>
        Image:
        <input
          value={person.artwork.image}
          onChange={handleImageChange}
        />
      </label>
      <p>
        <i>{person.artwork.title}</i>
        {' by '}
        {person.name}
        <br />
        (located in {person.artwork.city})
      </p>
      <img
        src={person.artwork.image}
        alt={person.artwork.title}
      />
    </>
  );
}
```

```css
label { display: block; }
input { margin-left: 5px; margin-bottom: 5px; }
img { width: 200px; height: 200px; }
```

</Sandpack>

If copying objects in code gets tedious, you can use a library like [Immer](https://github.com/immerjs/use-immer) to reduce repetitive code:
如果在代码中复制对象感觉乏味，可以使用 [Immer](https://github.com/immerjs/use-immer) 之类的库来减少重复代码：

<Sandpack>

```js
import { useImmer } from 'use-immer';

export default function Form() {
  const [person, updatePerson] = useImmer({
    name: 'Niki de Saint Phalle',
    artwork: {
      title: 'Blue Nana',
      city: 'Hamburg',
      image: 'https://i.imgur.com/Sd1AgUOm.jpg',
    }
  });

  function handleNameChange(e) {
    updatePerson(draft => {
      draft.name = e.target.value;
    });
  }

  function handleTitleChange(e) {
    updatePerson(draft => {
      draft.artwork.title = e.target.value;
    });
  }

  function handleCityChange(e) {
    updatePerson(draft => {
      draft.artwork.city = e.target.value;
    });
  }

  function handleImageChange(e) {
    updatePerson(draft => {
      draft.artwork.image = e.target.value;
    });
  }

  return (
    <>
      <label>
        Name:
        <input
          value={person.name}
          onChange={handleNameChange}
        />
      </label>
      <label>
        Title:
        <input
          value={person.artwork.title}
          onChange={handleTitleChange}
        />
      </label>
      <label>
        City:
        <input
          value={person.artwork.city}
          onChange={handleCityChange}
        />
      </label>
      <label>
        Image:
        <input
          value={person.artwork.image}
          onChange={handleImageChange}
        />
      </label>
      <p>
        <i>{person.artwork.title}</i>
        {' by '}
        {person.name}
        <br />
        (located in {person.artwork.city})
      </p>
      <img
        src={person.artwork.image}
        alt={person.artwork.title}
      />
    </>
  );
}
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

```css
label { display: block; }
input { margin-left: 5px; margin-bottom: 5px; }
img { width: 200px; height: 200px; }
```

</Sandpack>

<LearnMore path="/learn/updating-objects-in-state">

Read **[Updating Objects in State](/learn/updating-objects-in-state)** to learn how to update objects correctly.
阅读 **[更新 state 中的对象](/learn/updating-objects-in-state)** 了解如何正确地更新对象。

</LearnMore>

## 更新 state 中的数组 | Updating arrays in state {/*updating-arrays-in-state*/}

Arrays are another type of mutable JavaScript objects you can store in state and should treat as read-only. Just like with objects, when you want to update an array stored in state, you need to create a new one (or make a copy of an existing one), and then set state to use the new array:
数组是另一种可以存在状态中的可变 JavaScript 对象，应将其视为只读。就像对象一样，当你想更新存在状态中的数组时，你需要创建一个新数组（或者复制现有数组），然后用新数组来更新状态。

<Sandpack>

```js
import { useState } from 'react';

const initialList = [
  { id: 0, title: 'Big Bellies', seen: false },
  { id: 1, title: 'Lunar Landscape', seen: false },
  { id: 2, title: 'Terracotta Army', seen: true },
];

export default function BucketList() {
  const [list, setList] = useState(
    initialList
  );

  function handleToggle(artworkId, nextSeen) {
    setList(list.map(artwork => {
      if (artwork.id === artworkId) {
        return { ...artwork, seen: nextSeen };
      } else {
        return artwork;
      }
    }));
  }

  return (
    <>
      <h1>Art Bucket List</h1>
      <h2>My list of art to see:</h2>
      <ItemList
        artworks={list}
        onToggle={handleToggle} />
    </>
  );
}

function ItemList({ artworks, onToggle }) {
  return (
    <ul>
      {artworks.map(artwork => (
        <li key={artwork.id}>
          <label>
            <input
              type="checkbox"
              checked={artwork.seen}
              onChange={e => {
                onToggle(
                  artwork.id,
                  e.target.checked
                );
              }}
            />
            {artwork.title}
          </label>
        </li>
      ))}
    </ul>
  );
}
```

</Sandpack>

If copying arrays in code gets tedious, you can use a library like [Immer](https://github.com/immerjs/use-immer) to reduce repetitive code:
如果在代码中复制数组感觉乏味，可以使用 [Immer](https://github.com/immerjs/use-immer) 之类的库来减少重复代码：

<Sandpack>

```js
import { useState } from 'react';
import { useImmer } from 'use-immer';

const initialList = [
  { id: 0, title: 'Big Bellies', seen: false },
  { id: 1, title: 'Lunar Landscape', seen: false },
  { id: 2, title: 'Terracotta Army', seen: true },
];

export default function BucketList() {
  const [list, updateList] = useImmer(initialList);

  function handleToggle(artworkId, nextSeen) {
    updateList(draft => {
      const artwork = draft.find(a =>
        a.id === artworkId
      );
      artwork.seen = nextSeen;
    });
  }

  return (
    <>
      <h1>Art Bucket List</h1>
      <h2>My list of art to see:</h2>
      <ItemList
        artworks={list}
        onToggle={handleToggle} />
    </>
  );
}

function ItemList({ artworks, onToggle }) {
  return (
    <ul>
      {artworks.map(artwork => (
        <li key={artwork.id}>
          <label>
            <input
              type="checkbox"
              checked={artwork.seen}
              onChange={e => {
                onToggle(
                  artwork.id,
                  e.target.checked
                );
              }}
            />
            {artwork.title}
          </label>
        </li>
      ))}
    </ul>
  );
}
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

<LearnMore path="/learn/updating-arrays-in-state">

Read **[Updating Arrays in State](/learn/updating-arrays-in-state)** to learn how to update arrays correctly.
阅读 **[更新 state 中的数组](/learn/updating-arrays-in-state)** 了解如何正确地更新数组。

</LearnMore>

## 下节预告 | What's next? {/*whats-next*/}

Head over to [Responding to Events](/learn/responding-to-events) to start reading this chapter page by page!
前往 [响应事件](/learn/responding-to-events) 开始逐页阅读本章！

Or, if you're already familiar with these topics, why not read about [Managing State](/learn/managing-state)?
或者，如果你已经熟悉这些主题，不妨看看 [状态管理](/learn/managing-state)
