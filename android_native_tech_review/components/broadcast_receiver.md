# BroadcastReceiver（系统事件接入与安全治理）

## 1. 广播的定位

BroadcastReceiver 适合做“轻入口”，不适合做“重处理”。
- 适合：接收系统或应用事件、快速路由任务
- 不适合：长耗时计算、网络大任务、复杂业务编排

原则：`onReceive()` 里只做轻量逻辑，重任务立刻转交 WorkManager/Service。

## 2. 注册策略

- 动态注册优先：生命周期可控，暴露面小
- 静态注册谨慎：仅用于系统允许且确有必要的广播
- Android 8+ 大量隐式广播限制，需按官方替代方案迁移

```kotlin
private val receiver = object : BroadcastReceiver() {
    override fun onReceive(context: Context, intent: Intent) {
        if (intent.action == ConnectivityManager.CONNECTIVITY_ACTION) {
            enqueueNetworkSync(context)
        }
    }
}

override fun onStart() {
    super.onStart()
    registerReceiver(receiver, IntentFilter(ConnectivityManager.CONNECTIVITY_ACTION))
}

override fun onStop() {
    unregisterReceiver(receiver)
    super.onStop()
}
```

## 3. 安全策略

### 3.1 发送端
- 自定义 action 使用应用包名前缀，避免冲突
- 敏感广播加权限或改为应用内总线
- 避免在广播中携带高敏明文数据

### 3.2 接收端
- 校验 `intent.action` 与参数完整性
- 对关键来源做签名级权限保护
- 默认关闭导出能力

### 3.3 有序广播注意事项
- 不滥用 `abortBroadcast()`
- 避免修改结果造成链路不可预测

## 4. 性能与稳定性

常见问题：
- 高频广播导致主线程抖动
- 忘记反注册导致泄漏
- 广播风暴触发重复任务

治理建议：
- 去重与节流（短窗口内合并事件）
- 重任务幂等（同一业务 key 只执行一次）
- 关键广播打点，追踪触发频率与耗时

## 5. 现代替代

- 应用内通信：优先 `Flow/SharedFlow`，减少系统广播依赖
- 后台执行：优先 WorkManager
- 跨进程事件：AIDL/Messenger（按需求）

## 6. 面试深问

- 为什么 `onReceive` 不能执行耗时任务？
- Android 8+ 后广播设计如何演进？
- 如何防止恶意应用伪造广播触发敏感行为？
