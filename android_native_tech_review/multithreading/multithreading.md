# Android 多线程与异步（现代实践）

## 1. 线程模型全景

Android 线程模型要点：
- 主线程：UI 绘制、输入事件、轻量调度
- Binder 线程池：系统服务 IPC 回调
- 业务线程池：IO/CPU 任务

核心原则：主线程只做必须在主线程做的事。

## 2. 不再推荐的方案

- `AsyncTask`：已废弃，生命周期与线程行为不可靠
- 无边界 `Thread` 创建：难治理，易导致资源争抢

推荐：Kotlin Coroutines + structured concurrency。

## 3. 协程调度策略

- `Dispatchers.Main`：UI 更新
- `Dispatchers.IO`：磁盘/网络
- `Dispatchers.Default`：CPU 密集
- 自定义 Dispatcher：隔离关键任务池

```kotlin
class UserRepository(
    private val io: CoroutineDispatcher = Dispatchers.IO
) {
    suspend fun loadUser(id: String): User = withContext(io) {
        api.fetchUser(id)
    }
}
```

## 4. 生命周期安全

```kotlin
lifecycleScope.launch {
    repeatOnLifecycle(Lifecycle.State.STARTED) {
        viewModel.uiState.collect { state -> render(state) }
    }
}
```

收益：页面不可见时自动停止收集，避免泄漏和无效渲染。

## 5. 并行与超时控制

```kotlin
suspend fun loadHomeData(): HomeData = coroutineScope {
    val profile = async { profileRepo.fetch() }
    val feed = async { feedRepo.fetch() }
    val banner = async { bannerRepo.fetch() }

    withTimeout(3000) {
        HomeData(profile.await(), feed.await(), banner.await())
    }
}
```

关键点：
- `coroutineScope` 子任务失败会取消兄弟任务（结构化并发）
- `withTimeout` 防止挂死

## 6. 取消传播与资源清理

```kotlin
suspend fun syncData() {
    try {
        // long running task
    } finally {
        // 释放文件句柄、关闭流、上报状态
    }
}
```

注意：不要吞掉 `CancellationException`。

## 7. Flow 背压与状态建模

- UI 状态流用 `StateFlow`
- 一次性事件（toast/navigation）用 `SharedFlow`
- 高频流使用 `debounce` / `sample` / `conflate`

```kotlin
val queryFlow = MutableStateFlow("")

val resultFlow = queryFlow
    .debounce(300)
    .distinctUntilChanged()
    .flatMapLatest { q -> repo.search(q) }
```

## 8. 线程安全与共享状态

- 优先不可变数据 + 单向数据流
- 必要时使用 `Mutex` 而不是粗粒度 `synchronized`
- 避免跨线程共享可变集合

```kotlin
private val mutex = Mutex()
private val cache = mutableMapOf<String, User>()

suspend fun putUser(user: User) {
    mutex.withLock { cache[user.id] = user }
}
```

## 9. ANR 排查思路

1. 看 ANR trace：主线程卡在何处
2. 判断类型：锁等待、IO 阻塞、Binder 阻塞、GC 抖动
3. 对应优化：
- IO 下沉后台线程
- 缩小锁范围
- 避免主线程 IPC 链过长
- 大对象分批处理

## 10. 线程池治理建议

- 按任务类型拆池：网络池、CPU 池、下载池
- 线程池参数随设备等级分层
- 对关键任务池加队列长度监控和拒绝策略

## 11. 面试深问

- 协程比 RxJava 的优势与局限？
- 如何避免 ViewModel 内部并发竞态？
- 如何定位偶发 ANR 但本地无法复现？
