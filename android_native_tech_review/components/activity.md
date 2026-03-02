# Activity组件

## 核心概念

### Activity
- **定义**：Android应用的基本构建块，代表一个具有用户界面的单一屏幕
- **作用**：处理用户交互，显示UI元素，管理生命周期
- **重要性**：是Android应用的核心组件，用户与应用交互的主要入口

### 生命周期
- ** onCreate()**：Activity创建时调用，进行初始化操作
- **onStart()**：Activity变为可见时调用
- **onResume()**：Activity获得焦点并开始与用户交互时调用
- **onPause()**：Activity失去焦点但仍然可见时调用
- **onStop()**：Activity完全不可见时调用
- **onDestroy()**：Activity被销毁时调用
- **onRestart()**：Activity从停止状态重新开始时调用

### 启动模式
- **standard**：默认模式，每次启动都创建新实例
- **singleTop**：如果Activity在栈顶，不创建新实例
- **singleTask**：整个任务栈中只有一个实例
- **singleInstance**：单独的任务栈，且只有一个实例

### Intent
- **定义**：组件间通信的消息对象
- **作用**：启动Activity、Service，传递数据
- **类型**：显式Intent（指定组件）、隐式Intent（指定动作）

## 实现原理

### 生命周期管理
- **由系统管理**：Activity的生命周期由Android系统管理
- **状态转换**：系统根据用户操作和系统事件触发生命周期方法
- **保存与恢复**：通过onSaveInstanceState()和onRestoreInstanceState()保存和恢复状态

### 启动模式实现
- **standard**：每次启动都创建新实例，放入任务栈顶
- **singleTop**：检查栈顶是否为该Activity，是则不创建新实例
- **singleTask**：检查任务栈中是否存在该Activity，存在则将其移至栈顶
- **singleInstance**：创建新的任务栈，且该任务栈中只有一个实例

### Intent传递机制
- **显式Intent**：通过组件名直接指定目标组件
- **隐式Intent**：通过动作、数据、类型等匹配目标组件
- **Intent Filter**：组件声明自己能处理的Intent

### 状态管理
- **保存状态**：在onSaveInstanceState()中保存临时状态
- **恢复状态**：在onCreate()或onRestoreInstanceState()中恢复状态
- **持久化存储**：使用SharedPreferences、数据库等持久化存储数据

## 使用场景

### 基本使用
- **创建Activity**：继承Activity或AppCompatActivity
- **声明Activity**：在AndroidManifest.xml中声明
- **启动Activity**：使用Intent启动Activity
- **传递数据**：通过Intent传递数据

### 生命周期使用
- **初始化**：在onCreate()中进行初始化操作
- **资源管理**：在onStart()和onStop()中管理可见性相关资源
- **交互管理**：在onResume()和onPause()中管理交互相关资源
- **资源释放**：在onDestroy()中释放资源

### 启动模式使用
- **standard**：适用于大多数场景，如列表项详情页
- **singleTop**：适用于接收通知启动的Activity
- **singleTask**：适用于应用的主界面
- **singleInstance**：适用于系统级应用，如拨号、闹钟等

### 数据传递
- **基本数据**：通过Intent.putExtra()传递基本类型数据
- **对象**：通过Serializable或Parcelable传递对象
- **返回结果**：通过startActivityForResult()和onActivityResult()传递返回结果

## 常见问题及解决方案

### 生命周期问题
- **问题**：Activity被意外销毁后状态丢失
  **解决方案**：在onSaveInstanceState()中保存状态，在onCreate()或onRestoreInstanceState()中恢复状态

- **问题**：后台Activity被系统回收
  **解决方案**：使用onSaveInstanceState()保存状态，合理使用Service处理后台任务

- **问题**：生命周期方法调用顺序混乱
  **解决方案**：理解生命周期方法的调用时机，避免在生命周期方法中进行耗时操作

### 启动模式问题
- **问题**：启动模式选择不当导致Activity栈混乱
  **解决方案**：根据具体场景选择合适的启动模式

- **问题**：singleTask模式下Intent数据传递失败
  **解决方案**：重写onNewIntent()方法处理新的Intent

- **问题**：使用singleInstance模式导致任务栈管理复杂
  **解决方案**：谨慎使用singleInstance模式，只在必要时使用

### 内存管理问题
- **问题**：Activity内存泄漏
  **解决方案**：避免静态引用Activity，及时释放资源，使用WeakReference

- **问题**：Bitmap内存占用过大
  **解决方案**：合理使用Bitmap，及时回收，使用图片加载库

- **问题**：后台Activity占用内存
  **解决方案**：在onStop()中释放不必要的资源，使用onTrimMemory()处理内存紧张情况

### 配置变更问题
- **问题**：屏幕旋转等配置变更导致Activity重建
  **解决方案**：使用onSaveInstanceState()保存状态，或在AndroidManifest.xml中设置android:configChanges

## 代码示例

### 基本Activity创建
```java
public class MainActivity extends AppCompatActivity {
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        
        // 初始化操作
        initViews();
        initData();
    }
    
    private void initViews() {
        // 初始化UI组件
    }
    
    private void initData() {
        // 初始化数据
    }
}
```

### 生命周期方法重写
```java
public class LifecycleActivity extends AppCompatActivity {
    private static final String TAG = "LifecycleActivity";
    
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_lifecycle);
        Log.d(TAG, "onCreate");
    }
    
    @Override
    protected void onStart() {
        super.onStart();
        Log.d(TAG, "onStart");
    }
    
    @Override
    protected void onResume() {
        super.onResume();
        Log.d(TAG, "onResume");
    }
    
    @Override
    protected void onPause() {
        super.onPause();
        Log.d(TAG, "onPause");
    }
    
    @Override
    protected void onStop() {
        super.onStop();
        Log.d(TAG, "onStop");
    }
    
    @Override
    protected void onDestroy() {
        super.onDestroy();
        Log.d(TAG, "onDestroy");
    }
    
    @Override
    protected void onRestart() {
        super.onRestart();
        Log.d(TAG, "onRestart");
    }
}
```

### 状态保存与恢复
```java
public class StateActivity extends AppCompatActivity {
    private static final String KEY_COUNTER = "counter";
    private int counter = 0;
    private TextView counterTextView;
    
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_state);
        
        counterTextView = findViewById(R.id.counter_text);
        
        // 恢复状态
        if (savedInstanceState != null) {
            counter = savedInstanceState.getInt(KEY_COUNTER, 0);
            updateCounter();
        }
        
        findViewById(R.id.increment_button).setOnClickListener(v -> {
            counter++;
            updateCounter();
        });
    }
    
    @Override
    protected void onSaveInstanceState(Bundle outState) {
        super.onSaveInstanceState(outState);
        outState.putInt(KEY_COUNTER, counter);
    }
    
    @Override
    protected void onRestoreInstanceState(Bundle savedInstanceState) {
        super.onRestoreInstanceState(savedInstanceState);
        counter = savedInstanceState.getInt(KEY_COUNTER, 0);
        updateCounter();
    }
    
    private void updateCounter() {
        counterTextView.setText(String.valueOf(counter));
    }
}
```

### 启动Activity并传递数据
```java
// 启动Activity
Intent intent = new Intent(this, SecondActivity.class);
intent.putExtra("name", "John");
intent.putExtra("age", 30);
startActivity(intent);

// 接收数据
public class SecondActivity extends AppCompatActivity {
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_second);
        
        Intent intent = getIntent();
        String name = intent.getStringExtra("name");
        int age = intent.getIntExtra("age", 0);
        
        TextView textView = findViewById(R.id.data_text);
        textView.setText("Name: " + name + ", Age: " + age);
    }
}
```

### 启动Activity并获取返回结果
```java
// 启动Activity
private static final int REQUEST_CODE = 1;

Intent intent = new Intent(this, ResultActivity.class);
startActivityForResult(intent, REQUEST_CODE);

// 处理返回结果
@Override
protected void onActivityResult(int requestCode, int resultCode, Intent data) {
    super.onActivityResult(requestCode, resultCode, data);
    
    if (requestCode == REQUEST_CODE && resultCode == RESULT_OK) {
        String result = data.getStringExtra("result");
        TextView resultTextView = findViewById(R.id.result_text);
        resultTextView.setText("Result: " + result);
    }
}

// 返回结果
public class ResultActivity extends AppCompatActivity {
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_result);
        
        findViewById(R.id.ok_button).setOnClickListener(v -> {
            Intent intent = new Intent();
            intent.putExtra("result", "OK");
            setResult(RESULT_OK, intent);
            finish();
        });
        
        findViewById(R.id.cancel_button).setOnClickListener(v -> {
            setResult(RESULT_CANCELED);
            finish();
        });
    }
}
```

## 面试常考问题及参考答案

### 基础理论

**1. Activity的生命周期有哪些方法？它们的调用顺序是什么？**

**答案**：Activity的生命周期方法包括：
- `onCreate()`：Activity创建时调用
- `onStart()`：Activity变为可见时调用
- `onResume()`：Activity获得焦点并开始与用户交互时调用
- `onPause()`：Activity失去焦点但仍然可见时调用
- `onStop()`：Activity完全不可见时调用
- `onDestroy()`：Activity被销毁时调用
- `onRestart()`：Activity从停止状态重新开始时调用

调用顺序：
- 首次启动：onCreate() → onStart() → onResume()
- 按下Home键：onPause() → onStop()
- 从Home键返回：onRestart() → onStart() → onResume()
- 按下Back键：onPause() → onStop() → onDestroy()

**2. Activity的启动模式有哪些？它们的区别是什么？**

**答案**：Activity的启动模式包括：
- **standard**：默认模式，每次启动都创建新实例
- **singleTop**：如果Activity在栈顶，不创建新实例
- **singleTask**：整个任务栈中只有一个实例
- **singleInstance**：单独的任务栈，且只有一个实例

区别：
- standard：每次启动都创建新实例，适用于大多数场景
- singleTop：避免栈顶重复创建相同Activity，适用于接收通知启动的Activity
- singleTask：确保整个任务栈中只有一个实例，适用于应用的主界面
- singleInstance：单独的任务栈，适用于系统级应用，如拨号、闹钟等

**3. 如何在Activity之间传递数据？**

**答案**：
- 使用Intent.putExtra()传递基本类型数据
- 使用Serializable或Parcelable传递对象
- 使用startActivityForResult()和onActivityResult()传递返回结果
- 使用静态变量或全局变量传递数据（不推荐）
- 使用EventBus等第三方库传递数据

### 实际应用

**4. 如何处理Activity的配置变更？**

**答案**：
- 使用onSaveInstanceState()保存状态，在onCreate()或onRestoreInstanceState()中恢复状态
- 在AndroidManifest.xml中设置android:configChanges属性，指定不需要系统重新创建Activity的配置变更
- 使用ViewModel保存UI相关的数据，避免配置变更时数据丢失

**5. 如何避免Activity内存泄漏？**

**答案**：
- 避免静态引用Activity
- 及时释放资源，如在onDestroy()中取消网络请求、注销广播接收器等
- 使用WeakReference引用Activity
- 避免在Activity中创建匿名内部类，如必须创建，确保在适当的时候释放
- 使用Application Context而不是Activity Context

**6. 如何处理Activity被系统回收的情况？**

**答案**：
- 在onSaveInstanceState()中保存必要的状态数据
- 在onCreate()中检查savedInstanceState是否为null，如果不为null则恢复状态
- 使用Service处理后台任务，避免依赖Activity的生命周期
- 合理使用onTrimMemory()方法处理内存紧张情况

### 性能优化

**7. 如何优化Activity的启动速度？**

**答案**：
- 减少onCreate()中的耗时操作
- 使用异步加载数据
- 延迟加载非关键资源
- 合理使用启动模式
- 优化布局层级，减少View的嵌套
- 使用Splash Screen提升用户体验

**8. 如何减少Activity的内存占用？**

**答案**：
- 及时释放资源，如在onStop()中释放不必要的资源
- 合理使用Bitmap，及时回收
- 使用图片加载库，如Glide、Picasso等
- 避免内存泄漏
- 使用onTrimMemory()方法处理内存紧张情况

### 架构设计

**9. 如何设计一个可扩展的Activity架构？**

**答案**：
- 采用MVC、MVP或MVVM架构
- 抽取BaseActivity，封装通用功能
- 使用Fragment替代Activity处理复杂UI
- 合理使用Intent传递数据，避免过度耦合
- 使用依赖注入框架，如Dagger2
- 采用模块化设计，将功能拆分为独立模块

**10. Activity与Fragment的区别是什么？**

**答案**：
- **Activity**：代表一个完整的屏幕，有自己的生命周期，是应用的基本构建块
- **Fragment**：代表Activity中的一部分UI，依赖于Activity，有自己的生命周期
- **区别**：
  - Activity是独立的，Fragment依赖于Activity
  - 一个Activity可以包含多个Fragment
  - Fragment的生命周期受Activity影响
  - Fragment更适合处理复杂的UI布局
  - Fragment可以在不同的Activity中重用