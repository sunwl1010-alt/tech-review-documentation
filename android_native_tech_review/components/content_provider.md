# ContentProvider（跨进程数据共享与权限模型）

## 1. 适用场景边界

ContentProvider 不是通用数据库访问层，而是“跨进程共享协议层”。
适合：
- 应用间数据共享（含受控授权）
- 与系统组件对齐的数据接口（如媒体、文件）

不适合：
- 仅应用内访问（直接 Room/Repository 更简单）

## 2. URI 设计规范

推荐 URI 语义化：
- 集合：`content://authority/items`
- 单项：`content://authority/items/{id}`
- 子资源：`content://authority/items/{id}/tags`

配套要求：
- MIME type 明确区分 `dir` 与 `item`
- 路由规则集中管理，避免散落字符串

## 3. 权限模型

### 3.1 全局权限
- `readPermission` / `writePermission`
- 高敏数据用 `signature` 级权限

### 3.2 URI 级临时授权
- 通过 `FLAG_GRANT_READ_URI_PERMISSION` 下发最小授权
- 授权范围尽可能细到单文件/单记录

### 3.3 FileProvider 实践
- 不直接暴露 `file://` 路径
- 使用 `FileProvider` + 临时 URI 权限共享文件

## 4. 一致性与事务

ContentProvider 需保证批量操作一致性：
- 实现 `applyBatch()` 使用事务包裹
- 对并发读写加版本戳或冲突策略
- 修改后调用 `notifyChange()` 保证订阅方感知

```kotlin
override fun applyBatch(operations: ArrayList<ContentProviderOperation>): Array<ContentProviderResult> {
    val db = helper.writableDatabase
    db.beginTransaction()
    return try {
        val results = super.applyBatch(operations)
        db.setTransactionSuccessful()
        results
    } finally {
        db.endTransaction()
    }
}
```

## 5. 性能治理

- 查询必须可分页（`limit/offset` 或 keyset）
- 常用条件建立索引
- 返回列最小化（projection）
- Cursor 必须可关闭且通知 URI 正确

## 6. 安全常见坑

- 错误导出：Provider 意外被外部读写
- SQL 拼接：selection 拼接导致注入风险
- 路径穿越：文件 URI 未做合法性校验

防御建议：
- 一律参数化查询
- 严格白名单路径
- 安全测试覆盖越权、注入、批量异常回滚

## 7. 面试深问

- 何时应该用 ContentProvider，而不是 API/SDK？
- 如何设计 Provider 的权限与授权粒度？
- `notifyChange()` 在复杂缓存体系里的作用是什么？
