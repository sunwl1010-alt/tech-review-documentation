# SharedPreferences存储

## 核心概念

### SharedPreferences
- **定义**：Android系统中用于存储轻量级键值对数据的机制
- **作用**：存储应用的配置信息、用户偏好设置等
- **特点**：
  - 简单易用
  - 存储在XML文件中
  - 适合存储小量数据
  - 不适合存储复杂数据

### 数据结构
- **键值对**：使用String类型的键和各种基本类型的值
- **支持的数据类型**：String、int、float、long、boolean、Set<String>
- **存储位置**：/data/data/包名/shared_prefs/目录下的XML文件

### 模式
- **MODE_PRIVATE**：默认模式，只有当前应用可以访问
- **MODE_WORLD_READABLE**：已废弃，其他应用可以读取
- **MODE_WORLD_WRITEABLE**：已废弃，其他应用可以写入
- **MODE_MULTI_PROCESS**：已废弃，用于多进程访问

### 编辑操作
- **Editor**：用于编辑SharedPreferences中的数据
- **提交方式**：
  - commit()：同步提交，返回布尔值表示是否成功
  - apply()：异步提交，无返回值

## 实现原理

### 存储机制
- **XML文件**：将键值对存储在XML文件中
- **文件格式**：根元素为<map>，每个键值对为一个<string>、<int>等元素
- **读取过程**：首次读取时加载整个XML文件到内存
- **写入过程**：通过Editor修改内存中的数据，然后写入XML文件

### 编辑机制
- **Editor**：提供putXxx()方法添加或修改数据
- **remove()**：移除指定键的数据
- **clear()**：清空所有数据
- **commit()**：同步写入磁盘，返回布尔值表示是否成功
- **apply()**：异步写入磁盘，无返回值，优先于commit()

### 监听机制
- **OnSharedPreferenceChangeListener**：监听SharedPreferences的变化
- **registerOnSharedPreferenceChangeListener()**：注册监听器
- **unregisterOnSharedPreferenceChangeListener()**：注销监听器

### 多进程访问
- **MODE_MULTI_PROCESS**：已废弃
- **解决方案**：使用ContentProvider或其他跨进程通信机制

## 使用场景

### 应用配置
- **主题设置**：存储用户选择的主题
- **语言设置**：存储用户选择的语言
- **字体大小**：存储用户选择的字体大小
- **通知设置**：存储通知开关状态

### 用户偏好
- **登录状态**：存储用户是否自动登录
- **记住密码**：存储密码（不推荐，不安全）
- **搜索历史**：存储用户的搜索历史
- **浏览历史**：存储用户的浏览历史

### 应用状态
- **上次登录时间**：存储用户上次登录的时间
- **应用版本**：存储应用的版本号
- **首次启动**：存储是否首次启动
- **引导完成**：存储是否完成引导

### 临时数据
- **表单数据**：存储未提交的表单数据
- **会话信息**：存储临时会话信息

## 常见问题及解决方案

### 性能问题
- **问题**：频繁读写SharedPreferences影响性能
  **解决方案**：减少读写频率，使用apply()异步提交

- **问题**：存储数据过大导致加载缓慢
  **解决方案**：只存储必要的数据，使用其他存储方式存储大量数据

- **问题**：多线程并发访问导致数据不一致
  **解决方案**：使用同步机制，或使用apply()确保异步操作

### 安全问题
- **问题**：存储敏感数据不安全
  **解决方案**：使用EncryptedSharedPreferences加密存储，或使用其他安全存储方式

- **问题**：明文存储在XML文件中
  **解决方案**：使用加密，或避免存储敏感数据

- **问题**：应用卸载后数据仍然存在
  **解决方案**：在应用卸载时清理数据，或使用Context.MODE_PRIVATE

### 数据一致性问题
- **问题**：commit()和apply()的区别
  **解决方案**：了解两者的区别，根据需要选择合适的方法

- **问题**：多进程访问数据不一致
  **解决方案**：使用ContentProvider或其他跨进程通信机制

- **问题**：数据丢失
  **解决方案**：定期备份数据，使用事务确保数据完整性

### 版本兼容性问题
- **问题**：不同Android版本的SharedPreferences行为不同
  **解决方案**：适配不同版本的行为，使用最新的API

- **问题**：MODE_WORLD_READABLE和MODE_WORLD_WRITEABLE已废弃
  **解决方案**：使用其他方式实现跨应用数据共享

## 代码示例

### 基本使用
```java
// 获取SharedPreferences实例
SharedPreferences sharedPreferences = getSharedPreferences("my_preferences", Context.MODE_PRIVATE);

// 写入数据
SharedPreferences.Editor editor = sharedPreferences.edit();
editor.putString("username", "John");
editor.putInt("age", 30);
editor.putBoolean("isLoggedIn", true);
editor.apply(); // 异步提交
// editor.commit(); // 同步提交

// 读取数据
String username = sharedPreferences.getString("username", "default");
int age = sharedPreferences.getInt("age", 0);
boolean isLoggedIn = sharedPreferences.getBoolean("isLoggedIn", false);

// 移除数据
editor.remove("username");
editor.apply();

// 清空数据
editor.clear();
editor.apply();
```

### 监听变化
```java
// 注册监听器
SharedPreferences sharedPreferences = getSharedPreferences("my_preferences", Context.MODE_PRIVATE);
SharedPreferences.OnSharedPreferenceChangeListener listener = new SharedPreferences.OnSharedPreferenceChangeListener() {
    @Override
    public void onSharedPreferenceChanged(SharedPreferences sharedPreferences, String key) {
        if ("username".equals(key)) {
            String newUsername = sharedPreferences.getString(key, "");
            Log.d("SharedPreferences", "Username changed to: " + newUsername);
        }
    }
};
sharedPreferences.registerOnSharedPreferenceChangeListener(listener);

// 注销监听器
// sharedPreferences.unregisterOnSharedPreferenceChangeListener(listener);
```

### 存储复杂对象
```java
// 存储对象
public void saveUser(User user) {
    SharedPreferences sharedPreferences = getSharedPreferences("my_preferences", Context.MODE_PRIVATE);
    SharedPreferences.Editor editor = sharedPreferences.edit();
    
    // 使用Gson将对象转换为JSON字符串
    Gson gson = new Gson();
    String userJson = gson.toJson(user);
    
    editor.putString("user", userJson);
    editor.apply();
}

// 读取对象
public User getUser() {
    SharedPreferences sharedPreferences = getSharedPreferences("my_preferences", Context.MODE_PRIVATE);
    String userJson = sharedPreferences.getString("user", "");
    
    if (!userJson.isEmpty()) {
        Gson gson = new Gson();
        return gson.fromJson(userJson, User.class);
    }
    return null;
}

// User类
class User {
    private String name;
    private int age;
    private String email;
    
    // 构造方法、getter和setter
}
```

### 使用EncryptedSharedPreferences
```java
// 添加依赖
// implementation "androidx.security:security-crypto:1.1.0"

// 创建EncryptedSharedPreferences
MasterKey masterKey = new MasterKey.Builder(context)
        .setKeyScheme(MasterKey.KeyScheme.AES256_GCM)
        .build();

SharedPreferences sharedPreferences = EncryptedSharedPreferences.create(
        context,
        "encrypted_preferences",
        masterKey,
        EncryptedSharedPreferences.PrefKeyEncryptionScheme.AES256_SIV,
        EncryptedSharedPreferences.PrefValueEncryptionScheme.AES256_GCM
);

// 使用方式与普通SharedPreferences相同
SharedPreferences.Editor editor = sharedPreferences.edit();
editor.putString("sensitive_data", "password123");
editor.apply();

String sensitiveData = sharedPreferences.getString("sensitive_data", "");
```

### 工具类封装
```java
public class PrefUtils {
    private static final String PREF_NAME = "app_preferences";
    
    // 写入数据
    public static void putString(Context context, String key, String value) {
        SharedPreferences sharedPreferences = context.getSharedPreferences(PREF_NAME, Context.MODE_PRIVATE);
        SharedPreferences.Editor editor = sharedPreferences.edit();
        editor.putString(key, value);
        editor.apply();
    }
    
    public static void putInt(Context context, String key, int value) {
        SharedPreferences sharedPreferences = context.getSharedPreferences(PREF_NAME, Context.MODE_PRIVATE);
        SharedPreferences.Editor editor = sharedPreferences.edit();
        editor.putInt(key, value);
        editor.apply();
    }
    
    public static void putBoolean(Context context, String key, boolean value) {
        SharedPreferences sharedPreferences = context.getSharedPreferences(PREF_NAME, Context.MODE_PRIVATE);
        SharedPreferences.Editor editor = sharedPreferences.edit();
        editor.putBoolean(key, value);
        editor.apply();
    }
    
    // 读取数据
    public static String getString(Context context, String key, String defaultValue) {
        SharedPreferences sharedPreferences = context.getSharedPreferences(PREF_NAME, Context.MODE_PRIVATE);
        return sharedPreferences.getString(key, defaultValue);
    }
    
    public static int getInt(Context context, String key, int defaultValue) {
        SharedPreferences sharedPreferences = context.getSharedPreferences(PREF_NAME, Context.MODE_PRIVATE);
        return sharedPreferences.getInt(key, defaultValue);
    }
    
    public static boolean getBoolean(Context context, String key, boolean defaultValue) {
        SharedPreferences sharedPreferences = context.getSharedPreferences(PREF_NAME, Context.MODE_PRIVATE);
        return sharedPreferences.getBoolean(key, defaultValue);
    }
    
    // 移除数据
    public static void remove(Context context, String key) {
        SharedPreferences sharedPreferences = context.getSharedPreferences(PREF_NAME, Context.MODE_PRIVATE);
        SharedPreferences.Editor editor = sharedPreferences.edit();
        editor.remove(key);
        editor.apply();
    }
    
    // 清空数据
    public static void clear(Context context) {
        SharedPreferences sharedPreferences = context.getSharedPreferences(PREF_NAME, Context.MODE_PRIVATE);
        SharedPreferences.Editor editor = sharedPreferences.edit();
        editor.clear();
        editor.apply();
    }
}

// 使用工具类
PrefUtils.putString(context, "username", "John");
String username = PrefUtils.getString(context, "username", "");
```

## 面试常考问题及参考答案

### 基础理论

**1. SharedPreferences的工作原理是什么？**

**答案**：SharedPreferences将数据存储在XML文件中，位于/data/data/包名/shared_prefs/目录下。首次读取时，会将整个XML文件加载到内存中。写入数据时，通过Editor修改内存中的数据，然后通过commit()同步写入磁盘或apply()异步写入磁盘。

**2. commit()和apply()的区别是什么？**

**答案**：
- **commit()**：同步写入磁盘，返回布尔值表示是否成功
- **apply()**：异步写入磁盘，无返回值
- **apply()的优势**：不会阻塞主线程，性能更好
- **commit()的优势**：可以知道操作是否成功

**3. SharedPreferences适合存储什么类型的数据？**

**答案**：SharedPreferences适合存储轻量级的键值对数据，如应用配置、用户偏好设置等。支持的数据类型包括String、int、float、long、boolean和Set<String>。不适合存储复杂数据和大量数据。

### 实际应用

**4. 如何存储复杂对象到SharedPreferences？**

**答案**：可以使用JSON序列化将复杂对象转换为字符串，然后存储到SharedPreferences中。常用的JSON库有Gson、Jackson等。

**5. 如何加密存储敏感数据？**

**答案**：使用EncryptedSharedPreferences，它是AndroidX Security库提供的加密存储解决方案，可以对SharedPreferences中的数据进行加密。

**6. 如何监听SharedPreferences的变化？**

**答案**：实现OnSharedPreferenceChangeListener接口，然后通过registerOnSharedPreferenceChangeListener()方法注册监听器，当SharedPreferences中的数据发生变化时，会调用onSharedPreferenceChanged()方法。

### 性能优化

**7. 如何优化SharedPreferences的性能？**

**答案**：
- 使用apply()异步提交，避免阻塞主线程
- 减少读写频率，批量操作
- 只存储必要的数据，避免存储大量数据
- 使用多个SharedPreferences文件，按功能分类存储

**8. 为什么SharedPreferences不适合存储大量数据？**

**答案**：
- SharedPreferences会将整个XML文件加载到内存中，存储大量数据会占用过多内存
- 写入操作会涉及磁盘I/O，大量数据会影响性能
- XML格式存储效率不高，不适合存储大量数据

### 安全问题

**9. SharedPreferences的安全隐患有哪些？如何解决？**

**答案**：
- **安全隐患**：
  - 数据存储在明文XML文件中
  - 其他应用可能通过root权限访问
  - 不适合存储敏感数据

- **解决方案**：
  - 使用EncryptedSharedPreferences加密存储
  - 避免存储敏感数据
  - 使用其他安全存储方式，如Keystore

**10. 如何在多进程中使用SharedPreferences？**

**答案**：
- MODE_MULTI_PROCESS已废弃，不建议使用
- 可以使用ContentProvider实现跨进程数据共享
- 可以使用其他跨进程通信机制，如AIDL
- 可以使用文件锁确保数据一致性

### 架构设计

**11. 如何设计一个可扩展的SharedPreferences工具类？**

**答案**：
- 封装常用的读写方法
- 支持不同类型的数据存储
- 提供加密存储选项
- 支持批量操作
- 提供监听机制
- 支持默认值设置

**12. SharedPreferences与其他存储方式的对比？**

**答案**：
- **SharedPreferences**：适合存储轻量级键值对数据，简单易用
- **文件存储**：适合存储大量数据，如文本文件、图片等
- **SQLite**：适合存储结构化数据，支持复杂查询
- **Room**：SQLite的封装，提供更友好的API
- **ContentProvider**：适合跨应用数据共享
- **Keystore**：适合存储加密密钥和敏感数据

根据数据类型和使用场景选择合适的存储方式。对于简单的配置信息和用户偏好，SharedPreferences是最佳选择。