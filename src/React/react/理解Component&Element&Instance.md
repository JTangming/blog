在理解 react 原理或者阅读源码过程中，需要整明白这几个概念，Component, Element, Instance 之间有什么区别和联系？

- Element 是一个纯 JavaScript Object
对于每一个组件来说，render 函数返回的也正是一个 element，而不是真正的 DOM 节点。JSX 是 React.createElement(type, props, ...children) 函数的语法糖（[try it out](https://link.zhihu.com/?target=https%3A//babeljs.io/repl/)），createElement 函数最终返回的是 Element。它的结构是：
```js
{
  type: 'div',
  props: {
    className,
    children,
  }
}
```

> Element 的 type 可以是 string , 也可以是 function| class。

> 其中 children 的数据结构是单个 Element 或 Element 数组，这样的数据结构满足了递归的需要。

- Component
组件(Component)有以下几种类型：
    - DOMComponent:，如 `div, span, ul` 等
    - Composite Component（复合组件），分为 functional component 和 class component
    - TextComponent，即 number or string
    
- Instance 是 Component 的实例化之后的对象
在 React 中 instances 的概念并不是十分重要，因为我们只需通过 Elements 来描述界面 UI 或者声明 Component，React 自己会管理 Component 的 instance。通过实例化 Component 后，我们通过 this 能做许多的操作，如 setState。

> 注，functional component 是纯函数，它们并没有实例

### Reference
- [React Components, Elements, and Instances](https://reactjs.org/blog/2015/12/18/react-components-elements-and-instances.html)