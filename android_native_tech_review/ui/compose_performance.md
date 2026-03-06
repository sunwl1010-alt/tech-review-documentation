# Jetpack Compose 性能与重组优化面试专题

## 1. 先给结论

Compose 性能问题的核心通常不是“Compose 慢”，而是“重组范围失控 + 状态设计不合理 + 列表项不稳定”。  
优化目标是减少不必要重组、降低每帧工作量、保持状态与 UI 映射清晰。

## 2. 重组（Recomposition）到底是什么

重组是 Compose 在状态变化时重新执行受影响的 Composable。  
关键点：
- 重组是正常行为，不是 bug
- 问题在于“无效重组”太多
- 要关注“重组范围”和“重组频率”

## 3. 稳定性（Stability）与参数设计

Compose 会根据参数稳定性决定是否可跳过重组。

实践建议：
- UI 参数尽量使用不可变数据类
- 避免把频繁变化的大对象整体传入
- 列表 item 使用稳定 key

```kotlin
LazyColumn {
    items(items = uiState.list, key = { it.id }) { item ->
        FeedItemRow(item)
    }
}
```

## 4. 常见性能陷阱

### 4.1 在 Composable 中做重计算
- 例如复杂过滤、排序、格式化每次重组都执行
- 解决：移到 ViewModel 或 `remember/derivedStateOf`

### 4.2 状态粒度过粗
- 一个状态变化触发整页重组
- 解决：拆分子状态，局部更新

### 4.3 列表项缺少 key
- 导致复用差、滚动抖动
- 解决：提供稳定 key

### 4.4 Lambda/对象每次新建
- 造成不必要参数变化
- 解决：`remember` 缓存或提到外层

## 5. remember / derivedStateOf / rememberUpdatedState

- `remember`：缓存计算结果，避免每次重组重复创建
- `derivedStateOf`：基于源状态派生，按需触发
- `rememberUpdatedState`：在副作用中拿到最新值，避免闭包旧值问题

```kotlin
val filtered by remember(uiState.query, uiState.items) {
    derivedStateOf { uiState.items.filter { it.name.contains(uiState.query) } }
}
```

## 6. 副作用 API 正确使用

- `LaunchedEffect(key)`：key 变更时重启协程
- `DisposableEffect`：绑定/解绑资源
- `SideEffect`：每次成功重组后执行

常见错误：
- key 设计不当导致 effect 频繁重启
- 在 Composable 顶层直接发网络请求

## 7. Compose 与 ViewModel 协作

推荐模式：
- ViewModel 管业务状态
- Compose 只做状态渲染与事件上报
- 不在 UI 层拼业务逻辑

```kotlin
@Composable
fun UserScreen(vm: UserViewModel) {
    val state by vm.uiState.collectAsStateWithLifecycle()
    UserContent(state = state, onRetry = vm::retry)
}
```

## 8. Lazy 列表优化清单

- 使用 `LazyColumn/LazyVerticalGrid`
- item 提供稳定 key
- 避免 item 内部重计算
- 图片加载使用缓存库并限制尺寸
- 避免嵌套滚动导致测量复杂度上升

## 9. 可观测与排查工具

- Layout Inspector（Compose tree）
- Recomposition counts（重组计数）
- Macrobenchmark（滚动与启动基准）
- Perfetto（主线程、渲染线程耗时）

面试加分点：优化前后给出指标对比，而不是只说“更流畅了”。

## 10. 高频追问

### Q1：重组越少越好吗？
答：不是。关键是“必要重组要快，无效重组要少”。

### Q2：为什么 stable key 对列表很重要？
答：它决定 item 身份，影响复用、动画和状态保留。

### Q3：`remember` 会不会导致状态过期？
答：会，若 key 设计不当会缓存旧值。要根据依赖设置 key 或用 `rememberUpdatedState`。

### Q4：Compose 性能差如何排查顺序？
答：先看重组范围，再看主线程耗时，再看列表项和图片链路。

## 11. 60 秒答题模板

- 结论：Compose 性能优化核心是控制重组范围与频率。
- 依据：稳定参数、状态拆分、副作用 key 合理、列表 key 稳定。
- 落地：我会先做重组计数和 Perfetto 定位，再做局部重构与基准回归。
- 边界：只靠 `remember` 不能解决全部性能问题，主线程重任务仍会掉帧。
