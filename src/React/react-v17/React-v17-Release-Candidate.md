### React v17.0 Release Candidate: No New Features

原文：https://reactjs.org/blog/2020/08/10/react-v17-rc.html

React 于 2020.08.10 发布了 React v17.0 Release Candidate，距离上一个 release 大版本已有两年半的时间，下文将描述这个主版本的功能和大家的一些预期，以及如何尝试使用这个版本。

#### 没有新特性
React 17 release 不是一个寻常的版本，因为没有新加任何面向开发人员的特性（developer-facing features），相反，这个 release 版本主要 focus 在使得 React 自升级更加简单（making it easier to upgrade React itself）。

我们一直围绕 react 新特性积极地开展工作，React 17 是我们战略中重要的一环（to roll them out without leaving anyone behind）。

实际上，React 17 是一个”stepping stone“版本，可以更安全地将由一个版本的React管理的树嵌入到由其他版本的React管理的树中。

#### 渐进式升级
在过去 7 年中， React 升级采用 all-or-nothing 的方式，要么依旧沿用旧版，要么将整个应用全部升级，这可不是渐进式的玩法。

这种局面到目前为止得到解决了，但是我们正陷入“全有或全无”升级策略的局限。一些 API 改变了，例如，废弃了 [legacy context API](https://reactjs.org/docs/legacy-context.html)，这不可能以自动化的方式完成。即使目前很多应用已经不再使用这个 API，但我们依然在 React 中保留它。我们选择无限期地在 React 中支持，还是已有的一些应用程序不再兼容新 React 特性，这两个选项都不是很好。

所以，我们想提供另一种选择。

React 17 支持渐进式升级（React 17 enables gradual React upgrades），当你更新如从 React 15 更新到 16（或从 16 -> 17），通常是直接升级整个应用，这在大多情况是有效的，但是，如果代码库是在几年前编写的，并且没有维护不再频繁，它可能会变得越来越具有挑战性。尽管可以使用不同的版本维护，但是在 React 17 之前的版本，存在很大的隐患。

我们正在 React 17 修复这些问题，这就意味着在 React 18 或者后边的特性版本出来后，你将由更多的选择。第一个选择是立即升级整个应用程序，就像以前升级的套路一样。但是，你也可以选择一点点的升级上来，例如，你可以将应用大部分程序迁移到 React 18，但任然可以保留 React 17 上的一些如延迟加载的对话框或子路由。

这并不意味着不得不逐步升级，对于大多数应用程序，一次性升级可能是最好的选择。

要启用渐进式更新，我们需要对 React 事件系统进行一些更改。React 17 作为一个主版本是因为有潜在的 breaking change，事实上，我们也确实做了大量的组件改动，以使我们的 React 17 能够顺利的升级。

#### 渐进升级的例子
我们准备了一个 [example repository](https://github.com/reactjs/react-gradual-upgrade-demo/)来演示了如何在必要时延迟加载旧版本的React。这个 demo 使用  Create React App，但是可能像之前一样需要借助一些工具。

> 我们已将其他更改推迟到React 17之后。此版本的目标是实现逐步升级。如果升级到React 17太困难了，那将无法实现其目标。

#### 更改事件委托
在 React 17 event handlers 将不再绑定到 document元素上，取而代之的是，它将被绑定到根 DOM 容器上。
```js
const rootNode = document.getElementById('root');
ReactDOM.render(<App />, rootNode);
```

在 React 16 或更早的版本, React 将通过 `document.addEventListener()` 监听绑定事件，而 React 17 调用 `rootNode.addEventListener()`。

![react_17_delegation](https://reactjs.org/static/bb4b10114882a50090b8ff61b3c4d0fd/31868/react_17_delegation.png)

为什么做这样的改动？主要是为了 react tree 管理更加安全，没有嵌套带来的安全隐患。这就是为什么建议尽可能的更新到 React 17 版本。

**修复潜在问题**
与以往的重大变更一样，这个版本也做了很多的代码改动，调整了至少有 10 个模块的代码来支持这个版本。

例如，通过`document.addEventListener(...)`来监听一个事件，在 React 16 中，即使在回调函数中通过`e.stopPropagation()`来阻止冒泡我们依旧在 document 能收到事件，而在 React 17 中则不然，React 17 的合成事件更加接近于原生的事件。

#### 其它 breaking changes
React 17 尽可能的保持变动最小，例如，不会移除任何旧版已经废弃的 API，但是，它确实包括其他一些重大更改，根据我们的经验，这些更改相对安全。由于这些原因，我们必须在100,000多个组件中调整少于20个。

##### 与浏览器对齐
我们对事件系统进行了一些较小的更改：
- onScroll事件不再冒泡以防止常见的混乱
- React onFocus 和 onBlur 事件改为使用原生的 focusin 和 focusout 事件
- 捕获阶段事件（例如onClickCapture）现在使用实际的浏览器捕获阶段侦听器。

##### 没有事件池
React 17 移除了事件池，事件池并不能带来性能改观且干扰用户对事件的理解。举个 react 16 的例子：
```js
function handleChange(e) {
  setData(data => ({
    ...data,
    // This crashes in React 16 and earlier:
    text: e.target.value
  }));
}
```
例子中，如果不使用`e.persist()`则事件引用是不行的。在 React 17 中得以改正，旧版的事件池已经完全移除了。

##### Effect Cleanup Timing
例如使用 useEffect 来管理副作用：
```js
useEffect(() => {
  // This is the effect itself.
  return () => {
    // This is its cleanup.
  };
});
```

在 React 16 中相当于是在 `componentWillUnmount` 中来处理副作用，但是这是个同步的操作，一定程度是能影响用户体验的，在 React 17 中改成了异步的方式，也就是说在组件卸载中的状态时，清理副作用的操作将会在屏幕渲染完成后操作。

> React 16 中可以使用 useLayoutEffect 来确保页面不卡顿

##### 组件返回undefined处理
在 React 16 及更早的版本，如果在组件中返回undefined则会出错，如：
```js
function Button() {
  return; // Error: Nothing was returned from render
}
```
或
```js
function Button() {
  // We forgot to write return, so this component returns undefined.
  // React surfaces this as an error instead of ignoring it.
  <button />;
}
```

在函数组件或者类组件中是可以有错误边界帮忙处理，报错后沿用之前的页面来展示，但是使用了 forwardRef 和 memo 包装的组件则真的会出问题了，React 17 对此做了修复，维持和类、函数组件一致的效果。

```js
let Button = memo(() => {
  // We forgot to write return, so this component returns undefined.
  // React 17 surfaces this as an error instead of ignoring it.
  <button />;
});
```

##### Native Component Stacks
In React 17, the component stacks are generated using a different mechanism that stitches them together from the regular native JavaScript stacks. This lets you get the fully symbolicated React component stack traces in a production environment.

##### Removing Private Exports

In React 17, these private exports have been removed. As far as we’re aware, React Native for Web was the only project using them, and they have already completed a migration to a different approach that doesn’t depend on those private exports.

#### 总结
