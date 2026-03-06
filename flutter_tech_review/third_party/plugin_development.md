# Flutter 插件开发与 Federated Plugin

## 一、为什么插件开发值得学

当现成三方库满足不了需求时，团队就会遇到两类问题：

1. 需要自己封装原生能力
2. 需要做跨平台插件

这时就不是“怎么用插件”，而是“怎么写插件”。

## 二、package、plugin、federated plugin 的区别

### package

纯 Dart / Flutter 代码库，不一定涉及平台原生能力。

### plugin

包含 Flutter 侧 API 和至少一个平台侧实现。

### federated plugin

把插件拆成多个包：

1. app-facing package
2. platform interface
3. 各平台实现包

## 三、为什么会有 federated plugin

因为当插件支持的平台越来越多时，把所有实现塞在一个包里会变得难维护。

federated plugin 的价值在于：

1. 平台实现解耦
2. 更容易扩展新平台
3. 更利于社区协作

## 四、一个插件通常包含什么

### Flutter 侧

- 对外 API
- 参数校验
- 错误映射

### 平台侧

- Android Kotlin / Java 实现
- iOS Swift / Objective-C 实现
- 可能还包括 Web / Desktop 实现

### 通信方式

常见是：

- `MethodChannel`
- `EventChannel`
- `BasicMessageChannel`

也可以是 FFI。

## 五、什么时候用 Platform Channel 写插件

适合：

1. 调系统 API
2. 访问设备能力
3. 接入平台生命周期

例如：

- 相机
- 定位
- 通知
- 生物识别

## 六、什么时候插件更适合用 FFI

适合：

1. 调 C / C++ / Rust 动态库
2. 高性能底层逻辑
3. 尽量减少平台间重复实现

### 关键区别

- Platform Channel 更偏平台能力接入
- FFI 更偏底层原生库复用

## 七、插件设计的关键点

### 1. API 稳定

Flutter 侧 API 不要频繁变。

### 2. 错误边界清晰

平台错误不要直接裸抛到底层细节，最好转成统一错误码或异常。

### 3. 平台差异可感知

如果某平台不支持某能力，要有明确行为：

- 抛出未实现
- 返回受控错误

### 4. 生命周期正确

特别是：

- 监听器
- 回调
- 资源释放

## 八、插件测试怎么做

插件测试通常至少要分三层：

1. Flutter 侧单元测试
2. 平台接口层测试
3. 平台端或集成验证

### 原则

不要只测 Flutter 包装层，不测平台行为。

## 九、插件文档为什么重要

插件如果没有完整文档，实际接入成本会很高。

至少要写清：

1. 支持哪些平台
2. 需要哪些权限
3. 初始化方式
4. 平台差异
5. 已知限制

## 十、常见误区

### 1. 直接把原生细节暴露给 Flutter 层

这会让 API 很难维护。

### 2. 所有平台逻辑全塞进一个包

平台越多，维护越差。

### 3. 忽略 Web / Desktop 的空实现策略

如果插件声称跨平台，就要明确不支持平台的行为边界。

## 十一、面试高频问答

### 1. package 和 plugin 的区别是什么？

package 通常是纯 Dart / Flutter 代码，plugin 则包含 Flutter 侧 API 和平台原生实现。

### 2. federated plugin 的价值是什么？

把平台接口和平台实现解耦，便于扩展多平台和独立维护。

### 3. 什么时候写插件更适合用 Platform Channel？

当需求依赖系统 API 或平台特有能力时。

### 4. 什么时候更适合 FFI？

当核心逻辑来自动态库或高性能原生模块，需要直接调用底层能力时。

## 十二、推荐回答模板

如果面试官问“Flutter 插件怎么设计”，比较成熟的回答方式是：

1. 先判断需求属于平台能力接入还是底层库调用
2. 平台能力优先考虑 Platform Channel，底层库优先考虑 FFI
3. 对多平台插件，优先考虑 federated plugin 结构
4. 在 API、错误边界、生命周期和平台差异上做清晰设计

这样回答会比“我会写 MethodChannel”更完整。
