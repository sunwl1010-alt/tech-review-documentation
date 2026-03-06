# SQLite（底层能力与遗留系统治理）

## 1. 何时仍需要直接用 SQLite

尽管 Room 是首选，但以下场景可考虑直接 SQLite：
- 极致性能/定制 SQL 优化需求
- 历史项目已深度依赖 `SQLiteOpenHelper`
- 需要与旧表结构兼容且迁移成本高

## 2. SQLiteOpenHelper 工程规范

- 单一入口管理 DB 版本与升级脚本
- 升级脚本必须幂等、可重复执行
- 每次版本变更记录 DDL 与回滚方案

```kotlin
class AppDbHelper(context: Context) : SQLiteOpenHelper(context, "app.db", null, 4) {
    override fun onCreate(db: SQLiteDatabase) {
        db.execSQL("""
            CREATE TABLE users(
                id INTEGER PRIMARY KEY,
                name TEXT NOT NULL,
                email TEXT NOT NULL UNIQUE
            )
        """.trimIndent())
    }

    override fun onUpgrade(db: SQLiteDatabase, oldVersion: Int, newVersion: Int) {
        if (oldVersion < 4) {
            db.execSQL("CREATE INDEX IF NOT EXISTS idx_users_email ON users(email)")
        }
    }
}
```

## 3. 事务与批量写入

- 批量写必须包事务
- 大批量写入使用 `beginTransactionNonExclusive()` 结合分批
- 每批次失败要可重试或补偿

## 4. SQL 安全

- 一律参数化查询，禁止字符串拼接
- 对动态排序字段使用白名单
- 高敏表访问增加审计日志

## 5. 性能关键点

- 索引不是越多越好，需平衡写放大
- 复杂联表提前做 explain 分析
- 热路径避免 N+1 查询
- WAL 模式可提升并发读写表现

## 6. 并发与锁

SQLite 单写多读，常见问题是写锁竞争。治理思路：
- 缩短事务时间
- 避免在事务内做网络调用
- 串行化关键写路径

## 7. 从 SQLite 迁移到 Room 的分步方案

1. 保留原表结构，先引入 Room 映射旧表
2. 先迁移读路径，再迁移写路径
3. 最后移除直接 SQLite 调用，统一 Repository

## 8. 面试深问

- 直接 SQLite 与 Room 的取舍标准是什么？
- 如何排查线上偶发数据库锁超时？
- 迁移过程中如何保证数据一致性与可回退？
