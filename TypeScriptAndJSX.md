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

|模式     |输入           |输出
|---------|---------------|---------
|preserve |&lt;div/&gt;   |&lt;div/&gt;
|react    |&lt;div/&gt;   |React.createElement("div")

你可以在命令行中使用 `--jsx` 参数或者在 [tsconfig.json](https://github.com/Microsoft/TypeScript/wiki/tsconfig.json) 中指定工作模式。

*注意，编译生成的代码中，`React` 关键字是硬编码的，所以要保证全局变量中的 React 可用，并且 R 是大写的，`react` 就不行了。*

## `as` 运算符

由于 TypeScript 使用尖括号做类型声明，那么在编译的时候，类型声明会和 `JSX` 的语法冲突。比如以下代码：

```tsx
var foo1 = <foo>bar
</foo>
```

这个代码是准备创建一个 JSX 元素，还是想要声明 `bar` 变量是 `foo` 类型的，而第二行只是个无效表达式？为了让这个问题简化，`.tsx` 文件里不支持尖括号类型声明语法。所以如果上面的代码是在 `.tsx` 文件里的话，表示的是创建了一个 JSX 元素，而如果在 `.ts` 文件里，就会报错。为了弥补 `.tsx` 文件中类型声明语法的缺失，添加了一个新的类型声明操作符：`as`。

```tsx
var foo1 = bar as foo;
```

`as` 操作符在 `.ts` 文件和 `.tsx` 文件中都是可用的。

## 类型检查

要是没有类型检查，JSX 和 TypeScript 搞基有什么意义？多亏有了完整的类型检查，世界变得精彩纷呈。

首先，要区别内置元素和自定义元素。给定一个 JSX 表达式 `<expr />`，这个 `expr` 到底该对应环境里的一个内置元素（比如 DOM 环境下的 div 或者 span）还是应该对应一个自己写的自定义组件？区分这单很重要，因为：

1. 对于 React 来说，内置元素会编译成字符串，比如 `React.createComponent('div')`，而自定义元素则不是：`React.createElement(MyComponent)`
2. 传递给 JSX 标签的属性类型的查找方式是不同的。内置元素支持的属性应该是内置的，而自定义组件的属性取决于 props 的定义。

对于这两者，TypeScript 和 React 使用[同样的区分方式](http://facebook.github.io/react/docs/jsx-in-depth.html#html-tags-vs.-react-components)：内置元素的标签以小写字母开头，自定义元素的标签使用大写字母开头。

## 内置元素

内置元素会在接口 `JSX.InterinsicElements` 中定义。默认情况下，如果这个接口并没有定义，那么所有的内置元素都不会进行类型检查。然而，如果你定义了这个接口，那么内置元素将会在接口的属性中定义。

```tsx
declare module JSX {  
    interface IntrinsicElements {
        foo: any
    }
}

<foo />; // ok  
<bar />; // error  
```

在上述示例中，`foo` 可以通过类型检查但是 `bar` 不行。因为 `bar` 并没有在内置元素接口中定义到。

*注意：你也可以在 `JSX.IntrinsicElements` 接口中定义一个字符串索引器来匹配任意的内置元素*

> 笔者注：`JSX.IntrinsicElements` 在 Dom 环境中的定义[已经在 DefinitelyTyped 中描述好了](https://github.com/borisyankov/DefinitelyTyped/blob/master/react/react.d.ts#L1903-L2037)。大家可以直接使用。

## 自定义元素

自定义元素可以简单地通过标识符来查找到。

```tsx
import MyComponent = require('./myComponent');

<MyComponent />; // ok  
<SomeOtherComponent />; // error  
```

我们是可以限制自定义元素的类型的。然而，为了说明白，我们先要介绍两个概念：*元素类类型(Element Class Type)*和*元素实例类型(Element Instance Type)*。

元素类类型很简单，对于 `<Expr>` 组件，类类型就是 `Expr` 的类型。所以在上面的例子中，如果 `MyCompoent` 是一个 ES6 的类，那么 `<MyComponent>` 的类类型就是这个类。如果 `MyComponent` 是一个工厂方法，那么 `<MyCompoent>` 的类类型就是这个方法。

一旦元素类类型确定了，元素的实例类型就由类的调用签名和构造签名共同决定。在看上述例子，如果 `MyComponent` 是一个 ES6 的类，那么实例类型就是这个类的实例的类型，如果是个工厂方法，实例类型则是这个方法的返回值。是不是有点绕？我们来看看下面这个例子：

```tsx
class MyComponent {  
  render() {}
}

// 使用构造函数
var myComponent = new MyComponent();

// 元素类类型 => MyComponent
// 元素示例类型 => { render: () => void }

function MyFactoryFunction() {  
  return { 
    render: () => {
    } 
  }
}

// 使用工厂方法调用
var myComponent = MyFactoryFunction();

// 元素类类型 => FactoryFunction
// 元素实例类型 => { render: () => void }
```

现在元素实例类型是一个有趣的类型，它要求满足 `JSX.ElementClass` 的接口，否则将会报错。默认情况下，`JSX.ElementClass` 就是 `{}`，不过我们是可以认为增加限制，让它适应 JSX 的接口。

```tsx
declare module JSX {  
  interface ElementClass {
    render: any;
  }
}

class MyComponent {  
  render() {}
}
function MyFactoryFunction() {  
  return { render: () => {} }
}

<MyComponent />; // ok  
<MyFactoryFunction />; // ok

class NotAValidComponent {}  
function NotAValidFactoryFunction() {  
    return {};
}

<NotAValidComponent />; // error  
<NotAValidFactoryFunction />; // error  
```

*译者注：`JSX.ElementClass` 同样[在 DefinitelyTyped 中定义好了](https://github.com/borisyankov/DefinitelyTyped/blob/master/react/react.d.ts#L1898-L1900)，大家通过 tsd 或者 nuget 可以下载下来使用*

## 属性类型检查

做属性的类型检查，第一部就是要确定*元素属性类型*。对于内置元素和自定义元素，确定方式有一些区别。

对于内置类型，就是 `JSX.IntrinsicElements` 上的属性类型。

```tsx
declare module JSX {  
  interface IntrinsicElements {
    foo: { bar?: boolean }
  }
}

// `foo` 的元素属性类型是 `{bar?: boolean}`
<foo bar />;  
```

对于自定义元素，情况复杂一点点。它的类型是之前讨论的*元素实例类型*的一个属性决定的。你问我是哪个属性？哈哈，你可以自己决定！在 `JSX.ElementAttributesProperty` 上定义唯一一个属性，这个属性就是决定自定义元素属性类型的属性。*（译者：好绕，还是看示例）*

```tsx
declare module JSX {  
  interface ElementAttributesProperty {
    props; // 指定使用哪个属性名称
  }
}

class MyComponent {  
  // 在元素实例类型上定义这个属性
  props: {
    foo?: string;
  }
}

// `MyComponent` 的元素属性类型就是 `{foo?: string}`
<MyComponent foo="bar" />  
```

上面的例子可以明显看到，元素属性类型就是用于对 JAX 的元素做类型检查的。支持可选和必选属性。

```tsx
declare module JSX {  
  interface IntrinsicElements {
    foo: { requiredProp: string; optionalProp?: number }
  }
}

<foo requiredProp="bar" />; // ok  
<foo requiredProp="bar" optionalProp={0} />; // ok  
<foo />; // error, requiredProp is missing  
<foo requiredProp={0} />; // error, requiredProp should be a string  
<foo requiredProp="bar" unknownProp />; // error, unknownProp does not exist  
<foo requiredProp="bar" some-unknown-prop />; // ok, because `some-unknown-prop` is not a valid identifier  
```

*注意：如果一个属性名称不是一个合法的 JS 标识符（比如 `data-*` 属性），那么这个属性在进行类型检查的时候不会认为是错误，而是直接跳过。*

属性集也是支持的：

```tsx
var props = { requiredProp: 'bar' };  
<foo {...props} />; // ok

var badProps = {};  
<foo {...badProps} />; // error  
```

## JSX 的类型

好了，现在我们可以写 JSX 并且有了元素和属性的类型检查，但是 JSX 本身的类型是什么呢？默认来说，JSX 的类型是 `any`。你可以通过 `JSX.Element` 接口自定义这个类型。然而，我们从这个接口是不可能知道元素、元素的或者子节点的类型信息的。这是个黑盒。

## 内嵌 TypeScript

JSX 在 Javascript 里允许通过花括号 `{ }` 内嵌 JavaScript 代码。JSX 在 TypeScript 里同样允许你这样做，只不过是内嵌 TypeScript 代码。这就意味着这些 JSX 内嵌的 TypeScript 代码同样支持转译功能以及类型检查。

```tsx
var a = <div>  
  {['foo', 'bar'].map(i => <span>{i/2}</span>)}
</div>
```

上面的代码会报错，因为你不能使用字符串来除以一个数值。当使用 `preserve` 模式的时候，输出是这样的：

```tsx
var a = <div>  
  {['foo', 'bar'].map(function (i) { return <span>{i / 2}</span>; })}
</div>
```

## React 集成

TypeScript 对 JSX 的支持的设计是不考虑其使用方式的。然而，React 始终是主要的使用方。我年初[做过一个关于集成 React 和 TypeScript 的演讲](https://www.youtube.com/watch?v=9PTa9-PPVAc)。很多观点如今依然适用。[React 的 DefiniteTyped 仓库](https://github.com/borisyankov/DefinitelyTyped/tree/master/react)的最近更新中集成了 `JSX.IntrinsicElements` 和 `JSX.ElementAttributeProperty` 的概念。

```tsx
/// <reference path="react.d.ts" />

interface Props {  
  foo: string;
}

class MyComponent extends React.Component<Props, {}> {  
  render() {
    return <span>{this.props.foo}</span>
  }
}

<MyComponent foo="bar" />; // ok  
<MyComponent foo={0} />; // error  
```

## 更多资源

这篇文章中的很多信息都来自 TypeScript 的 [Issue #3203](https://github.com/Microsoft/TypeScript/issues/3203)。不过，这个讨论已经持续了很长时间并且扩散到很多个地方。我调了几个个人认为值得深挖的：

* http://typescript.codeplex.com/workitem/2608
* https://github.com/facebook/react/issues/759
* https://github.com/Microsoft/TypeScript/issues/296
* https://github.com/Microsoft/TypeScript/pull/3201
* https://github.com/Microsoft/TypeScript/issues/3203
* https://github.com/Microsoft/TypeScript/pull/3564
