---
title: 编辑器设置 | Editor Setup
translators:
  - dahui4dev
  - QC-L
---

<Intro>

A properly configured editor can make code clearer to read and faster to write. It can even help you catch bugs as you write them! If this is your first time setting up an editor or you're looking to tune up your current editor, we have a few recommendations.

正确的编辑器配置可以开发事半功倍。并且还可以在编码时帮你提示错误！如果这是你第一次配置编辑器，或者想对现有编辑器进行调整，我们有一些建议可供参考。

</Intro>

<YouWillLearn>

* What the most popular editors are
* 最受欢迎的编辑器是哪些
* How to format your code automatically
* 如何自动格式化你的代码

</YouWillLearn>

## 你的编辑器 | Your editor {/*your-editor*/}

[VS Code](https://code.visualstudio.com/) is one of the most popular editors in use today. It has a large marketplace of extensions and integrates well with popular services like GitHub. Most of the features listed below can be added to VS Code as extensions as well, making it highly configurable!

[VS Code](https://code.visualstudio.com/) 是现如今最流行的编辑器之一。它拥有庞大的扩展市场，同时可以与 GitHub 等流行服务完美集成。下面列出的大多数功能都可以作为扩展添加到 VS Code 中，使其具有极高的可配置性！

Other popular text editors used in the React community include:

React 社区中其他较为流行的文本编辑器包括：

* [WebStorm](https://www.jetbrains.com/webstorm/) is an integrated development environment designed specifically for JavaScript.
* [WebStorm](https://www.jetbrains.com/webstorm/) 是专为 JavaScript 设计的集成开发环境。
* [Sublime Text](https://www.sublimetext.com/) has support for JSX and TypeScript, [syntax highlighting](https://stackoverflow.com/a/70960574/458193) and autocomplete built in.
* [Sublime Text](https://www.sublimetext.com/) 支持 JSX 和 TypeScript，内置[语法高亮](https://stackoverflow.com/a/70960574/458193)和代码自动补全功能。
* [Vim](https://www.vim.org/) is a highly configurable text editor built to make creating and changing any kind of text very efficient. It is included as "vi" with most UNIX systems and with Apple OS X.
* [Vim](https://www.vim.org/) 是一个高度可配置的文本编辑器，可以非常高效地创建和更改任何类型的文本。它作为 “vi” 包含在大多数 UNIX 系统和 Apple OS X 中。

## 推荐的文本编辑器功能 | Recommended text editor features {/*recommended-text-editor-features*/}

Some editors come with these features built in, but others might require adding an extension. Check to see what support your editor of choice provides to be sure!

有些编辑器内置了这些功能，但某些编辑器可能需要添加扩展。你需要确认你的编辑器支持了哪些能力。

### 代码检查 | Linting {/*linting*/}

Code linters find problems in your code as you write, helping you fix them early. [ESLint](https://eslint.org/) is a popular, open source linter for JavaScript. 

代码检查（Code linters）可以在你编写代码时，发现代码中的问题，以帮你尽早修复。[ESLint](https://eslint.org/) 是一款流行且开源的 JavaScript 代码检查工具。

* [Install ESLint with the recommended configuration for React](https://www.npmjs.com/package/eslint-config-react-app) (be sure you have [Node installed!](https://nodejs.org/en/download/current/))
* [使用 React 的推荐配置安装 ESLint](https://www.npmjs.com/package/eslint-config-react-app) （确保你已经安装了 [Node](https://nodejs.org/en/download/current/)）
* [Integrate ESLint in VSCode with the official extension](https://marketplace.visualstudio.com/items?itemName=dbaeumer.vscode-eslint)
* [安装 VSCode 中的官方 ESLint 扩展](https://marketplace.visualstudio.com/items?itemName=dbaeumer.vscode-eslint)

**Make sure that you've enabled all the [`eslint-plugin-react-hooks`](https://www.npmjs.com/package/eslint-plugin-react-hooks) rules for your project.** They are essential and catch the most severe bugs early. The recommended [`eslint-config-react-app`](https://www.npmjs.com/package/eslint-config-react-app) preset already includes them.

**请确保你已经为你的项目启用了 [`eslint-plugin-react-hooks`](https://www.npmjs.com/package/eslint-plugin-react-hooks) 规则**。这在 React 项目中是必备的，同时能帮助你及早的捕获较为严重的 bug。我们推荐的 [`eslint-config-react-app`](https://www.npmjs.com/package/eslint-config-react-app) preset 中已经集成了该规则。

### 格式化 | Formatting {/*formatting*/}

The last thing you want to do when sharing your code with another contributor is get into a discussion about [tabs vs spaces](https://www.google.com/search?q=tabs+vs+spaces)! Fortunately, [Prettier](https://prettier.io/) will clean up your code by reformatting it to conform to preset, configurable rules. Run Prettier, and all your tabs will be converted to spaces—and your indentation, quotes, etc will also all be changed to conform to the configuration. In the ideal setup, Prettier will run when you save your file, quickly making these edits for you.

与其他贡献者共享代码时，你最不想做的事就是争论代码缩进应该使用 [tabs 还是空格](https://www.google.com/search?q=tabs+vs+spaces)！幸好，[Prettier](https://prettier.io/) 会根据预设配置的规则重新格式化代码，以保证代码整洁。运行 Prettier，你的所有 tabs 都将转换为空格，同时缩进、引号等也都将根据你的配置而改变。理想状态下，当你保存文件时，Prettier 会自动执行格式化操作。

You can install the [Prettier extension in VSCode](https://marketplace.visualstudio.com/items?itemName=esbenp.prettier-vscode) by following these steps:

你可以为 [VSCode 安装 Prettier 扩展](https://marketplace.visualstudio.com/items?itemName=esbenp.prettier-vscode)，具体步骤如下：

1. Launch VS Code
- 启动 VS Code
- Use Quick Open (press Ctrl/Cmd+P)
2. 使用快速打开（使用快捷键 `Ctrl/Cmd + P`）
- Paste in `ext install esbenp.prettier-vscode`
3. 粘贴 `ext install esbenp.prettier-vscode`
4. Press Enter
- 按回车键

#### 保存并自动格式化 | Formatting on save {/*formatting-on-save*/}

Ideally, you should format your code on every save. VS Code has settings for this!

理想情况下，你应该在每次保存时自动格式化代码。VS Code 就包含该配置!

1. In VS Code, press `CTRL/CMD + SHIFT + P`.
- 在 VS Code, 按快捷键 `Ctrl/Cmd + Shift + P`.
2. Type "settings"
- 输入 "settings"
3. Hit Enter
- 按回车键
4. In the search bar, type "format on save"
- 在搜索栏, 输入 "format on save"
5. Be sure the "format on save" option is ticked!
- 确保勾选 "format on save" 选项！

> If your ESLint preset has formatting rules, they may conflict with Prettier. We recommend disabling all formatting rules in your ESLint preset using [`eslint-config-prettier`](https://github.com/prettier/eslint-config-prettier) so that ESLint is *only* used for catching logical mistakes. If you want to enforce that files are formatted before a pull request is merged, use [`prettier --check`](https://prettier.io/docs/en/cli.html#--check) for your continuous integration.
>
> 如果你的 ESLint 预设包含格式化规则，它们可能会与 Prettier 发生冲突。我们建议使用 [`eslint-config-prettier`](https://github.com/prettier/eslint-config-prettier) 禁用你 ESLint 预设中的所有格式化规则，这样 ESLint 就只用于捕捉逻辑错误。如果你想在合并 PR 前强制执行文件的格式化，请在你的 CI 中使用 [`prettier --check`](https://prettier.io/docs/en/cli.html#--check) 命令。
