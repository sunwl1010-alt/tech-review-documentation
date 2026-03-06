# WorkManager 进阶专题（约束、重试、唯一任务）

## 1. 结论先说

WorkManager 适合“需要可靠执行、可延迟、受系统约束”的后台任务，不适合强实时交互。

## 2. 三个高频能力

- Constraints：网络、电量、充电状态等执行条件
- Backoff：失败重试策略（线性/指数）
- Unique Work：避免重复入队和任务风暴

## 3. 约束与重试示例

```kotlin
val constraints = Constraints.Builder()
    .setRequiredNetworkType(NetworkType.UNMETERED)
    .setRequiresBatteryNotLow(true)
    .build()

val request = OneTimeWorkRequestBuilder<SyncWorker>()
    .setConstraints(constraints)
    .setBackoffCriteria(
        BackoffPolicy.EXPONENTIAL,
        30, TimeUnit.SECONDS
    )
    .build()

WorkManager.getInstance(context)
    .enqueueUniqueWork(
        "daily_sync",
        ExistingWorkPolicy.KEEP,
        request
    )
```

## 4. 任务幂等与数据一致性

- Worker 应支持重复执行（幂等）
- 输入参数要带业务 key，避免重复副作用
- 失败要区分 `Result.retry()` 与 `Result.failure()`

## 5. 常见追问

### Q1：为什么不用 AlarmManager + Service？
答：WorkManager 自动处理系统版本差异与后台限制，可靠性和维护性更好。

### Q2：`KEEP` 和 `REPLACE` 怎么选？
答：同业务语义重复任务一般 `KEEP`，需要强制最新配置时 `REPLACE`。

### Q3：如何观测任务状态？
答：通过 `getWorkInfoByIdLiveData/Flow` 观察 `ENQUEUED/RUNNING/SUCCEEDED/FAILED`。

## 6. 60秒答题模板

- 结论：WorkManager 是后台可靠任务首选。
- 依据：支持系统约束、重试策略、唯一任务治理。
- 落地：我会把同步类任务做成幂等 Worker，并用 UniqueWork 防重复。
- 边界：实时性要求很高的任务应考虑前台服务。
