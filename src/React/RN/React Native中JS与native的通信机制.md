> 本文不是读源码并对其进行一个详解或整理回放，而是通过理解 RN 整个通信机制以便理解其核心，贯穿始末。

RN 的通信机制涉及 Objective-C/Java、C++、JavaScript 间的交互，这一节先着重介绍 bridge(以 iOS 端为例)。

在开始前，我们先以 NativeModules 数据交互来做对 JS 调用原生模块方法实现 JS to Native 的数据通信。

原生侧定义的可供 RN 调用的模块及其方法，shrinkToFloatView 旨在唤起一个浮窗效果，示例代码如下：
```objc
//  GFCloudManagerModule.h
#import <React/RCTBridgeModule.h>

@interface GFCloudManagerModule : NSObject <RCTBridgeModule>
@end

//  GFCloudManagerModule.m
#import "GFCloudManagerModule.h"

@implementation GFCloudManagerModule
RCT_EXPORT_MODULE(GFCloudManager)

RCT_EXPORT_METHOD(shrinkToFloatView) {
    dispatch_async(dispatch_get_main_queue(), ^{
    [[GFRemoteVideoAssistantManager sharedInstance] fullScreenShrinkToFloatBall];
    });
}
@end
```

在 JS 侧调用原生模块并执行 shrinkToFloatView 方法：
```js
import { NativeModules } from 'react-native';
var GFCloudManager = NativeModules.GFCloudManager;
GFCloudManager.shrinkToFloatView();
```

以上例子实现了 JS 到 Native 的数据交互通信，理论上讲，JS 和 OC 谁也不是认识谁，怎么做到模块和方法互调的呢？答案是 bridge，交互的原理图如下：
![bridge](./../../../../blog/images/rn/bridge.jpg)

接下来看看它做了什么，是怎样实现数据互通的。

React Native 用 iOS 自带的 JavaScriptCore 作为 JS 的解析引擎，要实现 JS to Native 的通信并非难事，主要有途径也就是传统的 jsBridge，参考[JS 与 Native 的几种通信方案](https://awhisper.github.io/2018/01/02/hybrid-jscomunication/)。但 RN 并没有使用到 JSC 提供的可以让 JS 和 OC 互调的特性，而是自己实现了一套通信机制。

### 模块配置信息(JSON messages)

首先 OC 侧要告诉 JS 它有哪些模块，模块里包含有哪些方法，JS 调用原生模块的方法是建立在有这些方法后才有可能让其去调用这些方法。

我们知道，要将 Native module(类、接口)曝露给 JS，module 需要实现 RCTBridgeModule 协议，并且在实现中要插入 RCT_EXPORT_MODULE 宏，具体曝露的方法也需要通过 RCT_EXPORT_METHOD 宏定义。

先来了解上面提到的两个关键的宏，用作收集所有需要暴露给 JS 的类和方法，将此宏放在将要实现的类中，在初始化加载时能自动向 bridge 注册相应的 module。

TODO: 收集 ModuleID、MethodID、CallbackID and args.

#### 编译时
RCT_EXPORT_MODULE以及RCT_EXPORT_METHOD宏属于编译阶段的处理
#### 注册NativeModule
通过实例化 RCTBridge、RCTCxxBridge

 // Initialize all native modules that cannot be loaded lazily
  [self _initModulesWithDispatchGroup:prepareBridge];

所有需要曝露给 JS 的 module 都已注册完成，并以RCTModuleData格式存储在RCTCxxBridge中。

#### Instance
_initializeBridge方法做的最重要的事情就是初始化Instance实例_reactInstance，此过程将所有曝露给 JS 的 module 由RCTModuleData格式转化为ModuleRegistry格式传入Instance。

// This is async, but any calls into JS are blocked by the m_syncReady CV in Instance
- (std::shared_ptr<ModuleRegistry>)_buildModuleRegistry

#### JSCExecutor
JSCExecutor的构造函数做了一条非常重要的事情：在 JS Context 中设置了一个全局代理nativeModuleProxy，其最终指向JSCExecutor类的getNativeModule方法。

#### NativeToJsBridge
NativeToJsBridge是 Native to JS 的桥接,所有从 Native 到 JS 的调用都是从NativeToJsBridge中的接口发出去的。

在构造函数中也将native module 注册信息转换为JSCNativeModules格式存储了下来。

JS 是怎样把数据传给 OC，让 OC 去调相应方法的？

JS 代码是运行在 JS 线程而非 main thread，JS 不会主动传递数据给 OC，在调 OC 方法时，会在上述前面步骤收集到的 ModuleID、MethodID 等数据加到一个队列里，等 OC 过来调JS的任意方法时，再把这个队列返回给 OC，此时OC再执行这个队列里要调用的方法。

native开发里，什么时候会执行代码？只在有事件触发的时候，这个事件可以是启动事件，触摸事件，timer事件，系统事件，回调事件。而在React Native里，这些事件发生时OC都会调用JS相应的模块方法去处理，处理完这些事件后再执行JS想让OC执行的方法，而没有事件发生的时候，是不会执行任何代码的，这跟native开发里事件响应机制是一致的。

这一过程不必纠结，大致是 RCTBridge、RCTCxxBridge、NativeToJSBridge、JSExecutor、JsToNativeBridge的初始化，这里仅作为一个了解。

### Bridge 的调用流程
1. JS端调用某个OC模块暴露出来的方法;
2. 把上一步的调用分解为ModuleName,MethodName,arguments，再扔给MessageQueue处理，在这一步把JS的callback函数缓存在MessageQueue的一个成员变量里;
3. 在通过保存在MessageQueue的模块配置表把上一步传进来的ModuleName和MethodName转为ModuleID和MethodID;
4.把上述步骤得到的ModuleID,MethodId,CallbackID和其他参数argus传给OC;
5. OC接收到消息，通过模块配置表拿到对应的模块和方法；
6. 根据 RCTModuleMethod 对JS传过来的每一个参数进行处理，执行对应模块对应的方法；
7. 


整个流程就是这样，简单概括下，差不多就是：JS函数调用转ModuleID/MethodID -> callback转CallbackID -> OC根据ID拿到方法 -> 处理参数 -> 调用OC方法 -> 回调CallbackID -> JS通过CallbackID拿到callback执行