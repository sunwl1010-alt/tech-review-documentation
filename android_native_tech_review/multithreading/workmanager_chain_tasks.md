# WorkManager 链式任务与可靠执行案例

## 1. 什么时候用链式任务

当任务有明确依赖关系时：
- 先拉配置
- 再下载资源
- 再校验并入库

链式任务比手写回调更稳，失败恢复和状态观测更一致。

## 2. 基础链式示例

```kotlin
val fetchConfig = OneTimeWorkRequestBuilder<FetchConfigWorker>().build()
val download = OneTimeWorkRequestBuilder<DownloadWorker>().build()
val verify = OneTimeWorkRequestBuilder<VerifyWorker>().build()

WorkManager.getInstance(context)
    .beginUniqueWork("startup_pipeline", ExistingWorkPolicy.REPLACE, fetchConfig)
    .then(download)
    .then(verify)
    .enqueue()
```

## 3. 输入输出传递

- 每个 Worker 通过 `Data` 输入输出关键字段
- 不要传大对象（Data 有体积限制）
- 复杂结果写本地数据库，再传 ID

## 4. 失败与重试策略

- 短暂故障：`Result.retry()` + backoff
- 参数错误：`Result.failure()`
- 下游依赖：上游失败则链路中断

## 5. 幂等设计（高频）

- 每步任务要支持重复执行
- 使用业务唯一键避免重复副作用
- 关键写入做去重约束

## 6. 观测与调试

- 通过 `WorkInfo` 观察每一步状态
- 关键链路写审计日志（workerName、attempt、errorCode）
- 提供手动重试入口（仅关键流程）

## 7. 常见追问

### Q1：链式任务与协程串行有什么区别？
答：WorkManager 可跨进程/跨重启恢复，协程通常受进程生命周期限制。

### Q2：何时用 `REPLACE`，何时用 `KEEP`？
答：配置类任务用 `REPLACE` 保证最新；幂等同步可用 `KEEP` 防重复。

## 8. 60 秒答题模板

- 结论：有依赖关系的后台流程优先链式 WorkManager。
- 依据：可恢复、可观测、可重试，且受系统调度约束。
- 落地：我会做幂等和唯一任务治理，避免风暴与重复副作用。
