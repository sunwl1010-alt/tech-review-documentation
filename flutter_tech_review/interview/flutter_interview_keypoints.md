# Flutter 面试高频考点

## 一、底层原理高频题

### 1. Flutter 为什么能跨平台保持一致 UI？

核心回答：

- Flutter 不是主要依赖平台原生控件绘制
- Flutter 自己维护 Widget、RenderObject 和渲染管线
- 最终通过引擎把绘制命令交给底层渲染后端输出

追问点：

- 这和 React Native 的思路有什么区别？
- 这样做的优劣势是什么？

### 2. Flutter 中 Widget、Element、RenderObject 的区别是什么？

核心回答：

- Widget：不可变配置
- Element：运行时树节点，负责生命周期和复用
- RenderObject：负责布局、绘制、命中测试

高频追问：

- 为什么 Widget 可以频繁创建？
- 哪一层最重？

### 3. Flutter 一帧是怎么产生的？

核心回答：

`build -> layout -> paint -> compositing -> raster -> GPU 显示`

高频追问：

- 哪一步最容易导致 UI 卡顿？
- 哪一步最容易导致 GPU / Raster 卡顿？

### 4. Skia 在 Flutter 中负责什么？

核心回答：

- Skia 是 Flutter 长期依赖的图形渲染引擎
- 负责路径、文本、图片、裁剪、渐变、阴影等图形输出
- Flutter framework 负责声明和组织 UI，Skia 负责真正绘制

高频追问：

- Skia 和 Flutter Engine 的关系是什么？
- Skia 和 Impeller 有什么区别？

## 二、渲染与性能高频题

### 5. Flutter 为什么会掉帧？

核心回答：

- 一帧需要在预算时间内完成
- 60Hz 下约 16.67ms，120Hz 下约 8.33ms
- UI Thread 或 Raster Thread 任一超时都可能掉帧

常见原因：

- build 过重
- layout 复杂
- 图片过大
- 透明层和模糊太多
- 主 isolate 有大计算

### 6. `const` 为什么能优化性能？

核心回答：

- 减少重复创建相同 Widget 配置
- 有助于框架识别稳定子树
- 降低不必要 rebuild 成本

注意：

- `const` 不是万能优化
- 真正大头仍然可能在布局、绘制和栅格化

### 7. `RepaintBoundary` 的作用是什么？

核心回答：

- 隔离重绘边界
- 避免局部动画带动整块区域 repaint

追问点：

- 为什么滥用会变差？

应答：

- 因为会增加 Layer 数量、内存占用和合成成本

### 8. 如何定位 Flutter 性能问题？

核心回答：

1. 用 profile 模式复现
2. 用 DevTools 看线程和帧时间
3. 判断是 UI 卡还是 Raster 卡
4. 再针对 build/layout/paint 或图片/图层问题处理

## 三、状态管理高频题

### 9. 你怎么选 Provider、Bloc、GetX、Riverpod？

建议回答方式：

- 小中型项目、团队简单：Provider / Riverpod
- 复杂业务流、事件驱动强：Bloc
- 强调开发效率、接受约定式风格：GetX
- 更关注可测试性、依赖管理和组合能力：Riverpod

不要回答成“哪个好用就用哪个”。

### 10. 为什么状态管理不只是选库？

核心回答：

状态管理的本质是：

- 状态边界怎么划分
- 生命周期怎么管理
- UI 和业务怎么解耦
- 异步状态如何表达

库只是实现手段，不是设计本身。

## 四、异步与并发高频题

### 11. Dart 的 event loop 和 microtask 有什么区别？

核心回答：

- microtask 优先级高于 event queue
- `Future.microtask` 会先于普通事件执行
- 如果滥用 microtask，可能影响 UI 响应

### 12. isolate 和线程是什么关系？

核心回答：

- isolate 是 Dart 的并发模型单位
- 每个 isolate 有自己的内存，不共享堆
- 通信依赖消息传递

面试里不要直接把 isolate 说成“就是线程”，这不严谨。

### 13. 什么任务适合放到 isolate？

适合：

- 大 JSON 解析
- 图片处理
- 加密压缩
- 大批量排序聚合

不适合：

- 很轻的小任务
- 强依赖 UI 上下文的逻辑

## 五、平台交互高频题

### 14. MethodChannel、EventChannel、BasicMessageChannel 的区别？

核心回答：

- `MethodChannel`：方法调用，请求-响应
- `EventChannel`：事件流，适合持续推送
- `BasicMessageChannel`：双向消息，适合自定义协议

### 15. 什么时候用 Platform Channel，什么时候用 FFI？

建议回答：

- 调系统能力、平台 API、生命周期相关：Platform Channel
- 调原生算法库、C/Rust 动态库、高性能核心逻辑：FFI

### 16. FFI 最大的风险是什么？

核心回答：

- 函数签名不匹配直接崩溃
- 内存管理责任变复杂
- 原生崩溃无法像 Dart 异常那样温和处理

## 六、工程化高频题

### 17. Flutter 项目怎么分层？

比较稳妥的回答：

- 表现层：页面、状态、交互
- 领域层：用例、业务规则
- 数据层：仓库、数据源、DTO

如果项目规模更小，也可以采用 feature-first 组织，而不是强行完整 clean architecture。

### 18. 如何做异常处理？

回答要点：

- UI 错误提示与日志上报分层
- 网络错误、业务错误、未知错误分层
- 全局捕获 `FlutterError.onError`
- isolate 和异步错误要单独兜底

### 19. Flutter 如何做启动优化？

回答要点：

- 首屏最小化
- 非关键初始化延后
- 大资源延迟加载
- 避免启动阶段重 CPU 任务
- 路由和依赖注入懒加载

## 七、回答策略

面试回答不要只给结论，要给“判断依据”。

例如被问“怎么优化性能”，更好的回答结构是：

1. 先区分 UI Thread 和 Raster Thread
2. 再分析 build、layout、paint、图片、图层等问题
3. 最后给对应优化动作

这样比“我会用 const、拆组件”强很多。

## 八、建议背熟的主线

下面这条主线建议你能口述清楚：

1. Flutter 自己掌控渲染，不主要依赖原生控件
2. Widget 是配置，Element 管生命周期，RenderObject 负责布局绘制
3. 一帧经过 build、layout、paint、compositing、raster
4. UI Thread 和 Raster Thread 任一超时都可能掉帧
5. Platform Channel 和 FFI 分别解决不同类型的平台互操作

这五条一旦讲稳，Flutter 面试的主干基本就稳了。
