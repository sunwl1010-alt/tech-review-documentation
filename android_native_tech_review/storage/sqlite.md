# SQLite存储

## 核心概念

### SQLite
- **定义**：轻量级的关系型数据库，内置在Android系统中
- **作用**：存储结构化数据，支持复杂查询
- **特点**：
  - 轻量级，无需独立服务器
  - 支持SQL语句
  - 事务支持
  - 跨平台

### SQLiteOpenHelper
- **定义**：Android提供的SQLite数据库辅助类
- **作用**：管理数据库的创建和版本升级
- **方法**：
  - onCreate()：创建数据库时调用
  - onUpgrade()：数据库版本升级时调用
  - getReadableDatabase()：获取只读数据库
  - getWritableDatabase()：获取可写数据库

### 数据库操作
- **CRUD**：创建（Create）、读取（Read）、更新（Update）、删除（Delete）
- **SQL语句**：使用标准SQL语句执行操作
- **Cursor**：结果集，用于遍历查询结果

### 事务
- **定义**：一组原子性的数据库操作
- **作用**：确保数据一致性
- **方法**：beginTransaction()、setTransactionSuccessful()、endTransaction()

## 实现原理

### 数据库创建
- **SQLiteOpenHelper**：通过构造函数指定数据库名称和版本
- **onCreate()**：执行CREATE TABLE语句创建表
- **数据库文件**：存储在/data/data/包名/databases/目录下

### 数据库升级
- **版本号**：通过构造函数指定版本号
- **onUpgrade()**：当版本号增加时调用，执行升级操作
- **升级策略**：根据需要修改表结构，迁移数据

### 数据操作
- **插入**：使用INSERT语句或ContentValues
- **查询**：使用SELECT语句，返回Cursor
- **更新**：使用UPDATE语句或ContentValues
- **删除**：使用DELETE语句

### 事务处理
- **开始事务**：调用beginTransaction()
- **执行操作**：执行数据库操作
- **标记成功**：调用setTransactionSuccessful()
- **结束事务**：调用endTransaction()，如果标记成功则提交，否则回滚

## 使用场景

### 结构化数据
- **用户信息**：存储用户的详细信息
- **应用设置**：存储应用的复杂设置
- **业务数据**：存储应用的业务数据
- **缓存数据**：缓存网络数据

### 复杂查询
- **多表关联**：需要关联查询多个表的数据
- **排序筛选**：需要排序和筛选数据
- **统计分析**：需要统计和分析数据

### 离线数据
- **离线模式**：应用在离线状态下仍能使用
- **数据同步**：与服务器数据同步
- **本地缓存**：缓存服务器数据，减少网络请求

### 大量数据
- **历史记录**：存储用户的历史记录
- **日志数据**：存储应用的日志数据
- **媒体库**：存储媒体文件的元数据

## 常见问题及解决方案

### 性能问题
- **问题**：查询速度慢
  **解决方案**：创建索引，优化SQL语句，使用事务

- **问题**：内存占用高
  **解决方案**：及时关闭Cursor，使用分页查询

- **问题**：并发访问冲突
  **解决方案**：使用事务，合理使用锁

### 数据库升级问题
- **问题**：升级过程中数据丢失
  **解决方案**：在onUpgrade()中备份和恢复数据

- **问题**：版本号管理混乱
  **解决方案**：建立版本管理策略，记录每个版本的变更

- **问题**：升级失败导致应用崩溃
  **解决方案**：使用事务确保升级过程的原子性

### 线程安全问题
- **问题**：多线程并发访问数据库
  **解决方案**：使用单例模式，同步访问，或使用Room的线程安全机制

- **问题**：Cursor未关闭导致内存泄漏
  **解决方案**：使用try-finally确保Cursor关闭

### 数据一致性问题
- **问题**：事务未正确处理
  **解决方案**：正确使用beginTransaction()、setTransactionSuccessful()、endTransaction()

- **问题**：数据写入失败
  **解决方案**：检查返回值，处理异常

## 代码示例

### SQLiteOpenHelper实现
```java
public class DBHelper extends SQLiteOpenHelper {
    private static final String DATABASE_NAME = "mydb.db";
    private static final int DATABASE_VERSION = 1;
    
    // 表名
    public static final String TABLE_USERS = "users";
    
    // 列名
    public static final String COLUMN_ID = "_id";
    public static final String COLUMN_NAME = "name";
    public static final String COLUMN_EMAIL = "email";
    public static final String COLUMN_AGE = "age";
    
    // 创建表语句
    private static final String CREATE_TABLE_USERS = "CREATE TABLE " + TABLE_USERS + " (" +
            COLUMN_ID + " INTEGER PRIMARY KEY AUTOINCREMENT, " +
            COLUMN_NAME + " TEXT NOT NULL, " +
            COLUMN_EMAIL + " TEXT UNIQUE, " +
            COLUMN_AGE + " INTEGER);";
    
    public DBHelper(Context context) {
        super(context, DATABASE_NAME, null, DATABASE_VERSION);
    }
    
    @Override
    public void onCreate(SQLiteDatabase db) {
        db.execSQL(CREATE_TABLE_USERS);
    }
    
    @Override
    public void onUpgrade(SQLiteDatabase db, int oldVersion, int newVersion) {
        // 升级逻辑
        if (oldVersion < 2) {
            // 版本2的升级操作
        }
    }
}
```

### 基本CRUD操作
```java
public class UserDAO {
    private DBHelper dbHelper;
    
    public UserDAO(Context context) {
        dbHelper = new DBHelper(context);
    }
    
    // 插入用户
    public long insertUser(User user) {
        SQLiteDatabase db = dbHelper.getWritableDatabase();
        ContentValues values = new ContentValues();
        values.put(DBHelper.COLUMN_NAME, user.getName());
        values.put(DBHelper.COLUMN_EMAIL, user.getEmail());
        values.put(DBHelper.COLUMN_AGE, user.getAge());
        long id = db.insert(DBHelper.TABLE_USERS, null, values);
        db.close();
        return id;
    }
    
    // 查询所有用户
    public List<User> getAllUsers() {
        List<User> users = new ArrayList<>();
        SQLiteDatabase db = dbHelper.getReadableDatabase();
        Cursor cursor = db.query(DBHelper.TABLE_USERS, null, null, null, null, null, DBHelper.COLUMN_NAME + " ASC");
        
        if (cursor != null) {
            while (cursor.moveToNext()) {
                int id = cursor.getInt(cursor.getColumnIndex(DBHelper.COLUMN_ID));
                String name = cursor.getString(cursor.getColumnIndex(DBHelper.COLUMN_NAME));
                String email = cursor.getString(cursor.getColumnIndex(DBHelper.COLUMN_EMAIL));
                int age = cursor.getInt(cursor.getColumnIndex(DBHelper.COLUMN_AGE));
                users.add(new User(id, name, email, age));
            }
            cursor.close();
        }
        db.close();
        return users;
    }
    
    // 根据ID查询用户
    public User getUserById(int id) {
        SQLiteDatabase db = dbHelper.getReadableDatabase();
        Cursor cursor = db.query(DBHelper.TABLE_USERS, null, DBHelper.COLUMN_ID + "=?", new String[]{String.valueOf(id)}, null, null, null);
        User user = null;
        
        if (cursor != null && cursor.moveToFirst()) {
            String name = cursor.getString(cursor.getColumnIndex(DBHelper.COLUMN_NAME));
            String email = cursor.getString(cursor.getColumnIndex(DBHelper.COLUMN_EMAIL));
            int age = cursor.getInt(cursor.getColumnIndex(DBHelper.COLUMN_AGE));
            user = new User(id, name, email, age);
            cursor.close();
        }
        db.close();
        return user;
    }
    
    // 更新用户
    public int updateUser(User user) {
        SQLiteDatabase db = dbHelper.getWritableDatabase();
        ContentValues values = new ContentValues();
        values.put(DBHelper.COLUMN_NAME, user.getName());
        values.put(DBHelper.COLUMN_EMAIL, user.getEmail());
        values.put(DBHelper.COLUMN_AGE, user.getAge());
        int rows = db.update(DBHelper.TABLE_USERS, values, DBHelper.COLUMN_ID + "=?", new String[]{String.valueOf(user.getId())});
        db.close();
        return rows;
    }
    
    // 删除用户
    public int deleteUser(int id) {
        SQLiteDatabase db = dbHelper.getWritableDatabase();
        int rows = db.delete(DBHelper.TABLE_USERS, DBHelper.COLUMN_ID + "=?", new String[]{String.valueOf(id)});
        db.close();
        return rows;
    }
}

// User类
class User {
    private int id;
    private String name;
    private String email;
    private int age;
    
    // 构造方法、getter和setter
    public User(int id, String name, String email, int age) {
        this.id = id;
        this.name = name;
        this.email = email;
        this.age = age;
    }
    
    public int getId() { return id; }
    public void setId(int id) { this.id = id; }
    public String getName() { return name; }
    public void setName(String name) { this.name = name; }
    public String getEmail() { return email; }
    public void setEmail(String email) { this.email = email; }
    public int getAge() { return age; }
    public void setAge(int age) { this.age = age; }
}
```

### 事务处理
```java
public void batchInsertUsers(List<User> users) {
    SQLiteDatabase db = dbHelper.getWritableDatabase();
    db.beginTransaction();
    try {
        for (User user : users) {
            ContentValues values = new ContentValues();
            values.put(DBHelper.COLUMN_NAME, user.getName());
            values.put(DBHelper.COLUMN_EMAIL, user.getEmail());
            values.put(DBHelper.COLUMN_AGE, user.getAge());
            db.insert(DBHelper.TABLE_USERS, null, values);
        }
        db.setTransactionSuccessful();
    } finally {
        db.endTransaction();
        db.close();
    }
}
```

### 高级查询
```java
// 按年龄查询用户
public List<User> getUsersByAge(int minAge, int maxAge) {
    List<User> users = new ArrayList<>();
    SQLiteDatabase db = dbHelper.getReadableDatabase();
    String selection = DBHelper.COLUMN_AGE + " BETWEEN ? AND ?";
    String[] selectionArgs = {String.valueOf(minAge), String.valueOf(maxAge)};
    Cursor cursor = db.query(DBHelper.TABLE_USERS, null, selection, selectionArgs, null, null, DBHelper.COLUMN_AGE + " ASC");
    
    if (cursor != null) {
        while (cursor.moveToNext()) {
            int id = cursor.getInt(cursor.getColumnIndex(DBHelper.COLUMN_ID));
            String name = cursor.getString(cursor.getColumnIndex(DBHelper.COLUMN_NAME));
            String email = cursor.getString(cursor.getColumnIndex(DBHelper.COLUMN_EMAIL));
            int age = cursor.getInt(cursor.getColumnIndex(DBHelper.COLUMN_AGE));
            users.add(new User(id, name, email, age));
        }
        cursor.close();
    }
    db.close();
    return users;
}

// 使用原始SQL查询
public List<User> getUsersWithNamePrefix(String prefix) {
    List<User> users = new ArrayList<>();
    SQLiteDatabase db = dbHelper.getReadableDatabase();
    String sql = "SELECT * FROM " + DBHelper.TABLE_USERS + " WHERE " + DBHelper.COLUMN_NAME + " LIKE ? ORDER BY " + DBHelper.COLUMN_NAME;
    Cursor cursor = db.rawQuery(sql, new String[]{prefix + "%"});
    
    if (cursor != null) {
        while (cursor.moveToNext()) {
            int id = cursor.getInt(cursor.getColumnIndex(DBHelper.COLUMN_ID));
            String name = cursor.getString(cursor.getColumnIndex(DBHelper.COLUMN_NAME));
            String email = cursor.getString(cursor.getColumnIndex(DBHelper.COLUMN_EMAIL));
            int age = cursor.getInt(cursor.getColumnIndex(DBHelper.COLUMN_AGE));
            users.add(new User(id, name, email, age));
        }
        cursor.close();
    }
    db.close();
    return users;
}
```

### 数据库升级
```java
@Override
public void onUpgrade(SQLiteDatabase db, int oldVersion, int newVersion) {
    switch (oldVersion) {
        case 1:
            // 版本1到版本2的升级
            db.execSQL("ALTER TABLE " + TABLE_USERS + " ADD COLUMN " + COLUMN_PHONE + " TEXT");
            // 继续处理其他版本升级
        case 2:
            // 版本2到版本3的升级
            db.execSQL("CREATE TABLE IF NOT EXISTS " + TABLE_ADDRESSES + " (" +
                    "_id INTEGER PRIMARY KEY AUTOINCREMENT, " +
                    "user_id INTEGER, " +
                    "address TEXT, " +
                    "FOREIGN KEY (user_id) REFERENCES " + TABLE_USERS + "(_id)" +
                    ");");
            // 继续处理其他版本升级
    }
}
```

## 面试常考问题及参考答案

### 基础理论

**1. SQLite的特点是什么？**

**答案**：
- 轻量级，无需独立服务器
- 支持标准SQL语句
- 事务支持
- 跨平台
- 内置在Android系统中
- 适合存储结构化数据

**2. SQLiteOpenHelper的作用是什么？**

**答案**：SQLiteOpenHelper是Android提供的SQLite数据库辅助类，用于管理数据库的创建和版本升级。它提供了onCreate()方法在数据库首次创建时调用，onUpgrade()方法在数据库版本升级时调用，以及getReadableDatabase()和getWritableDatabase()方法获取数据库实例。

**3. 如何创建和升级SQLite数据库？**

**答案**：
- **创建数据库**：继承SQLiteOpenHelper，在onCreate()方法中执行CREATE TABLE语句
- **升级数据库**：在SQLiteOpenHelper的构造函数中增加版本号，在onUpgrade()方法中执行升级操作，如ALTER TABLE或CREATE TABLE

### 实际应用

**4. 如何执行数据库的CRUD操作？**

**答案**：
- **插入**：使用insert()方法或execSQL()执行INSERT语句
- **查询**：使用query()方法或rawQuery()执行SELECT语句，返回Cursor
- **更新**：使用update()方法或execSQL()执行UPDATE语句
- **删除**：使用delete()方法或execSQL()执行DELETE语句

**5. 如何处理数据库事务？**

**答案**：
- 调用beginTransaction()开始事务
- 执行数据库操作
- 调用setTransactionSuccessful()标记事务成功
- 调用endTransaction()结束事务， 如果标记成功则提交，否则回滚

**6. 如何优化SQLite的性能？**

**答案**：
- 创建索引加速查询
- 使用事务减少I/O操作
- 优化SQL语句
- 合理设计表结构
- 及时关闭Cursor
- 使用分页查询处理大量数据

### 性能优化

**7. 如何处理大量数据的查询？**

**答案**：
- 使用分页查询，限制返回数据量
- 创建合适的索引
- 优化SQL语句
- 使用事务处理批量操作
- 考虑使用ContentProvider

**8. 如何避免Cursor内存泄漏？**

**答案**：
- 使用try-finally确保Cursor关闭
- 不要在Cursor未关闭的情况下离开方法
- 在Activity的onDestroy()中关闭Cursor
- 使用CursorLoader自动管理Cursor生命周期

### 安全问题

**9. 如何防止SQL注入攻击？**

**答案**：
- 使用参数化查询，避免直接拼接SQL语句
- 使用ContentValues或SQLiteDatabase的insert()、update()、delete()方法
- 对用户输入进行验证和过滤

**10. 如何加密SQLite数据库？**

**答案**：
- 使用SQLCipher库加密数据库
- 在创建数据库时设置密码
- 注意：加密会影响性能

### 架构设计

**11. 如何设计一个可扩展的SQLite数据库架构？**

**答案**：
- 采用DAO模式封装数据库操作
- 使用SQLiteOpenHelper管理数据库版本
- 合理设计表结构和关系
- 实现数据库升级策略
- 提供异步操作接口
- 考虑使用Room替代原生SQLite

**12. SQLite与其他存储方式的对比？**

**答案**：
- **SQLite**：适合存储结构化数据，支持复杂查询，但操作复杂
- **SharedPreferences**：适合存储轻量级键值对，简单易用但不适合大量数据
- **文件存储**：适合存储大文件、二进制数据，灵活但查询效率低
- **Room**：SQLite的封装，提供更友好的API，支持LiveData和RxJava

根据数据类型和使用场景选择合适的存储方式。对于结构化数据和复杂查询，SQLite是最佳选择。