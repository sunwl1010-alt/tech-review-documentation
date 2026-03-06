# Flutter 架构深入

## 一、为什么 Flutter 架构经常被问

Flutter 架构是所有后续知识的起点。

如果这部分只会回答“Flutter 分为 framework、engine、platform”，面试通常会继续追问：

1. framework 和 engine 的边界是什么？
2. Skia 属于哪一层？
3. Flutter 为什么能跨平台保持一致渲染？
4. Widget 和平台原生控件到底是什么关系？
5. JIT、AOT、Dart VM 在 Flutter 中各自承担什么角色？

所以这篇文档的目标不是记名词，而是建立可展开的理解框架。

## 二、Flutter 架构总览

Flutter 可以分成三层：

1. Framework 层
2. Engine 层
3. Embedder / Platform 层

可以用一句话记：

`Framework 负责描述 UI 和业务组织，Engine 负责执行渲染与运行时能力，Embedder 负责接入具体平台。`

## 三、Framework 层

Framework 主要由 Dart 编写，是开发者日常接触最多的一层。

### Framework 主要包含什么

1. Widget 系统
2. 渲染抽象
3. 手势系统
4. 动画系统
5. 路由与导航
6. 状态管理基础能力
7. Material 和 Cupertino 组件库

### 这一层的职责

- 描述页面长什么样
- 管理状态变化后如何重建 UI
- 把声明式 Widget 转换为可布局、可绘制的对象结构

### 关键认知

Framework 并不直接负责“把像素画到屏幕上”，它更像是负责组织和描述。

## 四、Engine 层

Engine 主要由 C++ 实现，是 Flutter 真正执行渲染和运行时能力的核心。

### Engine 主要包含什么

1. Dart Runtime
2. 文本布局
3. 图像解码
4. 合成器
5. 渲染后端
6. 平台通道基础设施
7. 调度和线程模型

### Engine 的职责

- 执行 Dart 代码
- 处理一帧的渲染提交
- 调用底层图形库完成绘制
- 管理资源与线程

## 五、Skia 在哪里

Skia 位于 Engine 的渲染能力部分。

它负责：

- 路径绘制
- 文本渲染
- 图片绘制
- 裁剪
- 渐变
- 阴影
- 混合

高频面试表述：

Flutter 不是直接把 Widget 画到屏幕上，而是先通过 framework 组织出渲染信息，再由 engine 调用 Skia 这类渲染后端完成真实绘制。

## 六、Embedder / Platform 层

这一层负责把 Flutter 接到具体平台运行环境里。

例如：

- Android Embedder
- iOS Embedder
- Windows Embedder
- macOS Embedder
- Linux Embedder

### 它主要负责什么

1. 创建窗口和 Surface
2. 接入平台事件循环
3. 转发触摸、键盘、系统事件
4. 提供 VSync 信号
5. 管理平台线程
6. 对接原生能力

### 为什么这一层重要

因为 Flutter 虽然追求跨平台一致性，但最终还是要运行在具体平台之上。

平台嵌入层就是桥梁。

## 七、Flutter 为什么能跨平台一致

这道题非常高频。

核心原因是：

- Flutter 绝大多数组件不是平台原生控件
- 它自己维护布局和绘制系统
- 它自己控制渲染链路
- 不把最终外观交给 Android View 或 iOS UIView 自己决定

这也是 Flutter 和 React Native 的核心差异之一。

### 优势

1. 跨平台一致性强
2. 自定义 UI 能力强
3. 复杂动画可控性高

### 代价

1. 引擎体积更大
2. 与平台控件体系天然隔离
3. 平台视图混排更复杂

## 八、Dart Runtime、JIT、AOT

### Dart Runtime 做什么

- 执行 Dart 代码
- 管理内存和垃圾回收
- 支持 isolate
- 支持调试和运行时能力

### JIT

主要用于开发阶段。

优点：

- 支持 hot reload
- 修改后反馈快

### AOT

主要用于发布阶段。

优点：

- 启动更快
- 运行性能更稳定
- 减少运行时编译开销

### 面试不要答浅

不是简单说：

- JIT 开发用
- AOT 发布用

更完整的理解是：

Flutter 开发效率很大程度上受益于 JIT 的快速增量反馈，而线上性能和启动体验又依赖 AOT 编译结果。

## 九、从代码到屏幕的执行路径

这是架构题里最关键的主线。

一条相对完整的链路是：

1. 你写 Dart 代码和 Widget
2. Framework 构建 Widget Tree
3. Element Tree 管理生命周期与复用
4. RenderObject Tree 负责布局和绘制
5. Layer Tree 负责合成边界
6. Engine 接收 Scene 信息
7. Raster Thread 调用渲染后端输出像素
8. 平台层提交到系统窗口显示

能讲清这条链，架构问题基本不会失分太多。

## 十、Framework 为什么能声明式开发

Flutter 的声明式本质不是“语法好看”，而是：

- UI 是状态的映射
- 状态变化后重建描述
- 框架负责比较和复用运行时对象

所以开发者更多关注：

- 当前状态是什么
- 当前状态应该对应什么 UI

而不是手动一点点操作原生视图树。

## 十一、Flutter 和原生平台的关系

Flutter 不是完全脱离平台。

它依然依赖平台去完成：

1. 窗口创建
2. 生命周期管理
3. 输入事件分发
4. 权限和系统 API 访问
5. 平台特有能力调用

所以 Flutter 的跨平台不是“不要平台”，而是“尽量把 UI 主导权从平台拿回来”。

## 十二、Platform Channel 在架构中的位置

Platform Channel 是 Framework / Engine 与原生平台沟通的重要机制之一。

常见形式：

- `MethodChannel`
- `EventChannel`
- `BasicMessageChannel`

适合：

- 调系统 API
- 访问设备能力
- 接平台生命周期回调

不适合拿来做高频大量底层计算，这类场景更适合 FFI。

## 十三、FFI 在架构中的位置

FFI 解决的是 Dart 与原生动态库的直接互操作。

适合：

- C/C++ 库
- Rust 动态库
- 高性能计算
- 音视频、图像、加密等底层模块

要区分清楚：

- Platform Channel 偏平台能力交互
- FFI 偏原生库直接调用

## 十四、线程模型与架构的关系

Flutter 架构面试经常会追到线程模型。

你需要知道常见线程：

1. UI Thread
2. Raster Thread
3. Platform Thread
4. IO Thread

### 为什么这很重要

因为很多性能问题本质上不是“代码写得不好”，而是：

- UI 线程做了重任务
- Raster 线程栅格化压力太大
- 平台视图混合过重

架构理解不到线程层，就很难答好性能题。

## 十五、Flutter 架构的核心优势

### 1. 一致性强

跨平台 UI 更统一。

### 2. 自定义能力强

对复杂 UI 和动画更友好。

### 3. 开发体验好

hot reload 提升迭代效率。

### 4. 体系完整

从 Widget 到渲染、从平台交互到多平台支持，链路比较完整。

## 十六、Flutter 架构的核心限制

### 1. 引擎体积

相比纯原生页面或极轻量方案更重。

### 2. 平台嵌入复杂度

与系统原生视图深度混排时更容易遇到问题。

### 3. 需要理解自己的渲染体系

Flutter 不是只学控件 API 就够了，越往中高级走，越需要理解它的底层模型。

## 十七、面试高频问答

### 1. Flutter 架构分几层？

Framework、Engine、Embedder / Platform 三层。

### 2. Flutter 为什么能跨平台一致渲染？

因为它自己掌控布局和绘制链路，而不是主要依赖平台原生控件。

### 3. Skia 属于哪一层？

属于 Engine 渲染能力的一部分，负责真实图形绘制。

### 4. JIT 和 AOT 的区别是什么？

JIT 更适合开发期快速反馈和 hot reload，AOT 更适合发布期性能和启动体验。

### 5. Flutter 和原生平台是什么关系？

Flutter 自己掌控大部分 UI 渲染，但仍依赖平台提供窗口、事件循环、系统 API 和设备能力。

### 6. Platform Channel 和 FFI 的区别是什么？

Platform Channel 用于 Flutter 与平台代码通信，FFI 用于 Dart 与原生动态库直接调用。

## 十八、建议记忆主线

如果要把这篇内容压缩成一条面试可复述主线，可以记成：

`Framework 负责描述和组织 UI，Engine 负责执行和渲染，Embedder 负责接入平台；Flutter 通过自有渲染体系而不是依赖原生控件实现跨平台一致性。`
