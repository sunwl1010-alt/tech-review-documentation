# MVP（遗留架构维护与升级）

## 1. MVP 的价值与代价

价值：
- 相比 MVC，职责分离更清晰
- Presenter 可测试性更好

代价：
- 接口与样板代码较多
- 生命周期绑定复杂，易出现内存泄漏
- 状态表达能力弱，复杂页面容易“回调地狱”

## 2. MVP 常见失败模式

- Presenter 持有 View 强引用导致泄漏
- 多异步回调竞态，UI 状态错乱
- 一个 Presenter 管太多场景

## 3. 维护期最佳实践

- View 接口最小化：只保留渲染能力
- Presenter 内禁止直接持有 Android Context（必要时用抽象）
- 引入调度器/协程统一异步管理
- Presenter 按场景拆分，避免超级 Presenter

## 4. 迁移路径（MVP -> MVVM）

1. 保留 Repository，不动数据层
2. 把 Presenter 的状态抽象为 `UiState`
3. 新增 ViewModel 复用原业务逻辑
4. 页面逐步从回调改为状态流收集
5. 旧 Presenter 下线并删除 View 接口冗余

## 5. 兼容迁移示例思路

- 先在 Presenter 输出 `State` 对象
- Activity 既可走旧回调，也可订阅新状态
- 双轨运行一段时间后切换至 ViewModel

## 6. 面试深问

- MVP 为什么在复杂页面下维护成本高？
- MVP 迁移 MVVM 如何避免一次性重构风险？
- 如何判断一个模块“值得迁移”？
