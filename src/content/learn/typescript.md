---
title: 使用 TypeScript | Using TypeScript
re: https://github.com/reactjs/react.dev/issues/5960
---

<Intro>

TypeScript is a popular way to add type definitions to JavaScript codebases. Out of the box, TypeScript [supports JSX](/learn/writing-markup-with-jsx) and you can get full React Web support by adding [`@types/react`](https://www.npmjs.com/package/@types/react) and [`@types/react-dom`](https://www.npmjs.com/package/@types/react-dom) to your project.
TypeScript 是一种向 JavaScript 代码添加类型定义的常用方法。TypeScript 天然支持 JSX——只需在项目中添加 [`@types/react`](https://www.npmjs.com/package/@types/react) 和 [`@types/react-dom`](https://www.npmjs.com/package/@types/react-dom) 即可获得完整的 React Web 支持。

</Intro>

<YouWillLearn>

* [TypeScript with React Components](/learn/typescript#typescript-with-react-components)
* [在 React 组件中使用 TypeScript](/learn/typescript#typescript-with-react-components)
* [Examples of typing with Hooks](/learn/typescript#example-hooks)
* [带有 Hook 的类型示例](/learn/typescript#example-hooks)
* [Common types from `@types/react`](/learn/typescript/#useful-types)
* [来自 `@types/react` 的常见类型](/learn/typescript/#useful-types)
* [Further learning locations](/learn/typescript/#further-learning)
* [更多学习资源](/learn/typescript/#further-learning)

</YouWillLearn>

## 安装 | Installation {/*installation*/}

All [production-grade React frameworks](/learn/start-a-new-react-project#production-grade-react-frameworks) offer support for using TypeScript. Follow the framework specific guide for installation:

所有的 [生产级 React 框架](/learn/start-a-new-react-project#production-grade-react-framework) 都支持使用 TypeScript。请按照框架特定的指南进行安装：

- [Next.js](https://nextjs.org/docs/app/building-your-application/configuring/typescript)
- [Remix](https://remix.run/docs/en/1.19.2/guides/typescript)
- [Gatsby](https://www.gatsbyjs.com/docs/how-to/custom-configuration/typescript/)
- [Expo](https://docs.expo.dev/guides/typescript/)

### 在现有 React 项目中添加 TypeScript | Adding TypeScript to an existing React project {/*adding-typescript-to-an-existing-react-project*/}

To install the latest version of React's type definitions:
使用下面命令安装最新版本的 React 类型定义：

<TerminalBlock>
npm install @types/react @types/react-dom
</TerminalBlock>

The following compiler options need to be set in your `tsconfig.json`:
然后在 `tsconfig.json` 中设置以下编译器选项：

1. `dom` must be included in [`lib`](https://www.typescriptlang.org/tsconfig/#lib) (Note: If no `lib` option is specified, `dom` is included by default).
- 必须在 [`lib`](https://www.typescriptlang.org/tsconfig/#lib) 中包含 `dom`（注意：如果没有指定 `lib` 选项，默认情况下会包含 `dom`）。
2. [`jsx`](https://www.typescriptlang.org/tsconfig/#jsx) must be set to one of the valid options. `preserve` should suffice for most applications.
  If you're publishing a library, consult the [`jsx` documentation](https://www.typescriptlang.org/tsconfig/#jsx) on what value to choose.
- [`jsx`](https://www.typescriptlang.org/tsconfig/#jsx) 必须设置为一个有效的选项。对于大多数应用程序，`preserve` 应该足够了。如果你正在发布一个库，请查阅 [`jsx` 文档](https://www.typescriptlang.org/tsconfig/#jsx) 以选择合适的值。

## 在 React 组件中使用 TypeScript | TypeScript with React Components {/*typescript-with-react-components*/}

<Note>

Every file containing JSX must use the `.tsx` file extension. This is a TypeScript-specific extension that tells TypeScript that this file contains JSX.
每个包含 JSX 的文件都必须使用 `.tsx` 文件扩展名。这是一个 TypeScript 特定的扩展，告诉 TypeScript 该文件包含 JSX。

</Note>

Writing TypeScript with React is very similar to writing JavaScript with React. The key difference when working with a component is that you can provide types for your component's props. These types can be used for correctness checking and providing inline documentation in editors.
使用 TypeScript 编写 React 与使用 JavaScript 编写 React 非常相似。与组件一起工作时的关键区别是，你可以为组件的 props 提供类型。这些类型可用于正确性检查，并在编辑器中提供内联文档。

Taking the [`MyButton` component](/learn#components) from the [Quick Start](/learn) guide, we can add a type describing the `title` for the button:
以 [快速入门](/learn) 指南中的 [`MyButton` 组件](/learn#components) 为例，我们可以为按钮的 `title` 添加一个描述类型：

<Sandpack>

```tsx src/App.tsx active
function MyButton({ title }: { title: string }) {
  return (
    <button>{title}</button>
  );
}

export default function MyApp() {
  return (
    <div>
      <h1>欢迎来到我的应用</h1>
      <MyButton title="我是一个按钮" />
    </div>
  );
}
```

```js src/App.js hidden
import AppTSX from "./App.tsx";
export default App = AppTSX;
```
</Sandpack>

 <Note>

These sandboxes can handle TypeScript code, but they do not run the type-checker. This means you can amend the TypeScript sandboxes to learn, but you won't get any type errors or warnings. To get type-checking, you can use the [TypeScript Playground](https://www.typescriptlang.org/play) or use a more fully-featured online sandbox.
这些沙盒可以处理 TypeScript 代码，但它们不运行类型检查器。这意味着你可以修改 TypeScript 沙盒以进行学习，但你不会收到任何类型错误或警告。要进行类型检查，你可以使用 [TypeScript Playground](https://www.typescriptlang.org/play) 或使用更完整的在线沙盒。

</Note>

This inline syntax is the simplest way to provide types for a component, though once you start to have a few fields to describe it can become unwieldy. Instead, you can use an `interface` or `type` to describe the component's props:
这种内联语法是为组件提供类型的最简单方法，但是一旦你开始描述几个字段，它可能变得难以管理。相反，你可以使用 `interface` 或 `type` 来描述组件的 props：

<Sandpack>

```tsx src/App.tsx active
interface MyButtonProps {
  /** The text to display inside the button */
  /** 按钮文字 */
  title: string;
  /** Whether the button can be interacted with */
  /** 按钮是否禁用 */
  disabled: boolean;
}

function MyButton({ title, disabled }: MyButtonProps) {
  return (
    <button disabled={disabled}>{title}</button>
  );
}

export default function MyApp() {
  return (
    <div>
      <h1>Welcome to my app</h1>
      <MyButton title="我是一个禁用按钮" disabled={true}/>
    </div>
  );
}
```

```js src/App.js hidden
import AppTSX from "./App.tsx";
export default App = AppTSX;
```

</Sandpack>

The type describing your component's props can be as simple or as complex as you need, though they should be an object type described with either a `type` or `interface`. You can learn about how TypeScript describes objects in [Object Types](https://www.typescriptlang.org/docs/handbook/2/objects.html) but you may also be interested in using [Union Types](https://www.typescriptlang.org/docs/handbook/2/everyday-types.html#union-types) to describe a prop that can be one of a few different types and the [Creating Types from Types](https://www.typescriptlang.org/docs/handbook/2/types-from-types.html) guide for more advanced use cases.
描述组件 props 的类型可以根据需要变得简单或复杂，但它们应该是使用 `type` 或 `interface` 描述的对象类型。你可以在 [对象类型](https://www.typescriptlang.org/docs/handbook/2/objects.html) 中了解 TypeScript 如何描述对象，但你可能还对使用 [联合类型](https://www.typescriptlang.org/docs/handbook/2/everyday-types.html#union-types) 描述可以是几种不同类型之一的 prop，以及在 [从类型创建类型](https://www.typescriptlang.org/docs/handbook/2/types-from-types.html) 指南中参考更高级的用例。


## Hooks 示例 | Example Hooks {/*example-hooks*/}

The type definitions from `@types/react` include types for the built-in Hooks, so you can use them in your components without any additional setup. They are built to take into account the code you write in your component, so you will get [inferred types](https://www.typescriptlang.org/docs/handbook/type-inference.html) a lot of the time and ideally do not need to handle the minutiae of providing the types. 

来自 `@types/react` 的类型定义包括内置的 Hook，因此你可以在组件中使用它们，无需任何额外设置。它们是根据你在组件中编写的代码构建的，所以你会得到很多 [类型推断](https://www.typescriptlang.org/docs/handbook/type-inference.html)，并且理想情况下不需要处理提供类型的细节。

However, we can look at a few examples of how to provide types for Hooks.
但是，我们可以看一下如何为 Hook 提供类型的几个示例。

### `useState` {/*typing-usestate*/}

The [`useState` Hook](/reference/react/useState) will re-use the value passed in as the initial state to determine what the type of the value should be. For example:
[`useState` Hook](/reference/react/useState) 会重用作为初始 state 传入的值以确定值的类型。例如：

```ts
// Infer the type as "boolean"
// 推断类型为 "boolean"
const [enabled, setEnabled] = useState(false);
```

This will assign the type of `boolean` to `enabled`, and `setEnabled` will be a function accepting either a `boolean` argument, or a function that returns a `boolean`. If you want to explicitly provide a type for the state, you can do so by providing a type argument to the `useState` call:
这将为 `enabled` 分配 `boolean` 类型，而 `setEnabled` 将是一个接受 `boolean` 参数的函数，或者返回 `boolean` 的函数。如果你想为 state 显式提供一个类型，你可以通过为 `useState` 调用提供一个类型参数来实现：

```ts
// Explicitly set the type to "boolean"
// 显式设置类型为 "boolean"
const [enabled, setEnabled] = useState<boolean>(false);
```

This isn't very useful in this case, but a common case where you may want to provide a type is when you have a union type. For example, `status` here can be one of a few different strings:
在这种情况下，这并不是很有用，但是当你有一个联合类型时，你可能想要提供一个 `type`。例如，这里的 `status` 可以是几个不同的字符串之一：

```ts
type Status = "idle" | "loading" | "success" | "error";

const [status, setStatus] = useState<Status>("idle");
```

Or, as recommended in [Principles for structuring state](/learn/choosing-the-state-structure#principles-for-structuring-state), you can group related state as an object and describe the different possibilities via object types:
或者，如 [选择 state 结构原则](/learn/choosing-the-state-structure#principles-for-structuring-state) 中推荐的，你可以将相关的 state 作为一个对象分组，并通过对象类型描述不同的可能性：

```ts
type RequestState =
  | { status: 'idle' }
  | { status: 'loading' }
  | { status: 'success', data: any }
  | { status: 'error', error: Error };

const [requestState, setRequestState] = useState<RequestState>({ status: 'idle' });
```

### `useReducer` {/*typing-usereducer*/}

The [`useReducer` Hook](/reference/react/useReducer) is a more complex Hook that takes a reducer function and an initial state. The types for the reducer function are inferred from the initial state. You can optionally provide a type argument to the `useReducer` call to provide a type for the state, but it is often better to set the type on the initial state instead:
[`useReducer`](/reference/react/useReducer) 是一个更复杂的 Hook，它接受一个 reducer 函数和一个初始 state 作为参数，并将从初始 state 推断出 reducer 函数的类型。你可以选择性地为 `useReducer` 提供类型参数以为 state 提供类型。但是更好的做法仍然是在初始 state 上添加类型：

<Sandpack>

```tsx src/App.tsx active
import {useReducer} from 'react';

interface State {
   count: number 
};

type CounterAction =
  | { type: "reset" }
  | { type: "setCount"; value: State["count"] }

const initialState: State = { count: 0 };

function stateReducer(state: State, action: CounterAction): State {
  switch (action.type) {
    case "reset":
      return initialState;
    case "setCount":
      return { ...state, count: action.value };
    default:
      throw new Error("Unknown action");
  }
}

export default function App() {
  const [state, dispatch] = useReducer(stateReducer, initialState);

  const addFive = () => dispatch({ type: "setCount", value: state.count + 5 });
  const reset = () => dispatch({ type: "reset" });

  return (
    <div>
      <h1>欢迎来到我的计数器</h1>

      <p>计数： {state.count}</p>
      <button onClick={addFive}>加 5</button>
      <button onClick={reset}>重置</button>
    </div>
  );
}

```

```js src/App.js hidden
import AppTSX from "./App.tsx";
export default App = AppTSX;
```

</Sandpack>


We are using TypeScript in a few key places:
我们在几个关键位置使用了 TypeScript：

 - `interface State` describes the shape of the reducer's state.
 - `interface State` 描述了 reducer state 的类型。
 - `type CounterAction` describes the different actions which can be dispatched to the reducer.
 - `type CounterAction` 描述了可以 dispatch 至 reducer 的不同 action。
 - `const initialState: State` provides a type for the initial state, and also the type which is used by `useReducer` by default.
 - `const initialState: State` 为初始 state 提供类型，并且也将成为 `useReducer` 默认使用的类型。
 - `stateReducer(state: State, action: CounterAction): State` sets the types for the reducer function's arguments and return value.
 - `stateReducer(state: State, action: CounterAction): State` 设置了 reducer 函数参数和返回值的类型。

A more explicit alternative to setting the type on `initialState` is to provide a type argument to `useReducer`:
除了在 `initialState` 上设置类型外，一个更明确的替代方法是为 `useReducer` 提供一个类型参数：

```ts
import { stateReducer, State } from './your-reducer-implementation';

const initialState = { count: 0 };

export default function App() {
  const [state, dispatch] = useReducer<State>(stateReducer, initialState);
}
```

### `useContext` {/*typing-usecontext*/}

The [`useContext` Hook](/reference/react/useContext) is a technique for passing data down the component tree without having to pass props through components. It is used by creating a provider component and often by creating a Hook to consume the value in a child component.
[`useContext`](/reference/react/useContext) 是一种无需通过组件传递 props 而可以直接在组件树中传递数据的技术。它是通过创建 provider 组件使用，通常还会创建一个 Hook 以在子组件中使用该值。

The type of the value provided by the context is inferred from the value passed to the `createContext` call:
从传递给 `createContext` 调用的值推断 context 提供的值的类型：

<Sandpack>

```tsx src/App.tsx active
import { createContext, useContext, useState } from 'react';

type Theme = "light" | "dark" | "system";
const ThemeContext = createContext<Theme>("system");

const useGetTheme = () => useContext(ThemeContext);

export default function MyApp() {
  const [theme, setTheme] = useState<Theme>('light');

  return (
    <ThemeContext.Provider value={theme}>
      <MyComponent />
    </ThemeContext.Provider>
  )
}

function MyComponent() {
  const theme = useGetTheme();

  return (
    <div>
      <p>当前主题：{theme}</p>
    </div>
  )
}
```

```js src/App.js hidden
import AppTSX from "./App.tsx";
export default App = AppTSX;
```

</Sandpack>

This technique works when you have a default value which makes sense - but there are occasionally cases when you do not, and in those cases `null` can feel reasonable as a default value. However, to allow the type-system to understand your code, you need to explicitly set `ContextShape | null` on the `createContext`. 
当你没有一个合理的默认值时，这种技术是有效的，而在这些情况下，`null` 作为默认值可能感觉是合理的。但是，为了让类型系统理解你的代码，你需要在 `createContext` 上显式设置 `ContextShape | null`。

This causes the issue that you need to eliminate the `| null` in the type for context consumers. Our recommendation is to have the Hook do a runtime check for it's existence and throw an error when not present:
这会导致一个问题，你需要在 context consumer 中消除 `| null` 的类型。我们建议让 Hook 在运行时检查它的存在，并在不存在时抛出一个错误：

```js {5, 16-20}
import { createContext, useContext, useState, useMemo } from 'react';

// This is a simpler example, but you can imagine a more complex object here
// 这是一个简单的示例，但你可以想象一个更复杂的对象
type ComplexObject = {
  kind: string
};

// The context is created with `| null` in the type, to accurately reflect the default value.
// 上下文在类型中创建为 `| null`，以准确反映默认值。
const Context = createContext<ComplexObject | null>(null);

// The context is created with `| null` in the type, to accurately reflect the default value.
// 这个 Hook 会在运行时检查 context 是否存在，并在不存在时抛出一个错误。
const useGetComplexObject = () => {
  const object = useContext(Context);
  if (!object) { throw new Error("useGetComplexObject must be used within a Provider") }
  return object;
}

export default function MyApp() {
  const object = useMemo(() => ({ kind: "complex" }), []);

  return (
    <Context.Provider value={object}>
      <MyComponent />
    </Context.Provider>
  )
}

function MyComponent() {
  const object = useGetComplexObject();

  return (
    <div>
      <p>Current object: {object.kind}</p>
    </div>
  )
}
```

### `useMemo` {/*typing-usememo*/}

the [`usememo`](/reference/react/usememo) hooks will create/re-access a memorized value from a function call, re-running the function only when dependencies passed as the 2nd parameter are changed. the result of calling the hook is inferred from the return value from the function in the first parameter. you can be more explicit by providing a type argument to the hook.
[`useMemo`](/reference/react/useMemo) 会从函数调用中创建/重新访问记忆化值，只有在第二个参数中传入的依赖项发生变化时，才会重新运行该函数。函数的类型是根据第一个参数中函数的返回值进行推断的，如果希望明确指定，可以为该 Hook 提供一个类型参数以指定函数类型。

```ts
// The type of visibleTodos is inferred from the return value of filterTodos
// 从 filterTodos 的返回值推断 visibleTodos 的类型
const visibleTodos = useMemo(() => filterTodos(todos, tab), [todos, tab]);
```


### `useCallback` {/*typing-usecallback*/}

The [`useCallback`](/reference/react/useCallback) provide a stable reference to a function as long as the dependencies passed into the second parameter are the same. Like `useMemo`, the function's type is inferred from the return value of the function in the first parameter, and you can be more explicit by providing a type argument to the Hook.
[`useCallback`](/reference/react/useCallback) 会在第二个参数中传入的依赖项保持不变的情况下，为函数提供相同的引用。与 `useMemo` 类似，函数的类型是根据第一个参数中函数的返回值进行推断的，如果希望明确指定，可以为这个 Hook 提供一个类型参数以指定函数类型。


```ts
const handleClick = useCallback(() => {
  // ...
}, [todos]);
```

When working in TypeScript strict mode `useCallback` requires adding types for the parameters in your callback. This is because the type of the callback is inferred from the return value of the function, and without parameters the type cannot be fully understood.
当在 TypeScript 严格模式下，使用 `useCallback` 需要为回调函数中的参数添加类型注解。这是因为回调函数的类型是根据函数的返回值进行推断的——如果没有参数，那么类型就不能完全理解。

Depending on your code-style preferences, you could use the `*EventHandler` functions from the React types to provide the type for the event handler at the same time as defining the callback: 
根据自身的代码风格偏好，你可以使用 React 类型中的 `*EventHandler` 函数以在定义回调函数的同时为事件处理程序提供类型注解：

```ts
import { useState, useCallback } from 'react';

export default function Form() {
  const [value, setValue] = useState("Change me");

  const handleChange = useCallback<React.ChangeEventHandler<HTMLInputElement>>((event) => {
    setValue(event.currentTarget.value);
  }, [setValue])
  
  return (
    <>
      <input value={value} onChange={handleChange} />
      <p>值： {value}</p>
    </>
  );
}
```

## 常用类型 | Useful Types {/*useful-types*/}

There is quite an expansive set of types which come from the `@types/react` package, it is worth a read when you feel comfortable with how React and TypeScript interact. You can find them [in React's folder in DefinitelyTyped](https://github.com/DefinitelyTyped/DefinitelyTyped/blob/master/types/react/index.d.ts). We will cover a few of the more common types here.
当逐渐适应 React 和 TypeScript 的搭配使用后, 可以尝试阅读 `@types/react`，此库提供了一整套类型。你可以在 [DefinitelyTyped 的 React 目录中](https://github.com/DefinitelyTyped/DefinitelyTyped/blob/master/types/react/index.d.ts) 找到它们。我们将在这里介绍一些更常见的类型。

### DOM 事件 | DOM Events {/*typing-dom-events*/}

When working with DOM events in React, the type of the event can often be inferred from the event handler. However, when you want to extract a function to be passed to an event handler, you will need to explicitly set the type of the event.
在 React 中处理 DOM 事件时，事件的类型通常可以从事件处理程序中推断出来。但是，当你想提取一个函数以传递给事件处理程序时，你需要明确设置事件的类型。

<Sandpack>

```tsx src/App.tsx active
import { useState } from 'react';

export default function Form() {
  const [value, setValue] = useState("Change me");

  function handleChange(event: React.ChangeEvent<HTMLInputElement>) {
    setValue(event.currentTarget.value);
  }

  return (
    <>
      <input value={value} onChange={handleChange} />
      <p>值： {value}</p>
    </>
  );
}
```

```js src/App.js hidden
import AppTSX from "./App.tsx";
export default App = AppTSX;
```

</Sandpack>

There are many types of events provided in the React types - the full list can be found [here](https://github.com/DefinitelyTyped/DefinitelyTyped/blob/b580df54c0819ec9df62b0835a315dd48b8594a9/types/react/index.d.ts#L1247C1-L1373) which is based on the [most popular events from the DOM](https://developer.mozilla.org/en-US/docs/Web/Events).
React 类型中提供了许多事件类型 —— 完整列表可以在 [这里](https://github.com/DefinitelyTyped/DefinitelyTyped/blob/b580df54c0819ec9df62b0835a315dd48b8594a9/types/react/index.d.ts#L1247C1-L1373) 查看，它基于 [DOM 的常用事件](https://developer.mozilla.org/zh-CN/docs/Web/Events)。

When determining the type you are looking for you can first look at the hover information for the event handler you are using, which will show the type of the event.
当你需要确定某个类型时，可以先将鼠标悬停在你使用的事件处理器上，这样可以查看到事件的具体类型。

If you need to use an event that is not included in this list, you can use the `React.SyntheticEvent` type, which is the base type for all events.
当你需要使用不包含在此列表中的事件时，你可以使用 `React.SyntheticEvent` 类型，这是所有事件的基类型。

### 子元素 | Children {/*typing-children*/}

There are two common paths to describing the children of a component. The first is to use the `React.ReactNode` type, which is a union of all the possible types that can be passed as children in JSX:
描述组件的子元素有两种常见方法。第一种是使用 `React.ReactNode` 类型，这是可以在 JSX 中作为子元素传递的所有可能类型的并集：

```ts
interface ModalRendererProps {
  title: string;
  children: React.ReactNode;
}
```

This is a very broad definition of children. The second is to use the `React.ReactElement` type, which is only JSX elements and not JavaScript primitives like strings or numbers:
这是对子元素的一个非常宽泛的定义。第二种方法是使用 `React.ReactElement` 类型，它只包括 JSX 元素，而不包括 JavaScript 原始类型，如 string 或 number：

```ts
interface ModalRendererProps {
  title: string;
  children: React.ReactElement;
}
```

Note, that you cannot use TypeScript to describe that the children are a certain type of JSX elements, so you cannot use the type-system to describe a component which only accepts `<li>` children. 
注意，你不能使用 TypeScript 来描述子元素是某种类型的 JSX 元素，所以你不能使用类型系统来描述一个只接受 `<li>` 子元素的组件。

You can see an example of both `React.ReactNode` and `React.ReactElement` with the type-checker in [this TypeScript playground](https://www.typescriptlang.org/play?#code/JYWwDg9gTgLgBAJQKYEMDG8BmUIjgIilQ3wChSB6CxYmAOmXRgDkIATJOdNJMGAZzgwAFpxAR+8YADswAVwGkZMJFEzpOjDKw4AFHGEEBvUnDhphwADZsi0gFw0mDWjqQBuUgF9yaCNMlENzgAXjgACjADfkctFnYkfQhDAEpQgD44AB42YAA3dKMo5P46C2tbJGkvLIpcgt9-QLi3AEEwMFCItJDMrPTTbIQ3dKywdIB5aU4kKyQQKpha8drhhIGzLLWODbNs3b3s8YAxKBQAcwXpAThMaGWDvbH0gFloGbmrgQfBzYpd1YjQZbEYARkB6zMwO2SHSAAlZlYIBCdtCRkZpHIrFYahQYQD8UYYFA5EhcfjyGYqHAXnJAsIUHlOOUbHYhMIIHJzsI0Qk4P9SLUBuRqXEXEwAKKfRZcNA8PiCfxWACecAAUgBlAAacFm80W-CU11U6h4TgwUv11yShjgJjMLMqDnN9Dilq+nh8pD8AXgCHdMrCkWisVoAet0R6fXqhWKhjKllZVVxMcavpd4Zg7U6Qaj+2hmdG4zeRF10uu-Aeq0LBfLMEe-V+T2L7zLVu+FBWLdLeq+lc7DYFf39deFVOotMCACNOCh1dq219a+30uC8YWoZsRyuEdjkevR8uvoVMdjyTWt4WiSSydXD4NqZP4AymeZE072ZzuUeZQKheQgA).
你可以在这个 [TypeScript playground](https://www.typescriptlang.org/play?#code/JYWwDg9gTgLgBAJQKYEMDG8BmUIjgIilQ3wChSB6CxYmAOmXRgDkIATJOdNJMGAZzgwAFpxAR+8YADswAVwGkZMJFEzpOjDKw4AFHGEEBvUnDhphwADZsi0gFw0mDWjqQBuUgF9yaCNMlENzgAXjgACjADfkctFnYkfQhDAEpQgD44AB42YAA3dKMo5P46C2tbJGkvLIpcgt9-QLi3AEEwMFCItJDMrPTTbIQ3dKywdIB5aU4kKyQQKpha8drhhIGzLLWODbNs3b3s8YAxKBQAcwXpAThMaGWDvbH0gFloGbmrgQfBzYpd1YjQZbEYARkB6zMwO2SHSAAlZlYIBCdtCRkZpHIrFYahQYQD8UYYFA5EhcfjyGYqHAXnJAsIUHlOOUbHYhMIIHJzsI0Qk4P9SLUBuRqXEXEwAKKfRZcNA8PiCfxWACecAAUgBlAAacFm80W-CU11U6h4TgwUv11yShjgJjMLMqDnN9Dilq+nh8pD8AXgCHdMrCkWisVoAet0R6fXqhWKhjKllZVVxMcavpd4Zg7U6Qaj+2hmdG4zeRF10uu-Aeq0LBfLMEe-V+T2L7zLVu+FBWLdLeq+lc7DYFf39deFVOotMCACNOCh1dq219a+30uC8YWoZsRyuEdjkevR8uvoVMdjyTWt4WiSSydXD4NqZP4AymeZE072ZzuUeZQKheQgA) 中查看 `React.ReactNode` 和 `React.ReactElement` 的示例，并使用类型检查器进行验证。

### 样式属性 | Style Props {/*typing-style-props*/}

When using inline styles in React, you can use `React.CSSProperties` to describe the object passed to the `style` prop. This type is a union of all the possible CSS properties, and is a good way to ensure you are passing valid CSS properties to the `style` prop, and to get auto-complete in your editor.
当在 React 中使用内联样式时，你可以使用 `React.CSSProperties` 来描述传递给 `style` 属性的对象。这个类型是所有可能的 CSS 属性的并集，它能确保你传递给 `style` 属性的是有效的 CSS 属性，并且你能在编辑器中获得样式编码提示。

```ts
interface MyComponentProps {
  style: React.CSSProperties;
}
```

## 更多学习资源 | Further learning {/*further-learning*/}

This guide has covered the basics of using TypeScript with React, but there is a lot more to learn.
Individual API pages on the docs may contain more in-depth documentation on how to use them with TypeScript.
本指南已经介绍了如何在 React 中使用 TypeScript 的基础知识，但还有更多内容等待学习。官网中的单个 API 页面或许包含了如何与 TypeScript 一起使用它们的更深入的说明。
文档中的各个 API 页面可能会包含更深入的说明，介绍如何在 TypeScript 中使用它们。

We recommend the following resources:
我们推荐以下资源：

 - [The TypeScript handbook](https://www.typescriptlang.org/docs/handbook/) is the official documentation for TypeScript, and covers most key language features.
 - [TypeScript 官方文档](https://www.typescriptlang.org/docs/handbook/) 涵盖了大多数关键的语言特性。

 - [The TypeScript release notes](https://devblogs.microsoft.com/typescript/) cover new features in depth.
 - [TypeScript 发布日志](https://devblogs.microsoft.com/typescript/) 深入介绍了每一个新特性。

 - [React TypeScript Cheatsheet](https://react-typescript-cheatsheet.netlify.app/) is a community-maintained cheatsheet for using TypeScript with React, covering a lot of useful edge cases and providing more breadth than this document.
 - [React TypeScript Cheatsheet](https://react-typescript-cheatsheet.netlify.app/) 是一个社区维护的，用于在 React 中使用 TypeScript 的速查表，涵盖了许多有用的边界情况，并提供了比本文更广泛全面的内容。

 - [TypeScript Community Discord](https://discord.com/invite/typescript) is a great place to ask questions and get help with TypeScript and React issues.
 - [TypeScript Community Discord](https://discord.com/invite/typescript) 是一个提问并获得有关 TypeScript 和 React issues 帮助的好地方。
