# Flutter Riverpod 深入

## 一、为什么要学 Riverpod

Riverpod 可以看成是对 Provider 思路的一次升级，它重点解决的问题不是“少写几行代码”，而是：

- 依赖关系更清晰
- 生命周期更可控
- 不强依赖 `BuildContext`
- 可测试性更好
- 组合异步状态更自然

面试里如果只会说“Riverpod 比 Provider 好用”，这是不够的。更准确的说法是：

Riverpod 把状态和依赖管理从 Widget 树依赖里抽离出来，让状态模型更接近“可组合的依赖图”。

## 二、Riverpod 的核心思想

Riverpod 的核心不是 Widget，而是 `Provider`。

这里的 Provider 不是老 Provider 包里的那个 `InheritedWidget` 包装器，而是“状态/依赖的声明单元”。

可以把它理解成：

- 输入：依赖和参数
- 输出：状态或对象
- 管理：创建、缓存、监听、销毁

## 三、为什么说它比 Provider 更彻底

传统 Provider 的问题主要在于：

- 强依赖 `BuildContext`
- 容易出现 provider scope 位置错误
- 依赖读取和 Widget 树耦合较强

Riverpod 的改进点：

1. 不依赖 `BuildContext` 读取状态
2. Provider 是一等公民，不只是 Widget 树注入手段
3. 生命周期和依赖关系显式可追踪
4. 更适合测试和模块化

## 四、Riverpod 常见 Provider 类型

### 1. Provider

用于提供只读依赖，例如：

- repository
- service
- formatter
- use case

### 2. StateProvider

适合简单可变状态，例如：

- tab 索引
- 开关
- 筛选条件

不要把复杂业务状态都塞进 `StateProvider`，这是常见误用。

### 3. FutureProvider

适合异步一次性数据获取，例如：

- 请求用户信息
- 拉取配置

### 4. StreamProvider

适合持续数据流，例如：

- WebSocket
- 数据库监听
- 定位变化

### 5. StateNotifierProvider

适合需要更清晰状态变更入口的中大型场景。

通常配合：

- `StateNotifier`
- 不可变状态对象
- 明确的更新方法

### 6. NotifierProvider / AsyncNotifierProvider

新版 Riverpod 中更推荐的方式，适合统一管理同步和异步业务状态。

## 五、为什么 Riverpod 适合中大型项目

### 1. 依赖图清晰

Provider 之间可以显式依赖：

- repository 依赖 datasource
- use case 依赖 repository
- view model 依赖 use case

这种结构比把依赖到处挂在 Widget 树里更容易维护。

### 2. 生命周期清晰

Riverpod 可以决定对象什么时候：

- 初始化
- 缓存
- 自动销毁
- 重新计算

### 3. 可测试性强

测试时可以 override provider：

- 替换真实 repository
- 注入 mock service
- 控制异步返回

这对面试来说是加分点，因为说明你不是只会写 UI。

## 六、`ref.watch`、`ref.read`、`ref.listen` 的区别

这是高频考点。

### `ref.watch`

- 监听 provider
- provider 变化时当前依赖方会更新
- 通常用于 UI 渲染

### `ref.read`

- 只读取当前值
- 不建立监听关系
- 常用于按钮点击、事件回调、一次性读取

### `ref.listen`

- 监听状态变化并执行副作用
- 常用于 toast、跳转、日志上报

面试里要能说清：

`watch` 负责“渲染依赖”，`listen` 负责“副作用响应”，`read` 负责“一次性访问”。

## 七、Riverpod 中的异步状态建模

Riverpod 异步状态的一大优势是状态表达更完整。

你不应该只用一个 `bool isLoading` 去描述异步流程，更稳妥的方式是明确区分：

- loading
- data
- error

这也是 `AsyncValue` 的价值。

### 为什么 `AsyncValue` 很重要

它强迫你面对异步状态的完整性：

- 数据没回来怎么办
- 请求失败怎么办
- 重试中怎么显示

这比手写散落的 `loading/error/data` 变量更不容易失控。

## 八、自动销毁和缓存

### `autoDispose`

适用于短生命周期状态，例如：

- 页面退出后就不需要保留的数据
- 搜索页临时条件
- 临时详情请求

### 需要注意的问题

如果滥用 `autoDispose`，可能导致：

- 切页回来重复请求
- 状态被意外销毁
- 用户输入丢失

所以要结合业务判断是否需要缓存。

## 九、Riverpod 的典型架构落地

一个相对稳的结构可以是：

1. `Provider` 提供 repository / service
2. `Notifier` 管理页面状态
3. UI 用 `watch` 渲染
4. UI 用 `listen` 处理副作用

这样做的好处：

- 业务逻辑不堆进 Widget
- 状态来源清晰
- 依赖注入简单

## 十、Riverpod 常见误区

### 1. 用 `StateProvider` 管理复杂业务状态

这会让状态更新逻辑散落在页面里，最后变得不可维护。

### 2. 把副作用写进 build

例如：

- 请求接口
- 弹 toast
- 页面跳转

这些都不该在 `build` 里做。

### 3. 滥用全局 provider

不是所有状态都要做成全局。页面局部状态、一次性输入状态，不要无脑上升。

### 4. 忽略 provider 生命周期

如果不关注缓存和销毁策略，Riverpod 也一样会出现重复请求和状态错乱。

## 十一、Riverpod 与 Provider / Bloc / GetX 对比

### 对比 Provider

- Riverpod：依赖关系更清晰，可测试性更强
- Provider：更轻，入门简单，但复杂场景容易扩散

### 对比 Bloc

- Riverpod：写法更灵活，依赖注入和组合更自然
- Bloc：事件流和状态流边界更硬，更适合流程型复杂业务

### 对比 GetX

- Riverpod：约束更清晰，可维护性更稳
- GetX：开发速度快，但大型团队里一致性更依赖约束

## 十二、面试高频问答

### 1. Riverpod 为什么不依赖 BuildContext？

因为它把状态访问从 Widget 树结构中解耦出来，provider 本身就是依赖声明与状态容器，不需要通过 context 向上查找。

### 2. `watch` 和 `read` 的区别是什么？

`watch` 会建立依赖关系并在变更时触发更新，`read` 只做一次性读取，不会触发后续更新。

### 3. Riverpod 为什么更容易测试？

因为 provider 可以被 override，测试中可以替换真实依赖，隔离网络、数据库和平台服务。

### 4. 什么时候不适合用 Riverpod？

如果项目很小、状态非常简单、团队成员对 Provider 更熟悉，那么强行切 Riverpod 不一定带来实际收益。

### 5. Riverpod 的核心优势是什么？

不是“语法新”，而是依赖管理、生命周期管理、异步状态表达和可测试性更完整。

## 十三、面试回答模板

如果面试官问“为什么选 Riverpod”，比较成熟的回答方式是：

1. 它不依赖 `BuildContext`，依赖关系更清晰
2. provider 生命周期可控，适合中大型模块
3. 异步状态表达更完整，尤其是 `AsyncValue`
4. 测试时可以方便替换依赖

这样回答比单纯说“更方便”更有说服力。
