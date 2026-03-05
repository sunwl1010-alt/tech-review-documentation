# 本地存储

## SharedPreferences

### 基本使用

```dart
// 添加依赖
// dependencies:
//   shared_preferences: ^2.0.15

import 'package:shared_preferences/shared_preferences.dart';

// 存储数据
Future<void> saveData() async {
  final prefs = await SharedPreferences.getInstance();
  
  // 存储字符串
  await prefs.setString('username', 'flutter');
  
  // 存储整数
  await prefs.setInt('age', 30);
  
  // 存储布尔值
  await prefs.setBool('isLoggedIn', true);
  
  // 存储双精度浮点数
  await prefs.setDouble('score', 95.5);
  
  // 存储字符串列表
  await prefs.setStringList('languages', ['Dart', 'Flutter', 'Java']);
}

// 读取数据
Future<void> loadData() async {
  final prefs = await SharedPreferences.getInstance();
  
  // 读取字符串
  final username = prefs.getString('username') ?? 'Guest';
  
  // 读取整数
  final age = prefs.getInt('age') ?? 0;
  
  // 读取布尔值
  final isLoggedIn = prefs.getBool('isLoggedIn') ?? false;
  
  // 读取双精度浮点数
  final score = prefs.getDouble('score') ?? 0.0;
  
  // 读取字符串列表
  final languages = prefs.getStringList('languages') ?? [];
  
  print('Username: $username');
  print('Age: $age');
  print('Is Logged In: $isLoggedIn');
  print('Score: $score');
  print('Languages: $languages');
}

// 删除数据
Future<void> removeData() async {
  final prefs = await SharedPreferences.getInstance();
  
  // 删除单个键
  await prefs.remove('username');
  
  // 清除所有数据
  await prefs.clear();
}

// 检查键是否存在
Future<void> checkKey() async {
  final prefs = await SharedPreferences.getInstance();
  final exists = prefs.containsKey('username');
  print('Username exists: $exists');
}
```

### 最佳实践
- **键名管理**：使用常量定义键名，避免硬编码
- **数据类型**：确保存储和读取的数据类型一致
- **错误处理**：添加 try-catch 处理可能的异常
- **数据大小**：避免存储过大的数据，SharedPreferences 适合存储小数据

## SQLite

### 基本使用

```dart
// 添加依赖
// dependencies:
//   sqflite: ^2.0.3
//   path: ^1.8.2

import 'dart:async';
import 'package:sqflite/sqflite.dart';
import 'package:path/path.dart';

class DatabaseHelper {
  static Database? _database;
  
  Future<Database> get database async {
    if (_database != null) return _database!;
    
    _database = await initDatabase();
    return _database!;
  }
  
  Future<Database> initDatabase() async {
    final dbPath = await getDatabasesPath();
    final path = join(dbPath, 'example.db');
    
    return await openDatabase(
      path,
      version: 1,
      onCreate: (db, version) {
        return db.execute(
          'CREATE TABLE users(id INTEGER PRIMARY KEY AUTOINCREMENT, name TEXT, email TEXT)',
        );
      },
    );
  }
  
  // 插入数据
  Future<void> insertUser(Map<String, dynamic> user) async {
    final db = await database;
    await db.insert('users', user);
  }
  
  // 查询所有数据
  Future<List<Map<String, dynamic>>> getUsers() async {
    final db = await database;
    return await db.query('users');
  }
  
  // 根据 ID 查询
  Future<Map<String, dynamic>?> getUser(int id) async {
    final db = await database;
    final results = await db.query('users', where: 'id = ?', whereArgs: [id]);
    if (results.isNotEmpty) {
      return results.first;
    }
    return null;
  }
  
  // 更新数据
  Future<void> updateUser(int id, Map<String, dynamic> user) async {
    final db = await database;
    await db.update('users', user, where: 'id = ?', whereArgs: [id]);
  }
  
  // 删除数据
  Future<void> deleteUser(int id) async {
    final db = await database;
    await db.delete('users', where: 'id = ?', whereArgs: [id]);
  }
  
  // 关闭数据库
  Future<void> close() async {
    final db = await database;
    db.close();
  }
}

// 使用
final dbHelper = DatabaseHelper();

// 插入用户
await dbHelper.insertUser({'name': 'John', 'email': 'john@example.com'});

// 获取所有用户
final users = await dbHelper.getUsers();
print(users);

// 更新用户
await dbHelper.updateUser(1, {'name': 'John Doe', 'email': 'john.doe@example.com'});

// 删除用户
await dbHelper.deleteUser(1);

// 关闭数据库
await dbHelper.close();
```

### 最佳实践
- **数据库管理**：使用单例模式管理数据库实例
- **事务**：使用事务确保数据一致性
- **错误处理**：添加 try-catch 处理数据库操作异常
- **版本管理**：正确处理数据库版本升级

## Hive

### 基本使用

```dart
// 添加依赖
// dependencies:
//   hive: ^2.2.3
//   hive_flutter: ^1.1.0

import 'package:hive/hive.dart';
import 'package:hive_flutter/hive_flutter.dart';

// 初始化 Hive
Future<void> initHive() async {
  await Hive.initFlutter();
  
  // 注册适配器（如果需要）
  // Hive.registerAdapter(UserAdapter());
  
  // 打开盒子
  await Hive.openBox('users');
  await Hive.openBox('settings');
}

// 存储数据
void saveData() {
  final usersBox = Hive.box('users');
  final settingsBox = Hive.box('settings');
  
  // 存储数据
  usersBox.put('user1', {'name': 'John', 'age': 30});
  settingsBox.put('theme', 'dark');
  settingsBox.put('notifications', true);
}

// 读取数据
void loadData() {
  final usersBox = Hive.box('users');
  final settingsBox = Hive.box('settings');
  
  // 读取数据
  final user = usersBox.get('user1');
  final theme = settingsBox.get('theme', defaultValue: 'light');
  final notifications = settingsBox.get('notifications', defaultValue: false);
  
  print('User: $user');
  print('Theme: $theme');
  print('Notifications: $notifications');
}

// 删除数据
void removeData() {
  final usersBox = Hive.box('users');
  final settingsBox = Hive.box('settings');
  
  // 删除单个键
  usersBox.delete('user1');
  
  // 清除所有数据
  settingsBox.clear();
}

// 使用自定义对象
@HiveType(typeId: 0)
class User {
  @HiveField(0)
  final String name;
  
  @HiveField(1)
  final int age;
  
  User(this.name, this.age);
}

// 注册适配器
class UserAdapter extends TypeAdapter<User> {
  @override
  final int typeId = 0;
  
  @override
  User read(BinaryReader reader) {
    return User(reader.readString(), reader.readInt());
  }
  
  @override
  void write(BinaryWriter writer, User obj) {
    writer.writeString(obj.name);
    writer.writeInt(obj.age);
  }
}

// 使用自定义对象
void useCustomObject() async {
  // 注册适配器
  Hive.registerAdapter(UserAdapter());
  
  // 打开盒子
  final box = await Hive.openBox<User>('userBox');
  
  // 存储对象
  final user = User('John', 30);
  await box.put('user1', user);
  
  // 读取对象
  final storedUser = box.get('user1');
  print('User: ${storedUser?.name}, ${storedUser?.age}');
}
```

### 最佳实践
- **初始化**：在应用启动时初始化 Hive
- **适配器**：为自定义对象创建适配器
- **盒子管理**：根据数据类型使用不同的盒子
- **错误处理**：添加 try-catch 处理可能的异常

## Isar

### 基本使用

```dart
// 添加依赖
// dependencies:
//   isar: ^3.0.5
//   isar_flutter_libs: ^3.0.5

import 'package:isar/isar.dart';
import 'package:path_provider/path_provider.dart';

// 定义模型
@collection
class User {
  Id id = Isar.autoIncrement;
  
  String? name;
  int? age;
  String? email;
}

// 初始化 Isar
Future<Isar> initIsar() async {
  final dir = await getApplicationDocumentsDirectory();
  
  return await Isar.open(
    [UserSchema],
    directory: dir.path,
  );
}

// 增删改查
Future<void> isarOperations() async {
  final isar = await initIsar();
  
  // 插入数据
  final user = User()
    ..name = 'John'
    ..age = 30
    ..email = 'john@example.com';
  
  await isar.writeTxn(() async {
    await isar.users.put(user);
  });
  
  // 查询数据
  final allUsers = await isar.users.where().findAll();
  print(allUsers);
  
  // 根据 ID 查询
  final foundUser = await isar.users.get(user.id);
  print(foundUser);
  
  // 更新数据
  await isar.writeTxn(() async {
    foundUser?.name = 'John Doe';
    await isar.users.put(foundUser!);
  });
  
  // 删除数据
  await isar.writeTxn(() async {
    await isar.users.delete(user.id);
  });
  
  // 关闭 Isar
  await isar.close();
}

// 高级查询
Future<void> advancedQueries() async {
  final isar = await initIsar();
  
  // 条件查询
  final users = await isar.users
    .filter()
    .ageGreaterThan(25)
    .nameStartsWith('J')
    .findAll();
  
  print(users);
  
  // 排序
  final sortedUsers = await isar.users
    .where()
    .sortByAgeDesc()
    .findAll();
  
  print(sortedUsers);
  
  await isar.close();
}
```

### 最佳实践
- **模型定义**：使用 `@collection` 注解定义模型
- **事务**：使用 `writeTxn` 确保数据一致性
- **索引**：为常用查询字段添加索引
- **错误处理**：添加 try-catch 处理可能的异常

## 最佳实践

1. **选择合适的存储方案**：
   - 小数据：SharedPreferences
   - 结构化数据：SQLite、Hive、Isar
   - 大型数据：文件存储

2. **数据加密**：
   - 对敏感数据进行加密
   - 使用 secure_storage 库存储敏感信息

3. **性能优化**：
   - 批量操作
   - 合理使用事务
   - 避免频繁读写

4. **错误处理**：
   - 捕获并处理存储操作异常
   - 提供降级方案

5. **数据迁移**：
   - 处理应用版本升级时的数据迁移
   - 备份重要数据

## 常见问题

1. **数据丢失**：
   - 未正确处理事务
   - 应用崩溃导致数据未保存

2. **性能问题**：
   - 频繁读写操作
   - 未使用索引
   - 处理大量数据时的内存占用

3. **兼容性问题**：
   - 不同平台的存储路径差异
   - 不同 Flutter 版本的 API 变化

4. **安全问题**：
   - 存储敏感信息未加密
   - 权限配置不当

5. **数据一致性**：
   - 并发操作导致数据不一致
   - 未使用事务

## 面试题

1. **Q**: Flutter 中有哪些本地存储方案？
   **A**: SharedPreferences、SQLite、Hive、Isar、文件存储

2. **Q**: SharedPreferences 和 SQLite 的区别是什么？
   **A**: SharedPreferences 适合存储小数据，如设置、用户偏好等；SQLite 适合存储结构化的复杂数据

3. **Q**: Hive 和 SQLite 相比有什么优势？
   **A**: Hive 是 NoSQL 数据库，读写速度快，适合存储复杂对象，不需要 SQL 语句

4. **Q**: 如何处理本地存储的异常？
   **A**: 使用 try-catch 捕获异常，提供降级方案，确保应用正常运行

5. **Q**: 如何保证本地存储的数据安全？
   **A**: 对敏感数据进行加密，使用 secure_storage 库，合理设置文件权限