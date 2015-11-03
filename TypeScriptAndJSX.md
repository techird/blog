> 之前的一篇翻译[安利了一下 TypeScript](https://github.com/techird/blog/issues/1)，今天再给大家翻译一篇 TypeScript 和 JSX 结合使用的安利文。
> 原文地址：http://www.jbrantly.com/typescript-and-jsx/

对 JSX 的支持已经在 TypeScript 官方落地了！非常感谢 [Ryan Cavanaugh](https://twitter.com/SeaRyanC) 和 [François de Campredon](https://twitter.com/fdecampredon) 的大力推动。在这篇文章中，笔者打算跟大家探索一下如何把 JSX 和 TypeScript 的第一特性——静态类型检查完美结合使用。

## 发展历史

当我们开始在 [AssureSign](https://www.assuresign.com/) 项目中实验性地使用 React 的时候，已经走到了 TypeScript 的路子上。然而当我们碰到 JSX 的时候，一道石墙突然在我们面前挡道。在 [Github 中已经有一个历史悠久的 Issue](https://github.com/facebook/react/issues/759) 去反映这个问题，然而一直没有得出实质性的解决方案，或者干脆让你“不要使用 JSX”了。这对我来说不可接受，所以我[用了一点黑魔法](https://github.com/facebook/react/issues/759#issuecomment-40954893)来解决这个问题。虽然可以玩起来，然而这个方法太挫了，并且没有类型检查。[François de Campredon](https://twitter.com/fdecampredon) 后来创建了一个 [jsx-typescript](https://github.com/fdecampredon/jsx-typescript) 的项目来证明其实 TypeScript 是可以支持 JSX 的。然后，很突然，[会得到 TypeScript 官方支持](https://github.com/Microsoft/TypeScript/issues/296#issuecomment-89266813)的说法就开始起来了。三个月后，它就[真的支持](https://github.com/Microsoft/TypeScript/pull/3564#event-343293342)了。

## 安装

~~目前 JSX 还没有稳定版本，所以你需要去拉取最新代码。~~
JSX 已经在 TypeScript 1.6 或者更高的版本上支持。

```sh
$ npm install typescript
```

## 基础使用

使用 JSX 之前，你需要做两件事情：

1. 文件使用 `.tsx` 作为扩展名
2. 开启 TypeScript 的 `jsx` 选项

TypeScript 发布了两种 JSX 的工作模式：`preserve` 模式和 `react` 模式。这两个模式只影响编译策略。`preserve` 模式会保留 JSX 文件，以便日后进一步被 [babel](http://facebook.github.io/react/blog/2015/06/12/deprecating-jstransform-and-react-tools.html) 使用，输出的文件是 `.jsx` 格式的。而 `react` 模式则会直接编译成 `React.createElement`，在使用之前就不需要再做 JSX 转换了，输出的文件是 `.js` 格式的。

|模式     |输入     |输出
|---------|---------|---------
|preserve |<div/>   |<div/>
|react    |<div/>   |React.createElement("div")
