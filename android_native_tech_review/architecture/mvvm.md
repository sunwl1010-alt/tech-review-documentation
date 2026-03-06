# MVVM 架构（进阶落地）

## 1. MVVM 的目标
MVVM 不是“用了 ViewModel 就算完成”，核心目标是：
- UI 状态可预测
- 业务逻辑可测试
- 数据流可追踪
- 模块边界可演进

## 2. 推荐分层关系

- View：Activity/Fragment/Compose，仅做 UI 映射与事件上报
- ViewModel：状态机 + 事件处理 + UseCase 编排
- Model：Repository/DataSource/Entity

依赖方向：View -> ViewModel -> Domain/Repository。

## 3. 状态建模优先于回调堆叠

```kotlin
data class UserUiState(
    val loading: Boolean = false,
    val data: User? = null,
    val error: String? = null
)

sealed interface UserIntent {
    data class Load(val id: String) : UserIntent
    data object Retry : UserIntent
}
```

## 4. ViewModel 实现模板

```kotlin
@HiltViewModel
class UserViewModel @Inject constructor(
    private val fetchUser: FetchUserProfileUseCase
) : ViewModel() {

    private val _uiState = MutableStateFlow(UserUiState())
    val uiState: StateFlow<UserUiState> = _uiState

    fun dispatch(intent: UserIntent) {
        when (intent) {
            is UserIntent.Load -> load(intent.id)
            UserIntent.Retry -> _uiState.value.data?.id?.let(::load)
        }
    }

    private fun load(id: String) {
        viewModelScope.launch {
            _uiState.update { it.copy(loading = true, error = null) }
            when (val result = fetchUser(id)) {
                is Result.Success -> _uiState.update {
                    it.copy(loading = false, data = result.value)
                }
                is Result.Failure -> _uiState.update {
                    it.copy(loading = false, error = result.error.message)
                }
            }
        }
    }
}
```

## 5. View 层收集策略

```kotlin
lifecycleScope.launch {
    repeatOnLifecycle(Lifecycle.State.STARTED) {
        viewModel.uiState.collect { render(it) }
    }
}
```

不要在 View 里直接调用 Repository，避免绕开状态机。

## 6. LiveData 与 StateFlow 取舍

- 新项目优先 `StateFlow/SharedFlow`
- 存量项目可逐步从 LiveData 迁移
- 一次性事件不要滥用 LiveData（易粘性事件）

## 7. 常见反模式

- ViewModel 返回 `MutableLiveData` 给 View（破坏封装）
- ViewModel 持有 Activity/Fragment 引用（内存泄漏）
- 把 DTO 直接暴露给 UI（模型污染）
- UI 层写 if-else 业务规则（职责错位）

## 8. 测试策略

### 8.1 ViewModel 单测
- 给定 intent
- 断言状态流转顺序（loading -> success/error）

### 8.2 Repository 单测
- Mock 网络与本地源
- 覆盖缓存命中、失效、异常分支

## 9. 与 Clean Architecture 配合

- UseCase 提供业务语义
- ViewModel 只编排，不承载底层实现
- Repository 负责数据策略

这让 MVVM 从“页面模式”升级为“工程体系”。

## 10. 面试深问

- MVVM 与 MVI 的边界是什么？
- 如何解决复杂页面的状态爆炸？
- 如何把 MVVM 落地到遗留项目而不大改？
