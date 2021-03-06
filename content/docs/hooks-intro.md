---
id: hooks-intro
title: Introducing Hooks
permalink: docs/hooks-intro.html
next: hooks-overview.html
---

*Hooks*是React v16.7.0-alpha中加入的新特性。它可以让你在class以外使用state和其他React特性。你可以在[这里](https://github.com/reactjs/rfcs/pull/68)看到关于它的一些讨论。

```js{4,5}
import { useState } from 'react';

function Example() {
  // Declare a new state variable, which we'll call "count"
  const [count, setCount] = useState(0);

  return (
    <div>
      <p>You clicked {count} times</p>
      <button onClick={() => setCount(count + 1)}>
        Click me
      </button>
    </div>
  );
}
```

`useState`这个方法是我们接触到的第一个"hook"。我们用它完成了一个最简单的组件，接下来我们会看到更多有趣的应用。

**你可以立即跳到[下一页](/docs/hooks-overview.html)开始学习Hooks。** 在这一页，我们将继续解释为什么我们要在React中引入Hooks，以及它们将如何帮助你写出超棒的React应用。

## 视频介绍

在React Conf 2018，Sophie Alpert 和 Dan Abramov介绍了Hooks，然后Ryan Florence演示了如何用它们重构我们的应用。你可以在这里看到这个视频：

<br>

<iframe width="650" height="366" src="//www.youtube.com/embed/dpw9EHDh2bM" frameborder="0" allowfullscreen></iframe>

## No Breaking Changes

在我们继续学习之前，你需要注意：

* **完全可选** 如果你喜欢Hooks，你可以立即在一些组件和已经存在的代码中使用它们。不过如果你不喜欢也不要紧，你完全没必要学习或者使用它们。
* **100% 向后兼容** Hooks不包含任何爆炸性的更新。
* **立即可用** Hooks现在已经包含在alpha版本中。而且我们期望在接受社区反馈后把他们加入到React 16.7版本中。

**classes不会被移除** 你可以在这一页的[最后一节](#gradual-adoption-strategy)读到更多关于Hooks的渐进策略。

**Hooks不会影响你对React的理解** 恰恰相反，Hooks为React的这些概念提供了更直接的API。之后你将会看到，有了Hooks，你可以以一种更加强大的方式将props, state, context, refs 和生命周期整合起来。

**如果你只是想要学学Hooks如何使用，你可以直接跳到[下一页](/docs/hooks-overview.html)。** 当然你也可以继续读这一页。你将会在这里了解到更多我们引入Hooks的原因，以及要如何在已有的代码上使用Hooks重构我们的应用。

## 动机

Hooks解决了我们在React发布至今的五年来遇到的一系列看似不相关的问题。不论你是刚刚开始学习React，或是一直在用它，甚至你只是在使用与React具有相似组件模型的框架，你一定或多或少注意到这些问题。

### 跨组件复用stateful logic(包含状态的逻辑)十分困难

React没有提供一种将可复用的行为“attach”到组件上的方式（比如redux的connect方法）。如果你已经使用了一段时间的React，你可能对[render props](/docs/render-props.html) 和 [高阶组件](/docs/higher-order-components.html)有一定的了解，它们的出现就是为了解决逻辑复用的问题。但是这些模式都要求你重新构造你的组件，这可能会非常麻烦。在很多典型的React组件中，你可以在React DevTool里看到我们的组件被层层叠叠的providers, consumers, 高阶组件, render props, 和其他抽象层包裹。当然你可以通过[筛选功能](https://github.com/facebook/react-devtools/pull/503)把它们全部都过滤掉，但是这种现象也指出了一些更深层次的问题：React需要一些更好的底层元素来复用stateful logic.

使用Hooks，你可以在将含有state的逻辑从组件中抽象出来，这将可以让这些逻辑容易被测试。同时，**Hooks可以帮助你在不重写组件结构的情况下复用这些逻辑。** 这样这些逻辑就可以跨组件复用，甚至你可以将他们分享到社区中。

我们将在[自定义Hooks](/docs/hooks-custom.html)中继续这一部分的讨论。

### 复杂的组件难以理解

我们在刚开始构建我们的组件时它们往往很简单，然而随着开发的进展它们会变得越来越大、越来越混乱，各种逻辑在组件中散落的到处都是。每个生命周期钩子中都包含了一堆互不相关的逻辑。比如我们常常在`componentDidMount` 和 `componentDidUpdate` 中拉取数据，同时`compnentDidMount` 方法可能又包含一些不相干的逻辑，比如设置事件监听（之后需要在 `componentWillUnmount` 中清除）。最终的结果是强相关的代码被分离，反而是不相关的代码被组合在了一起。这显然会导致大量错误。

在许多情况下，我们也不太可能将这些组件分解成更小的组件，因为stateful logic到处都是。测试它们也很困难。这是许多人喜欢将React与单独的状态管理库结合使用的原因之一。然而，这通常会引入太多的抽象，需要在不同的文件之间跳转，并且使得重用组件更加困难。

为了解决这个问题，**Hooks允许您根据相关部分(例如设置订阅或获取数据)将一个组件分割成更小的函数**，而不是强制基于生命周期方法进行分割。您还可以选择使用一个reducer来管理组件的本地状态，以使其更加可预测。

我们将在[使用Effect Hook](/docs/hooks-effect.html#tip-use-multiple-effects-to-separate-concerns)中继续这一部分的讨论。

### 不止是用户，机器也对Classes难以理解

据我们观察，classes是学习React的最大障碍。您必须了解`this`如何在JavaScript中工作，这与它在大多数语言中的工作方式非常不同。必须记住绑定事件处理程序。没有稳定的[语法提案](https://babeljs.io/docs/en/babel-plugin-transform-class-properties/)，代码非常冗长。尽管人们可以很好地理解props、state和自顶向下的数据流，但仍然要与类做斗争。React中功能和类组件的区别以及何时使用每种组件都会导致有经验的React开发人员之间的分歧。

除此以外，React已经发布了五年，我们还想在未来的五年让他保持稳定。就像在[Svelte](https://svelte.technology/), [Angular](https://angular.io/), [Glimmer](https://glimmerjs.com/)和其他框架中展示的那样，组件的提前编译潜力巨大。尤其是在它不局限于模板的时候。最近我们在测试使用[Prepack](https://prepack.io/)来进行[component folding](https://github.com/facebook/react/issues/7323)，而且我们已经初步看到了成果。然而我们发现class组件可能会导致一些让我们做的这些优化白费的编码模式。类也为今天的工具带来了不少的issue。比如，classes不能很好的被minify，同时他们也造成了太多不必要的组件更新。我们想要提供一种便于优化的API。

To solve these problems, **Hooks let you use more of React's features without classes.** Conceptually, React components have always been closer to functions. Hooks embrace functions, but without sacrificing the practical spirit of React. Hooks provide access to imperative escape hatches and don't require you to learn complex functional or reactive programming techniques.

>Examples
>
>[Hooks at a Glance](/docs/hooks-overview.html) is a good place to start learning Hooks.

## Gradual Adoption Strategy

>**TLDR: There are no plans to remove classes from React.**

We know that React developers are focused on shipping products and don't have time to look into every new API that's being released. Hooks are very new, and it might be better to wait for more examples and tutorials before considering learning or adopting them.

We also understand that the bar for adding a new primitive to React is extremely high. For curious readers, we have prepared a [detailed RFC](https://github.com/reactjs/rfcs/pull/68) that dives into motivation with more details, and provides extra perspective on the specific design decisions and related prior art.

**Crucially, Hooks work side-by-side with existing code so you can adopt them gradually.** We are sharing this experimental API to get early feedback from those in the community who are interested in shaping the future of React — and we will iterate on Hooks in the open.

Finally, there is no rush to migrate to Hooks. We recommend avoiding any "big rewrites", especially for existing, complex class components. It takes a bit of a mindshift to start "thinking in Hooks". In our experience, it's best to practice using Hooks in new and non-critical components first, and ensure that everybody on your team feels comfortable with them. After you give Hooks a try, please feel free to [send us feedback](https://github.com/facebook/react/issues/new), positive or negative.

We intend for Hooks to cover all existing use cases for classes, but **we will keep supporting class components for the foreseeable future.** At Facebook, we have tens of thousands of components written as classes, and we have absolutely no plans to rewrite them. Instead, we are starting to use Hooks in the new code side by side with classes.

## Next Steps

By the end of this page, you should have a rough idea of what problems Hooks are solving, but many details are probably unclear. Don't worry! **Let's now go to [the next page](/docs/hooks-overview.html) where we start learning about Hooks by example.**
