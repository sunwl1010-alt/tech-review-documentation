# Flutter 架构

## 框架层

### Widgets 层
- **核心组件**：Text、Image、Container 等基础 Widget
- **布局组件**：Row、Column、Stack、Flex 等
- **Material/Cupertino**：Material Design 和 iOS 风格的 UI 组件

### Rendering 层
- **RenderObject**：负责布局和绘制
- **Layout**：计算 Widget 位置和大小
- **Painting**：绘制 Widget 内容

### Foundation 层
- **动画**：Animation、AnimationController
- **手势**：GestureDetector、InkWell
- **主题**：ThemeData、ColorScheme

## 引擎层

### Skia
- **图形渲染引擎**：负责 2D 图形绘制
- **跨平台**：在不同平台上提供一致的渲染效果
- **性能优化**：硬件加速、图层合成

### Dart VM
- **JIT**：开发时使用即时编译，支持热重载
- **AOT**：发布时使用预编译，提高性能
- **垃圾回收**：自动内存管理

### 平台嵌入层
- **Android**：通过 JNI 与原生代码交互
- **iOS**：通过 Objective-C/Swift 与原生代码交互
- **Web**：通过 CanvasKit 或 DOM 渲染

## 平台层

### 平台特定代码
- **Android**：Java/Kotlin 代码
- **iOS**：Objective-C/Swift 代码
- **Web**：JavaScript/TypeScript 代码
- **桌面**：C++/C# 代码

### 平台服务
- **传感器**：加速度计、陀螺仪等
- **相机**：访问设备相机
- **定位**：获取设备位置
- **存储**：访问本地存储

## 架构优势

1. **跨平台一致性**：一套代码在多个平台运行
2. **高性能**：Skia 渲染引擎提供流畅的 UI
3. **热重载**：快速开发和调试
4. **丰富的组件库**：Material 和 Cupertino 组件
5. **灵活的布局系统**：响应式布局

## 架构流程图

```
┌─────────────────────────────────────────────────┐
│                 Flutter App                   │
├─────────────────────────────────────────────────┤
│             Framework Layer                   │
│  ┌─────────────┐ ┌─────────────┐ ┌─────────────┐
│  │  Widgets    │ │ Rendering   │ │ Foundation  │
│  └─────────────┘ └─────────────┘ └─────────────┘
├─────────────────────────────────────────────────┤
│             Engine Layer                      │
│  ┌─────────────┐ ┌─────────────┐ ┌─────────────┐
│  │    Skia     │ │  Dart VM    │ │ Platform    │
│  └─────────────┘ └─────────────┘ └─────────────┘
├─────────────────────────────────────────────────┤
│             Platform Layer                    │
│  ┌─────────────┐ ┌─────────────┐ ┌─────────────┐
│  │   Android   │ │    iOS      │ │    Web      │
│  └─────────────┘ └─────────────┘ └─────────────┘
└─────────────────────────────────────────────────┘
```

## 最佳实践

1. **理解 Widget 生命周期**：掌握 StatelessWidget 和 StatefulWidget 的生命周期
2. **合理使用状态管理**：根据应用复杂度选择合适的状态管理方案
3. **优化渲染性能**：避免不必要的重建和重绘
4. **平台特定代码隔离**：使用 Platform Channel 处理平台特定功能
5. **遵循 Material Design 或 Cupertino 设计规范**：提供一致的用户体验

## 常见问题

1. **性能问题**：Widget 重建频繁、布局层次过深
2. **内存泄漏**：未正确处理 Stream、Timer 等资源
3. **平台兼容性**：不同平台的行为差异
4. **热重载失效**：代码结构问题导致热重载不生效
5. **构建失败**：依赖冲突、版本不兼容

## 面试题

1. **Q**: Flutter 的架构分层是怎样的？
   **A**: Flutter 分为三层：框架层（Widgets、Rendering、Foundation）、引擎层（Skia、Dart VM、平台嵌入层）和平台层（平台特定代码和服务）

2. **Q**: Flutter 如何实现跨平台渲染？
   **A**: 使用 Skia 图形渲染引擎，在不同平台上提供一致的渲染效果

3. **Q**: JIT 和 AOT 有什么区别？
   **A**: JIT（即时编译）用于开发时，支持热重载；AOT（预编译）用于发布时，提高性能

4. **Q**: Flutter 的热重载原理是什么？
   **A**: 基于 Dart 的 JIT 编译器，在代码变更时只重新编译修改的部分，保持应用状态

5. **Q**: 如何处理 Flutter 与原生平台的交互？
   **A**: 使用 Platform Channel（MethodChannel、EventChannel、BasicMessageChannel）