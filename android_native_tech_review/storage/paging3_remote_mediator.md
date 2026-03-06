# Paging3 RemoteMediator 实战专题

## 1. 场景定位

RemoteMediator 用于“本地数据库 + 远端分页接口”协同：
- 首屏先读本地，提升可用性
- 后台拉远端并写库
- 列表展示统一来自本地 PagingSource

## 2. 架构图（文字）

UI -> Pager -> Room PagingSource  
RemoteMediator <-> Remote API  
RemoteMediator -> Room（事务写入）

关键点：UI 不直接消费远端分页结果，统一消费数据库数据。

## 3. 实战模板

```kotlin
@OptIn(ExperimentalPagingApi::class)
class FeedRemoteMediator(
    private val db: AppDb,
    private val api: FeedApi
) : RemoteMediator<Int, FeedEntity>() {

    override suspend fun load(
        loadType: LoadType,
        state: PagingState<Int, FeedEntity>
    ): MediatorResult {
        return try {
            val page = when (loadType) {
                LoadType.REFRESH -> 1
                LoadType.PREPEND -> return MediatorResult.Success(endOfPaginationReached = true)
                LoadType.APPEND -> {
                    val last = state.lastItemOrNull() ?: return MediatorResult.Success(true)
                    last.page + 1
                }
            }

            val resp = api.getFeed(page, state.config.pageSize)
            db.withTransaction {
                if (loadType == LoadType.REFRESH) db.feedDao().clearAll()
                db.feedDao().insertAll(resp.items.map { it.toEntity(page) })
            }
            MediatorResult.Success(endOfPaginationReached = resp.items.isEmpty())
        } catch (e: Exception) {
            MediatorResult.Error(e)
        }
    }
}
```

## 4. 关键设计点

- 刷新时是否清表：按业务决定（时间线型通常清理）
- 页码来源：不要依赖 UI，基于本地 remote keys 或最后 item 计算
- 写库必须事务化，避免分页状态与数据不同步

## 5. 常见故障

- APPEND 死循环：endOfPagination 判断错误
- 刷新后重复数据：缺少主键约束或 merge 策略
- 列表闪烁：UI 直接混用本地和远端源

## 6. 面试追问

### Q1：为什么 RemoteMediator 比“先请求再 submitList”更好？
答：统一数据源、离线可用、状态一致性更强。

### Q2：何时不需要 RemoteMediator？
答：纯在线短列表、无本地缓存诉求场景可直接 PagingSource。

## 7. 60 秒答题模板

- 结论：RemoteMediator 适合离线优先分页。
- 依据：本地单一数据源 + 远端增量写入 + 事务一致性。
- 落地：我会把刷新、追加和 endOfPagination 策略显式化并做集成测试。
