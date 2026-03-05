# 多线程与异步

## Thread

### 基本使用

```kotlin
// 创建并启动线程
val thread = Thread {
    // 后台执行的任务
    Thread.sleep(1000)
    println("Thread executed")
}
thread.start()

// 等待线程完成
thread.join()
```

### 线程状态

- **新建**：线程已创建但未启动
- **就绪**：线程已启动，等待CPU调度
- **运行**：线程正在执行
- **阻塞**：线程等待资源或条件
- **死亡**：线程执行完毕或异常终止

### 线程安全

```kotlin
// 使用 synchronized 关键字
class Counter {
    private var count = 0
    
    @Synchronized
    fun increment() {
        count++
    }
    
    fun getCount(): Int {
        return count
    }
}

// 使用 Lock
val lock = ReentrantLock()
fun doSomething() {
    lock.lock()
    try {
        // 临界区
    } finally {
        lock.unlock()
    }
}
```

## Handler

### 基本概念

- **Handler**：处理消息和 Runnable
- **Looper**：消息循环器，处理消息队列
- **MessageQueue**：消息队列，存储待处理的消息

### 常用方法

```kotlin
// 在主线程中创建 Handler
val handler = Handler(Looper.getMainLooper())

// 延迟执行
handler.postDelayed({
    // 在主线程执行
    textView.text = "Updated"
}, 1000)

// 发送消息
val message = Message.obtain()
message.what = 1
message.obj = "Hello"
handler.sendMessage(message)

// 处理消息
val handlerWithCallback = object : Handler(Looper.getMainLooper()) {
    override fun handleMessage(msg: Message) {
        when (msg.what) {
            1 -> println(msg.obj)
        }
    }
}
```

## AsyncTask

### 基本用法

```kotlin
class MyAsyncTask : AsyncTask<Void, Int, String>() {
    
    override fun onPreExecute() {
        // 执行前准备，在主线程
    }
    
    override fun doInBackground(vararg params: Void?): String {
        // 后台执行，在子线程
        for (i in 1..100) {
            publishProgress(i)
            Thread.sleep(100)
        }
        return "Task completed"
    }
    
    override fun onProgressUpdate(vararg values: Int?) {
        // 更新进度，在主线程
        progressBar.progress = values[0] ?: 0
    }
    
    override fun onPostExecute(result: String?) {
        // 执行完成，在主线程
        textView.text = result
    }
}

// 执行 AsyncTask
MyAsyncTask().execute()
```

### 注意事项

- **已弃用**：AsyncTask 在 API 30 中已被标记为弃用
- **内存泄漏**：内部类可能持有外部 Activity 引用
- **生命周期**：配置变更时可能导致问题

## ThreadPoolExecutor

### 核心参数

```kotlin
val executor = ThreadPoolExecutor(
    corePoolSize = 2,      // 核心线程数
    maximumPoolSize = 4,   // 最大线程数
    keepAliveTime = 60L,   // 线程存活时间
    unit = TimeUnit.SECONDS, // 时间单位
    workQueue = LinkedBlockingQueue<Runnable>() // 工作队列
)

// 提交任务
executor.execute {
    // 执行任务
}

// 关闭线程池
executor.shutdown()
```

### 线程池类型

- **FixedThreadPool**：固定大小的线程池
- **CachedThreadPool**：可缓存的线程池
- **ScheduledThreadPool**：定时任务线程池
- **SingleThreadExecutor**：单线程线程池

```kotlin
// 使用 Executors 工具类
val fixedThreadPool = Executors.newFixedThreadPool(4)
val cachedThreadPool = Executors.newCachedThreadPool()
val scheduledThreadPool = Executors.newScheduledThreadPool(4)
val singleThreadExecutor = Executors.newSingleThreadExecutor()
```

## Coroutine

### 基本概念

- **协程**：轻量级线程
- **挂起函数**：可以暂停执行并在稍后恢复的函数
- **调度器**：决定协程在哪个线程执行

### 基本使用

```kotlin
// 添加依赖
// implementation "org.jetbrains.kotlinx:kotlinx-coroutines-android:1.6.4"

// 启动协程
GlobalScope.launch {
    // 在后台线程执行
    val result = doNetworkRequest()
    withContext(Dispatchers.Main) {
        // 在主线程更新 UI
        textView.text = result
    }
}

// 挂起函数
suspend fun doNetworkRequest(): String {
    delay(1000) // 非阻塞延迟
    return "Network response"
}

// 使用 lifecycleScope (AndroidX)
class MyActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        
        lifecycleScope.launch {
            // 协程会在 Activity 销毁时自动取消
            val data = loadData()
            updateUI(data)
        }
    }
    
    private suspend fun loadData(): String {
        // 加载数据
        return "Data"
    }
}
```

### 协程调度器

- **Dispatchers.Main**：主线程，用于 UI 操作
- **Dispatchers.IO**：IO 操作线程，用于网络请求、文件操作
- **Dispatchers.Default**：默认调度器，用于 CPU 密集型任务
- **Dispatchers.Unconfined**：不受限制的调度器

### 取消协程

```kotlin
val job = GlobalScope.launch {
    repeat(1000) {
        if (isActive) {
            println("Working...")
            delay(500)
        }
    }
}

// 取消协程
job.cancel()
```

## 最佳实践

1. **选择合适的线程方案**：
   - 简单后台任务：Thread 或 Coroutine
   - UI 相关任务：Handler 或 Coroutine with Dispatchers.Main
   - 大量任务：ThreadPoolExecutor
   - 现代 Android 开发：优先使用 Coroutine

2. **避免 UI 线程阻塞**：
   - 耗时操作必须在后台线程执行
   - 使用 Handler 或 Coroutine 在主线程更新 UI

3. **处理线程安全**：
   - 使用 synchronized、Lock 或原子操作
   - 避免共享可变状态

4. **资源管理**：
   - 及时关闭线程池
   - 取消不需要的协程
   - 避免内存泄漏

5. **错误处理**：
   - 在后台线程捕获异常
   - 向主线程传递错误信息

## 常见问题

1. **ANR (Application Not Responding)**：在主线程执行耗时操作
2. **内存泄漏**：线程持有 Activity 引用
3. **线程安全问题**：共享资源未正确同步
4. **协程取消**：未正确处理取消信号
5. **线程池过载**：提交过多任务导致系统资源耗尽

## 面试题

1. **Q**: 主线程和子线程的区别是什么？
   **A**: 主线程负责 UI 渲染和用户交互，子线程用于执行耗时操作避免阻塞 UI

2. **Q**: Handler、Looper、MessageQueue 的关系是什么？
   **A**: Looper 负责循环处理 MessageQueue 中的消息，Handler 负责发送和处理消息

3. **Q**: AsyncTask 为什么被弃用？
   **A**: 存在内存泄漏风险、生命周期管理复杂、并行执行行为不可预测

4. **Q**: 协程相比线程有什么优势？
   **A**: 轻量级、挂起而非阻塞、简化异步代码、更好的错误处理

5. **Q**: 如何处理协程的取消？
   **A**: 使用 job.cancel() 取消协程，在协程中检查 isActive 状态，使用 withTimeout 等函数