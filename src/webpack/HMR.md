# HMR

模块热替换(HMR - hot module replacement)功能会在应用程序运行过程中，替换、添加或删除模块，而无需重新加载整个页面，主要有以下几种方式：

- 保留应用程序状态；
- 只更新变更内容；
- 在源代码中 CSS/JS 产生修改时，会立刻在浏览器中进行更新，这几乎相当于在浏览器 devtools 直接更改样式。





webpack-dev-middleware和webpack-dev-server的区别

其实就是因为webpack-dev-server只负责启动服务和前置准备工作，所有文件相关的操作都抽离到webpack-dev-middleware库了，主要是本地文件的编译和输出以及监听，无非就是职责的划分更清晰了。


// todo
- [轻松理解webpack热更新原理](https://juejin.cn/post/6844904008432222215)
- [官文](https://www.webpackjs.com/concepts/hot-module-replacement/)