# Flutter 渲染引擎与 Skia 深入

## 主题定位

这篇文档用于补齐 Flutter 底层原理，重点回答几个高频问题：

1. Flutter 为什么能跨平台保持一致渲染？
2. Skia 在 Flutter 里到底负责什么？
3. Widget、Element、RenderObject、Layer、Scene 之间是什么关系？
4. UI 线程和 Raster 线程分别做什么？
5. Flutter 为什么有时会掉帧？

## 一、Flutter 为什么能自己渲染

Flutter 的核心思路不是“调用平台原生控件拼 UI”，而是“自己掌控渲染管线”。

这意味着：

- 大部分控件不是 Android `View` 或 iOS `UIView`
- Flutter 自己计算布局
- Flutter 自己生成绘制指令
- Flutter 再把绘制指令交给底层渲染引擎执行

直接收益：

- 跨平台视觉一致性强
- 复杂动画更可控
- 自定义 UI 成本低

代价：

- 引擎体积更大
- 与平台原生控件体系天然存在隔离
- 需要额外处理平台嵌入和混合渲染

## 二、Flutter 引擎分层

Flutter 可以粗分为三层：

### 1. Framework

主要由 Dart 编写，包括：

- Widget 系统
- 手势系统
- 动画系统
- 状态管理基础设施
- 渲染抽象层

开发者平时主要接触的是这一层。

### 2. Engine

主要由 C++ 实现，包括：

- Dart Runtime
- 文本排版
- 图像解码
- 合成器
- 渲染后端
- 与平台窗口系统交互

Skia 主要工作在这一层。

### 3. Embedder

平台嵌入层，负责把 Flutter 引擎接到具体平台上，例如：

- Android
- iOS
- Windows
- macOS
- Linux

Embedder 负责：

- 创建窗口和 Surface
- 处理输入事件
- 对接平台消息循环
- 管理线程和 VSync

## 三、Skia 到底是什么

Skia 是一个 2D 图形引擎，Flutter 很长一段时间都依赖 Skia 作为核心渲染后端。

Skia 负责：

- 绘制文本
- 绘制路径、圆角、阴影、渐变
- 绘制位图和图片
- 执行裁剪、变换、混合
- 输出到 GPU 或 CPU 后端

你可以把它理解成：

- Flutter 负责“描述画什么”
- Skia 负责“真正把它画出来”

## 四、从 Widget 到像素的完整链路

Flutter 的一帧通常会经历下面几步：

1. 构建 Widget Tree
2. 生成或复用 Element Tree
3. 更新 RenderObject Tree
4. 执行 layout
5. 执行 paint
6. 生成 Layer Tree
7. 交给 Engine 合成
8. Raster 线程调用 Skia 输出像素
9. GPU 提交到屏幕显示

可以这样记：

- Widget：配置
- Element：实例与生命周期管理
- RenderObject：布局和绘制
- Layer：合成单元
- Scene：最终待提交画面

## 五、Widget、Element、RenderObject 的关系

### Widget

- 不可变
- 只是配置描述
- 轻量
- 频繁创建没问题

### Element

- Widget 的运行时实例
- 负责树结构和生命周期
- 决定旧 Widget 能否复用

### RenderObject

- 真正参与布局和绘制
- 负责大小、位置、命中测试、paint

面试里常见追问：

“为什么 Flutter 可以频繁 rebuild？”

答案核心是：

- rebuild 的是轻量 Widget
- Element 和 RenderObject 会尽可能复用
- 真正昂贵的是 layout、paint、compositing、raster，而不是单纯创建 Widget 对象

## 六、Layout 阶段做了什么

布局阶段的核心目标是：

- 父节点向子节点传约束 `Constraints`
- 子节点根据约束计算自己的尺寸 `Size`
- 父节点再决定子节点的位置

这套机制的本质是：

- 父给约束
- 子选尺寸
- 父定位置

高频考点：

1. 为什么 `ListView` 放在 `Column` 里常报约束错误？
2. 为什么 `Expanded` 必须放在 `Flex` 体系里？
3. 为什么无界约束会导致布局异常？

这些问题本质都和 `BoxConstraints` 有关。

## 七、Paint 阶段做了什么

Paint 阶段不会重新决定布局，只负责把已经算好的 RenderObject 画出来。

核心点：

- paint 依赖 layout 结果
- paint 可以被局部标脏
- paint 的输出不是最终像素，而是绘制指令

如果只是颜色变化、阴影变化、图片变化，通常是 `markNeedsPaint()`；
如果尺寸变化，则要 `markNeedsLayout()`。

## 八、Layer Tree 与合成

不是所有节点都会生成独立 Layer，只有需要“独立合成”的节点才会产生 Layer，例如：

- `Opacity`
- `Transform`
- `Clip*`
- `RepaintBoundary`
- 平台视图

Layer 的意义：

- 降低整棵树重复重绘成本
- 支持局部缓存
- 支持复杂特效合成

但 Layer 不是越多越好。

问题在于：

- Layer 太多会增加合成成本
- 离屏缓存会占内存
- 某些场景会引入额外 GPU 压力

## 九、线程模型

Flutter 性能面试几乎一定会问线程模型。

常见线程可以这样记：

### 1. UI Thread

负责：

- 执行 Dart 代码
- build
- layout
- paint
- 生成 Layer Tree

### 2. Raster Thread

负责：

- 把 Layer Tree 转成 GPU 可执行的绘制命令
- 调用 Skia 真正 rasterize

### 3. Platform Thread

负责：

- 平台消息循环
- 输入事件
- 插件回调
- 平台视图协调

### 4. IO Thread

负责：

- 图片解码
- 资源加载
- 磁盘或部分异步 IO

## 十、掉帧的本质

Flutter 一帧通常要在 VSync 周期内完成。

在 60Hz 屏幕下，一帧预算大约是 `16.67ms`。
在 120Hz 屏幕下，预算大约是 `8.33ms`。

只要 UI Thread 或 Raster Thread 有一方超时，就可能掉帧。

### 常见原因

1. `build` 过重
2. 布局层级太深
3. 图片解码太大
4. 过多透明层和裁剪
5. 平台视图混合过重
6. 主 isolate 执行大计算
7. 列表一次性创建过多元素

## 十一、Skia 与 Impeller

面试里现在经常会追问：

“Flutter 不是开始推进 Impeller 了吗？那 Skia 还重要吗？”

应答建议：

- Skia 仍然是理解 Flutter 渲染原理的核心入口
- Impeller 是 Flutter 为了解决某些 GPU 后端卡顿、shader 编译抖动等问题推进的新渲染实现
- 即使引入 Impeller，Flutter 的整体渲染流程、树结构分层、帧生产模型依然值得从 Skia 时代的架构去理解

不要把回答说成“Skia 过时了”，这是错误的。

## 十二、实际优化思路

### 1. 先判断瓶颈在哪条线程

- UI 卡顿：优先看 build/layout/同步计算
- Raster 卡顿：优先看阴影、模糊、裁剪、透明层、图片

### 2. 降低不必要的 rebuild

- 使用 `const`
- 缩小状态影响范围
- 拆分 Widget
- 避免整页刷新

### 3. 降低不必要的 repaint

- 使用 `RepaintBoundary`
- 避免动画带动大面积重绘
- 减少复杂装饰叠加

### 4. 把大计算移出主 isolate

- JSON 大对象解析
- 加密压缩
- 图片处理
- 大量排序和聚合

## 十三、面试高频问答

### 1. Flutter 为什么不直接用原生控件？

因为 Flutter 追求的是跨平台一致渲染和完整掌控 UI 管线，而不是依赖平台控件差异化实现。

### 2. Flutter 中 Widget、Element、RenderObject 的区别是什么？

- Widget 是配置
- Element 是运行时节点和生命周期承载
- RenderObject 负责布局和绘制

### 3. Flutter 的一帧经历了哪些阶段？

build -> layout -> paint -> compositing -> raster -> GPU 提交。

### 4. Skia 在 Flutter 中的职责是什么？

负责把引擎提交的绘制信息转成真正的图形输出，包括文本、路径、位图、裁剪和混合。

### 5. 为什么 rebuild 不一定贵？

因为 Widget 很轻，真正昂贵的是后续 layout、paint、合成和 raster。

### 6. `RepaintBoundary` 的作用是什么？

隔离重绘边界，减少父子区域互相牵连的 repaint，但滥用会增加 Layer 成本。

## 十四、追问点清单

如果你要把这部分准备到面试可用，至少还要能继续回答这些追问：

1. 什么情况下会触发重新 layout？
2. 什么情况下只会 repaint 不会 relayout？
3. `Opacity` 为什么可能影响性能？
4. 平台视图为什么常常更难优化？
5. 为什么大图经常拖慢 Raster Thread？
6. isolate 和线程是什么关系？

## 十五、记忆框架

你可以用一条线记忆：

`Widget 配置 -> Element 管生命周期 -> RenderObject 做布局绘制 -> Layer 做合成 -> Skia 做栅格化 -> GPU 出图`

只要这条线讲清楚，Flutter 底层题大多数不会答偏。
