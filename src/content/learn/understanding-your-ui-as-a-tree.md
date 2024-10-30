---
title: 将 UI 视为树 | Understanding Your UI as a Tree
---

<Intro>

Your React app is taking shape with many components being nested within each other. How does React keep track of your app's component structure?
当 React 应用程序逐渐成形时，许多组件会出现嵌套。那么 React 是如何跟踪应用程序组件结构的？

React, and many other UI libraries, model UI as a tree. Thinking of your app as a tree is useful for understanding the relationship between components. This understanding will help you debug future concepts like performance and state management.
React 以及许多其他 UI 库，将 UI 建模为树。将应用程序视为树对于理解组件之间的关系以及调试性能和状态管理等未来将会遇到的一些概念非常有用。

</Intro>

<YouWillLearn>

* How React "sees" component structures
* React 如何看待组件结构
* What a render tree is and what it is useful for
* 渲染树是什么以及它有什么用处
* What a module dependency tree is and what it is useful for
* 模块依赖树是什么以及它有什么用处

</YouWillLearn>

## 将 UI 视为树 | Your UI as a tree {/*your-ui-as-a-tree*/}

Trees are a relationship model between items and UI is often represented using tree structures. For example, browsers use tree structures to model HTML ([DOM](https://developer.mozilla.org/docs/Web/API/Document_Object_Model/Introduction)) and CSS ([CSSOM](https://developer.mozilla.org/docs/Web/API/CSS_Object_Model)). Mobile platforms also use trees to represent their view hierarchy.
树是项目和 UI 之间的关系模型，通常使用树结构来表示 UI。例如，浏览器使用树结构来建模 HTML（[DOM](https://developer.mozilla.org/docs/Web/API/Document_Object_Model/Introduction)）与CSS（[CSSOM](https://developer.mozilla.org/docs/Web/API/CSS_Object_Model)）。移动平台也使用树来表示其视图层次结构。

<Diagram name="preserving_state_dom_tree" height={193} width={864} alt="水平排列的三个部分的图表。第一部分有三个垂直堆叠的矩形，并分别标记为 Component A、Component B 和 Component C。向下一个窗格过渡的是一个带有 React 标志的箭头，标记为 React。中间部分包含一棵组件树，根节点标记为 A，有两个子节点分别标记为 B 和 C。下一个部分再次使用带有 React 标志的箭头进行过渡，标记为 React DOM。第三和最后一个部分是浏览器的线框图，包含一棵有 8 个节点的树，其中只有一个子集被突出显示（表示中间部分的子树）。| Diagram with three sections arranged horizontally. In the first section, there are three rectangles stacked vertically, with labels 'Component A', 'Component B', and 'Component C'. Transitioning to the next pane is an arrow with the React logo on top labeled 'React'. The middle section contains a tree of components, with the root labeled 'A' and two children labeled 'B' and 'C'. The next section is again transitioned using an arrow with the React logo on top labeled 'React DOM'. The third and final section is a wireframe of a browser, containing a tree of 8 nodes, which has only a subset highlighted (indicating the subtree from the middle section).
">

React creates a UI tree from your components. In this example, the UI tree is then used to render to the DOM.
React 从组件中创建 UI 树。在这个示例中，UI 树最后会用于渲染 DOM。
</Diagram>

Like browsers and mobile platforms, React also uses tree structures to manage and model the relationship between components in a React app. These trees are useful tools to understand how data flows through a React app and how to optimize rendering and app size.
与浏览器和移动平台一样，React 还使用树结构来管理和建模 React 应用程序中组件之间的关系。这些树是有用的工具，用于理解数据如何在 React 应用程序中流动以及如何优化呈现和应用程序大小。

## 渲染树 | The Render Tree {/*the-render-tree*/}

A major feature of components is the ability to compose components of other components. As we [nest components](/learn/your-first-component#nesting-and-organizing-components), we have the concept of parent and child components, where each parent component may itself be a child of another component.
组件的一个主要特性是能够由其他组件组合而成。在 [嵌套组件](/learn/your-first-component#nesting-and-organizing-components) 中有父组件和子组件的概念，其中每个父组件本身可能是另一个组件的子组件。

When we render a React app, we can model this relationship in a tree, known as the render tree.
当渲染 React 应用程序时，可以在一个称为渲染树的树中建模这种关系。

Here is a React app that renders inspirational quotes.
下面的 React 应用程序渲染了一些鼓舞人心的引语。

<Sandpack>

```js src/App.js
import FancyText from './FancyText';
import InspirationGenerator from './InspirationGenerator';
import Copyright from './Copyright';

export default function App() {
  return (
    <>
      <FancyText title text="Get Inspired App" />
      <InspirationGenerator>
        <Copyright year={2004} />
      </InspirationGenerator>
    </>
  );
}

```

```js src/FancyText.js
export default function FancyText({title, text}) {
  return title
    ? <h1 className='fancy title'>{text}</h1>
    : <h3 className='fancy cursive'>{text}</h3>
}
```

```js src/InspirationGenerator.js
import * as React from 'react';
import quotes from './quotes';
import FancyText from './FancyText';

export default function InspirationGenerator({children}) {
  const [index, setIndex] = React.useState(0);
  const quote = quotes[index];
  const next = () => setIndex((index + 1) % quotes.length);

  return (
    <>
      <p>Your inspirational quote is:</p>
      <FancyText text={quote} />
      <button onClick={next}>Inspire me again</button>
      {children}
    </>
  );
}
```

```js src/Copyright.js
export default function Copyright({year}) {
  return <p className='small'>©️ {year}</p>;
}
```

```js src/quotes.js
export default [
  "Don’t let yesterday take up too much of today.” — Will Rogers",
  "Ambition is putting a ladder against the sky.",
  "A joy that's shared is a joy made double.",
  ];
```

```css
.fancy {
  font-family: 'Georgia';
}
.title {
  color: #007AA3;
  text-decoration: underline;
}
.cursive {
  font-style: italic;
}
.small {
  font-size: 10px;
}
```

</Sandpack>

<Diagram name="render_tree" height={250} width={500} alt="带有五个节点的树形图。每个节点代表一个组件。树的根是 App，从它延伸出两条箭头，分别指向 InspirationGenerator 和 FancyText。这些箭头标有 renders 一词。InspirationGenerator 节点还有两个箭头指向节点 FancyText 和 Copyright。| Tree graph with five nodes. Each node represents a component. The root of the tree is App, with two arrows extending from it to 'InspirationGenerator' and 'FancyText'. The arrows are labelled with the word 'renders'. 'InspirationGenerator' node also has two arrows pointing to nodes 'FancyText' and 'Copyright'.">

React creates a *render tree*, a UI tree, composed of the rendered components.
React 创建的 UI 树是由渲染过的组件构成的，被称为渲染树。

</Diagram>

From the example app, we can construct the above render tree.
通过示例应用程序，可以构建上面的渲染树。

The tree is composed of nodes, each of which represents a component. `App`, `FancyText`, `Copyright`, to name a few, are all nodes in our tree.
这棵树由节点组成，每个节点代表一个组件。例如，`App`、`FancyText`、`Copyright` 等都是我们树中的节点。

The root node in a React render tree is the [root component](/learn/importing-and-exporting-components#the-root-component-file) of the app. In this case, the root component is `App` and it is the first component React renders. Each arrow in the tree points from a parent component to a child component.
在 React 渲染树中，根节点是应用程序的 [根组件](/learn/importing-and-exporting-components#the-root-component-file)。在这种情况下，根组件是 `App`，它是 React 渲染的第一个组件。树中的每个箭头从父组件指向子组件。

<DeepDive>

#### 那么渲染树中的 HTML 标签在哪里呢？| Where are the HTML tags in the render tree? {/*where-are-the-html-elements-in-the-render-tree*/}

You'll notice in the above render tree, there is no mention of the HTML tags that each component renders. This is because the render tree is only composed of React [components](learn/your-first-component#components-ui-building-blocks).
也许会注意到在上面的渲染树中，没有提到每个组件渲染的 HTML 标签。这是因为渲染树仅由 React [组件](learn/your-first-component#components-ui-building-blocks) 组成。

React, as a UI framework, is platform agnostic. On react.dev, we showcase examples that render to the web, which uses HTML markup as its UI primitives. But a React app could just as likely render to a mobile or desktop platform, which may use different UI primitives like [UIView](https://developer.apple.com/documentation/uikit/uiview) or [FrameworkElement](https://learn.microsoft.com/en-us/dotnet/api/system.windows.frameworkelement?view=windowsdesktop-7.0).
React 是跨平台的 UI 框架。react.dev 展示了一些渲染到使用 HTML 标签作为 UI 原语的 web 的示例。但是 React 应用程序同样可以渲染到移动设备或桌面平台，这些平台可能使用不同的 UI 原语，如 [UIView](https://developer.apple.com/documentation/uikit/uiview) 或 [FrameworkElement](https://learn.microsoft.com/en-us/dotnet/api/system.windows.frameworkelement?view=windowsdesktop-7.0)。

These platform UI primitives are not a part of React. React render trees can provide insight to our React app regardless of what platform your app renders to.
这些平台 UI 原语不是 React 的一部分。无论应用程序渲染到哪个平台，React 渲染树都可以为 React 应用程序提供见解。

</DeepDive>

A render tree represents a single render pass of a React application. With [conditional rendering](/learn/conditional-rendering), a parent component may render different children depending on the data passed.
渲染树表示 React 应用程序的单个渲染过程。在 [条件渲染](/learn/conditional-rendering) 中，父组件可以根据传递的数据渲染不同的子组件。

We can update the app to conditionally render either an inspirational quote or color.
我们可以更新应用程序以有条件地渲染励志语录或颜色。

<Sandpack>

```js src/App.js
import FancyText from './FancyText';
import InspirationGenerator from './InspirationGenerator';
import Copyright from './Copyright';

export default function App() {
  return (
    <>
      <FancyText title text="Get Inspired App" />
      <InspirationGenerator>
        <Copyright year={2004} />
      </InspirationGenerator>
    </>
  );
}

```

```js src/FancyText.js
export default function FancyText({title, text}) {
  return title
    ? <h1 className='fancy title'>{text}</h1>
    : <h3 className='fancy cursive'>{text}</h3>
}
```

```js src/Color.js
export default function Color({value}) {
  return <div className="colorbox" style={{backgroundColor: value}} />
}
```

```js src/InspirationGenerator.js
import * as React from 'react';
import inspirations from './inspirations';
import FancyText from './FancyText';
import Color from './Color';

export default function InspirationGenerator({children}) {
  const [index, setIndex] = React.useState(0);
  const inspiration = inspirations[index];
  const next = () => setIndex((index + 1) % inspirations.length);

  return (
    <>
      <p>Your inspirational {inspiration.type} is:</p>
      {inspiration.type === 'quote'
      ? <FancyText text={inspiration.value} />
      : <Color value={inspiration.value} />}

      <button onClick={next}>Inspire me again</button>
      {children}
    </>
  );
}
```

```js src/Copyright.js
export default function Copyright({year}) {
  return <p className='small'>©️ {year}</p>;
}
```

```js src/inspirations.js
export default [
  {type: 'quote', value: "Don’t let yesterday take up too much of today.” — Will Rogers"},
  {type: 'color', value: "#B73636"},
  {type: 'quote', value: "Ambition is putting a ladder against the sky."},
  {type: 'color', value: "#256266"},
  {type: 'quote', value: "A joy that's shared is a joy made double."},
  {type: 'color', value: "#F9F2B4"},
];
```

```css
.fancy {
  font-family: 'Georgia';
}
.title {
  color: #007AA3;
  text-decoration: underline;
}
.cursive {
  font-style: italic;
}
.small {
  font-size: 10px;
}
.colorbox {
  height: 100px;
  width: 100px;
  margin: 8px;
}
```
</Sandpack>

<Diagram name="conditional_render_tree" height={250} width={561} alt="带有六个节点的树形图。树的顶部节点标有 App ，有两个箭头指向标有 InspirationGenerator 和 FancyText 的节点。箭头是实线，标有 renders 一词。InspirationGenerator 节点还有三个箭头。指向 FancyText 和 Color 节点的箭头是虚线，标有 renders?。最后一个箭头指向标有 Copyright 的节点，是实线，标有 renders 一词。| Tree graph with six nodes. The top node of the tree is labelled 'App' with two arrows extending to nodes labelled 'InspirationGenerator' and 'FancyText'. The arrows are solid lines and are labelled with the word 'renders'. 'InspirationGenerator' node also has three arrows. The arrows to nodes 'FancyText' and 'Color' are dashed and labelled with 'renders?'. The last arrow points to the node labelled 'Copyright' and is solid and labelled with 'renders'.">

With conditional rendering, across different renders, the render tree may render different components.
在条件渲染的不同渲染过程中，渲染树可能会渲染不同的组件。

</Diagram>

In this example, depending on what `inspiration.type` is, we may render `<FancyText>` or `<Color>`. The render tree may be different for each render pass.
在这个示例中，根据 `inspiration.type` 的值可能会渲染 `<FancyText>` 或 `<Color>`。每次渲染过程的渲染树可能都不同。

Although render trees may differ across render passes, these trees are generally helpful for identifying what the *top-level* and *leaf components* are in a React app. Top-level components are the components nearest to the root component and affect the rendering performance of all the components beneath them and often contain the most complexity. Leaf components are near the bottom of the tree and have no child components and are often frequently re-rendered.
尽管渲染树可能在不同的渲染过程中有所不同，但通常这些树有助于识别 React 应用程序中的顶级和叶子组件。顶级组件是离根组件最近的组件，它们影响其下所有组件的渲染性能，通常包含最多复杂性。叶子组件位于树的底部，没有子组件，通常会频繁重新渲染。

Identifying these categories of components are useful for understanding data flow and performance of your app.
识别这些组件类别有助于理解应用程序的数据流和性能。

## 模块依赖树 | The Module Dependency Tree {/*the-module-dependency-tree*/}

Another relationship in a React app that can be modeled with a tree are an app's module dependencies. As we [break up our components](/learn/importing-and-exporting-components#exporting-and-importing-a-component) and logic into separate files, we create [JS modules](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Modules) where we may export components, functions, or constants.
在 React 应用程序中，可以使用树来建模的另一个关系是应用程序的模块依赖关系。当 [拆分组件](/learn/importing-and-exporting-components#exporting-and-importing-a-component) 和逻辑到不同的文件中时，就创建了 [JavaScript 模块](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Modules)，在这些模块中可以导出组件、函数或常量。

Each node in a module dependency tree is a module and each branch represents an `import` statement in that module.
模块依赖树中的每个节点都是一个模块，每个分支代表该模块中的 `import` 语句。

If we take the previous Inspirations app, we can build a module dependency tree, or dependency tree for short.
以之前的 Inspirations 应用程序为例，可以构建一个模块依赖树，简称依赖树。

<Diagram name="module_dependency_tree" height={250} width={658} alt="带有七个节点的树形图。每个节点都标有一个模块名称。树的顶层节点标有 App.js。有三条箭头指向模块 InspirationGenerator.js、FancyText.js 和 Copyright.js，箭头上标有 imports。从 InspirationGenerator.js 节点出发，有三条箭头分别指向三个模块：FancyText.js、Color.js 和 inspirations.js，箭头上标有 imports。| A tree graph with seven nodes. Each node is labelled with a module name. The top level node of the tree is labelled 'App.js'. There are three arrows pointing to the modules 'InspirationGenerator.js', 'FancyText.js' and 'Copyright.js' and the arrows are labelled with 'imports'. From the 'InspirationGenerator.js' node, there are three arrows that extend to three modules: 'FancyText.js', 'Color.js', and 'inspirations.js'. The arrows are labelled with 'imports'.">

The module dependency tree for the Inspirations app.
Inspirations 应用程序的模块依赖树。

</Diagram>

The root node of the tree is the root module, also known as the entrypoint file. It often is the module that contains the root component.
树的根节点是根模块，也称为入口文件。它通常包含根组件的模块。

Comparing to the render tree of the same app, there are similar structures but some notable differences:
与同一应用程序的渲染树相比，存在相似的结构，但也有一些显著的差异：

* The nodes that make-up the tree represent modules, not components.
* 构成树的节点代表模块，而不是组件。
* Non-component modules, like `inspirations.js`, are also represented in this tree. The render tree only encapsulates components.
* 非组件模块，如 `inspirations.js`，在这个树中也有所体现。渲染树仅封装组件。
* `Copyright.js` appears under `App.js` but in the render tree, `Copyright`, the component, appears as a child of `InspirationGenerator`. This is because `InspirationGenerator` accepts JSX as [children props](/learn/passing-props-to-a-component#passing-jsx-as-children), so it renders `Copyright` as a child component but does not import the module.
* `Copyright.js` 出现在 `App.js` 下，但在渲染树中，`Copyright` 作为 `InspirationGenerator` 的子组件出现。这是因为 `InspirationGenerator` 接受 JSX 作为 [children props](/learn/passing-props-to-a-component#passing-jsx-as-children)，因此它将 `Copyright` 作为子组件渲染，但不导入该模块。

Dependency trees are useful to determine what modules are necessary to run your React app. When building a React app for production, there is typically a build step that will bundle all the necessary JavaScript to ship to the client. The tool responsible for this is called a [bundler](https://developer.mozilla.org/en-US/docs/Learn/Tools_and_testing/Understanding_client-side_tools/Overview#the_modern_tooling_ecosystem), and bundlers will use the dependency tree to determine what modules should be included.
依赖树对于确定运行 React 应用程序所需的模块非常有用。在为生产环境构建 React 应用程序时，通常会有一个构建步骤，该步骤将捆绑所有必要的 JavaScript 以供客户端使用。负责此操作的工具称为 [bundler（捆绑器）](https://developer.mozilla.org/en-US/docs/Learn/Tools_and_testing/Understanding_client-side_tools/Overview#the_modern_tooling_ecosystem)，并且 bundler 将使用依赖树来确定应包含哪些模块。

As your app grows, often the bundle size does too. Large bundle sizes are expensive for a client to download and run. Large bundle sizes can delay the time for your UI to get drawn. Getting a sense of your app's dependency tree may help with debugging these issues.
随着应用程序的增长，捆绑包大小通常也会增加。大型捆绑包大小对于客户端来说下载和运行成本高昂，并延迟 UI 绘制的时间。了解应用程序的依赖树可能有助于调试这些问题。

[comment]: <> (也许应该深入介绍条件导入)

<Recap>

* Trees are a common way to represent the relationship between entities. They are often used to model UI.
* 树是表示实体之间关系的常见方式，它们经常用于建模 UI。
* Render trees represent the nested relationship between React components across a single render.
* 渲染树表示单次渲染中 React 组件之间的嵌套关系。
* With conditional rendering, the render tree may change across different renders. With different prop values, components may render different children components.
* 使用条件渲染，渲染树可能会在不同的渲染过程中发生变化。使用不同的属性值，组件可能会渲染不同的子组件。
* Render trees help identify what the top-level and leaf components are. Top-level components affect the rendering performance of all components beneath them and leaf components are often re-rendered frequently. Identifying them is useful for understanding and debugging rendering performance.
* 渲染树有助于识别顶级组件和叶子组件。顶级组件会影响其下所有组件的渲染性能，而叶子组件通常会频繁重新渲染。识别它们有助于理解和调试渲染性能问题。
* Dependency trees represent the module dependencies in a React app.
* 依赖树表示 React 应用程序中的模块依赖关系。
* Dependency trees are used by build tools to bundle the necessary code to ship an app.
* 构建工具使用依赖树来捆绑必要的代码以部署应用程序。
* Dependency trees are useful for debugging large bundle sizes that slow time to paint and expose opportunities for optimizing what code is bundled.
* 依赖树有助于调试大型捆绑包带来的渲染速度过慢的问题，以及发现哪些捆绑代码可以被优化。

</Recap>

[TODO]: <> (Add challenges)
