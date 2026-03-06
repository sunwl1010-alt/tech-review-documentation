# Activity 深度实践

## 1. Activity 的真实职责

Activity 应聚焦三件事：
- 生命周期边界管理
- UI 容器与导航协调
- 系统回调入口（权限、结果、配置变化）

不建议在 Activity 中承载：
- 复杂业务逻辑
- 网络请求编排
- 持久化策略

## 2. 生命周期与资源绑定

推荐将资源按生命周期分层：
- `onCreate/onDestroy`：一次性初始化与释放（binding、adapter）
- `onStart/onStop`：可见期资源（location listener、camera preview）
- `onResume/onPause`：交互期资源（sensor、动画、输入焦点）

## 3. 状态恢复策略

状态分三类：
- 瞬时 UI 状态：`savedInstanceState`
- 可重建状态：`ViewModel`
- 持久状态：数据库/磁盘

```kotlin
override fun onSaveInstanceState(outState: Bundle) {
    outState.putString("query", binding.searchInput.text.toString())
    super.onSaveInstanceState(outState)
}
```

## 4. 结果回传：Activity Result API

```kotlin
private val pickImage = registerForActivityResult(
    ActivityResultContracts.GetContent()
) { uri ->
    if (uri != null) viewModel.onImagePicked(uri)
}

private fun openPicker() {
    pickImage.launch("image/*")
}
```

避免使用已过时的 `startActivityForResult`。

## 5. 启动模式与任务栈设计

- `standard`：默认，最通用
- `singleTop`：通知跳转详情等“栈顶复用”场景
- `singleTask`：主路由页或外部 deep link 汇聚页

设计原则：
- 不要滥用 `singleTask/singleInstance`
- 明确“返回路径”是产品体验，不只是技术实现

## 6. Deep Link 与安全

- `intent-filter` 精确匹配 host/path
- 校验参数合法性，防止注入和越权
- 对敏感页面增加登录态与权限二次校验

## 7. 配置变化与多窗口

- 屏幕旋转/字体缩放/语言切换都可能触发重建
- 不建议通过 `configChanges` 粗暴拦截所有变化
- 使用 `ViewModel + SavedStateHandle` 保持状态连续性

## 8. 常见问题与修复

### 8.1 内存泄漏
- 静态持有 Activity
- 长生命周期对象持有 View 引用
- 监听器未注销

### 8.2 启动闪屏过长
- 首屏同步初始化过重
- 主线程阻塞 I/O

### 8.3 返回栈异常
- 混用 launchMode + flag 无统一规范

建议建立“导航规范文档”，统一 flag 组合。

## 9. 实战模板

```kotlin
@AndroidEntryPoint
class ProfileActivity : AppCompatActivity() {

    private lateinit var binding: ActivityProfileBinding
    private val viewModel: ProfileViewModel by viewModels()

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        binding = ActivityProfileBinding.inflate(layoutInflater)
        setContentView(binding.root)

        lifecycleScope.launch {
            repeatOnLifecycle(Lifecycle.State.STARTED) {
                viewModel.uiState.collect(::render)
            }
        }

        binding.btnRetry.setOnClickListener { viewModel.retry() }
    }

    private fun render(state: ProfileUiState) {
        // 只做 UI 映射，不放业务决策
    }
}
```

## 10. 面试深问

- Activity 与 Fragment 职责怎么划分？
- 复杂任务栈如何设计才能兼顾通知跳转和返回体验？
- 为什么不建议在 Activity 内直接发网络请求？
