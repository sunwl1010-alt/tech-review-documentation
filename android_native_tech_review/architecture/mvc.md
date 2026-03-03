# MVC 架构

## 核心概念

MVC (Model-View-Controller) 是一种经典的软件架构模式，将应用程序分为三个主要组件：

- **Model**: 数据模型，负责数据的存储和业务逻辑
- **View**: 视图，负责界面展示
- **Controller**: 控制器，负责处理用户输入并更新模型和视图

## 实现原理

在 Android 中，MVC 架构的实现如下：

1. **Model**: 通常是数据模型类、数据库操作类、网络请求类等
2. **View**: 对应 Android 中的 Activity、Fragment、XML 布局等
3. **Controller**: 通常是 Activity 或 Fragment 中的逻辑代码

## 基本使用

### 1. Model 层

```java
// 数据模型
public class User {
    private String name;
    private int age;
    
    // 构造方法、getter、setter 省略
}

// 数据仓库
public class UserRepository {
    public User getUserById(int id) {
        // 从数据库或网络获取用户数据
        return new User("John", 25);
    }
    
    public void saveUser(User user) {
        // 保存用户数据到数据库或网络
    }
}
```

### 2. View 层

```xml
<!-- activity_main.xml -->
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:padding="16dp">
    
    <TextView
        android:id="@+id/tv_user_info"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="User Info"
        android:textSize="18sp"/>
    
    <Button
        android:id="@+id/btn_load_user"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Load User"/>
</LinearLayout>
```

### 3. Controller 层

```java
public class MainActivity extends AppCompatActivity {
    private TextView tvUserInfo;
    private Button btnLoadUser;
    private UserRepository userRepository;
    
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        
        // 初始化 View
        tvUserInfo = findViewById(R.id.tv_user_info);
        btnLoadUser = findViewById(R.id.btn_load_user);
        
        // 初始化 Model
        userRepository = new UserRepository();
        
        // 设置点击事件
        btnLoadUser.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                // 处理用户输入
                User user = userRepository.getUserById(1);
                // 更新视图
                tvUserInfo.setText("Name: " + user.getName() + ", Age: " + user.getAge());
            }
        });
    }
}
```

## 高级特性

### 1. 事件总线

使用 EventBus 或 LocalBroadcastManager 来处理组件间的通信：

```java
// 使用 EventBus
public class UserEvent {
    private User user;
    
    public UserEvent(User user) {
        this.user = user;
    }
    
    public User getUser() {
        return user;
    }
}

// 在 Controller 中发送事件
EventBus.getDefault().post(new UserEvent(user));

// 在 View 中接收事件
@Subscribe
public void onUserEvent(UserEvent event) {
    User user = event.getUser();
    tvUserInfo.setText("Name: " + user.getName() + ", Age: " + user.getAge());
}
```

### 2. 异步操作

使用 AsyncTask 或线程池处理异步操作：

```java
private class LoadUserTask extends AsyncTask<Integer, Void, User> {
    @Override
    protected User doInBackground(Integer... params) {
        int userId = params[0];
        return userRepository.getUserById(userId);
    }
    
    @Override
    protected void onPostExecute(User user) {
        tvUserInfo.setText("Name: " + user.getName() + ", Age: " + user.getAge());
    }
}

// 调用
new LoadUserTask().execute(1);
```

## 最佳实践

1. **职责分离**: 确保 Model、View、Controller 各自的职责明确
2. **接口定义**: 使用接口定义 Model 和 View 的交互
3. **数据绑定**: 考虑使用数据绑定库简化 View 更新
4. **依赖注入**: 使用依赖注入框架管理依赖关系
5. **测试**: 编写单元测试和集成测试

## 常见问题及解决方案

### 1. 控制器臃肿

**问题**: Activity 或 Fragment 中的代码过于复杂，职责不明确

**解决方案**: 
- 将业务逻辑移到 Model 层
- 使用 Presenter 或 ViewModel 分离逻辑
- 采用模块化设计

### 2. 数据与视图同步

**问题**: 数据更新后视图没有及时更新

**解决方案**: 
- 使用观察者模式
- 实现数据绑定
- 确保在主线程更新 UI

### 3. 代码复用性差

**问题**: 相似功能的代码重复出现

**解决方案**: 
- 提取公共代码到工具类
- 使用继承或组合
- 采用设计模式提高代码复用性

## 面试问题及答案

### 1. MVC 架构的优缺点是什么？

**优点**:
- 职责分离，代码结构清晰
- 易于理解和维护
- 适合小型应用

**缺点**:
- 控制器可能会变得臃肿
- 视图和控制器耦合度高
- 不适合大型复杂应用

### 2. MVC 和 MVP 的区别是什么？

**MVC**:
- 控制器直接更新视图
- 视图和控制器耦合度高
- 适合小型应用

**MVP**:
- Presenter 作为中间层，隔离视图和模型
- 视图和 Presenter 通过接口通信
- 适合中型应用

### 3. 如何在 Android 中实现 MVC 架构？

- **Model**: 数据模型、数据仓库、网络请求等
- **View**: Activity、Fragment、XML 布局等
- **Controller**: Activity 或 Fragment 中的逻辑代码
- 使用接口定义交互
- 采用事件总线或回调机制处理通信

### 4. 如何解决 MVC 架构中控制器臃肿的问题？

- 将业务逻辑移到 Model 层
- 引入 Presenter 或 ViewModel
- 采用模块化设计
- 使用依赖注入框架

### 5. MVC 架构适合什么样的应用？

- 小型应用
- 功能相对简单的应用
- 快速开发的原型应用
- 对架构要求不高的应用

## 总结

MVC 架构是一种经典的软件架构模式，适合小型 Android 应用。它将应用分为 Model、View、Controller 三个组件，实现了职责分离，使代码结构更加清晰。

在实际开发中，应根据应用的规模和复杂度选择合适的架构模式。对于小型应用，MVC 是一个不错的选择；对于中型应用，可以考虑 MVP 或 MVVM；对于大型应用，则可以采用清洁架构或组件化架构。

无论采用哪种架构模式，都应遵循职责分离、代码复用、可测试性等原则，以提高代码质量和开发效率。