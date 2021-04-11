# 基本概念

loader 用于转换某些类型的模块，如用于对模块的源代码进行转换。loader 可以将文件从不同的语言（如 TypeScript）转换为 JavaScript 或将内联图像转换为 data URL，loader 甚至允许你直接在 JavaScript 模块中 import CSS文件！

Plugin插件则可以用于执行范围更广的任务，目的在于解决 loader 无法实现的其他事，包括：打包优化，资源管理，注入环境变量。

webpack 插件是一个具有 apply 方法的 JavaScript 对象。apply 方法会被 webpack compiler 调用，并且在整个编译生命周期都可以访问 compiler 对象。

runtime 和 manifest

runtime 在浏览器运行过程中，在模块交互时，连接模块所需的加载和解析逻辑。

manifest 数据用途即管理 webpack 所有所需模块之间的交互。当 compiler 开始执行、解析和映射应用程序时，它会保留所有模块的详细要点，这个数据集合称为 "manifest"。

当完成打包并发送到浏览器时，runtime 会通过 manifest 来解析和加载模块。

而每个模块间的依赖关系，则依赖于AST语法树。每个模块文件在通过Loader解析完成之后，会通过[acorn库](https://github.com/acornjs/acorn)生成模块代码的AST语法树，通过语法树就可以分析这个模块是否还有依赖的模块，进而继续循环执行下一个模块的编译解析。

如 import 或 require 语句现在都已经转换为 __webpack_require__ 方法，此方法指向模块标识符(module identifier)。通过使用 manifest 中的数据，runtime 将能够检索这些标识符，找出每个标识符背后对应的模块。

在webpack源码中主要依赖于 compiler 和 compilation 两个核心对象实现。

- compiler 对象是一个全局单例，包含了完整的 webpack 配置，他负责把控整个 webpack 打包的构建流程，即负责文件监听和编译。
- compilation 对象是每一次构建的上下文对象，它包含了当次构建所需要的所有信息，每次热更新和重新构建，compiler 都会重新生成一个新的 compilation 对象，负责此次更新的构建过程。
