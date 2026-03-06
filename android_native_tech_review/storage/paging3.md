# Paging3 面试专题（分页架构与性能）

## 1. 结论先说

Paging3 的核心价值不是“少写分页代码”，而是统一分页状态管理、加载策略和错误恢复机制，避免列表分页逻辑散落在 UI 层。

## 2. 核心组件

- `PagingSource<Key, Value>`：定义单次分页加载
- `Pager`：配置分页行为并产出 `Flow<PagingData<T>>`
- `PagingDataAdapter`：消费分页数据并绑定 RecyclerView
- `RemoteMediator`：本地数据库 + 远端接口双源协同

## 3. 基础接入模板

```kotlin
class FeedPagingSource(
    private val api: FeedApi
) : PagingSource<Int, FeedItem>() {
    override suspend fun load(params: LoadParams<Int>): LoadResult<Int, FeedItem> {
        val page = params.key ?: 1
        return try {
            val resp = api.getFeed(page = page, pageSize = params.loadSize)
            LoadResult.Page(
                data = resp.items,
                prevKey = if (page == 1) null else page - 1,
                nextKey = if (resp.items.isEmpty()) null else page + 1
            )
        } catch (e: Exception) {
            LoadResult.Error(e)
        }
    }

    override fun getRefreshKey(state: PagingState<Int, FeedItem>): Int? {
        val anchor = state.anchorPosition ?: return null
        val page = state.closestPageToPosition(anchor)
        return page?.prevKey?.plus(1) ?: page?.nextKey?.minus(1)
    }
}
```

## 4. 常见面试追问

### Q1：为什么分页不建议自己维护 pageIndex？
答：手写分页常见状态错乱、重复请求、刷新回滚问题，Paging3 已封装加载状态机。

### Q2：`getRefreshKey` 为什么重要？
答：它决定刷新后锚点恢复位置，不正确会导致刷新后跳页。

### Q3：何时用 RemoteMediator？
答：离线优先或需要本地缓存一致性时用；纯在线列表可直接 PagingSource。

## 5. 性能与稳定性要点

- 提供稳定 item id，减少列表抖动
- 避免 item bind 中重计算
- 对错误状态提供 retry，不要全页重建
- 控制 `pageSize/prefetchDistance`，按数据体量调优

## 6. 60秒答题模板

- 结论：Paging3 用状态机解决分页复杂度，核心在 `PagingSource + Pager + Adapter`。
- 依据：统一加载状态、自动处理刷新与追加、可接入本地缓存。
- 落地：列表页用 PagingDataFlow，错误态局部重试，关键参数做压测调优。
- 边界：简单静态列表不用 Paging3，避免过度引入复杂度。
