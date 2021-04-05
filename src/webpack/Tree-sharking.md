 tree shaking，你必须……

使用 ES2015 模块语法（即 import 和 export）。
在项目 package.json 文件中，添加一个 "sideEffects" 入口。
引入一个能够删除未引用代码(dead code)的压缩工具(minifier)（例如 UglifyJSPlugin）。

// todo
- [Webpack 中的 sideEffects 到底该怎么用？](https://github.com/kuitos/kuitos.github.io/issues/41)
- [你的Tree-Shaking并没什么卵用](https://zhuanlan.zhihu.com/p/32831172)
