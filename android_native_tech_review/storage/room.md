# Room（数据层一致性、迁移与性能实践）

## 1. Room 的工程价值

Room 的核心价值不是“写 SQL 更少”，而是：
- 编译期校验 SQL，降低线上语法错误
- 明确 schema 演进路径，减少升级事故
- 与协程/Flow 集成，简化响应式数据流

## 2. 分层落地建议

- DAO：只做数据访问，不做业务逻辑
- LocalDataSource：组合 DAO，封装本地策略
- Repository：协调本地与远端，定义缓存策略

## 3. 事务与一致性

多表写入必须事务化：

```kotlin
@Dao
interface OrderDao {
    @Insert
    suspend fun insertOrder(order: OrderEntity)

    @Insert
    suspend fun insertItems(items: List<OrderItemEntity>)

    @Transaction
    suspend fun saveOrder(order: OrderEntity, items: List<OrderItemEntity>) {
        insertOrder(order)
        insertItems(items)
    }
}
```

要点：
- 领域一致性操作必须放同一事务
- 避免“先写主表再异步写子表”导致脏状态

## 4. Migration 策略

### 4.1 强制显式迁移
- 禁止线上使用 `fallbackToDestructiveMigration()`
- 每次版本升级必须提供 migration 脚本与测试

```kotlin
val MIGRATION_3_4 = object : Migration(3, 4) {
    override fun migrate(db: SupportSQLiteDatabase) {
        db.execSQL("ALTER TABLE users ADD COLUMN avatar TEXT")
        db.execSQL("CREATE INDEX IF NOT EXISTS idx_users_email ON users(email)")
    }
}
```

### 4.2 自动迁移的边界
- 简单加列可考虑 AutoMigration
- 涉及数据回填、表拆分、语义变更应使用手写迁移

## 5. 查询性能

- 高频查询建立复合索引
- 列表页禁止 `SELECT *`
- 大查询分页 + 懒加载
- 用 `EXPLAIN QUERY PLAN` 检查索引命中

## 6. Flow 与线程模型

- DAO `suspend` 方法默认放 IO 调度
- `Flow` 查询可自动感知表变化
- 避免在主线程做大结果集映射

## 7. 数据模型治理

- Entity 与 Domain Model 分离
- Mapper 层集中维护，避免 UI 直接依赖 Entity
- 枚举/状态字段要做向前兼容设计

## 8. 测试策略

- Migration test：旧版本 -> 新版本数据完整性
- DAO test：边界条件与冲突策略
- Repository test：缓存命中、失效、降级路径

## 9. 面试深问

- Room 迁移如何做到可回滚与可验证？
- 如何平衡 SQL 性能与代码可维护性？
- 为什么不建议让 UI 直接依赖 Entity？
