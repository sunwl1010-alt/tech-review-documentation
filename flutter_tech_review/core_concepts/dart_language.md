# Dart 语言

## 为什么 Flutter 开发必须把 Dart 学扎实
Flutter 不只是“用 Dart 写 UI”。Dart 的类型系统、对象模型、空安全、异步语义，都会直接影响 Flutter 工程质量。

Flutter 里很多常见问题，本质上都和 Dart 语言理解不够有关：

- 状态写法混乱，来自对象与不可变建模不清晰
- 空指针和类型错误，来自空安全使用不规范
- 异步逻辑难维护，来自 `Future` / `Stream` 理解停留在表面
- 性能问题，来自频繁创建对象、滥用闭包、错误的数据结构
- 可维护性差，来自泛型、抽象、扩展能力没有真正掌握

所以 Dart 不是前置语法，而是 Flutter 工程能力的一部分。

## Dart 的定位
Dart 是一门面向对象、强类型、支持 JIT 和 AOT 的语言。它在 Flutter 里的几个关键价值：

- 开发期支持 JIT，配合热重载提升效率
- 发布期支持 AOT，提升启动与运行性能
- 自带空安全，降低空指针风险
- 内置异步模型，适合 UI 驱动的事件式开发
- 语法相对现代，适合写业务也适合写基础设施

结合 Flutter 看，Dart 的价值不是“语法简单”，而是它和 Flutter Framework、Engine 的配合非常紧。

## 变量与不可变性

### `var`、显式类型、`dynamic`
```dart
var name = 'Flutter';
String title = 'Review';
dynamic value = 42;
```

工程原则：

- 局部变量优先用 `var`，前提是类型一眼可见
- 公共 API、字段、返回值优先显式类型
- 少用 `dynamic`，它会把很多错误拖到运行时

### `final` 和 `const`
这是 Dart 面试高频题。

- `final`：运行时赋值一次，之后不可变
- `const`：编译期常量，值在编译阶段确定

```dart
final now = DateTime.now();
const secondsPerMinute = 60;
```

理解重点：

- `final` 对象引用不可变，不代表对象内部状态不可变
- `const` 更强，常用于常量对象和编译期固定值
- Flutter 里正确使用 `const` 可以减少重复创建 Widget

## 基本类型与常用集合
Dart 常见核心类型：

- `int`
- `double`
- `num`
- `bool`
- `String`
- `List`
- `Set`
- `Map`

```dart
final numbers = <int>[1, 2, 3];
final tags = <String>{'flutter', 'dart'};
final user = <String, dynamic>{
  'id': 1,
  'name': 'sun',
};
```

工程上要注意：

- `List` 有序可重复
- `Set` 无序且唯一，适合去重
- `Map` 是键值结构，不要滥用 `Map<String, dynamic>` 代替实体类
- 在 Flutter 项目里，能建模成类就不要长期透传裸 `Map`

## 函数
Dart 的函数是一等公民，可以作为参数、返回值和变量。

### 普通函数与箭头函数
```dart
String greet(String name) {
  return 'Hello, $name';
}

int add(int a, int b) => a + b;
```

### 位置参数与命名参数
```dart
void logMessage(String message, {String level = 'info', bool report = false}) {
  print('[$level] $message');
}
```

命名参数在 Flutter 里非常常见，因为 Widget 构造函数大量依赖命名参数来提升可读性。

### 可空与必填
```dart
void createUser({required String name, int? age}) {
  print('$name - $age');
}
```

工程建议：

- 对外 API 优先命名参数
- 核心参数用 `required`
- 避免用布尔参数堆叠复杂语义

## 控制流
Dart 有常规的 `if`、`for`、`while`、`switch`，也支持更适合声明式写法的集合内控制流。

```dart
final showVip = true;
final actions = [
  'profile',
  if (showVip) 'vip',
  for (final tab in ['feed', 'message']) tab,
];
```

这在 Flutter UI 构建里非常有用，因为 Widget 列表经常需要按条件和循环动态生成。

## 面向对象
Dart 是单继承，但支持接口实现、抽象类、mixin、扩展方法等能力。

### 类与构造函数
```dart
class User {
  final int id;
  final String name;

  const User({required this.id, required this.name});
}
```

### 抽象类
```dart
abstract class Logger {
  void log(String message);
}
```

### 接口实现
Dart 里每个类都隐式定义接口。

```dart
class ConsoleLogger implements Logger {
  @override
  void log(String message) {
    print(message);
  }
}
```

### mixin
适合复用行为，不适合承载混乱业务。

```dart
mixin Cacheable {
  void clearCache() {
    print('cache cleared');
  }
}

class UserRepository with Cacheable {}
```

### extension
适合给已有类型补充工具能力。

```dart
extension StringX on String {
  bool get isBlank => trim().isEmpty;
}
```

Flutter 实战里，extension 经常用于：

- `BuildContext` 辅助能力
- 字符串、时间、数值格式化
- 领域模型工具方法

## 空安全
空安全是 Dart 最重要的语言特性之一，也是面试高频题。

### 可空与不可空
```dart
String name = 'dart';
String? nickname;
```

- `String` 默认不能为 `null`
- `String?` 允许为 `null`

### 常见操作符
```dart
String? input;
final text = input ?? 'default';
final length = input?.length;
final sure = input!;
```

含义：

- `??`：左边为空就用右边默认值
- `?.`：安全访问
- `!`：非空断言，表示“我确定这里不为空”

### 类型提升
```dart
void printLength(String? value) {
  if (value != null) {
    print(value.length);
  }
}
```

`if (value != null)` 后，Dart 会把 `value` 提升为非空类型。

### 工程建议

- 少用 `!`，因为它只是把风险延后到运行时
- 进入边界层时尽早做判空和数据校验
- 把“可能为空”建模清楚，不要靠调用方猜

## 泛型
泛型的意义不是“写法高级”，而是复用同时保持类型安全。

### 泛型函数
```dart
T firstItem<T>(List<T> items) {
  if (items.isEmpty) {
    throw StateError('items is empty');
  }
  return items.first;
}
```

### 泛型类
```dart
class ApiResponse<T> {
  final T data;
  final String message;

  const ApiResponse({required this.data, required this.message});
}
```

### 泛型约束
```dart
abstract class Entity {
  int get id;
}

class Repository<T extends Entity> {
  T save(T entity) {
    return entity;
  }
}
```

Flutter 项目里，泛型常见于：

- 网络响应封装
- repository 抽象
- 状态管理基类
- 通用组件与列表模型

## 异步基础
`dart_advanced.md` 已经专门讲了 event loop、microtask、isolate，这里重点保留语言层的基础理解。

### Future
`Future` 表示未来某个时间点会拿到一个结果。

```dart
Future<String> fetchProfile() async {
  await Future.delayed(const Duration(milliseconds: 300));
  return 'profile';
}
```

常见使用方式：

```dart
final profile = await fetchProfile();
```

### `async` / `await`
它本质上是异步代码的语法糖，让异步逻辑更接近同步写法。

工程建议：

- 业务代码优先 `async` / `await`
- 需要链式组合时再考虑 `then`
- 异常处理优先 `try/catch`

### Stream
`Stream` 适合一段时间内持续产生多个值的场景。

```dart
Stream<int> counter() async* {
  for (var i = 1; i <= 3; i++) {
    yield i;
  }
}
```

典型场景：

- 输入监听
- WebSocket
- 下载进度
- 原生事件流
- Bloc / Rx 风格状态流

### 异步错误处理
```dart
Future<void> load() async {
  try {
    final data = await fetchProfile();
    print(data);
  } catch (e, stackTrace) {
    print('error: $e');
    print(stackTrace);
  }
}
```

原则：

- 不要吞异常
- 业务错误和系统错误分层处理
- 需要日志时保留 `stackTrace`

## 运算符与级联
Dart 有几个 Flutter 开发里很常见的语法点。

### `??=`
```dart
String? message;
message ??= 'hello';
```

### 级联运算符 `..`
```dart
final buffer = StringBuffer()
  ..write('flutter ')
  ..write('review');
```

它适合连续配置同一个对象。Flutter 里构建复杂对象时偶尔会用到，但不要为了“炫技”降低可读性。

## 常见语言误区

### 1. 误以为 `final` 就是深度不可变
不是。`final` 只是引用不可重新赋值。

```dart
final list = [1, 2];
list.add(3);
```

上面是合法的，因为变的是对象内部状态，不是引用本身。

### 2. 滥用 `dynamic`
`dynamic` 会让静态检查失效，很多错误只能运行时发现。业务代码里应尽量把 `dynamic` 限制在边界层，例如 JSON 解析入口。

### 3. 过度依赖 `Map<String, dynamic>`
短期方便，长期可维护性很差。尤其是复杂业务对象，应该尽快转换成实体类。

### 4. 空安全下到处写 `!`
这说明模型设计有问题，或者边界校验没做好。`!` 应该是少量、可证明安全的点状使用。

### 5. Stream 不取消订阅
这会导致页面销毁后仍然接收数据，进而造成内存泄漏或状态异常。

## Flutter 场景里的 Dart 最佳实践

1. 优先不可变建模
2. 公共 API 尽量强类型化
3. 页面层少碰 `dynamic`
4. 对异步错误做显式处理
5. 命名参数优先于位置参数
6. 用实体类承接 JSON，而不是页面直接处理裸数据
7. 把 extension 用在提升可读性，而不是制造魔法

## 面试高频题

### 1. `final` 和 `const` 的区别是什么
标准回答：

- `final` 是运行时赋值一次
- `const` 是编译期常量
- `const` 更适合常量值和常量对象
- 在 Flutter 里合理使用 `const` 还能减少重复创建 Widget

### 2. Dart 的空安全解决了什么问题
标准回答：

- 它通过类型系统把“是否允许为空”前置到编译期
- 默认类型不可空，可空类型要显式写成 `Type?`
- 配合 `??`、`?.`、类型提升等机制，减少空指针异常
- 但如果滥用 `!`，仍然会把风险放回运行时

### 3. `Future` 和 `Stream` 的区别是什么
标准回答：

- `Future` 表示单次异步结果
- `Stream` 表示一段时间内持续产生多个结果
- 请求详情页通常更像 `Future`
- WebSocket、下载进度、事件流通常更像 `Stream`

### 4. 为什么 Flutter 项目里要少用 `dynamic`
标准回答：

- 因为 `dynamic` 会绕过大部分静态类型检查
- 会让错误在运行时才暴露
- 会削弱重构能力和 IDE 提示能力
- 应该把它限制在 JSON 边界、插件边界等少数位置

### 5. mixin 和继承的区别是什么
标准回答：

- 继承强调 is-a 关系，表达类型层级
- mixin 强调行为复用，不建立严格层级关系
- Flutter 里 mixin 常用于生命周期辅助、缓存能力、日志能力等横切行为

## 面试答题模板

### 题目：Dart 语言有哪些特性对 Flutter 很重要
可以按这个结构回答：

1. 强类型和空安全，保证工程稳定性
2. `final` / `const` 和不可变建模，影响 Widget 构建与状态管理
3. 命名参数和声明式风格，很适合 Flutter UI 组织
4. `Future` / `Stream` 提供异步能力
5. 泛型、mixin、extension 提供复用和抽象能力

### 题目：你在 Flutter 项目里怎么体现 Dart 基本功
可以按这个结构回答：

1. 页面层尽量强类型，不透传 `dynamic`
2. JSON 尽快映射为实体类
3. 状态模型尽量不可变
4. 异步统一处理错误和取消逻辑
5. 公共能力通过泛型、抽象类、extension 做清晰封装

## 复习重点

- 说清 `final`、`const`、`dynamic` 的边界
- 理解空安全不是语法糖，而是类型系统约束
- 知道 `Future` 和 `Stream` 的使用场景
- 能解释泛型、mixin、extension 在 Flutter 工程里的实际价值
- 能把 Dart 语言特性和 Flutter 开发场景联系起来
