---
title: React 开发者工具 | React Developer Tools
translators:
  - dahui4dev
---

<Intro>

Use React Developer Tools to inspect React [components](/learn/your-first-component), edit [props](/learn/passing-props-to-a-component) and [state](/learn/state-a-components-memory), and identify performance problems.
使用 React 开发者工具检查 React [components](/learn/your-first-component)，编辑 [props](/learn/passing-props-to-a-component) 和 [state](/learn/state-a-components-memory)，并识别性能问题。

</Intro>

<YouWillLearn>

* How to install React Developer Tools
* 如何安装 React 开发者工具

</YouWillLearn>

## 浏览器扩展 | Browser extension {/*browser-extension*/}

The easiest way to debug websites built with React is to install the React Developer Tools browser extension. It is available for several popular browsers:
调试 React 构建的网站最简单的办法就是安装 React 开发者工具浏览器扩展。它可用于几种流行的浏览器：

* [Install for **Chrome**](https://chrome.google.com/webstore/detail/react-developer-tools/fmkadmapgofadopljbjfkapdkoienihi?hl=en)
* [安装 **Chrome** 扩展](https://chrome.google.com/webstore/detail/react-developer-tools/fmkadmapgofadopljbjfkapdkoienihi?hl=en)
* [Install for **Firefox**](https://addons.mozilla.org/en-US/firefox/addon/react-devtools/)
* [安装 **Firefox** 扩展](https://addons.mozilla.org/zh-CN/firefox/addon/react-devtools/)
* [Install for **Edge**](https://microsoftedge.microsoft.com/addons/detail/react-developer-tools/gpphkfbcpidddadnkolkpfckpihlkkil)
* [安装 **Edge** 扩展](https://microsoftedge.microsoft.com/addons/detail/react-developer-tools/gpphkfbcpidddadnkolkpfckpihlkkil)

Now, if you visit a website **built with React,** you will see the _Components_ and _Profiler_ panels.
现在，如果你访问一个用 **React 构建** 的网站，你将看到 **Components** 和 **Profiler** 面板。

![React Developer Tools extension](/images/docs/react-devtools-extension.png)

### Safari 和其他浏览器 | Safari and other browsers {/*safari-and-other-browsers*/}
For other browsers (for example, Safari), install the [`react-devtools`](https://www.npmjs.com/package/react-devtools) npm package:
为其他浏览器（比如，Safari），安装 [`react-devtools`](https://www.npmjs.com/package/react-devtools) npm 包:
```bash
# Yarn
yarn global add react-devtools

# Npm
npm install -g react-devtools
```

Next open the developer tools from the terminal:
接下来从终端打开开发者工具：
```bash
react-devtools
```

Then connect your website by adding the following `<script>` tag to the beginning of your website's `<head>`:
然后通过将以下 `<script>` 标签添加到你网站的 `<head>` 开头来连接你的网站： 
```html {3}
<html>
  <head>
    <script src="http://localhost:8097"></script>
```

Reload your website in the browser now to view it in developer tools.
现在在浏览器里刷新你的网站，你可以在开发者工具里查看它。

![React Developer Tools standalone](/images/docs/react-devtools-standalone.png)

## 移动端（React Native）| Mobile (React Native) {/*mobile-react-native*/}
React Developer Tools can be used to inspect apps built with [React Native](https://reactnative.dev/) as well.
React 开发者工具同样可检查用 [React Native](https://reactnative.dev/) 构建的应用程序。

The easiest way to use React Developer Tools is to install it globally:
使用 React 开发者工具最简单的办法就是全局安装它：
```bash
# Yarn
yarn global add react-devtools

# Npm
npm install -g react-devtools
```

Next open the developer tools from the terminal.
接下来从终端打开开发者工具：
```bash
react-devtools
```

It should connect to any local React Native app that's running.
它应该可以连接到任何正在运行的本地 React Native 应用程序。

> Try reloading the app if developer tools doesn't connect after a few seconds.
> 如果几秒钟后开发者工具未连接，请尝试重新加载应用程序。

[Learn more about debugging React Native.](https://reactnative.dev/docs/debugging)
[了解有关调试 React Native 的更多信息](https://reactnative.dev/docs/debugging)。
