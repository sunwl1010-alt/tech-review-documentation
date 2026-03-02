# BroadcastReceiver组件

## 核心概念

### BroadcastReceiver
- **定义**：Android系统中的一种组件，用于接收和处理广播消息
- **作用**：响应系统事件或应用事件，如网络状态变化、电池电量变化等
- **类型**：
  - 静态注册：在AndroidManifest.xml中声明
  - 动态注册：在代码中注册

### 广播类型
- **标准广播**：完全异步，所有接收器同时接收
- **有序广播**：同步执行，按优先级顺序接收
- **本地广播**：只在应用内部传递

### Intent Filter
- **定义**：用于指定BroadcastReceiver可以接收的广播类型
- **作用**：过滤不符合条件的广播
- **属性**：action、category、data等

### PendingIntent
- **定义**：一种延迟执行的Intent
- **作用**：用于在未来某个时间点执行操作
- **类型**：Activity、Service、BroadcastReceiver

## 实现原理

### 广播发送机制
- **发送广播**：通过Context.sendBroadcast()发送标准广播
- **发送有序广播**：通过Context.sendOrderedBroadcast()发送有序广播
- **发送本地广播**：通过LocalBroadcastManager.sendBroadcast()发送本地广播

### 广播接收机制
- **静态注册**：系统在应用安装时注册，即使应用未启动也能接收广播
- **动态注册**：在代码中注册，应用启动后才能接收广播，需要手动注销
- **广播过滤**：通过Intent Filter过滤不符合条件的广播

### 有序广播处理
- **优先级**：通过android:priority属性设置接收器优先级
- **中断广播**：通过abortBroadcast()中断广播传递
- **修改广播**：通过setResultData()修改广播数据
- **获取结果**：通过getResultData()获取上一个接收器的结果

### 本地广播实现
- **LocalBroadcastManager**：应用内部的广播管理器
- **安全性**：只在应用内部传递，避免广播泄露
- **效率**：比系统广播更高效，因为不需要跨进程通信

## 使用场景

### 系统广播
- **网络状态变化**：监听网络连接状态
- **电池电量变化**：监听电池电量和充电状态
- **屏幕状态变化**：监听屏幕开/关状态
- **应用安装/卸载**：监听应用安装和卸载事件

### 应用广播
- **自定义广播**：应用内部或应用之间的通信
- **事件通知**：通知应用其他部分某个事件发生
- **状态同步**：同步应用不同组件的状态

### 有序广播
- **权限验证**：通过优先级验证广播数据
- **数据处理**：按顺序处理广播数据
- **结果传递**：在接收器之间传递处理结果

### 本地广播
- **应用内部通信**：应用内部组件之间的通信
- **敏感数据传递**：传递敏感数据，避免泄露
- **高频广播**：发送频率较高的广播

## 常见问题及解决方案

### 注册问题
- **问题**：静态注册的BroadcastReceiver不接收广播
  **解决方案**：检查Intent Filter配置，确保广播动作正确

- **问题**：动态注册的BroadcastReceiver忘记注销
  **解决方案**：在onPause()或onDestroy()中注销BroadcastReceiver

- **问题**：Android 8.0及以上版本静态注册的BroadcastReceiver不工作
  **解决方案**：使用动态注册，或使用JobScheduler替代

### 性能问题
- **问题**：广播接收导致ANR（应用无响应）
  **解决方案**：在BroadcastReceiver中避免执行耗时操作，使用Service处理

- **问题**：广播过于频繁导致性能下降
  **解决方案**：使用LocalBroadcastManager，或优化广播发送频率

- **问题**：广播接收器占用过多内存
  **解决方案**：及时注销BroadcastReceiver，避免内存泄漏

### 安全问题
- **问题**：广播被其他应用接收
  **解决方案**：使用本地广播，或添加权限保护

- **问题**：广播发送敏感数据
  **解决方案**：使用LocalBroadcastManager，或加密数据

- **问题**：恶意应用发送伪造广播
  **解决方案**：添加权限验证，或使用签名验证

### 兼容性问题
- **问题**：Android版本差异导致广播行为不同
  **解决方案**：适配不同Android版本的广播行为，如Android 8.0的隐式广播限制

- **问题**：某些系统广播在不同Android版本中行为不同
  **解决方案**：查阅官方文档，了解不同版本的变化

## 代码示例

### 静态注册BroadcastReceiver
```xml
<!-- AndroidManifest.xml -->
<receiver android:name=".MyBroadcastReceiver">
    <intent-filter>
        <action android:name="android.intent.action.BOOT_COMPLETED" />
        <action android:name="android.net.conn.CONNECTIVITY_CHANGE" />
    </intent-filter>
</receiver>
```

```java
public class MyBroadcastReceiver extends BroadcastReceiver {
    private static final String TAG = "MyBroadcastReceiver";
    
    @Override
    public void onReceive(Context context, Intent intent) {
        String action = intent.getAction();
        Log.d(TAG, "Received broadcast: " + action);
        
        if (Intent.ACTION_BOOT_COMPLETED.equals(action)) {
            // 处理开机启动
            Log.d(TAG, "Boot completed");
        } else if (ConnectivityManager.CONNECTIVITY_ACTION.equals(action)) {
            // 处理网络状态变化
            ConnectivityManager cm = (ConnectivityManager) context.getSystemService(Context.CONNECTIVITY_SERVICE);
            NetworkInfo activeNetwork = cm.getActiveNetworkInfo();
            boolean isConnected = activeNetwork != null && activeNetwork.isConnectedOrConnecting();
            Log.d(TAG, "Network connected: " + isConnected);
        }
    }
}
```

### 动态注册BroadcastReceiver
```java
public class MainActivity extends AppCompatActivity {
    private static final String TAG = "MainActivity";
    private NetworkChangeReceiver networkChangeReceiver;
    
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
    }
    
    @Override
    protected void onStart() {
        super.onStart();
        // 注册广播接收器
        IntentFilter filter = new IntentFilter();
        filter.addAction(ConnectivityManager.CONNECTIVITY_ACTION);
        networkChangeReceiver = new NetworkChangeReceiver();
        registerReceiver(networkChangeReceiver, filter);
        Log.d(TAG, "Broadcast receiver registered");
    }
    
    @Override
    protected void onStop() {
        super.onStop();
        // 注销广播接收器
        if (networkChangeReceiver != null) {
            unregisterReceiver(networkChangeReceiver);
            networkChangeReceiver = null;
            Log.d(TAG, "Broadcast receiver unregistered");
        }
    }
    
    class NetworkChangeReceiver extends BroadcastReceiver {
        @Override
        public void onReceive(Context context, Intent intent) {
            ConnectivityManager cm = (ConnectivityManager) context.getSystemService(Context.CONNECTIVITY_SERVICE);
            NetworkInfo activeNetwork = cm.getActiveNetworkInfo();
            boolean isConnected = activeNetwork != null && activeNetwork.isConnectedOrConnecting();
            Log.d(TAG, "Network connected: " + isConnected);
            
            // 显示网络状态
            Toast.makeText(context, isConnected ? "Network connected" : "Network disconnected", Toast.LENGTH_SHORT).show();
        }
    }
}
```

### 发送和接收自定义广播
```java
// 发送广播
Intent intent = new Intent("com.example.MY_CUSTOM_ACTION");
intent.putExtra("message", "Hello from broadcast");
// 发送标准广播
sendBroadcast(intent);
// 发送有序广播
// sendOrderedBroadcast(intent, null);

// 接收广播
public class CustomBroadcastReceiver extends BroadcastReceiver {
    private static final String TAG = "CustomBroadcastReceiver";
    
    @Override
    public void onReceive(Context context, Intent intent) {
        if ("com.example.MY_CUSTOM_ACTION".equals(intent.getAction())) {
            String message = intent.getStringExtra("message");
            Log.d(TAG, "Received custom broadcast: " + message);
            Toast.makeText(context, "Received: " + message, Toast.LENGTH_SHORT).show();
        }
    }
}

// 动态注册自定义广播接收器
IntentFilter filter = new IntentFilter("com.example.MY_CUSTOM_ACTION");
registerReceiver(new CustomBroadcastReceiver(), filter);
```

### 使用LocalBroadcastManager
```java
public class MainActivity extends AppCompatActivity {
    private static final String TAG = "MainActivity";
    private LocalBroadcastManager localBroadcastManager;
    private BroadcastReceiver localReceiver;
    
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        
        // 获取LocalBroadcastManager实例
        localBroadcastManager = LocalBroadcastManager.getInstance(this);
        
        // 注册本地广播接收器
        IntentFilter filter = new IntentFilter("com.example.LOCAL_ACTION");
        localReceiver = new BroadcastReceiver() {
            @Override
            public void onReceive(Context context, Intent intent) {
                String message = intent.getStringExtra("message");
                Log.d(TAG, "Received local broadcast: " + message);
                Toast.makeText(context, "Local broadcast: " + message, Toast.LENGTH_SHORT).show();
            }
        };
        localBroadcastManager.registerReceiver(localReceiver, filter);
        
        // 发送本地广播
        findViewById(R.id.send_button).setOnClickListener(v -> {
            Intent intent = new Intent("com.example.LOCAL_ACTION");
            intent.putExtra("message", "Hello from local broadcast");
            localBroadcastManager.sendBroadcast(intent);
        });
    }
    
    @Override
    protected void onDestroy() {
        super.onDestroy();
        // 注销本地广播接收器
        if (localReceiver != null) {
            localBroadcastManager.unregisterReceiver(localReceiver);
        }
    }
}
```

### 有序广播示例
```java
// 发送有序广播
Intent intent = new Intent("com.example.ORDERED_ACTION");
intent.putExtra("data", "Original data");
sendOrderedBroadcast(intent, null);

// 高优先级接收器
public class HighPriorityReceiver extends BroadcastReceiver {
    @Override
    public void onReceive(Context context, Intent intent) {
        String data = getResultData();
        Log.d("HighPriorityReceiver", "Received: " + data);
        
        // 修改广播数据
        setResultData("Modified by high priority receiver");
    }
}

// 低优先级接收器
public class LowPriorityReceiver extends BroadcastReceiver {
    @Override
    public void onReceive(Context context, Intent intent) {
        String data = getResultData();
        Log.d("LowPriorityReceiver", "Received: " + data);
    }
}

// 注册有序广播接收器（在代码中设置优先级）
IntentFilter highFilter = new IntentFilter("com.example.ORDERED_ACTION");
highFilter.setPriority(100); // 高优先级
registerReceiver(new HighPriorityReceiver(), highFilter);

IntentFilter lowFilter = new IntentFilter("com.example.ORDERED_ACTION");
lowFilter.setPriority(50); // 低优先级
registerReceiver(new LowPriorityReceiver(), lowFilter);
```

## 面试常考问题及参考答案

### 基础理论

**1. BroadcastReceiver的生命周期是怎样的？**

**答案**：BroadcastReceiver的生命周期非常短暂，当接收到广播时，系统会创建BroadcastReceiver实例并调用onReceive()方法，onReceive()方法执行完毕后，BroadcastReceiver实例会被销毁。因此，BroadcastReceiver不适合执行耗时操作，否则会导致ANR（应用无响应）。

**2. 静态注册和动态注册BroadcastReceiver的区别是什么？**

**答案**：
- **静态注册**：在AndroidManifest.xml中声明，系统在应用安装时注册，即使应用未启动也能接收广播，适合接收系统广播
- **动态注册**：在代码中注册，应用启动后才能接收广播，需要手动注销，适合接收应用内部广播或需要根据应用状态决定是否接收的广播

**3. 广播的类型有哪些？它们的区别是什么？**

**答案**：
- **标准广播**：完全异步，所有接收器同时接收，效率高，但无法中断
- **有序广播**：同步执行，按优先级顺序接收，可以中断广播传递，修改广播数据
- **本地广播**：只在应用内部传递，安全性高，效率高，适合应用内部通信

### 实际应用

**4. 如何处理BroadcastReceiver中的耗时操作？**

**答案**：在BroadcastReceiver的onReceive()方法中，不应该执行耗时操作，否则会导致ANR。如果需要执行耗时操作，应该：
- 启动Service处理耗时操作
- 使用IntentService处理异步任务
- 使用Handler或线程处理短期耗时操作

**5. 如何在Android 8.0及以上版本使用BroadcastReceiver？**

**答案**：Android 8.0及以上版本对隐式广播进行了限制，静态注册的BroadcastReceiver无法接收大部分隐式广播。解决方案：
- 使用动态注册接收广播
- 使用JobScheduler替代广播
- 使用显式广播（指定组件名）
- 对于系统广播，使用PendingIntent

**6. 如何确保广播的安全性？**

**答案**：
- 使用LocalBroadcastManager发送本地广播
- 添加权限保护，限制只有具有特定权限的应用才能接收广播
- 使用签名验证，确保只有相同签名的应用才能接收广播
- 避免在广播中传递敏感数据

### 性能优化

**7. 如何优化BroadcastReceiver的性能？**

**答案**：
- 避免在onReceive()方法中执行耗时操作
- 使用LocalBroadcastManager发送本地广播
- 减少广播的发送频率
- 及时注销动态注册的BroadcastReceiver
- 避免使用静态注册的BroadcastReceiver接收频繁的广播

**8. 如何减少BroadcastReceiver的内存泄漏？**

**答案**：
- 及时注销动态注册的BroadcastReceiver
- 避免在BroadcastReceiver中持有Activity或Context的引用
- 使用WeakReference引用Context
- 避免在BroadcastReceiver中创建匿名内部类

### 架构设计

**9. 如何设计一个可扩展的BroadcastReceiver架构？**

**答案**：
- 抽象出广播处理的通用逻辑
- 使用接口定义广播处理的方法
- 实现广播的优先级管理
- 使用事件总线模式替代广播
- 采用模块化设计，将不同类型的广播处理分离

**10. BroadcastReceiver与EventBus的区别是什么？**

**答案**：
- **BroadcastReceiver**：系统级组件，支持跨应用通信，适合接收系统事件
- **EventBus**：第三方库，只在应用内部通信，适合应用内部组件间的通信
- **EventBus的优点**：
  - 代码更简洁，不需要注册和注销
  - 支持粘性事件
  - 性能更好，因为不需要跨进程通信
  - 类型安全，避免字符串常量错误
- **BroadcastReceiver的优点**：
  - 支持跨应用通信
  - 可以接收系统广播
  - 与系统集成更紧密