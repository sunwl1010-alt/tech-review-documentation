# Service 组件（边界、安全与可持续运行）

## 1. 先做职责边界

Service 不是“后台万能容器”，只适合承载以下职责：
- 与用户可感知的持续任务（前台服务）
- 需要跨页面共享的长生命周期任务（绑定服务）
- 与系统调度协作的可延迟任务（应优先 WorkManager，而非常驻 Service）

不建议放在 Service 的内容：
- 纯短任务（可直接协程 + ViewModel）
- 周期任务（优先 WorkManager）
- 与 UI 强耦合逻辑

## 2. 启动策略矩阵

- `startForegroundService`：音乐播放、导航、运动轨迹采集
- `bindService`：页面和后台引擎持续交互（播放器控制）
- `WorkManager`：可延迟、可重试、需系统约束的任务

选择原则：能不常驻就不常驻，能被系统调度就别自建保活。

## 3. Android 8+ 前台服务关键约束

- 启动后必须在短时间内调用 `startForeground()`
- 必须展示用户可理解的通知
- Android 13+ 通知权限缺失会影响可见性与体验

```kotlin
class SyncForegroundService : Service() {
    override fun onStartCommand(intent: Intent?, flags: Int, startId: Int): Int {
        val notification = buildNotification()
        startForeground(1001, notification)

        serviceScope.launch {
            runCatching { syncNow() }
                .onFailure { stopSelf(startId) }
        }
        return START_NOT_STICKY
    }

    override fun onBind(intent: Intent?): IBinder? = null
}
```

## 4. 安全基线

### 4.1 组件暴露
- 默认 `android:exported="false"`
- 需要跨应用调用时，必须加权限保护
- 明确 `intent-filter`，避免过宽匹配

### 4.2 调用方校验
- 校验调用包名/签名（高敏能力）
- 拒绝不可信 Intent 参数
- 避免通过隐式 Intent 启动敏感服务

### 4.3 前台服务类型
- 在 Manifest 指定 `foregroundServiceType`（如 `location`, `mediaPlayback`）
- 类型和实际行为不一致会带来系统限制与审核风险

## 5. 生命周期与资源管理

- `onCreate`：初始化线程池、仓储、通知渠道
- `onStartCommand`：只做参数解析和任务调度，不做阻塞操作
- `onDestroy`：取消协程、关闭连接、释放监听器

```kotlin
private val serviceScope = CoroutineScope(SupervisorJob() + Dispatchers.IO)

override fun onDestroy() {
    serviceScope.cancel()
    super.onDestroy()
}
```

## 6. 常见故障模式

- 重启风暴：`START_STICKY` 使用不当导致频繁重启
- 内存泄漏：Service 持有 Activity 引用
- ANR：在 `onStartCommand` 做同步 I/O
- 用户投诉：前台通知信息不透明或无法关闭

## 7. 与 WorkManager 的分工

建议规则：
- 用户实时可感知 + 持续运行：Foreground Service
- 可靠执行 + 系统约束：WorkManager
- 一次性短后台工作：协程

## 8. 面试深问

- 为什么很多后台任务应从 Service 迁移到 WorkManager？
- 如何保证前台服务“合规 + 不打扰用户”？
- 如何设计播放器服务以支持多页面控制与断点恢复？
