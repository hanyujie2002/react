---
title: React Compiler
---

<Intro>
This page will give you an introduction to React Compiler and how to try it out successfully.
本页面将为你介绍新的 React Compiler，以及如何成功试用。
</Intro>

<Wip>
This page will give you an introduction to React Compiler and how to try it out successfully.
这些文档仍在不断完善中。更多文档可在 [React Compiler 工作组代码库](https://github.com/reactwg/react-compiler/discussions) 中找到，并在这些文档更加稳定时被整合进来。
</Wip>

<YouWillLearn>

* Getting started with the compiler
* 开始使用 React Compiler
* Installing the compiler and ESLint plugin
* 安装 React Compiler 和 ESLint 插件
* Troubleshooting
* 疑难解答

</YouWillLearn>

<Note>
React Compiler is a new compiler currently in Beta, that we've open sourced to get early feedback from the community. While it has been used in production at companies like Meta, rolling out the compiler to production for your app will depend on the health of your codebase and how well you’ve followed the [Rules of React](/reference/rules).
React Compiler 是一个处于 Beta 阶段的新的编译器，我们将其开源以获取社区的早期反馈。虽然Meta 等公司已经在生产中使用它，但是能否在你的应用程序中使用它取决于代码库的健康状态以及你遵守 [React 规则](/reference/rules) 的程度。

The latest Beta release can be found with the `@beta` tag, and daily experimental releases with `@experimental`.
最新的 Beta 版本发布于 `@beta` 标签，每日实验版本发布于 `@experimental` 标签。
</Note>

React Compiler is a new compiler that we've open sourced to get early feedback from the community. It is a build-time only tool that automatically optimizes your React app. It works with plain JavaScript, and understands the [Rules of React](/reference/rules), so you don't need to rewrite any code to use it.
React Compiler 是一个新编译器，我们将其开源以获取社区的早期反馈。它是一个仅在构建时使用的工具，可以自动优化你的 React 应用程序。它可以与纯 JavaScript 一起使用，并且了解 [React 规则](/reference/rules)，因此你无需重写任何代码即可使用它。

The compiler also includes an [ESLint plugin](#installing-eslint-plugin-react-compiler) that surfaces the analysis from the compiler right in your editor. **We strongly recommend everyone use the linter today.** The linter does not require that you have the compiler installed, so you can use it even if you are not ready to try out the compiler.
编译器还包括一个 [ESLint 插件](#installing-eslint-plugin-react-compiler)，可以在你的编辑器中直接显示编译器的分析结果。**我们强烈建议大家使用 linter。** 不过 linter 并不需要安装编译器，因此即使你还没有准备好尝试编译器也可以使用它。

The compiler is currently released as `beta`, and is available to try out on React 17+ apps and libraries. To install the Beta:
编译器目前处于 `beta` 阶段，并且可以在 React 17+ 应用程序和库上使用。安装方式如下：

<TerminalBlock>
npm install -D babel-plugin-react-compiler@beta eslint-plugin-react-compiler@beta
</TerminalBlock>

Or, if you're using Yarn:
或者使用 Yarn：

<TerminalBlock>
yarn add -D babel-plugin-react-compiler@beta eslint-plugin-react-compiler@beta
</TerminalBlock>

If you are not using React 19 yet, please see [the section below](#using-react-compiler-with-react-17-or-18) for further instructions.
如果你还没有使用 React 19，请参考 [此内容](#using-react-compiler-with-react-17-or-18) 以获得进一步说明。

### 编译器是做什么的？| What does the compiler do? {/*what-does-the-compiler-do*/}

In order to optimize applications, React Compiler automatically memoizes your code. You may be familiar today with memoization through APIs such as `useMemo`, `useCallback`, and `React.memo`. With these APIs you can tell React that certain parts of your application don't need to recompute if their inputs haven't changed, reducing work on updates. While powerful, it's easy to forget to apply memoization or apply them incorrectly. This can lead to inefficient updates as React has to check parts of your UI that don't have any _meaningful_ changes.
为了优化应用程序，React Compiler 会自动对你的代码进行记忆化处理。你可能已经熟悉了像 `useMemo`、`useCallback` 和 `React.memo` 这样的 API，通过这些 API，你可以告诉 React，如果输入没有发生变化，应用程序的某些部分不需要重新计算，从而减少更新时的工作量。虽然这些功能很强大，但很容易忘记应用记忆化或者错误地应用它们。这可能会导致更新效率低下，因为 React 必须检查 UI 中没有任何**有意义**的更改的部分。

The compiler uses its knowledge of JavaScript and React's rules to automatically memoize values or groups of values within your components and hooks. If it detects breakages of the rules, it will automatically skip over just those components or hooks, and continue safely compiling other code.
编译器利用其对 JavaScript 和 React 规则的了解，自动对组件和钩子中的值或值组进行记忆化。如果它检测到规则的破坏，它将自动跳过那些组件或钩子，并继续安全地编译其他代码。

<Note>
React Compiler can statically detect when Rules of React are broken, and safely opt-out of optimizing just the affected components or hooks. It is not necessary for the compiler to optimize 100% of your codebase.
React Compiler 可以静态检测 React 规则何时被破坏，并安全地选择不优化受影响的组件或钩子。编译器没有必要对代码库进行 100% 的优化。
</Note>

If your codebase is already very well-memoized, you might not expect to see major performance improvements with the compiler. However, in practice memoizing the correct dependencies that cause performance issues is tricky to get right by hand.
如果你的代码库已经非常好地进行了记忆化处理，你可能不会指望通过编译器看到主要的性能改进。然而，在实践中，手动正确记忆化导致性能问题的依赖关系是很棘手的。

<DeepDive>
#### React Compiler 添加了什么样的记忆？| What kind of memoization does React Compiler add? {/*what-kind-of-memoization-does-react-compiler-add*/}

The initial release of React Compiler is primarily focused on **improving update performance** (re-rendering existing components), so it focuses on these two use cases:
React Compiler 的初始版本主要专注于**改善更新性能**（重新渲染现有组件），因此它专注于以下两种用例：

1. **Skipping cascading re-rendering of components**
- **跳过组件的级联重新渲染**
    * Re-rendering `<Parent />` causes many components in its component tree to re-render, even though only `<Parent />` has changed
    * 重新渲染 `<Parent />` 会导致其组件树中的许多组件重新渲染，即使只有 `<Parent />` 发生了变化
2. **Skipping expensive calculations from outside of React**
- **从 React 外部跳过昂贵计算**
    * For example, calling `expensivelyProcessAReallyLargeArrayOfObjects()` inside of your component or hook that needs that data
    * 例如，在需要该数据的组件或钩子内部调用 `expensivelyProcessAReallyLargeArrayOfObjects()`

#### 优化重新渲染 | Optimizing Re-renders {/*optimizing-re-renders*/}

React lets you express your UI as a function of their current state (more concretely: their props, state, and context). In its current implementation, when a component's state changes, React will re-render that component _and all of its children_ — unless you have applied some form of manual memoization with `useMemo()`, `useCallback()`, or `React.memo()`. For example, in the following example, `<MessageButton>` will re-render whenever `<FriendList>`'s state changes:
React 允许你将你的 UI 表达为它们当前状态的函数（更具体地说：它们的属性、状态和上下文）。在当前的实现中，当组件的状态发生变化时，React 将重新渲染该组件及其**所有子组件**——除非你使用 `useMemo()`、`useCallback()` 或 `React.memo()` 应用了某种形式的手动记忆。例如，在以下示例中，每当 `<FriendList>` 的状态发生变化时，`<MessageButton>` 将重新呈现：

```javascript
function FriendList({ friends }) {
  const onlineCount = useFriendOnlineCount();
  if (friends.length === 0) {
    return <NoFriends />;
  }
  return (
    <div>
      <span>{onlineCount} online</span>
      {friends.map((friend) => (
        <FriendListCard key={friend.id} friend={friend} />
      ))}
      <MessageButton />
    </div>
  );
}
```
[**在 React Compiler Playground 中查看此示例** | _See this example in the React Compiler Playground_](https://playground.react.dev/#N4Igzg9grgTgxgUxALhAMygOzgFwJYSYAEAYjHgpgCYAyeYOAFMEWuZVWEQL4CURwADrEicQgyKEANnkwIAwtEw4iAXiJQwCMhWoB5TDLmKsTXgG5hRInjRFGbXZwB0UygHMcACzWr1ABn4hEWsYBBxYYgAeADkIHQ4uAHoAPksRbisiMIiYYkYs6yiqPAA3FMLrIiiwAAcAQ0wU4GlZBSUcbklDNqikusaKkKrgR0TnAFt62sYHdmp+VRT7SqrqhOo6Bnl6mCoiAGsEAE9VUfmqZzwqLrHqM7ubolTVol5eTOGigFkEMDB6u4EAAhKA4HCEZ5DNZ9ErlLIWYTcEDcIA)

React Compiler automatically applies the equivalent of manual memoization, ensuring that only the relevant parts of an app re-render as state changes, which is sometimes referred to as "fine-grained reactivity". In the above example, React Compiler determines that the return value of `<FriendListCard />` can be reused even as `friends` changes, and can avoid recreating this JSX _and_ avoid re-rendering `<MessageButton>` as the count changes.
React Compiler 会自动应用等效的手动记忆，确保只有应用的相关部分在状态发生变化时重新渲染，这有时被称为“细粒度反应”。在上面的例子中，React Compiler 确定 `<FriendListCard />` 的返回值即使在 `friends` 发生变化时也可以重用，并且可以避免重新创建此 JSX，**并**避免在 `onlineCount` 变化时重新渲染 `<MessageButton>`。

#### 昂贵计算也会被记忆 | Expensive calculations also get memoized {/*expensive-calculations-also-get-memoized*/}

The compiler can also automatically memoize for expensive calculations used during rendering:
编译器还可以自动记忆渲染过程中使用的昂贵计算：

```js
// **Not** memoized by React Compiler, since this is not a component or hook
// 由于这不是组件或钩子，React Compiler 不会进行记忆
function expensivelyProcessAReallyLargeArrayOfObjects() { /* ... */ }

// Memoized by React Compiler since this is a component
// 由 React Compiler 进行了记忆化，因为这是一个组件
function TableContainer({ items }) {
  // This function call would be memoized:
  // 这个函数调用将被记忆：
  const data = expensivelyProcessAReallyLargeArrayOfObjects(items);
  // ...
}
```
[**在 React Compiler Playground 中查看此示例** | _See this example in the React Compiler Playground_](https://playground.react.dev/#N4Igzg9grgTgxgUxALhAejQAgFTYHIQAuumAtgqRAJYBeCAJpgEYCemASggIZyGYDCEUgAcqAGwQwANJjBUAdokyEAFlTCZ1meUUxdMcIcIjyE8vhBiYVECAGsAOvIBmURYSonMCAB7CzcgBuCGIsAAowEIhgYACCnFxioQAyXDAA5gixMDBcLADyzvlMAFYIvGAAFACUmMCYaNiYAHStOFgAvk5OGJgAshTUdIysHNy8AkbikrIKSqpaWvqGIiZmhE6u7p7ymAAqXEwSguZcCpKV9VSEFBodtcBOmAYmYHz0XIT6ALzefgFUYKhCJRBAxeLcJIsVIZLI5PKFYplCqVa63aoAbm6u0wMAQhFguwAPPRAQA+YAfL4dIloUmBMlODogDpAA)

However, if `expensivelyProcessAReallyLargeArrayOfObjects` is truly an expensive function, you may want to consider implementing its own memoization outside of React, because:

但是，如果 `expensivelyProcessAReallyLargeArrayOfObjects` 确实是一个昂贵的函数，你可能需要考虑在 React 之外实现它自己的记忆，因为：

- React Compiler only memoizes React components and hooks, not every function
- React Compiler 只记住 React 组件和钩子，而不是每个函数
- React Compiler's memoization is not shared across multiple components or hooks
- React Compiler 的记忆不会在多个组件或钩子之间共享

So if `expensivelyProcessAReallyLargeArrayOfObjects` was used in many different components, even if the same exact items were passed down, that expensive calculation would be run repeatedly. We recommend [profiling](https://react.dev/reference/react/useMemo#how-to-tell-if-a-calculation-is-expensive) first to see if it really is that expensive before making code more complicated.
因此，如果在许多不同的组件中使用 `expensivelyProcessAReallyLargeArrayOfObjects`，即使传递相同的 `items`，那昂贵的计算也会被重复运行。我们建议先进行 [性能分析](/reference/react/useMemo#how-to-tell-if-a-calculation-is-expensive)，看看是否真的那么昂贵，然后再使代码更加复杂。
</DeepDive>

### 我应该尝试一下编译器吗？| Should I try out the compiler? {/*should-i-try-out-the-compiler*/}

Please note that the compiler is still in Beta and has many rough edges. While it has been used in production at companies like Meta, rolling out the compiler to production for your app will depend on the health of your codebase and how well you've followed the [Rules of React](/reference/rules).
请注意，编译器仍处于 Beta 阶段，存在许多不完善之处。虽然它已经在 Meta 等公司的生产环境中使用过，但将编译器应用于你的应用程序生产环境将取决于你的代码库的健康状况以及你是否遵循了 [React 的规则](/reference/rules)。

**You don't have to rush into using the compiler now. It's okay to wait until it reaches a stable release before adopting it.** However, we do appreciate trying it out in small experiments in your apps so that you can [provide feedback](#reporting-issues) to us to help make the compiler better.
**你现在不必急着使用编译器。在采用它之前等到它达到稳定版本是可以的。** 然而，我们确实赞赏在你的应用程序中进行小型实验，以便你可以向我们 [提供反馈](#reporting-issues)，帮助使编译器更好。

## 开始 | Getting Started {/*getting-started*/}

In addition to these docs, we recommend checking the [React Compiler Working Group](https://github.com/reactwg/react-compiler) for additional information and discussion about the compiler.
除了这些文档之外，我们还建议查看 [React Compiler 工作组](https://github.com/reactwg/react-compiler)，以获取有关编译器的更多信息和讨论。

### 安装 eslint-plugin-react-compiler | Installing eslint-plugin-react-compiler {/*installing-eslint-plugin-react-compiler*/}

React Compiler also powers an ESLint plugin. The ESLint plugin can be used **independently** of the compiler, meaning you can use the ESLint plugin even if you don't use the compiler.
React Compiler 还为 ESLint 插件提供支持。ESLint 插件可以**独立**于编译器使用，这意味着即使你不使用编译器，也可以使用 ESLint 插件。

<TerminalBlock>
npm install -D eslint-plugin-react-compiler@beta
</TerminalBlock>

Then, add it to your ESLint config:
然后，将其添加到你的 ESLint 配置中：

```js
import reactCompiler from 'eslint-plugin-react-compiler'

export default [
  {
    plugins: {
      'react-compiler': reactCompiler,
    },
    rules: {
      'react-compiler/react-compiler': 'error',
    },
  },
]
```

Or, in the deprecated eslintrc config format:
或者使用已弃用的 eslintrc 配置格式：

```js
module.exports = {
  plugins: [
    'eslint-plugin-react-compiler',
  ],
  rules: {
    'react-compiler/react-compiler': 'error',
  },
}
```

The ESLint plugin will display any violations of the rules of React in your editor. When it does this, it means that the compiler has skipped over optimizing that component or hook. This is perfectly okay, and the compiler can recover and continue optimizing other components in your codebase.
ESLint 插件将在编辑器中显示任何违反 React 规则的行为。当它这样做时，这意味着编译器跳过了优化该组件或钩子。这是完全可以的，编译器可以恢复并继续优化代码库中的其他组件。

<Note>
**You don't have to fix all ESLint violations straight away.** You can address them at your own pace to increase the amount of components and hooks being optimized, but it is not required to fix everything before you can use the compiler.
**你不必立即修复所有的违反 ESLint 规则的代码。** 你可以按照自己的节奏来处理它们，以增加被优化的组件和钩子的数量，但在你可以使用编译器之前并不需要修复所有问题。
</Note>

### 将编译器应用到你的代码库 | Rolling out the compiler to your codebase {/*using-the-compiler-effectively*/}

#### 现有项目 | Existing projects {/*existing-projects*/}
The compiler is designed to compile functional components and hooks that follow the [Rules of React](/reference/rules). It can also handle code that breaks those rules by bailing out (skipping over) those components or hooks. However, due to the flexible nature of JavaScript, the compiler cannot catch every possible violation and may compile with false negatives: that is, the compiler may accidentally compile a component/hook that breaks the Rules of React which can lead to undefined behavior.
编译器旨在编译遵循 React 规则的功能组件和钩子。它还可以处理违反这些规则的代码，通过跳过这些组件或钩子来终止执行。然而，由于JavaScript的灵活性，编译器无法捕捉到每一个可能的违规行为，可能会出现错误的负面编译：也就是说，编译器可能会意外地编译出一个违反 [React 规则](/reference/rules) 的组件或钩子，这可能导致未定义的行为。

For this reason, to adopt the compiler successfully on existing projects, we recommend running it on a small directory in your product code first. You can do this by configuring the compiler to only run on a specific set of directories:
因此，要在现有项目中成功采用编译器，我们建议你先在项目代码中的一个小目录中运行它。你可以通过将编译器配置为仅在一组特定的目录上运行来执行此操作：

```js {3}
const ReactCompilerConfig = {
  sources: (filename) => {
    return filename.indexOf('src/path/to/dir') !== -1;
  },
};
```

When you have more confidence with rolling out the compiler, you can expand coverage to other directories as well and slowly roll it out to your whole app.
当你对编译器更有信心时，你也可以将覆盖范围扩展到其他目录，并逐渐将其推出到整个应用程序。

#### 新项目 | New projects {/*new-projects*/}

If you're starting a new project, you can enable the compiler on your entire codebase, which is the default behavior.
如果你正在启动一个新项目，你可以在整个代码库上启用编译器，这是默认行为。

### 在 React 17 或 18 中使用 React Compiler | Using React Compiler with React 17 or 18 {/*using-react-compiler-with-react-17-or-18*/}

React Compiler works best with React 19 RC. If you are unable to upgrade, you can install the extra `react-compiler-runtime` package which will allow the compiled code to run on versions prior to 19. However, note that the minimum supported version is 17.
React Compiler 与 React 19 RC 配合使用效果最佳。如果你无法升级，可以安装额外的 `react-compiler-runtime` 包来编译代码并在 19 之前的版本上运行。 但请注意，支持的最低版本是 17。

<TerminalBlock>
npm install react-compiler-runtime@beta
</TerminalBlock>

You should also add the correct `target` to your compiler config, where `target` is the major version of React you are targeting:
你还应该在编译器配置中添加正确的 `target`，值为你所使用的 React 大版本。

```js {3}
// babel.config.js
const ReactCompilerConfig = {
  target: '18' // '17' | '18' | '19'
};

module.exports = function () {
  return {
    plugins: [
      ['babel-plugin-react-compiler', ReactCompilerConfig],
    ],
  };
};
```

### 在库中使用 Compiler | Using the compiler on libraries {/*using-the-compiler-on-libraries*/}

React Compiler can also be used to compile libraries. Because React Compiler needs to run on the original source code prior to any code transformations, it is not possible for an application's build pipeline to compile the libraries they use. Hence, our recommendation is for library maintainers to independently compile and test their libraries with the compiler, and ship compiled code to npm.
React Compiler 还可用于编译库。由于 React Compiler 需要在代码转换之前的源码上运行，因此应用程序无法使用 pipeline 来编译所使用的库。因此我们建议库维护人员使用编译器独立编译和测试他们的库，并将编译后的代码发布到 npm。

Because your code is pre-compiled, users of your library will not need to have the compiler enabled in order to benefit from the automatic memoization applied to your library. If your library targets apps not yet on React 19, specify a minimum [`target` and add `react-compiler-runtime` as a direct dependency](#using-react-compiler-with-react-17-or-18). The runtime package will use the correct implementation of APIs depending on the application's version, and polyfill the missing APIs if necessary.
由于库的代码是预编译的，因此用户无需启用 Compiler 即可从编译器的自动记忆化中受益。如果库的 target 不是 React 19，请指定一个最小的 [`target` 并且将 `react-compiler-runtime` 添加为直接依赖](#using-react-compiler-with-react-17-or-18)。这个运行时包将根据应用程序的版本使用正确的 API 实现，并在必要时填充缺失的 API。

Library code can often require more complex patterns and usage of escape hatches. For this reason, we recommend ensuring that you have sufficient testing in order to identify any issues that might arise from using the compiler on your library. If you identify any issues, you can always opt-out the specific components or hooks with the [`'use no memo'` directive](#something-is-not-working-after-compilation).
库代码通常需要更复杂的模式和脱围机制。因此我们建议你进行足够的测试，以便发现在在库中使用编译器时可能出现的任何问题。对于任何发现的问题都可以使用 [`'use no memo'` 指令](#something-is-not-working-after-compilation) 来选择退出对特定组件或 Hook 的自动记忆化。

Similarly to apps, it is not necessary to fully compile 100% of your components or hooks to see benefits in your library. A good starting point might be to identify the most performance sensitive parts of your library and ensuring that they don't break the [Rules of React](/reference/rules), which you can use `eslint-plugin-react-compiler` to identify.
与应用程序类似，无需 100% 编译组件或 Hook 就可以看到编译器带来的好处。一个好的起点可能是确定库中对性能最敏感的部分，并确保它们没有违反 [React 规则](/reference/rules)，你可以通过使用 `eslint-plugin-react-compiler` 来完成。

## 用法 | Usage {/*installation*/}

### Babel {/*usage-with-babel*/}

<TerminalBlock>
npm install babel-plugin-react-compiler@beta
</TerminalBlock>

The compiler includes a Babel plugin which you can use in your build pipeline to run the compiler.
编译器包含一个 Babel 插件，你可以在构建流水线中使用它来运行编译器。

After installing, add it to your Babel config. Please note that it's critical that the compiler run **first** in the pipeline:
安装后，请将其添加到你的 Babel 配置中。请注意，编译器必须**首先**在流水线中运行。

```js {7}
// babel.config.js
const ReactCompilerConfig = { /* ... */ };

module.exports = function () {
  return {
    plugins: [
      ['babel-plugin-react-compiler', ReactCompilerConfig], // 必须首先运行！
      // ...
    ],
  };
};
```

`babel-plugin-react-compiler` should run first before other Babel plugins as the compiler requires the input source information for sound analysis.
`babel-plugin-react-compiler` 应该在其他 Babel 插件之前运行，因为编译器需要输入源信息进行声音分析。

### Vite {/*usage-with-vite*/}

If you use Vite, you can add the plugin to vite-plugin-react:
如果你使用 Vite，你可以将插件添加到 vite-plugin-react 中：

```js {10}
// vite.config.js
const ReactCompilerConfig = { /* ... */ };

export default defineConfig(() => {
  return {
    plugins: [
      react({
        babel: {
          plugins: [
            ["babel-plugin-react-compiler", ReactCompilerConfig],
          ],
        },
      }),
    ],
    // ...
  };
});
```

### Next.js {/*usage-with-nextjs*/}

Please refer to the [Next.js docs](https://nextjs.org/docs/canary/app/api-reference/next-config-js/reactCompiler) for more information.
请参考 [Next.js 文档](https://nextjs.org/docs/canary/app/api-reference/next-config-js/reactCompiler) 来了解更多信息。

### Remix {/*usage-with-remix*/}
Install `vite-plugin-babel`, and add the compiler's Babel plugin to it:
安装 `vite-plugin-babel`, 并将编译器的 Babel 插件添加到其中：

<TerminalBlock>
npm install vite-plugin-babel
</TerminalBlock>

```js {2,14}
// vite.config.js
import babel from "vite-plugin-babel";

const ReactCompilerConfig = { /* ... */ };

export default defineConfig({
  plugins: [
    remix({ /* ... */}),
    babel({
      filter: /\.[jt]sx?$/,
      babelConfig: {
        presets: ["@babel/preset-typescript"], // 如果你使用 TypeScript
        plugins: [
          ["babel-plugin-react-compiler", ReactCompilerConfig],
        ],
      },
    }),
  ],
});
```

### Webpack {/*usage-with-webpack*/}

A community Webpack loader is [now available here](https://github.com/SukkaW/react-compiler-webpack).
由社区提供的 Webpack loader 可以 [在这里找到](https://github.com/SukkaW/react-compiler-webpack)。

### Expo {/*usage-with-expo*/}

Please refer to [Expo's docs](https://docs.expo.dev/preview/react-compiler/) to enable and use the React Compiler in Expo apps.
请参考 [Expo 文档](https://docs.expo.dev/preview/react-compiler/) 应用程序中启用和使用 React Compiler。

### Metro (React Native) {/*usage-with-react-native-metro*/}

React Native uses Babel via Metro, so refer to the [Usage with Babel](#usage-with-babel) section for installation instructions.
React Native 通过 Metro 使用 Babel，因此请参考 [使用 Babel](#usage-with-babel) 部分的安装说明。

### Rspack {/*usage-with-rspack*/}

Please refer to [Rspack's docs](https://rspack.dev/guide/tech/react#react-compiler) to enable and use the React Compiler in Rspack apps.
请参考 [Rspack 文档](https://rspack.dev/guide/tech/react#react-compiler) 以启用并在 Rspack 应用程序中使用 React Compiler。

### Rsbuild {/*usage-with-rsbuild*/}

Please refer to [Rsbuild's docs](https://rsbuild.dev/guide/framework/react#react-compiler) to enable and use the React Compiler in Rsbuild apps.
请参考 [Rsbuild 文档](https://rsbuild.dev/guide/framework/react#react-compiler) 以在 Rsbuild 应用程序中启用和使用 React Compiler。 

## 疑难解答 | Troubleshooting {/*troubleshooting*/}

To report issues, please first create a minimal repro on the [React Compiler Playground](https://playground.react.dev/) and include it in your bug report. You can open issues in the [facebook/react](https://github.com/facebook/react/issues) repo.
请先在 [React Compiler Playground](https://playground.react.dev/) 上创建一个最小的可复现问题，并将其包含在你的错误报告中。你可以在 [facebook/react](https://github.com/facebook/react/issues) 仓库中提交 issue。

You can also provide feedback in the React Compiler Working Group by applying to be a member. Please see [the README for more details on joining](https://github.com/reactwg/react-compiler).
你也可以通过申请成为成员，在 React Compiler 工作组中提供反馈意见。请查看 [README](https://github.com/reactwg/react-compiler) 以获取更多加入详情。

### 编译器假设什么？| What does the compiler assume? {/*what-does-the-compiler-assume*/}

React Compiler assumes that your code:
React Compiler 假设你的代码：

1. Is valid, semantic JavaScript.
- 是有效的，语义化的 JavaScript
2. Tests that nullable/optional values and properties are defined before accessing them (for example, by enabling [`strictNullChecks`](https://www.typescriptlang.org/tsconfig/#strictNullChecks) if using TypeScript), i.e., `if (object.nullableProperty) { object.nullableProperty.foo }` or with optional-chaining `object.nullableProperty?.foo`.
- 在访问可空/可选值和属性之前，测试它们是否已定义（例如，如果使用 TypeScript，则启用 [`strictNullChecks`](https://www.typescriptlang.org/tsconfig/#strictNullChecks)），即：`if (object.nullableProperty) { object.nullableProperty.foo }` 或者使用可选链 `object.nullableProperty?.foo`
3. Follows the [Rules of React](https://react.dev/reference/rules).
- 遵循 [React 规则](/reference/rules)

React Compiler can verify many of the Rules of React statically, and will safely skip compilation when it detects an error. To see the errors we recommend also installing [eslint-plugin-react-compiler](https://www.npmjs.com/package/eslint-plugin-react-compiler).
React Compiler 可以静态验证 React 的许多规则，并且在检测到错误时会安全地跳过编译。要查看错误，我们建议同时安装 [eslint-plugin-react-compiler](https://www.npmjs.com/package/eslint-plugin-react-compiler)。

### 我如何知道我的组件已被优化？| How do I know my components have been optimized? {/*how-do-i-know-my-components-have-been-optimized*/}

[React Devtools](/learn/react-developer-tools) (v5.0+) has built-in support for React Compiler and will display a "Memo ✨" badge next to components that have been optimized by the compiler.
[React 开发工具](/learn/react-developer-tools)（v5.0 及以上版本）对 React Compiler 有内置支持，并会在已被编译器优化的组件旁边显示“Memo ✨”徽章。

### 编译后某些内容无法正常工作 | Something is not working after compilation {/*something-is-not-working-after-compilation*/}
If you have eslint-plugin-react-compiler installed, the compiler will display any violations of the rules of React in your editor. When it does this, it means that the compiler has skipped over optimizing that component or hook. This is perfectly okay, and the compiler can recover and continue optimizing other components in your codebase. **You don't have to fix all ESLint violations straight away.** You can address them at your own pace to increase the amount of components and hooks being optimized.
如果你安装了 eslint-plugin-react-compiler ，编译器将在你的编辑器中显示任何违反 React 规则的情况。当它这样做时，意味着编译器跳过了对该组件或钩子的优化。这完全没问题，并且编译器可以恢复并继续优化你代码库中的其他组件。**你不必立即修复所有的违反 ESLint 规则的代码。** 你可以按照自己的节奏来处理它们，以增加被优化的组件和钩子的数量。

Due to the flexible and dynamic nature of JavaScript however, it's not possible to comprehensively detect all cases. Bugs and undefined behavior such as infinite loops may occur in those cases.
然而，由于 JavaScript 的灵活和动态性质，不可能全面检测到所有情况。在这些情况下，可能会出现错误和未定义的行为，例如无限循环。

If your app doesn't work properly after compilation and you aren't seeing any ESLint errors, the compiler may be incorrectly compiling your code. To confirm this, try to make the issue go away by aggressively opting out any component or hook you think might be related via the [`"use no memo"` directive](#opt-out-of-the-compiler-for-a-component).
如果你的应用在编译后无法正常工作，并且你没有看到任何 ESLint 错误，编译器可能错误地编译了你的代码。为了确认这一点，尝试通过积极选择你认为可能相关的任何组件或钩子，并通过 [`"use no memo"` 指令](#opt-out-of-the-compiler-for-a-component) 退出优化来解决问题。

```js {2}
function SuspiciousComponent() {
  "use no memo"; // 选择不让此组件由 React Compiler 进行编译 // opts out this component from being compiled by React Compiler
  // ...
}
```

<Note>
#### `"use no memo"` {/*use-no-memo*/}

`"use no memo"` is a _temporary_ escape hatch that lets you opt-out components and hooks from being compiled by the React Compiler. This directive is not meant to be long lived the same way as eg [`"use client"`](/reference/rsc/use-client) is.
`"use no memo"` 是一个**临时的**逃避机制，它允许你选择不让组件和钩子由 React Compiler 进行编译。此指令不像例如 [`"use client"`](/reference/rsc/use-client) 那样长期存在。

It is not recommended to reach for this directive unless it's strictly necessary. Once you opt-out a component or hook, it is opted-out forever until the directive is removed. This means that even if you fix the code, the compiler will still skip over compiling it unless you remove the directive.
除非绝对必要，否则不建议使用这个指令。一旦你选择退出一个组件或钩子，它将永久退出，直到指令被移除。这意味着即使你修复了代码，编译器仍然会跳过编译它，除非你移除指令。
</Note>

When you make the error go away, confirm that removing the opt out directive makes the issue come back. Then share a bug report with us (you can try to reduce it to a small repro, or if it's open source code you can also just paste the entire source) using the [React Compiler Playground](https://playground.react.dev) so we can identify and help fix the issue.
当你修复错误时，请确认删除退出指令是否会使问题重新出现。然后使用 [React Compiler Playground](https://playground.react.dev) 与我们分享一个错误报告（你可以尝试将其减少到一个小的重现，或者如果是开源代码，你也可以直接粘贴整个源代码），这样我们就可以识别并帮助解决问题。

### 其他问题 | Other issues {/*other-issues*/}

Please see https://github.com/reactwg/react-compiler/discussions/7.
请查阅 https://github.com/reactwg/react-compiler/discussions/7 。
