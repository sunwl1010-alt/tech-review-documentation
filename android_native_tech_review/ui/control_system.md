# Android 控件体系（状态、交互与可扩展设计）

## 1. 控件设计目标

高质量控件应满足：
- 状态一致：加载/成功/空态/错误态可预测
- 交互可靠：防抖、幂等、可恢复
- 易复用：可配置而不失控
- 易测试：行为可断言

## 2. 状态驱动 UI

不建议直接在多个点击回调里散改控件状态。推荐单一 `UiState` 映射：

```kotlin
data class LoginUiState(
    val loading: Boolean = false,
    val enableSubmit: Boolean = false,
    val errorText: String? = null
)
```

View 只做渲染：
- `loading` -> Progress 显示
- `enableSubmit` -> Button enabled
- `errorText` -> Error 提示

## 3. 输入控件治理

- 输入合法性：本地快速校验 + 服务端最终校验
- 输入节流：搜索框 debounce
- 焦点管理：避免键盘遮挡与焦点跳转异常

## 4. 点击防抖与幂等

```kotlin
fun View.setSafeClickListener(intervalMs: Long = 600, onClick: (View) -> Unit) {
    var lastClick = 0L
    setOnClickListener {
        val now = SystemClock.elapsedRealtime()
        if (now - lastClick >= intervalMs) {
            lastClick = now
            onClick(it)
        }
    }
}
```

## 5. 自定义控件边界

适合自定义控件：
- 业务通用但系统控件无法直接满足
- 交互复杂且跨页面重复出现

不适合：
- 仅单页面一次性需求
- 样式变化频繁但行为简单（优先组合现有控件）

## 6. 主题与设计系统

- 使用 Material3 主题 token（color/type/shape）
- 主题切换（明暗/品牌）通过 token 层实现
- 避免在代码里硬编码颜色和尺寸

## 7. 无障碍与国际化

- 所有可交互控件配置 `contentDescription`
- 触控区域 >= 48dp
- 文本不硬编码，支持多语言与 RTL

## 8. 常见问题

- 按钮重复点击导致重复请求
- 文本输入监听触发过密导致卡顿
- 自定义 View `onDraw` 做重计算

解决策略：
- 防抖 + 幂等 key
- 输入节流 + 后台计算
- 缓存绘制参数，减少每帧分配

## 9. 面试深问

- 自定义 View 的 `onMeasure/onLayout/onDraw` 如何分工？
- 如何设计一个可复用且可测试的业务控件？
- 为什么 UI 状态建模比回调堆叠更稳定？
