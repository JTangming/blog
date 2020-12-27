### Terminology(术语)

- JSI(JavaScript Interface): 它是一个统一的轻量级通用 API，理论上可用于任何 JavaScript VM(虚拟机)。
- TurboModules: NativeModules 的新架构，同样使用的是 JSI。
- CodeGen: a tool to "automate" the compatibility between JS and native side。

### 旧架构现状

所有的 UI 操作(like creating native views, managing children, etc)都被 UIManagerModule 这一个 native module 接收处理，即 React Reconciller 通过 Bridge 发送 ui 指令，在 UIManagerModule 接收并委派给 UIImplementation 来一次创建 shadow nodes。shadow nodes 一颗 layout tree，传递给 Yoga 布局引擎来进行进行计算各个节点的相对坐标。

学习材料
- [PPT](https://speakerdeck.com/kelset/react-native-past-future-and-present?slide=25)
- [Fabric relies on some new strategies](https://github.com/react-native-community/discussions-and-proposals/issues/4#issuecomment-413696591)