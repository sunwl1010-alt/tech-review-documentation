# MVP 架构

## 核心概念

MVP (Model-View-Presenter) 是一种软件架构模式，是 MVC 架构的改进版，将应用程序分为三个主要组件：

- **Model**: 数据模型，负责数据的存储和业务逻辑
- **View**: 视图，负责界面展示和用户交互
- **Presenter**:  presenter，作为中间层，连接视图和模型

## 实现原理

在 Android 中，MVP 架构的实现如下：

1. **Model**: 通常是数据模型类、数据库操作类、网络请求类等
2. **View**: 对应 Android 中的 Activity、Fragment，通过接口定义视图操作
3. **Presenter**: 处理业务逻辑，与 Model 交互，并通过 View 接口更新视图

## 基本使用

### 1. View 接口

```java
public interface UserView {
    void showLoading();
    void hideLoading();
    void showUserInfo(User user);
    void showError(String message);
}
```

### 2. Model 层

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

### 3. Presenter 层

```java
public class UserPresenter {
    private UserView view;
    private UserRepository repository;
    
    public UserPresenter(UserView view) {
        this.view = view;
        this.repository = new UserRepository();
    }
    
    public void loadUser(int id) {
        view.showLoading();
        
        // 模拟异步操作
        new Thread(new Runnable() {
            @Override
            public void run() {
                try {
                    Thread.sleep(1000);
                    final User user = repository.getUserById(id);
                    
                    // 在主线程更新视图
                    new Handler(Looper.getMainLooper()).post(new Runnable() {
                        @Override
                        public void run() {
                            view.hideLoading();
                            view.showUserInfo(user);
                        }
                    });
                } catch (InterruptedException e) {
                    e.printStackTrace();
                    new Handler(Looper.getMainLooper()).post(new Runnable() {
                        @Override
                        public void run() {
                            view.hideLoading();
                            view.showError("加载失败");
                        }
                    });
                }
            }
        }).start();
    }
    
    public void detachView() {
        this.view = null;
    }
}
```

### 4. View 实现

```java
public class MainActivity extends AppCompatActivity implements UserView {
    private TextView tvUserInfo;
    private Button btnLoadUser;
    private ProgressBar progressBar;
    private UserPresenter presenter;
    
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        
        // 初始化 View
        tvUserInfo = findViewById(R.id.tv_user_info);
        btnLoadUser = findViewById(R.id.btn_load_user);
        progressBar = findViewById(R.id.progress_bar);
        
        // 初始化 Presenter
        presenter = new UserPresenter(this);
        
        // 设置点击事件
        btnLoadUser.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                presenter.loadUser(1);
            }
        });
    }
    
    @Override
    public void showLoading() {
        progressBar.setVisibility(View.VISIBLE);
    }
    
    @Override
    public void hideLoading() {
        progressBar.setVisibility(View.GONE);
    }
    
    @Override
    public void showUserInfo(User user) {
        tvUserInfo.setText("Name: " + user.getName() + ", Age: " + user.getAge());
    }
    
    @Override
    public void showError(String message) {
        Toast.makeText(this, message, Toast.LENGTH_SHORT).show();
    }
    
    @Override
    protected void onDestroy() {
        super.onDestroy();
        presenter.detachView();
    }
}
```

## 高级特性

### 1. 生命周期管理

```java
public class BasePresenter<V> {
    private V view;
    
    public void attachView(V view) {
        this.view = view;
    }
    
    public void detachView() {
        this.view = null;
    }
    
    protected V getView() {
        return view;
    }
    
    protected boolean isViewAttached() {
        return view != null;
    }
}

// 使用
public class UserPresenter extends BasePresenter<UserView> {
    // 实现省略
    
    public void loadUser(int id) {
        if (isViewAttached()) {
            getView().showLoading();
        }
        
        // 异步操作
        // ...
        
        if (isViewAttached()) {
            getView().hideLoading();
            getView().showUserInfo(user);
        }
    }
}
```

### 2. 依赖注入

使用 Dagger 或 Koin 等依赖注入框架：

```java
// 使用 Dagger
@Component
public interface AppComponent {
    void inject(MainActivity activity);
}

@Module
public class UserModule {
    @Provides
    UserRepository provideUserRepository() {
        return new UserRepository();
    }
    
    @Provides
    UserPresenter provideUserPresenter(UserRepository repository, UserView view) {
        return new UserPresenter(repository, view);
    }
}
```

## 最佳实践

1. **接口分离**: 使用接口定义 View 和 Presenter 的交互
2. **生命周期管理**: 正确处理 View 的生命周期，避免内存泄漏
3. **线程管理**: 使用线程池或 RxJava 处理异步操作
4. **错误处理**: 统一处理错误和异常
5. **测试**: 便于单元测试，可单独测试 Presenter

## 常见问题及解决方案

### 1. 内存泄漏

**问题**: Presenter 持有 View 引用，导致内存泄漏

**解决方案**: 
- 在 View 的 onDestroy 方法中调用 presenter.detachView()
- 使用弱引用持有 View
- 避免在 Presenter 中执行长时间运行的操作

### 2. 代码复杂度

**问题**: Presenter 代码可能变得复杂

**解决方案**: 
- 将业务逻辑拆分为多个 Presenter
- 使用 UseCase 模式分离业务逻辑
- 采用模块化设计

### 3. 状态管理

**问题**: 视图状态管理复杂

**解决方案**: 
- 使用状态模式管理视图状态
- 实现状态持久化
- 采用事件总线传递状态变化

## 面试问题及答案

### 1. MVP 架构的优缺点是什么？

**优点**:
- 视图和业务逻辑分离，代码结构清晰
- 易于单元测试
- 适合中型应用
- 视图和 Presenter 通过接口通信，耦合度低

**缺点**:
- 代码量增加
- Presenter 可能变得臃肿
- 视图状态管理复杂

### 2. MVP 和 MVVM 的区别是什么？

**MVP**:
- Presenter 主动更新 View
- View 和 Presenter 通过接口通信
- 适合中型应用

**MVVM**:
- 使用数据绑定，View 自动更新
- ViewModel 不持有 View 引用
- 适合大型应用

### 3. 如何在 Android 中实现 MVP 架构？

- **Model**: 数据模型、数据仓库、网络请求等
- **View**: Activity、Fragment，通过接口定义视图操作
- **Presenter**: 处理业务逻辑，连接 View 和 Model
- 使用接口分离 View 和 Presenter
- 正确管理 View 的生命周期

### 4. 如何解决 MVP 架构中的内存泄漏问题？

- 在 View 的 onDestroy 方法中调用 presenter.detachView()
- 使用弱引用持有 View
- 避免在 Presenter 中执行长时间运行的操作
- 使用生命周期感知组件

### 5. MVP 架构适合什么样的应用？

- 中型应用
- 对代码质量和可测试性要求较高的应用
- 团队协作开发的应用
- 需要清晰代码结构的应用

## 总结

MVP 架构是一种改进的软件架构模式，适合中型 Android 应用。它通过 Presenter 作为中间层，实现了视图和业务逻辑的分离，提高了代码的可测试性和可维护性。

在实际开发中，应根据应用的规模和复杂度选择合适的架构模式。对于中型应用，MVP 是一个不错的选择；对于大型应用，则可以考虑 MVVM 或清洁架构。

无论采用哪种架构模式，都应遵循职责分离、代码复用、可测试性等原则，以提高代码质量和开发效率。