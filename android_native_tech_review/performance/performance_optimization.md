# 性能优化

## 内存优化

### 内存泄漏

#### 常见原因

- **静态变量**：持有 Activity 或 Context 引用
- **Handler**：未及时移除消息，持有外部类引用
- **线程**：线程未正确终止，持有 Activity 引用
- **监听器**：未正确注销监听器
- **资源未释放**：Bitmap、Cursor 等资源未关闭

#### 检测工具

- **Memory Profiler**：Android Studio 内置的内存分析工具
- **LeakCanary**：自动检测内存泄漏的库

#### 解决方案

```kotlin
// 使用弱引用
val weakReference = WeakReference<Context>(context)

// 正确处理 Handler
class MyActivity : AppCompatActivity() {
    private val handler = object : Handler(Looper.getMainLooper()) {
        override fun handleMessage(msg: Message) {
            // 处理消息
        }
    }
    
    override fun onDestroy() {
        super.onDestroy()
        handler.removeCallbacksAndMessages(null)
    }
}

// 正确关闭资源
fun loadBitmap(uri: Uri) {
    val inputStream = contentResolver.openInputStream(uri)
    try {
        val bitmap = BitmapFactory.decodeStream(inputStream)
        // 使用 bitmap
    } finally {
        inputStream?.close()
    }
}

// 使用 LeakCanary
// 添加依赖
// implementation 'com.squareup.leakcanary:leakcanary-android:2.10'
```

### 内存使用优化

1. **Bitmap 优化**：
   - 使用合适的图片尺寸
   - 及时回收 Bitmap
   - 使用 BitmapFactory.Options 减少内存占用

2. **对象池**：
   - 复用对象，减少 GC 压力
   - 使用 SparseArray、ArrayMap 等高效集合

3. **内存缓存**：
   - 使用 LruCache 缓存常用数据
   - 合理设置缓存大小

```kotlin
// Bitmap 优化
val options = BitmapFactory.Options()
options.inSampleSize = 2 // 缩小图片
options.inPreferredConfig = Bitmap.Config.RGB_565 // 减少每个像素的内存占用
val bitmap = BitmapFactory.decodeResource(resources, R.drawable.large_image, options)

// LruCache 缓存
val maxMemory = (Runtime.getRuntime().maxMemory() / 1024).toInt()
val cacheSize = maxMemory / 8
val bitmapCache = LruCache<String, Bitmap>(cacheSize) {
    it.byteCount / 1024
}

// 存储 Bitmap
bitmapCache.put("image_key", bitmap)

// 获取 Bitmap
val cachedBitmap = bitmapCache.get("image_key")
```

## UI 优化

### 布局优化

1. **减少布局层级**：
   - 使用 ConstraintLayout 代替嵌套的 LinearLayout 和 RelativeLayout
   - 避免过深的布局层次

2. **使用 ViewStub**：
   - 延迟加载不常用的布局
   - 减少初始布局加载时间

3. **使用 include 和 merge**：
   - 重用布局代码
   - 减少布局层级

```xml
<!-- 使用 ConstraintLayout -->
<androidx.constraintlayout.widget.ConstraintLayout
    android:layout_width="match_parent"
    android:layout_height="match_parent">
    
    <TextView
        android:id="@+id/textView"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent"/>
    
    <Button
        android:id="@+id/button"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintTop_toTopOf="parent"/>
</androidx.constraintlayout.widget.ConstraintLayout>

<!-- 使用 ViewStub -->
<ViewStub
    android:id="@+id/viewStub"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:layout="@layout/complex_layout"/>

<!-- 动态加载 ViewStub -->
val viewStub = findViewById<ViewStub>(R.id.viewStub)
val inflatedView = viewStub.inflate()
```

### 渲染优化

1. **避免过度绘制**：
   - 移除不必要的背景
   - 使用 clipRect 限制绘制区域
   - 减少透明视图

2. **使用硬件加速**：
   - 启用硬件加速提高渲染性能
   - 注意硬件加速的兼容性问题

3. **优化动画**：
   - 使用 Property Animation
   - 避免在动画中进行耗时操作
   - 使用硬件图层（hardware layer）

```kotlin
// 启用硬件图层
view.setLayerType(View.LAYER_TYPE_HARDWARE, null)

// 动画完成后禁用硬件图层
view.animate()
    .translationX(100f)
    .setDuration(500)
    .withEndAction {
        view.setLayerType(View.LAYER_TYPE_NONE, null)
    }
```

### 列表优化

1. **使用 RecyclerView**：
   - 视图复用机制
   - 高效的布局管理器

2. **优化适配器**：
   - 避免在 onBindViewHolder 中进行耗时操作
   - 使用 DiffUtil 优化数据更新
   - 实现 getItemViewType 处理不同类型的布局

3. **预加载**：
   - 提前加载即将显示的数据
   - 使用 RecyclerView 的 prefetch 机制

```kotlin
// 使用 DiffUtil
class MyDiffCallback(private val oldList: List<Item>, private val newList: List<Item>) : DiffUtil.Callback() {
    override fun getOldListSize() = oldList.size
    override fun getNewListSize() = newList.size
    
    override fun areItemsTheSame(oldItemPosition: Int, newItemPosition: Int): Boolean {
        return oldList[oldItemPosition].id == newList[newItemPosition].id
    }
    
    override fun areContentsTheSame(oldItemPosition: Int, newItemPosition: Int): Boolean {
        return oldList[oldItemPosition] == newList[newItemPosition]
    }
}

// 计算差异并更新列表
val diffResult = DiffUtil.calculateDiff(MyDiffCallback(oldItems, newItems))
diffResult.dispatchUpdatesTo(adapter)
```

## 网络优化

### 请求优化

1. **减少请求次数**：
   - 合并请求
   - 使用批量接口
   - 合理使用缓存

2. **优化请求大小**：
   - 使用压缩
   - 减少不必要的字段
   - 使用 JSON 或 Protocol Buffers

3. **网络库选择**：
   - OkHttp：支持连接池、拦截器
   - Retrofit：类型安全的 REST 客户端
   - Volley：适合频繁的小请求

### 缓存策略

1. **HTTP 缓存**：
   - 使用 Cache-Control 头
   - 实现离线缓存

2. **本地缓存**：
   - 使用 Room 或 SharedPreferences
   - 缓存热点数据

3. **图片缓存**：
   - 使用 Glide 或 Picasso
   - 二级缓存（内存 + 磁盘）

```kotlin
// OkHttp 缓存配置
val cacheSize = 10 * 1024 * 1024 // 10 MB
val cache = Cache(cacheDir, cacheSize.toLong())

val client = OkHttpClient.Builder()
    .cache(cache)
    .addInterceptor {
        val request = it.request()
        if (NetworkUtils.isNetworkAvailable(context)) {
            request.newBuilder().header("Cache-Control", "public, max-age=60").build()
        } else {
            request.newBuilder().header("Cache-Control", "public, only-if-cached, max-stale=2419200").build()
        }
        it.proceed(request)
    }
    .build()
```

### 网络状态感知

1. **监听网络变化**：
   - 注册网络状态广播
   - 使用 ConnectivityManager

2. **自适应网络策略**：
   - 根据网络类型调整请求策略
   - WiFi 下预加载更多数据
   - 移动网络下减少数据传输

```kotlin
// 监听网络状态
val connectivityManager = context.getSystemService(Context.CONNECTIVITY_SERVICE) as ConnectivityManager

if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.N) {
    connectivityManager.registerDefaultNetworkCallback(object : ConnectivityManager.NetworkCallback() {
        override fun onAvailable(network: Network) {
            // 网络可用
        }
        
        override fun onLost(network: Network) {
            // 网络丢失
        }
    })
}

// 检查网络类型
val networkCapabilities = connectivityManager.getNetworkCapabilities(connectivityManager.activeNetwork)
val isWifi = networkCapabilities?.hasTransport(NetworkCapabilities.TRANSPORT_WIFI) ?: false
val isCellular = networkCapabilities?.hasTransport(NetworkCapabilities.TRANSPORT_CELLULAR) ?: false
```

## 电池优化

### 减少唤醒

1. **使用 Doze 模式**：
   - 适应系统的省电模式
   - 避免频繁唤醒设备

2. **优化后台任务**：
   - 使用 WorkManager 调度后台任务
   - 避免使用 AlarmManager 频繁唤醒

3. **使用 JobScheduler**：
   - 批处理任务
   - 在合适的时机执行

```kotlin
// 使用 WorkManager
val workRequest = OneTimeWorkRequestBuilder<MyWorker>()
    .setConstraints(
        Constraints.Builder()
            .setRequiredNetworkType(NetworkType.CONNECTED)
            .setRequiresCharging(true)
            .build()
    )
    .build()

WorkManager.getInstance(context).enqueue(workRequest)

// 定义 Worker
class MyWorker(appContext: Context, workerParams: WorkerParameters) : Worker(appContext, workerParams) {
    override fun doWork(): Result {
        // 执行后台任务
        return Result.success()
    }
}
```

### 减少耗电操作

1. **优化定位**：
   - 使用合适的定位精度
   - 及时关闭定位服务
   - 使用 FusedLocationProviderClient

2. **优化网络操作**：
   - 批量处理网络请求
   - 使用压缩减少数据传输
   - 避免频繁的网络轮询

3. **优化硬件使用**：
   - 减少 CPU 密集型操作
   - 优化 GPU 渲染
   - 合理使用传感器

### 监控电池状态

1. **获取电池信息**：
   - 使用 BatteryManager
   - 监听电池状态变化

2. **适应电池状态**：
   - 低电量时减少后台活动
   - 充电时执行耗电操作

```kotlin
// 获取电池信息
val batteryManager = context.getSystemService(Context.BATTERY_SERVICE) as BatteryManager
val batteryLevel = batteryManager.getIntProperty(BatteryManager.BATTERY_PROPERTY_CAPACITY)
val isCharging = batteryManager.getIntProperty(BatteryManager.BATTERY_PROPERTY_STATUS) == BatteryManager.BATTERY_STATUS_CHARGING

// 监听电池状态
val batteryIntent = context.registerReceiver(null, IntentFilter(Intent.ACTION_BATTERY_CHANGED))
val level = batteryIntent?.getIntExtra(BatteryManager.EXTRA_LEVEL, -1) ?: -1
val scale = batteryIntent?.getIntExtra(BatteryManager.EXTRA_SCALE, -1) ?: -1
val batteryPercentage = level * 100 / scale
```

## 最佳实践

1. **性能分析**：
   - 使用 Android Studio Profiler 分析性能
   - 使用 Systrace 跟踪系统调用
   - 使用 Traceview 分析方法执行时间

2. **代码优化**：
   - 避免在主线程执行耗时操作
   - 优化循环和递归
   - 使用高效的数据结构

3. **构建优化**：
   - 使用 R8 或 ProGuard 混淆代码
   - 启用代码压缩和资源压缩
   - 优化构建配置

4. **监控与反馈**：
   - 集成性能监控工具
   - 收集用户反馈
   - 持续优化

## 常见问题

1. **ANR**：应用无响应，通常是主线程阻塞
2. **OOM**：内存溢出，通常是内存泄漏或内存使用过度
3. **卡顿**：UI 渲染不流畅，通常是布局复杂或主线程耗时操作
4. **电池消耗过快**：后台操作频繁或硬件使用不当
5. **网络请求缓慢**：请求过多或网络策略不当

## 面试题

1. **Q**: 如何检测内存泄漏？
   **A**: 使用 LeakCanary 自动检测，或使用 Android Studio 的 Memory Profiler 分析内存使用

2. **Q**: 如何优化 RecyclerView 的性能？
   **A**: 使用视图复用、优化适配器、使用 DiffUtil、实现 getItemViewType

3. **Q**: 如何减少应用的电池消耗？
   **A**: 使用 WorkManager 调度后台任务、优化定位服务、减少网络轮询、适应系统省电模式

4. **Q**: 如何优化应用的启动速度？
   **A**: 减少冷启动时间、优化布局加载、延迟初始化非必要组件、使用 Splash Screen

5. **Q**: 如何优化网络请求？
   **A**: 合并请求、使用缓存、优化请求大小、根据网络状态调整策略