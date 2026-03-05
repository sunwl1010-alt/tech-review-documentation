# Dart 语言

## 语法基础

### 变量和类型

```dart
// 变量声明
var name = 'Flutter'; // 类型推断
String message = 'Hello'; // 显式类型
final pi = 3.14; // 不可变变量
const maxSize = 100; // 编译时常量

// 基本类型
int age = 20;
double height = 1.75;
bool isActive = true;
String name = 'John';
```

### 函数

```dart
// 基本函数
String greet(String name) {
  return 'Hello, $name!';
}

// 箭头函数
int add(int a, int b) => a + b;

// 可选参数
void printInfo(String name, {int age, String address}) {
  print('Name: $name');
  if (age != null) print('Age: $age');
  if (address != null) print('Address: $address');
}

// 默认参数
void greet(String name, {String greeting = 'Hello'}) {
  print('$greeting, $name!');
}
```

### 控制流

```dart
// 条件语句
if (age >= 18) {
  print('Adult');
} else if (age >= 13) {
  print('Teenager');
} else {
  print('Child');
}

// 循环
for (int i = 0; i < 5; i++) {
  print(i);
}

// 遍历列表
var numbers = [1, 2, 3, 4, 5];
for (var number in numbers) {
  print(number);
}

// while 循环
int count = 0;
while (count < 5) {
  print(count);
  count++;
}
```

### 集合

```dart
// 列表
var fruits = ['apple', 'banana', 'orange'];
fruits.add('grape');
print(fruits[0]); // apple

// 集合
var uniqueFruits = {'apple', 'banana', 'orange'};
uniqueFruits.add('apple'); // 不会重复添加

// 映射
var person = {
  'name': 'John',
  'age': 30,
  'address': 'New York'
};
print(person['name']); // John
```

## 异步编程

### Future

```dart
// 创建 Future
Future<String> fetchData() {
  return Future.delayed(Duration(seconds: 2), () {
    return 'Data loaded';
  });
}

// 使用 then
fetchData().then((data) {
  print(data);
}).catchError((error) {
  print('Error: $error');
});

// 使用 async/await
Future<void> loadData() async {
  try {
    var data = await fetchData();
    print(data);
  } catch (error) {
    print('Error: $error');
  }
}
```

### Stream

```dart
// 创建 Stream
Stream<int> countStream(int max) async* {
  for (int i = 1; i <= max; i++) {
    await Future.delayed(Duration(seconds: 1));
    yield i;
  }
}

// 监听 Stream
countStream(5).listen(
  (value) => print('Value: $value'),
  onDone: () => print('Done'),
  onError: (error) => print('Error: $error'),
);

// 使用 async/await for
Future<void> processStream() async {
  await for (var value in countStream(3)) {
    print('Processed: $value');
  }
  print('Stream completed');
}
```

## 空安全

### 空安全操作符

```dart
// 可空类型
String? name; // 可以为 null
name = 'John';
name = null; // 允许

// 非空断言
String nonNullableName = name!; // 断言 name 不为 null

// 空值检查
String? nullableName;
String safeName = nullableName ?? 'Default'; // 如果为 null 使用默认值

// 条件访问
String? maybeName;
int? length = maybeName?.length; // 如果 maybeName 为 null，返回 null

// 级联条件访问
Person? person;
String? street = person?.address?.street;
```

### 类型提升

```dart
String? name;

if (name != null) {
  // 类型提升：name 在此作用域内被视为非空
  print(name.length);
}

// 使用 ??= 操作符
String? message;
message ??= 'Hello'; // 如果 message 为 null，赋值
```

## 泛型

### 泛型函数

```dart
T getFirst<T>(List<T> list) {
  if (list.isEmpty) throw Exception('List is empty');
  return list[0];
}

// 使用
var firstInt = getFirst([1, 2, 3]); // int
var firstString = getFirst(['a', 'b', 'c']); // String
```

### 泛型类

```dart
class Box<T> {
  T value;
  
  Box(this.value);
  
  T getValue() => value;
}

// 使用
var intBox = Box<int>(42);
var stringBox = Box<String>('Hello');
```

### 泛型约束

```dart
class Repository<T extends Entity> {
  void save(T entity) {
    // 保存实体
  }
}

abstract class Entity {
  int id;
  Entity(this.id);
}

class User extends Entity {
  String name;
  User(int id, this.name) : super(id);
}

// 使用
var userRepository = Repository<User>();
```

## 最佳实践

1. **使用类型推断**：优先使用 `var`，让 Dart 推断类型
2. **使用不可变变量**：优先使用 `final` 和 `const`
3. **空安全**：正确处理可空类型，避免空指针异常
4. **异步编程**：使用 `async/await` 使代码更清晰
5. **代码风格**：遵循 Dart 官方风格指南

## 常见问题

1. **空安全错误**：未正确处理可空类型
2. **异步错误处理**：未捕获 Future 或 Stream 中的错误
3. **内存泄漏**：未取消 Stream 订阅
4. **性能问题**：过度使用闭包或创建不必要的对象
5. **类型转换错误**：不正确的类型转换

## 面试题

1. **Q**: Dart 中的 `final` 和 `const` 有什么区别？
   **A**: `final` 变量只能赋值一次，在运行时初始化；`const` 变量是编译时常量，必须在编译时确定值

2. **Q**: Dart 如何处理异步操作？
   **A**: 使用 Future 处理一次性异步操作，使用 Stream 处理连续的异步事件，通过 async/await 使代码更清晰

3. **Q**: 什么是空安全？Dart 如何实现空安全？
   **A**: 空安全是一种类型系统特性，防止空指针异常。Dart 通过可空类型（`Type?`）、非空断言（`!`）、空值检查（`??`）等操作符实现

4. **Q**: 泛型在 Dart 中的作用是什么？
   **A**: 泛型允许编写可重用的代码，可以处理不同类型的数据，提高代码的类型安全性和可维护性

5. **Q**: Dart 中的 Mixin 是什么？如何使用？
   **A**: Mixin 是一种代码复用机制，允许一个类继承多个类的行为。使用 `mixin` 关键字定义，通过 `with` 关键字使用