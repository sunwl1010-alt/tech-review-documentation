# Activity 启动流程面试专题（AMS / WMS / 任务栈）

## 1. 面试先给结论

Activity 启动不是“单进程函数调用”，而是一条跨进程协作链路：
- App 进程发起启动请求
- System Server 中 AMS 负责调度与任务栈决策
- WMS 负责窗口与显示准备
- 目标 Activity 在主线程完成生命周期回调

理解这条链路，才能回答启动慢、任务栈异常、冷启动白屏等问题。

## 2. 简化启动链路（高频）

以 `startActivity` 为例：
1. 调用方执行 `startActivity`（Activity/Context）
2. 请求通过 Binder 进入 AMS（System Server）
3. AMS 做权限、Intent、launchMode、taskAffinity 等决策
4. 若目标进程未启动，先走 Zygote fork 目标进程
5. AMS 通知目标进程调度 ActivityThread
6. 主线程执行 `performLaunchActivity` -> `onCreate/onStart/onResume`
7. WMS 完成窗口附着与可见性切换

关键词：`AMS`、`ActivityThread`、`Instrumentation`、`WMS`。

## 3. 冷启动 / 热启动 / 温启动区别

- 冷启动：进程不存在，需要创建进程和 Application
- 温启动：进程在，但 Activity 需重建
- 热启动：Activity 已在内存中，恢复前台

面试答法：冷启动最慢，优化价值最高；热启动更关注任务栈和状态恢复。

## 4. AMS 在启动流程中的核心职责

- 任务栈（Task）选择与复用
- launchMode 与 intent flag 解析
- 进程存活性判断与拉起
- Activity 生命周期切换时序协调

常见问题：返回栈混乱通常不是“页面 bug”，而是任务栈策略没设计好。

## 5. WMS 相关高频点

WMS 不负责业务逻辑，主要负责窗口层级、动画、可见性和输入分发关联。  
启动白屏、闪屏异常、界面切换抖动常与窗口时序和首帧准备有关。

## 6. 任务栈与启动模式实战

- `standard`：默认，每次新建
- `singleTop`：栈顶复用，常见于通知跳转
- `singleTask`：任务内唯一，入口汇聚页常用
- `singleInstance`：隔离任务栈，慎用

建议：先设计用户返回路径，再确定 launchMode 与 flags。

## 7. 常见面试追问

### Q1：为什么启动一个 Activity 需要 AMS？
答：Activity 属于系统级组件调度对象，涉及权限、任务栈、跨进程生命周期，必须由 AMS 统一协调。

### Q2：启动慢主要慢在哪？
答：常见慢点在进程创建、Application 初始化、主线程阻塞、首屏资源加载和首帧渲染。

### Q3：如何解释“点击跳转后偶发白屏”？
答：通常是首帧准备慢、主线程阻塞或窗口可见时序问题。排查需结合启动打点 + Perfetto。

### Q4：通知点击后返回键行为异常怎么排查？
答：优先看 launchMode + flags + 任务栈历史，确认是否错误复用/新建 task。

## 8. 启动优化与流程关联

可按阶段优化：
- 进程阶段：减少 Application 阻塞初始化
- Activity 阶段：首屏只做必要同步任务
- 渲染阶段：降低首屏布局复杂度，避免主线程重 I/O
- 链路监控：分段打点（点击 -> 跳转 -> onCreate -> 首帧）

## 9. 易错点清单

- 只改 launchMode 不验证返回路径
- 在 `onCreate` 做大量同步 I/O
- 把业务初始化和 UI 初始化混在一起
- 忽略多进程场景下的启动差异

## 10. 60秒答题模板

- 结论：Activity 启动是 AMS/WMS/应用进程协同的跨进程链路。
- 依据：AMS 做任务栈和生命周期调度，WMS 管窗口显示，应用主线程执行生命周期。
- 落地：我会按启动阶段打点定位瓶颈，分离阻塞初始化并校正任务栈策略。
- 边界：仅调整 launchMode 无法根治慢启动与白屏，需要结合主线程和渲染链路优化。
