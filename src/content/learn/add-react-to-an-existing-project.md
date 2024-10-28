---
title: 将 React 添加到现有项目中 | Add React to an Existing Project
---

<Intro>

If you want to add some interactivity to your existing project, you don't have to rewrite it in React. Add React to your existing stack, and render interactive React components anywhere.

如果想对现有项目添加一些交互，不必使用 React 将其整个重写。只需将 React 添加到已有技术栈中，就可以在任何位置渲染交互式的 React 组件。

</Intro>

<Note>

**You need to install [Node.js](https://nodejs.org/en/) for local development.** Although you can [try React](/learn/installation#try-react) online or with a simple HTML page, realistically most JavaScript tooling you'll want to use for development requires Node.js.

**你需要安装 [Node.js](https://nodejs.org/zh-cn/) 以进行本地开发**。尽管可以使用 [在线演练场](/learn/installation#try-react) 或简单的 HTML 页面来尝试 React，但实际上大多数用于开发的 JavaScript 工具都需要 Node.js。

</Note>

## 在现有网站的子路由中使用 React | Using React for an entire subroute of your existing website {/*using-react-for-an-entire-subroute-of-your-existing-website*/}

Let's say you have an existing web app at `example.com` built with another server technology (like Rails), and you want to implement all routes starting with `example.com/some-app/` fully with React.

假设你在 `example.com` 部署了一个其他服务端技术（例如 Rails）构建的 Web 应用，但是你又想在 `example.com/some-app/` 部署一个 React 项目。

Here's how we recommend to set it up:

以下是推荐的配置方式：

1. **Build the React part of your app** using one of the [React-based frameworks](/learn/start-a-new-react-project).
- 使用一个 [基于 React 的框架](/learn/start-a-new-react-project) 构建 **应用的 React 部分**。
2. **Specify `/some-app` as the *base path*** in your framework's configuration (here's how: [Next.js](https://nextjs.org/docs/api-reference/next.config.js/basepath), [Gatsby](https://www.gatsbyjs.com/docs/how-to/previews-deploys-hosting/path-prefix/)).
- **在框架配置中将 `/some-app` 指定为基本路径**（这里有 [Next.js](https://nextjs.org/docs/api-reference/next.config.js/basepath) 与 [Gatsby](https://www.gatsbyjs.com/docs/how-to/previews-deploys-hosting/path-prefix/) 的配置样例）。
3. **Configure your server or a proxy** so that all requests under `/some-app/` are handled by your React app.
- **配置服务器或代理**，以便所有位于 `/some-app/` 下的请求都由 React 应用处理。

This ensures the React part of your app can [benefit from the best practices](/learn/start-a-new-react-project#can-i-use-react-without-a-framework) baked into those frameworks.

这可以确保应用的 React 部分可以受益于这些框架中内置的 [最佳实践](/learn/start-a-new-react-project#can-i-use-react-without-a-framework)。

Many React-based frameworks are full-stack and let your React app take advantage of the server. However, you can use the same approach even if you can't or don't want to run JavaScript on the server. In that case, serve the HTML/CSS/JS export ([`next export` output](https://nextjs.org/docs/advanced-features/static-html-export) for Next.js, default for Gatsby) at `/some-app/` instead.

许多基于 React 的框架都是全栈的，从而可以让你的 React 应用充分利用服务器。但是，即使无法或不想在服务器上运行 JavaScript，也可以使用相同的方法。在这种情况下，将 HTML/CSS/JS 导出（Next.js 的 [`next export` output](https://nextjs.org/docs/advanced-features/static-html-export)，Gatsby 的 default）替换为 `/some-app/`。

## 在现有页面的一部分中使用 React | Using React for a part of your existing page {/*using-react-for-a-part-of-your-existing-page*/}

Let's say you have an existing page built with another technology (either a server one like Rails, or a client one like Backbone), and you want to render interactive React components somewhere on that page. That's a common way to integrate React--in fact, it's how most React usage looked at Meta for many years!

假设有一个其他技术栈（无论是 Rails 这样的服务端技术，还是 Backbone 那样的客户端技术）构建的现有页面，并且想要在该页面的某个位置渲染交互式的 React 组件。这是集成 React 的常见方式——实际上，这也正是多年来大多数情况下 Meta 使用 React 的方式！

You can do this in two steps:

你可以分两步进行：

1. **Set up a JavaScript environment** that lets you use the [JSX syntax](/learn/writing-markup-with-jsx), split your code into modules with the [`import`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/import) / [`export`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/export) syntax, and use packages (for example, React) from the [npm](https://www.npmjs.com/) package registry.
- **配置 JavaScript 环境**，以便使用 [JSX 语法](/learn/writing-markup-with-jsx)、[`import`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/import) 和 [`export`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/export) 语法将代码拆分为模块，以及从 [npm](https://www.npmjs.com/) 包注册表中使用包（例如 React）。
2. **Render your React components** where you want to see them on the page.
- **在需要的位置渲染 React 组件**。

The exact approach depends on your existing page setup, so let's walk through some details.

确切的方法取决于现有的页面配置，因此让我们对一些细节进行说明。

### 步骤 1：配置模块化的 JavaScript 环境 | Step 1: Set up a modular JavaScript environment {/*step-1-set-up-a-modular-javascript-environment*/}

A modular JavaScript environment lets you write your React components in individual files, as opposed to writing all of your code in a single file. It also lets you use all the wonderful packages published by other developers on the [npm](https://www.npmjs.com/) registry--including React itself! How you do this depends on your existing setup:

模块化的 JavaScript 环境可以让你在单一的文件中编写 React 组件，而不是在一个文件中编写所有的代码。它还可以让你使用其他开发人员在 [npm](https://www.npmjs.com/) 注册表上发布的一些特别好用的包，包括 React！如何实现这一点取决于你现有的配置：

* **If your app is already split into files that use `import` statements,** try to use the setup you already have. Check whether writing `<div />` in your JS code causes a syntax error. If it causes a syntax error, you might need to [transform your JavaScript code with Babel](https://babeljs.io/setup), and enable the [Babel React preset](https://babeljs.io/docs/babel-preset-react) to use JSX.

* **如果你的应用已经使用 `import` 语句来分割成不同的文件，请尝试利用已有的配置**。检查在你的 JavaScript 代码中编写 `<div />`是否会导致语法错误。如果有语法错误，你可能需要使用 [Babel](https://babeljs.io/setup) 转换你的 JavaScript 代码，并启用 [Babel React preset](https://babeljs.io/docs/babel-preset-react) 来使用 JSX。

* **If your app doesn't have an existing setup for compiling JavaScript modules,** set it up with [Vite](https://vitejs.dev/). The Vite community maintains [many integrations with backend frameworks](https://github.com/vitejs/awesome-vite#integrations-with-backends), including Rails, Django, and Laravel. If your backend framework is not listed, [follow this guide](https://vitejs.dev/guide/backend-integration.html) to manually integrate Vite builds with your backend.

* **如果你的应用没有用于编译 JavaScript 模块的配置，请使用 [Vite](https://cn.vitejs.dev/) 进行配置**。Vite 社区维护了与后端框架（包括 Rails、Django 和 Laravel）的 [许多集成项目](https://github.com/vitejs/awesome-vite#integrations-with-backends)。如果你的后端框架没有列出，请 [按照此指南](https://cn.vitejs.dev/guide/backend-integration.html) 手动将 Vite 构建集成到你的后端。

To check whether your setup works, run this command in your project folder:

如果想要检查你的配置是否有效，可以在项目文件夹中运行以下命令：

<TerminalBlock>
npm install react react-dom
</TerminalBlock>

Then add these lines of code at the top of your main JavaScript file (it might be called `index.js` or `main.js`):

然后在你的 JavaScript 主文件（它可能被称为 `index.js` 或 `main.js`）的顶部添加以下代码：

<Sandpack>

```html index.html hidden
<!DOCTYPE html>
<html>
  <head><title>My app</title></head>
  <body>
    <!-- Your existing page content (in this example, it gets replaced) -->
    <!-- 你现有的页面内容（在这个例子中，它将被替换） -->
  </body>
</html>
```

```js src/index.js active
import { createRoot } from 'react-dom/client';

// Clear the existing HTML content
// 清除现有的 HTML 内容
document.body.innerHTML = '<div id="app"></div>';

// Render your React component instead
// 渲染你的 React 组件
const root = createRoot(document.getElementById('app'));
root.render(<h1>Hello, world</h1>);
```

</Sandpack>

If the entire content of your page was replaced by a "Hello, world!", everything worked! Keep reading.

如果页面的全部内容都被替换为“Hello, world!”，则一切正常！那么继续阅读。

<Note>

Integrating a modular JavaScript environment into an existing project for the first time can feel intimidating, but it's worth it! If you get stuck, try our [community resources](/community) or the [Vite Chat](https://chat.vitejs.dev/).

第一次将模块化 JavaScript 环境集成到现有项目中可能会让人感到害怕，但这是值得的！如果遇到困难，请尝试我们的 [社区资源](/community) 或 [Vite Chat](https://chat.vitejs.dev/)。

</Note>

### 步骤 2：在页面的任何位置渲染 React 组件 | Step 2: Render React components anywhere on the page {/*step-2-render-react-components-anywhere-on-the-page*/}

In the previous step, you put this code at the top of your main file:

在上一步中，此代码将被放在主文件的顶部：

```js
import { createRoot } from 'react-dom/client';

// Clear the existing HTML content
// 清除现有的 HTML 内容
document.body.innerHTML = '<div id="app"></div>';

// Render your React component instead
// 渲染你的 React 组件
const root = createRoot(document.getElementById('app'));
root.render(<h1>Hello, world</h1>);
```

Of course, you don't actually want to clear the existing HTML content!

当然，你实际上并不想清除现有的 HTML 内容！

Delete this code.

那么请删除此代码。

Instead, you probably want to render your React components in specific places in your HTML. Open your HTML page (or the server templates that generate it) and add a unique [`id`](https://developer.mozilla.org/en-US/docs/Web/HTML/Global_attributes/id) attribute to any tag, for example:

相反，你可能想要在 HTML 中特定的位置渲染 React 组件。打开 HTML 页面（或用于生成它的服务端模板），并向任意一个标签添加一个唯一的 [`id`](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Global_attributes/id) 属性，例如：

```html
<!-- ... 你的 HTML 代码某处 ... -->
<nav id="navigation"></nav>
<!-- ... 其他 HTML 代码 ... -->
```

This lets you find that HTML element with [`document.getElementById`](https://developer.mozilla.org/en-US/docs/Web/API/Document/getElementById) and pass it to [`createRoot`](/reference/react-dom/client/createRoot) so that you can render your own React component inside:

这样可以使用 [`document.getElementById`](https://developer.mozilla.org/zh-CN/docs/Web/API/Document/getElementById) 查找到该 HTML 元素，并将其传递给 [`createRoot`](/reference/react-dom/client/createRoot)，以便可以在其中渲染自己的 React 组件：

<Sandpack>

```html index.html
<!DOCTYPE html>
<html>
  <head><title>My app</title></head>
  <body>
    <p>This paragraph is a part of HTML.</p>
    <nav id="navigation"></nav>
    <p>This paragraph is also a part of HTML.</p>
  </body>
</html>
```

```js src/index.js active
import { createRoot } from 'react-dom/client';

function NavigationBar() {
  // TODO: Actually implement a navigation bar
  // TODO: 实际实现一个导航栏
  return <h1>Hello from React!</h1>;
}

const domNode = document.getElementById('navigation');
const root = createRoot(domNode);
root.render(<NavigationBar />);
```

</Sandpack>

Notice how the original HTML content from `index.html` is preserved, but your own `NavigationBar` React component now appears inside the `<nav id="navigation">` from your HTML. Read the [`createRoot` usage documentation](/reference/react-dom/client/createRoot#rendering-a-page-partially-built-with-react) to learn more about rendering React components inside an existing HTML page.

请注意 `index.html` 中的原始 HTML 内容是如何保留的，但现在你自己的 `NavigationBar` React 组件出现在 HTML 的 `<nav id="navigation">` 中。阅读 [`createRoot` 用法文档](/reference/react-dom/client/createRoot#rendering-a-page-partially-built-with-react) 以了解如何在现有 HTML 页面中渲染 React 组件。

When you adopt React in an existing project, it's common to start with small interactive components (like buttons), and then gradually keep "moving upwards" until eventually your entire page is built with React. If you ever reach that point, we recommend migrating to [a React framework](/learn/start-a-new-react-project) right after to get the most out of React.

当在现有项目中采用 React 时，通常会从小型交互式组件（例如按钮）开始，然后逐渐“向上移动”，直到最终整个页面都由 React 构建。到那个时候，我们建议立即迁移到 [一个 React 框架](/learn/start-a-new-react-project)，以充分利用 React 的优势。

## 在现有的原生移动应用中使用 React Native | Using React Native in an existing native mobile app {/*using-react-native-in-an-existing-native-mobile-app*/}

[React Native](https://reactnative.dev/) can also be integrated into existing native apps incrementally. If you have an existing native app for Android (Java or Kotlin) or iOS (Objective-C or Swift), [follow this guide](https://reactnative.dev/docs/integration-with-existing-apps) to add a React Native screen to it.

[React Native](https://reactnative.dev/) 也可以逐步集成到现有的原生应用中。如果已经有一个现有的 Android（Java 或 Kotlin）或 iOS（Objective-C 或 Swift）原生应用，请 [按照本指南](https://reactnative.dev/docs/integration-with-existing-apps) 将 React Native 添加到其中。
