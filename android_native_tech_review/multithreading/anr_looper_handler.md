# ANR 与消息机制面试专题（Looper / Handler / Binder）

## 1. ANR 本质

ANR（Application Not Responding）本质是：主线程在系统规定时间窗口内没有及时处理输入、广播或服务回调。

常见超时场景：
- Input dispatch 超时（用户点击/触摸无响应）
- BroadcastReceiver 超时（前台/后台广播处理超时）
- Service 执行超时（前台服务或绑定交互阻塞）

面试要点：ANR 不是“卡顿严重版”，而是“系统级超时判定”。

## 2. 主线程消息机制回顾

主线程机制由 `Looper + MessageQueue + Handler` 组成：
- Looper：不断轮询消息队列
- MessageQueue：按时间顺序管理消息
- Handler：发送和处理消息

核心约束：
- 主线程要持续“可轮询”
- 任何长耗时任务都会阻塞后续消息处理

## 3. 为什么会 ANR（按根因分类）

### 3.1 主线程阻塞
- 同步 I/O（文件、数据库、网络）
- 大 JSON 解析、Bitmap 解码
- 重布局/重绘制

### 3.2 锁竞争
- 主线程等待后台线程释放锁
- 多线程互相等待形成死锁

### 3.3 Binder 阻塞
- 主线程同步调用系统服务或远程进程
- 远端执行慢，主线程被动等待返回

### 3.4 GC/内存抖动
- 大量对象分配导致频繁 GC
- 主线程停顿放大事件处理延迟

## 4. 排查流程（实战）

1. 拿到 ANR traces：先看 main 线程栈顶
2. 判断类型：
   - `monitor`/`lock` 关键字 -> 锁等待
   - `BinderProxy.transact` -> Binder 阻塞
   - `InputDispatcher` 相关 -> 输入超时
3. 结合日志与埋点还原现场：页面、操作、网络状态、机型
4. 用 Perfetto/Systrace 验证修复收益

关键原则：先定位根因，再改代码；避免“猜测式优化”。

## 5. Handler 常见陷阱

- 非静态内部 Handler 持有 Activity 引用，导致泄漏
- 消息未清理，页面销毁后仍回调 UI
- 用 `postDelayed` 做关键业务定时但无生命周期治理

治理建议：
- 页面销毁时 `removeCallbacksAndMessages(null)`
- 生命周期感知任务优先协程 + `repeatOnLifecycle`
- 减少基于 Handler 的复杂状态流转

## 6. Binder 阻塞专项

ANR 面试常追问：为什么主线程会被 Binder 卡住？

原因：
- 主线程调用系统服务是同步事务
- 远端进程繁忙或锁竞争时，调用链会等待

规避策略：
- 避免主线程进行高频、重负载 Binder 调用
- 可批量的调用合并，减少跨进程往返
- 对关键调用做超时与降级（必要时转后台线程）

## 7. Broadcast / Service 场景防线

### BroadcastReceiver
- `onReceive` 仅做轻路由
- 重任务立即转 WorkManager/Service

### Service
- `onStartCommand` 不做阻塞
- 重任务放协程/线程池
- 前台服务通知及时可见，避免系统策略处罚

## 8. 代码级防御模板

```kotlin
class SafeActivity : AppCompatActivity() {
    private val uiHandler = Handler(Looper.getMainLooper())

    override fun onDestroy() {
        uiHandler.removeCallbacksAndMessages(null)
        super.onDestroy()
    }
}

class HomeViewModel : ViewModel() {
    fun loadData() {
        viewModelScope.launch {
            withContext(Dispatchers.IO) {
                // heavy I/O
            }
        }
    }
}
```

## 9. 面试高频追问

### Q1：卡顿和 ANR 的区别？
答：卡顿是帧耗时超预算（体验问题），ANR 是系统超时判定（稳定性问题），两者可相关但不等价。

### Q2：ANR traces 先看哪里？
答：先看 `main` 线程栈顶和阻塞点，再看是否有锁/Binder/IO 特征。

### Q3：如何避免 Broadcast 导致 ANR？
答：`onReceive` 不做重任务，转异步任务系统，并保证幂等与去重。

### Q4：为什么“把任务丢到线程池”不一定能解决 ANR？
答：如果主线程仍等待结果、持有锁或频繁 Binder 同步调用，ANR 依然会发生。

## 10. 60秒答题模板

- 结论：ANR 是主线程在系统超时窗口内未及时响应。
- 依据：消息循环被阻塞，常见是 IO、锁竞争、Binder 阻塞。
- 落地：先看 trace 定位，再做异步化、锁粒度优化和 Binder 调用治理。
- 边界：仅靠线程池不能兜底，必须保证主线程不等待重操作。
