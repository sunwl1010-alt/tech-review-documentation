# Widget 体系

## StatelessWidget

### 基本概念
- **无状态组件**：不可变，一旦创建就不能改变
- **生命周期**：创建 → 构建 → 销毁
- **使用场景**：展示静态内容，不依赖于状态变化

### 示例代码

```dart
class MyStatelessWidget extends StatelessWidget {
  final String title;
  final int count;

  const MyStatelessWidget({
    Key? key,
    required this.title,
    required this.count,
  }) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return Container(
      padding: EdgeInsets.all(16.0),
      child: Column(
        children: [
          Text(title, style: TextStyle(fontSize: 24)),
          Text('Count: $count'),
        ],
      ),
    );
  }
}

// 使用
MyStatelessWidget(
  title: 'Hello',
  count: 42,
)
```

### 最佳实践
- **不可变性**：所有字段都应该是 `final`
- **性能优化**：避免在 `build` 方法中创建复杂对象
- **职责单一**：每个 StatelessWidget 只负责一个功能

## StatefulWidget

### 基本概念
- **有状态组件**：可变，可以响应状态变化
- **生命周期**：创建 → 初始化状态 → 构建 → 状态更新 → 销毁
- **使用场景**：需要响应用户交互或数据变化的场景

### 生命周期

1. **createState()**：创建 State 对象
2. **initState()**：初始化状态，只调用一次
3. **build()**：构建 UI，状态变化时会重新调用
4. **didUpdateWidget()**：Widget 配置变化时调用
5. **setState()**：更新状态，触发重建
6. **dispose()**：清理资源，只调用一次

### 示例代码

```dart
class CounterWidget extends StatefulWidget {
  final String title;

  const CounterWidget({Key? key, required this.title}) : super(key: key);

  @override
  _CounterWidgetState createState() => _CounterWidgetState();
}

class _CounterWidgetState extends State<CounterWidget> {
  int _count = 0;

  @override
  void initState() {
    super.initState();
    // 初始化操作，如订阅流、初始化数据等
  }

  void _incrementCounter() {
    setState(() {
      _count++;
    });
  }

  @override
  void dispose() {
    // 清理资源，如取消订阅、关闭流等
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        Text(widget.title),
        Text('Count: $_count'),
        ElevatedButton(
          onPressed: _incrementCounter,
          child: Text('Increment'),
        ),
      ],
    );
  }
}

// 使用
CounterWidget(title: 'Counter')
```

### 最佳实践
- **状态管理**：只管理与当前 Widget 相关的状态
- **资源管理**：在 `dispose()` 中清理资源，避免内存泄漏
- **性能优化**：避免在 `build` 方法中执行耗时操作

## InheritedWidget

### 基本概念
- **数据共享**：在 Widget 树中向下传递数据
- **效率**：当数据变化时，只有依赖该数据的 Widget 会重建
- **使用场景**：共享主题、用户信息、应用配置等

### 示例代码

```dart
class AppTheme extends InheritedWidget {
  final ThemeData themeData;
  final Function(ThemeData) updateTheme;

  const AppTheme({
    Key? key,
    required this.themeData,
    required this.updateTheme,
    required Widget child,
  }) : super(key: key, child: child);

  static AppTheme of(BuildContext context) {
    final AppTheme? result = context.dependOnInheritedWidgetOfExactType<AppTheme>();
    assert(result != null, 'No AppTheme found in context');
    return result!;
  }

  @override
  bool updateShouldNotify(AppTheme oldWidget) {
    return themeData != oldWidget.themeData;
  }
}

// 使用
class MyApp extends StatefulWidget {
  @override
  _MyAppState createState() => _MyAppState();
}

class _MyAppState extends State<MyApp> {
  ThemeData _themeData = ThemeData.light();

  void _updateTheme(ThemeData newTheme) {
    setState(() {
      _themeData = newTheme;
    });
  }

  @override
  Widget build(BuildContext context) {
    return AppTheme(
      themeData: _themeData,
      updateTheme: _updateTheme,
      child: MaterialApp(
        theme: _themeData,
        home: HomePage(),
      ),
    );
  }
}

// 在子 Widget 中访问
class HomePage extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    final theme = AppTheme.of(context).themeData;
    final updateTheme = AppTheme.of(context).updateTheme;

    return Scaffold(
      appBar: AppBar(title: Text('Home')),
      body: Center(
        child: ElevatedButton(
          onPressed: () {
            updateTheme(ThemeData.dark());
          },
          child: Text('Switch to Dark Theme'),
        ),
      ),
    );
  }
}
```

### 最佳实践
- **轻量级数据**：只共享必要的数据，避免共享过大的对象
- **合理更新**：正确实现 `updateShouldNotify` 方法
- **性能考虑**：避免频繁更新 InheritedWidget 的数据

## RenderObjectWidget

### 基本概念
- **底层 Widget**：直接与渲染引擎交互
- **性能**：比普通 Widget 更接近底层，性能更高
- **使用场景**：需要自定义布局和绘制的场景

### 常见 RenderObjectWidget

1. **LeafRenderObjectWidget**：没有子节点的 RenderObjectWidget
   - 示例：`Text`、`Image`、`RawImage`

2. **SingleChildRenderObjectWidget**：只有一个子节点的 RenderObjectWidget
   - 示例：`Opacity`、`Transform`、`Container`

3. **MultiChildRenderObjectWidget**：有多个子节点的 RenderObjectWidget
   - 示例：`Row`、`Column`、`Stack`

### 自定义 RenderObjectWidget 示例

```dart
class CustomPaintWidget extends LeafRenderObjectWidget {
  final Color color;
  final double size;

  const CustomPaintWidget({
    Key? key,
    required this.color,
    required this.size,
  }) : super(key: key);

  @override
  RenderObject createRenderObject(BuildContext context) {
    return RenderCustomPaint(color: color, size: size);
  }

  @override
  void updateRenderObject(BuildContext context, RenderCustomPaint renderObject) {
    renderObject
      ..color = color
      ..size = size;
  }
}

class RenderCustomPaint extends RenderBox {
  Color _color;
  double _size;

  RenderCustomPaint({required Color color, required double size})
      : _color = color,
        _size = size;

  set color(Color value) {
    if (_color != value) {
      _color = value;
      markNeedsPaint();
    }
  }

  set size(double value) {
    if (_size != value) {
      _size = value;
      markNeedsLayout();
    }
  }

  @override
  void performLayout() {
    size = Size(_size, _size);
  }

  @override
  void paint(PaintingContext context, Offset offset) {
    final canvas = context.canvas;
    final paint = Paint()..color = _color;
    canvas.drawCircle(offset + Offset(_size / 2, _size / 2), _size / 2, paint);
  }
}

// 使用
CustomPaintWidget(
  color: Colors.blue,
  size: 100,
)
```

### 最佳实践
- **性能优化**：只在必要时使用 RenderObjectWidget
- **正确实现**：遵循 RenderObject 的生命周期和方法
- **测试**：充分测试自定义 RenderObject 的性能和正确性

## Widget 树

### 构建过程
1. **Widget 构建**：调用 `build()` 方法生成 Widget 树
2. **Element 树**：Widget 树对应的 Element 树，管理 Widget 的生命周期
3. **RenderObject 树**：负责布局和绘制

### 重建机制
- **Widget 不可变性**：Widget 是不可变的，状态变化时会创建新的 Widget
- **Element 复用**：Element 会尽可能复用，只更新需要变化的部分
- **RenderObject 复用**：RenderObject 也会尽可能复用，提高性能

## 最佳实践

1. **Widget 拆分**：将复杂 UI 拆分为多个小 Widget
2. **状态管理**：根据复杂度选择合适的状态管理方案
3. **性能优化**：
   - 使用 `const` 构造器创建不可变 Widget
   - 避免在 `build` 方法中创建复杂对象
   - 使用 `const` 修饰符优化重建
   - 使用 `RepaintBoundary` 隔离重绘

4. **代码组织**：
   - 按功能组织 Widget
   - 使用命名构造器提高可读性
   - 遵循 Dart 代码风格

## 常见问题

1. **性能问题**：
   - Widget 重建过于频繁
   - 布局层次过深
   - 在 `build` 方法中执行耗时操作

2. **内存泄漏**：
   - 未在 `dispose()` 中清理资源
   - 不正确的 Stream 订阅

3. **状态管理混乱**：
   - 状态管理层次不合理
   - 过度使用全局状态

4. **Widget 生命周期问题**：
   - 不了解 StatefulWidget 的生命周期
   - 在错误的生命周期方法中执行操作

## 面试题

1. **Q**: StatelessWidget 和 StatefulWidget 的区别是什么？
   **A**: StatelessWidget 是不可变的，一旦创建就不能改变；StatefulWidget 是可变的，可以响应状态变化

2. **Q**: InheritedWidget 的作用是什么？
   **A**: InheritedWidget 用于在 Widget 树中向下传递数据，当数据变化时，只有依赖该数据的 Widget 会重建

3. **Q**: Widget、Element 和 RenderObject 的关系是什么？
   **A**: Widget 是配置信息，Element 是 Widget 的实例，RenderObject 负责布局和绘制

4. **Q**: 如何优化 Flutter 应用的性能？
   **A**: 使用 const 构造器、避免在 build 方法中创建复杂对象、使用 RepaintBoundary 隔离重绘、优化状态管理

5. **Q**: StatefulWidget 的生命周期有哪些方法？
   **A**: createState()、initState()、build()、didUpdateWidget()、setState()、dispose()