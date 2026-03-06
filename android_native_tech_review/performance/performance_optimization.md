# Android 性能优化（工程化实战）

## 性能优化的正确目标
性能不是“局部调参”，而是围绕用户体验指标建立优化闭环。建议以以下指标为核心：

- 启动：冷启动时间、首帧时间、可交互时间
- 渲染：慢帧率、卡顿率、掉帧场景分布
- 内存：PSS、Java Heap、Native Heap、OOM 率
- 稳定性：ANR 率、崩溃率、后台保活成功率
- 能耗：前后台耗电、唤醒次数、网络与定位耗电占比

## 一、建立性能基线与预算

### 1.1 性能预算（示例）
- 冷启动（中端机）< 1800ms
- 首屏 95 分位帧耗时 < 16.6ms
- 单页面内存增量 < 80MB
- 前台 ANR < 0.1%

### 1.2 观测工具组合
- Android Studio Profiler：CPU/Memory/Network
- Macrobenchmark：启动与滑动基准测试
- Perfetto / System Tracing：线程调度与渲染链路
- Baseline Profiles：改善 ART 预编译命中
- 线上埋点：关键路径分阶段打点（Application -> Splash -> Home）

## 二、启动优化：拆解关键路径

### 2.1 冷启动阶段拆分
- `attachBaseContext`
- `Application#onCreate`
- 首个 Activity `onCreate/onStart/onResume`
- 首帧绘制与数据就绪

### 2.2 常见问题
- Application 初始化过重（数据库、SDK、网络预热都堆在主线程）
- 首屏同步 IO（读取大文件、解压、反射扫描）
- 多 SDK 争抢主线程

### 2.3 优化策略
- 同步改异步：非关键初始化延后到首帧之后
- 串行改并行：互不依赖任务并行初始化
- 主线程剥离：将可后台执行任务移至 `Dispatchers.Default/IO`
- 按场景懒加载：功能首次使用时再初始化

示例：启动任务分级调度
```kotlin
interface StartupTask {
    val name: String
    val runOnMainThread: Boolean
    val dependencies: List<String>
    suspend fun execute(context: Context)
}

class StartupManager(private val tasks: List<StartupTask>) {
    suspend fun run(context: Context) {
        val sorted = topologicalSort(tasks)
        coroutineScope {
            sorted.forEach { task ->
                if (task.runOnMainThread) {
                    withContext(Dispatchers.Main) { task.execute(context) }
                } else {
                    launch(Dispatchers.Default) { task.execute(context) }
                }
            }
        }
    }
}
```

## 三、渲染性能：围绕 16.6ms 预算做减法

### 3.1 卡顿根因定位
- 主线程阻塞：IO、锁竞争、复杂 JSON 解析
- 过度绘制：背景层级叠加、透明区域过多
- 布局抖动：多次 measure/layout，深层嵌套
- RecyclerView 绑定过重：图片解码、富文本计算

### 3.2 UI 优化清单
- 减少布局层级：优先 `ConstraintLayout` 或 Compose 合理拆分
- 避免 `onBindViewHolder` 做重计算
- 图片按需采样 + 缓存策略（磁盘 + 内存）
- 列表分页加载，避免一次性大数据渲染
- 复杂动画使用硬件加速与属性动画，避免 `invalidate` 风暴

### 3.3 Compose 场景补充
- 用 `remember` 缓存稳定对象
- 降低不必要重组（拆分可组合函数、使用 `derivedStateOf`）
- 对列表项传稳定数据，避免全量重组

## 四、内存优化：以“生命周期对齐”为核心

### 4.1 常见内存泄漏模式
- 单例持有 Activity/Fragment 引用
- 匿名内部类持有外部类
- Observer/Flow 未解绑
- WebView、Bitmap、Cursor、MediaCodec 未释放

### 4.2 工程化策略
- 组件级资源托管：在 `onStart/onStop` 或 `onCreate/onDestroy` 成对申请释放
- 生命周期感知：优先 `repeatOnLifecycle` 收集流
- 大对象池化与复用：避免频繁创建临时对象

示例：生命周期安全收集
```kotlin
lifecycleScope.launch {
    repeatOnLifecycle(Lifecycle.State.STARTED) {
        viewModel.uiState.collect { render(it) }
    }
}
```

### 4.3 Bitmap 策略
- 根据目标尺寸设置 `inSampleSize`
- 缩略图优先 `RGB_565`（视质量需求）
- 大图浏览使用分片或区域解码

## 五、网络性能：减少无效请求与重复开销

### 5.1 连接与协议优化
- 启用 HTTP/2，复用连接
- 合理设置连接池与超时
- 接入 Brotli/Gzip 压缩

### 5.2 请求治理
- 幂等请求做缓存与去重
- 列表接口分页 + 增量刷新
- 弱网重试要带退避（指数退避 + 抖动）

### 5.3 数据解析
- 大 JSON 改流式解析
- 关键字段提前抽取，非关键字段延迟解析

## 六、ANR 与线程模型优化

### 6.1 ANR 高发场景
- 主线程执行数据库查询
- `BroadcastReceiver` 中做耗时操作
- 锁竞争导致主线程等待

### 6.2 解决策略
- 主线程只做 UI 和轻量调度
- IO/CPU 任务明确线程池归属
- 避免全局锁，缩小锁粒度
- 给耗时路径加超时与熔断

## 七、包体积优化（影响下载与启动）

- 资源瘦身：移除无用资源、按密度拆分
- 代码瘦身：R8/Proguard，移除无用类
- Native 瘦身：按 ABI 拆分
- 依赖治理：避免“大而全”SDK

## 八、功耗优化（常被忽略）

- 合并后台任务，优先 `WorkManager`
- 减少高频定位与无意义轮询
- 网络批处理：可合并则合并
- 使用前台服务时严格控制生命周期

## 九、性能优化落地流程

1. 建立基线：基准测试 + 线上指标
2. 锁定瓶颈：火焰图、trace、埋点对齐
3. 小步迭代：每次只改一个变量
4. 回归验证：自动化 benchmark + 人工场景复测
5. 持续守护：发布门禁（性能回退阻断）

## 十、面试深问与答题框架

- 问：你如何优化冷启动？
  答题框架：启动分阶段 -> 识别主线程阻塞 -> 任务分级延迟 -> 指标回归验证。

- 问：遇到卡顿怎么系统排查？
  答题框架：先看慢帧分布 -> trace 定位主线程热点 -> 区分布局/绘制/业务阻塞 -> 验证收益。

- 问：如何防止性能优化回退？
  答题框架：预算阈值 + benchmark 自动化 + CI 门禁 + 线上监控告警。
