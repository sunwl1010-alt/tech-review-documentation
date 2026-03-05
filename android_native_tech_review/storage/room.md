# Room 持久化库

## 概述

Room 是 Android Jetpack 中的一个持久化库，它提供了一个抽象层，在 SQLite 的基础上提供了更高级的访问接口，使数据库操作更加简单和安全。

### 核心组件

1. **Entity**：代表数据库中的表结构
2. **DAO**（Data Access Object）：定义访问数据库的方法
3. **Database**：数据库持有者，管理数据库连接

## 基本使用

### 添加依赖

```gradle
dependencies {
    def room_version = "2.4.3"
    
    implementation "androidx.room:room-runtime:$room_version"
    annotationProcessor "androidx.room:room-compiler:$room_version"
    
    // Kotlin 使用 kapt 替代 annotationProcessor
    // kapt "androidx.room:room-compiler:$room_version"
    
    // 可选：RxJava 支持
    // implementation "androidx.room:room-rxjava2:$room_version"
    
    // 可选：Coroutines 支持
    // implementation "androidx.room:room-ktx:$room_version"
}
```

### 定义 Entity

```kotlin
@Entity(tableName = "users")
data class User(
    @PrimaryKey val id: Int,
    @ColumnInfo(name = "first_name") val firstName: String?,
    @ColumnInfo(name = "last_name") val lastName: String?
)
```

### 定义 DAO

```kotlin
@Dao
interface UserDao {
    @Query("SELECT * FROM users")
    fun getAll(): List<User>
    
    @Query("SELECT * FROM users WHERE id IN (:userIds)")
    fun loadAllByIds(userIds: IntArray): List<User>
    
    @Query("SELECT * FROM users WHERE first_name LIKE :first AND last_name LIKE :last LIMIT 1")
    fun findByName(first: String, last: String): User
    
    @Insert
    fun insertAll(vararg users: User)
    
    @Delete
    fun delete(user: User)
    
    @Update
    fun update(user: User)
}
```

### 定义 Database

```kotlin
@Database(entities = [User::class], version = 1)
abstract class AppDatabase : RoomDatabase() {
    abstract fun userDao(): UserDao
    
    companion object {
        @Volatile
        private var INSTANCE: AppDatabase? = null
        
        fun getDatabase(context: Context): AppDatabase {
            return INSTANCE ?: synchronized(this) {
                val instance = Room.databaseBuilder(
                    context.applicationContext,
                    AppDatabase::class.java,
                    "app_database"
                ).build()
                INSTANCE = instance
                instance
            }
        }
    }
}
```

### 使用 Room

```kotlin
// 获取数据库实例
val db = AppDatabase.getDatabase(context)
val userDao = db.userDao()

// 在后台线程中执行数据库操作
// 使用 Kotlin Coroutines
lifecycleScope.launch(Dispatchers.IO) {
    // 插入数据
    val user = User(1, "John", "Doe")
    userDao.insertAll(user)
    
    // 查询数据
    val users = userDao.getAll()
    
    // 更新数据
    user.firstName = "Jane"
    userDao.update(user)
    
    // 删除数据
    userDao.delete(user)
}
```

## 高级特性

### 关系

#### 一对一关系

```kotlin
@Entity
data class User(
    @PrimaryKey val userId: Int,
    val name: String
)

@Entity
data class UserDetail(
    @PrimaryKey val detailId: Int,
    val userId: Int,
    val address: String,
    @ForeignKey(entity = User::class, parentColumns = ["userId"], childColumns = ["userId"])
)

// 数据类，用于查询结果
class UserWithDetail(
    @Embedded val user: User,
    @Relation(
        parentColumn = "userId",
        entityColumn = "userId"
    )
    val detail: UserDetail
)

// 在 DAO 中定义查询
@Dao
interface UserDao {
    @Transaction
    @Query("SELECT * FROM User")
    fun getUsersWithDetails(): List<UserWithDetail>
}
```

#### 一对多关系

```kotlin
@Entity
data class User(
    @PrimaryKey val userId: Int,
    val name: String
)

@Entity
data class Pet(
    @PrimaryKey val petId: Int,
    val userId: Int,
    val name: String,
    @ForeignKey(entity = User::class, parentColumns = ["userId"], childColumns = ["userId"])
)

class UserWithPets(
    @Embedded val user: User,
    @Relation(
        parentColumn = "userId",
        entityColumn = "userId"
    )
    val pets: List<Pet>
)
```

### 类型转换器

```kotlin
class Converters {
    @TypeConverter
    fun fromTimestamp(value: Long?): Date? {
        return value?.let { Date(it) }
    }
    
    @TypeConverter
    fun dateToTimestamp(date: Date?): Long? {
        return date?.time
    }
}

// 在 Database 类中添加
@Database(entities = [User::class], version = 1)
@TypeConverters(Converters::class)
abstract class AppDatabase : RoomDatabase() {
    // ...
}
```

### 数据库迁移

```kotlin
val MIGRATION_1_2 = object : Migration(1, 2) {
    override fun migrate(database: SupportSQLiteDatabase) {
        database.execSQL("ALTER TABLE users ADD COLUMN age INTEGER")
    }
}

// 在构建数据库时添加迁移
val db = Room.databaseBuilder(
    context.applicationContext,
    AppDatabase::class.java,
    "app_database"
).addMigrations(MIGRATION_1_2).build()
```

## 最佳实践

1. **使用 Singleton 模式**：确保数据库实例的唯一性
2. **在后台线程执行操作**：避免阻塞主线程
3. **使用事务**：确保数据一致性
4. **合理设计 Entity**：优化表结构
5. **使用索引**：提高查询性能
6. **定期清理数据**：避免数据库过大

## 常见问题

1. **编译错误**：确保所有 Entity 和 DAO 都正确注解
2. **运行时错误**：检查数据库版本和迁移策略
3. **性能问题**：优化查询语句，使用适当的索引
4. **内存泄漏**：避免在 DAO 中持有 Context 引用

## 面试题

1. **Q**: Room 与 SQLite 有什么区别？
   **A**: Room 是在 SQLite 基础上的抽象层，提供了编译时检查、简化的数据库操作、与 LiveData 和 Coroutines 的集成等优势

2. **Q**: Room 的三个核心组件是什么？
   **A**: Entity（表结构）、DAO（数据访问对象）、Database（数据库持有者）

3. **Q**: 如何处理数据库版本升级？
   **A**: 使用 Room 的 Migration 类，定义从旧版本到新版本的迁移逻辑

4. **Q**: Room 如何支持关系查询？
   **A**: 使用 @Relation 注解和嵌入式对象（@Embedded）来处理一对一和一对多关系

5. **Q**: 为什么 Room 推荐使用 DAO 而不是直接执行 SQL？
   **A**: DAO 提供了类型安全的方法，编译时检查，避免了 SQL 注入的风险，使代码更易于维护