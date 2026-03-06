# Android 布局体系（性能与可维护性）

## 1. 布局不只是“排版”

布局直接影响：
- 首帧与滑动性能
- 可维护性与迭代速度
- 设备适配复杂度

建议目标：
- 层级可控
- 约束清晰
- 可测可演进

## 2. 布局选型决策

- 简单线性结构：`LinearLayout`
- 复杂相对关系：`ConstraintLayout`
- 列表页：`RecyclerView`
- 瀑布/网格：`GridLayoutManager` / `StaggeredGridLayoutManager`
- 大量动态 UI：优先 Compose（或混合）

避免：
- 为了“看起来简单”堆叠多层 LinearLayout
- 在 item 布局里嵌套过深导致测量成本高

## 3. 关键性能指标

- 帧预算：16.6ms（60fps）
- 过度绘制层数
- `measure/layout/draw` 耗时
- 首次布局次数与重布局频率

## 4. 常见性能问题与修复

### 4.1 层级过深
- 问题：频繁 measure/layout，滚动卡顿
- 方案：用 ConstraintLayout 扁平化层级

### 4.2 权重滥用
- 问题：`layout_weight` 导致二次测量
- 方案：改用约束或固定尺寸策略

### 4.3 列表 item 重布局
- 问题：bind 时改动过多 layout params
- 方案：预定义布局状态，减少运行时结构变化

## 5. RecyclerView 布局实战

- 使用 `ListAdapter + DiffUtil`，避免 `notifyDataSetChanged`
- 稳定 ID 提升复用命中
- 复杂 item 拆分 ViewHolder 类型

```kotlin
class FeedAdapter : ListAdapter<FeedItem, RecyclerView.ViewHolder>(Diff) {
    override fun getItemId(position: Int): Long = getItem(position).stableId
    init { setHasStableIds(true) }
}
```

## 6. 大屏与多尺寸适配

- 使用 `sw600dp`/`sw720dp` 资源目录
- 关键间距和字号通过 design token 管理
- 平板场景优先双栏布局，避免简单拉伸手机 UI

## 7. 可维护布局规范

- 命名统一：`screen_xxx.xml`、`item_xxx.xml`
- 复用通用组件：header/card/empty/error
- 避免魔法值：统一 dimens/colors/styles

## 8. Compose 混合场景建议

- 新模块优先 Compose，旧模块可渐进迁移
- 在 XML 中通过 `ComposeView` 接入
- 状态上移，避免 XML 与 Compose 双状态源

## 9. 面试深问

- ConstraintLayout 为什么通常比多层 LinearLayout 更优？
- RecyclerView 列表卡顿的系统排查顺序是什么？
- 多设备适配如何兼顾一致性与效率？
