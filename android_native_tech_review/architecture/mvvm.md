# MVVM 架构

## 核心概念

MVVM (Model-View-ViewModel) 是一种软件架构模式，是 MVP 架构的进一步发展，将应用程序分为三个主要组件：

- **Model**: 数据模型，负责数据的存储和业务逻辑
- **View**: 视图，负责界面展示和用户交互
- **ViewModel**: 视图模型，负责处理视图逻辑和数据绑定

## 实现原理

在 Android 中，MVVM 架构的实现如下：

1. **Model**: 通常是数据模型类、数据库操作类、网络请求类等
2. **View**: 对应 Android 中的 Activity、Fragment、XML 布局等
3. **ViewModel**: 处理业务逻辑，与 Model 交互，并通过数据绑定更新视图

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
    public LiveData<User> getUserById(int id) {
        MutableLiveData<User> data = new MutableLiveData<>();
        
        // 模拟异步操作
        new Thread(new Runnable() {
            @Override
            public void run() {
                try {
                    Thread.sleep(1000);
                    data.postValue(new User("John", 25));
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }).start();
        
        return data;
    }
    
    public void saveUser(User user) {
        // 保存用户数据到数据库或网络
    }
}
```

### 2. ViewModel 层

```java
public class UserViewModel extends ViewModel {
    private UserRepository repository;
    private MutableLiveData<Boolean> isLoading = new MutableLiveData<>();
    private MutableLiveData<String> error = new MutableLiveData<>();
    private LiveData<User> user;
    
    public UserViewModel() {
        repository = new UserRepository();
    }
    
    public void loadUser(int id) {
        isLoading.setValue(true);
        error.setValue(null);
        
        user = repository.getUserById(id);
        
        // 监听数据加载完成
        Transformations.map(user, new Function<User, User>() {
            @Override
            public User apply(User user) {
                isLoading.setValue(false);
                return user;
            }
        });
    }
    
    public LiveData<User> getUser() {
        return user;
    }
    
    public LiveData<Boolean> getIsLoading() {
        return isLoading;
    }
    
    public LiveData<String> getError() {
        return error;
    }
}
```

### 3. View 层

```xml
<!-- activity_main.xml -->
<layout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto">
    
    <data>
        <variable
            name="viewModel"
            type="com.example.mvvm.UserViewModel" />
    </data>
    
    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:orientation="vertical"
        android:padding="16dp">
        
        <ProgressBar
            android:id="@+id/progress_bar"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_gravity="center"
            android:visibility="@{viewModel.isLoading ? View.VISIBLE : View.GONE}" />
        
        <TextView
            android:id="@+id/tv_error"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:text="@{viewModel.error}"
            android:visibility="@{viewModel.error != null ? View.VISIBLE : View.GONE}" />
        
        <TextView
            android:id="@+id/tv_user_info"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:text="@{viewModel.user != null ? `Name: ${viewModel.user.name}, Age: ${viewModel.user.age}` : `User Info`}"
            android:textSize="18sp" />
        
        <Button
            android:id="@+id/btn_load_user"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="Load User"
            android:onClick="@{() -> viewModel.loadUser(1)}" />
    </LinearLayout>
</layout>
```

```java
public class MainActivity extends AppCompatActivity {
    private ActivityMainBinding binding;
    private UserViewModel viewModel;
    
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        
        // 初始化视图绑定
        binding = ActivityMainBinding.inflate(getLayoutInflater());
        setContentView(binding.getRoot());
        
        // 初始化 ViewModel
        viewModel = new ViewModelProvider(this).get(UserViewModel.class);
        
        // 绑定 ViewModel
        binding.setViewModel(viewModel);
        binding.setLifecycleOwner(this);
    }
}
```

## 高级特性

### 1. 生命周期感知

```java
public class UserViewModel extends ViewModel {
    private UserRepository repository;
    private MutableLiveData<User> user = new MutableLiveData<>();
    private CompositeDisposable disposables = new CompositeDisposable();
    
    public UserViewModel() {
        repository = new UserRepository();
    }
    
    public void loadUser(int id) {
        // 使用 RxJava 处理异步操作
        disposables.add(
            repository.getUserByIdRx(id)
                .subscribeOn(Schedulers.io())
                .observeOn(AndroidSchedulers.mainThread())
                .subscribe(
                    user -> this.user.setValue(user),
                    throwable -> {
                        // 处理错误
                    }
                )
        );
    }
    
    @Override
    protected void onCleared() {
        super.onCleared();
        disposables.clear();
    }
    
    public LiveData<User> getUser() {
        return user;
    }
}
```

### 2. 依赖注入

使用 Hilt 进行依赖注入：

```java
// 应用组件
@HiltAndroidApp
public class MyApplication extends Application {
}

// 模块
@Module
@InstallIn(ViewModelComponent.class)
public abstract class UserModule {
    @Binds
    public abstract UserRepository bindUserRepository(UserRepositoryImpl repository);
}

// ViewModel
@HiltViewModel
public class UserViewModel extends ViewModel {
    private final UserRepository repository;
    
    @Inject
    public UserViewModel(UserRepository repository) {
        this.repository = repository;
    }
    
    // 实现省略
}

// Activity
@AndroidEntryPoint
public class MainActivity extends AppCompatActivity {
    private UserViewModel viewModel;
    
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        // 初始化代码
        viewModel = new ViewModelProvider(this).get(UserViewModel.class);
    }
}
```

### 3. 状态管理

使用 StateFlow 或 LiveData 管理复杂状态：

```kotlin
// 使用 Kotlin + StateFlow
class UserViewModel(private val repository: UserRepository) : ViewModel() {
    private val _uiState = MutableStateFlow<UserUiState>(UserUiState.Loading)
    val uiState: StateFlow<UserUiState> = _uiState
    
    fun loadUser(id: Int) {
        viewModelScope.launch {
            _uiState.value = UserUiState.Loading
            try {
                val user = repository.getUserById(id)
                _uiState.value = UserUiState.Success(user)
            } catch (e: Exception) {
                _uiState.value = UserUiState.Error(e.message ?: "Unknown error")
            }
        }
    }
}

sealed class UserUiState {
    object Loading : UserUiState()
    data class Success(val user: User) : UserUiState()
    data class Error(val message: String) : UserUiState()
}
```

## 最佳实践

1. **数据绑定**: 使用 Android 数据绑定库或 Jetpack Compose
2. **生命周期管理**: 使用 ViewModel 和 LiveData 处理生命周期
3. **异步操作**: 使用 Kotlin Coroutines 或 RxJava 处理异步操作
4. **依赖注入**: 使用 Hilt 或 Koin 进行依赖注入
5. **状态管理**: 使用 StateFlow 或 LiveData 管理复杂状态
6. **测试**: 便于单元测试和 UI 测试

## 常见问题及解决方案

### 1. 内存泄漏

**问题**: ViewModel 持有 Context 引用，导致内存泄漏

**解决方案**: 
- ViewModel 不应持有 Context 引用
- 使用 Application Context 而非 Activity Context
- 使用 LifecycleOwner 管理生命周期

### 2. 数据绑定错误

**问题**: 数据绑定表达式错误或不更新

**解决方案**: 
- 检查绑定表达式语法
- 确保 LiveData 值正确更新
- 使用正确的 LifecycleOwner

### 3. 状态管理复杂

**问题**: 复杂应用的状态管理困难

**解决方案**: 
- 使用状态模式管理视图状态
- 采用单向数据流
- 使用 Redux 或 MVI 架构

### 4. 性能问题

**问题**: 数据绑定或 LiveData 导致性能问题

**解决方案**: 
- 避免在绑定表达式中执行复杂计算
- 使用 `binding.executePendingBindings()` 优化绑定
- 合理使用 `Transformations` 操作 LiveData

## 面试问题及答案

### 1. MVVM 架构的优缺点是什么？

**优点**:
- 视图和业务逻辑分离，代码结构清晰
- 使用数据绑定，减少样板代码
- 适合大型应用
- ViewModel 不持有 View 引用，避免内存泄漏
- 易于单元测试

**缺点**:
- 学习曲线较陡
- 数据绑定可能增加调试难度
- 对于小型应用可能过于复杂

### 2. MVVM 和 MVP 的区别是什么？

**MVVM**:
- 使用数据绑定，View 自动更新
- ViewModel 不持有 View 引用
- 适合大型应用
- 代码量相对较少

**MVP**:
- Presenter 主动更新 View
- View 和 Presenter 通过接口通信
- 适合中型应用
- 代码量相对较多

### 3. 如何在 Android 中实现 MVVM 架构？

- **Model**: 数据模型、数据仓库、网络请求等
- **View**: Activity、Fragment、XML 布局等
- **ViewModel**: 处理业务逻辑，与 Model 交互
- 使用数据绑定库或 Jetpack Compose
- 使用 ViewModel 和 LiveData 处理生命周期
- 使用 Kotlin Coroutines 或 RxJava 处理异步操作

### 4. 如何解决 MVVM 架构中的内存泄漏问题？

- ViewModel 不应持有 Context 引用
- 使用 Application Context 而非 Activity Context
- 使用 LifecycleOwner 管理生命周期
- 及时取消异步操作
- 使用弱引用持有必要的引用

### 5. MVVM 架构适合什么样的应用？

- 大型应用
- 对代码质量和可维护性要求较高的应用
- 团队协作开发的应用
- 需要清晰代码结构的应用
- 适合使用 Jetpack Compose 的现代 Android 应用

## 总结

MVVM 架构是一种现代化的软件架构模式，适合大型 Android 应用。它通过 ViewModel 和数据绑定，实现了视图和业务逻辑的分离，提高了代码的可测试性和可维护性。

在实际开发中，应根据应用的规模和复杂度选择合适的架构模式。对于大型应用，MVVM 是一个不错的选择；对于中型应用，可以考虑 MVP；对于小型应用，则可以采用 MVC。

无论采用哪种架构模式，都应遵循职责分离、代码复用、可测试性等原则，以提高代码质量和开发效率。