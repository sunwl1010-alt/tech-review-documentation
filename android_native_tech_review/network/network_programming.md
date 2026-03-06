# Android 网络编程（工程实战）

## 1. 目标与边界
网络层的目标不是“把请求发出去”，而是提供稳定、可观测、可演进的数据通道。一个可用的网络层至少要满足：

- 正确性：请求参数、签名、幂等、错误分类正确
- 稳定性：弱网可恢复，超时和重试可控
- 性能：连接复用、缓存合理、解析高效
- 可观测：链路追踪、错误埋点、慢请求分析

## 2. 推荐分层

- API 层：Retrofit interface，仅描述协议
- DataSource 层：封装请求与响应转换
- Repository 层：缓存与数据聚合策略
- Domain 层：UseCase，不暴露网络细节

## 3. OkHttp 基础配置模板

```kotlin
fun buildHttpClient(): OkHttpClient {
    return OkHttpClient.Builder()
        .connectTimeout(10, TimeUnit.SECONDS)
        .readTimeout(15, TimeUnit.SECONDS)
        .writeTimeout(15, TimeUnit.SECONDS)
        .callTimeout(20, TimeUnit.SECONDS)
        .connectionPool(ConnectionPool(10, 5, TimeUnit.MINUTES))
        .addInterceptor(AuthInterceptor())
        .addInterceptor(IdempotencyKeyInterceptor())
        .addNetworkInterceptor(HttpLoggingInterceptor().apply {
            level = HttpLoggingInterceptor.Level.BASIC
        })
        .build()
}
```

关键点：
- `callTimeout` 防止整条调用无限挂起。
- 业务拦截器做鉴权、签名、trace-id 注入。
- 日志拦截器在 release 环境要脱敏或关闭。

## 4. Retrofit + 统一结果模型

```kotlin
sealed interface ApiResult<out T> {
    data class Success<T>(val data: T, val code: Int) : ApiResult<T>
    data class Failure(val error: AppError, val code: Int? = null) : ApiResult<Nothing>
}

suspend fun <T> safeApiCall(block: suspend () -> Response<T>): ApiResult<T> {
    return try {
        val response = block()
        if (response.isSuccessful) {
            val body = response.body()
            if (body != null) ApiResult.Success(body, response.code())
            else ApiResult.Failure(AppError.EmptyBody, response.code())
        } else {
            ApiResult.Failure(AppError.Http(response.code()), response.code())
        }
    } catch (e: IOException) {
        ApiResult.Failure(AppError.Network)
    } catch (e: Exception) {
        ApiResult.Failure(AppError.Unknown)
    }
}
```

## 5. 重试与退避策略

适用场景：短暂网络抖动、网关超时、幂等接口失败。  
不适用场景：参数错误、鉴权失败、业务拒绝。

```kotlin
suspend fun <T> retryIO(
    times: Int = 3,
    initialDelayMs: Long = 300,
    block: suspend () -> T
): T {
    var delayMs = initialDelayMs
    repeat(times - 1) {
        try {
            return block()
        } catch (e: IOException) {
            delay(delayMs + Random.nextLong(0, 100))
            delayMs *= 2
        }
    }
    return block()
}
```

## 6. 缓存策略

- 强一致场景：直接走网络（支付、库存）
- 可接受短期陈旧：`stale-while-revalidate`
- 列表分页：本地缓存 + 增量刷新

OkHttp 缓存常见误区：
- 只配客户端缓存，不遵守服务端缓存头
- 把错误响应缓存下来导致脏读

## 7. 鉴权与安全

- token 自动续签：401 后单飞刷新，避免并发雪崩
- 请求签名：timestamp + nonce + body hash
- 证书 Pinning：双 pin 轮转，支持灰度
- 禁止明文流量（release）

## 8. 文件上传下载

### 上传
- 分片上传 + 断点续传
- 大文件使用流式读写，避免一次性读入内存
- 失败重试要幂等（uploadId + chunkIndex）

### 下载
- `Range` 断点下载
- 校验文件完整性（hash）
- 避免在主线程做 I/O

## 9. 可观测性

建议每个请求带以下字段：
- requestId / traceId
- apiName
- latencyMs
- networkType（WiFi/4G/5G）
- resultCode（HTTP/业务码）

这些数据用于：
- 慢接口定位
- 版本回归分析
- 弱网体验优化

## 10. 面试深问

- 如何设计 token 刷新避免并发 401 风暴？
  关键：单飞锁 + 队列等待 + 刷新失败快速失败。

- 如何处理“本地有缓存但服务端变更频繁”？
  关键：缓存过期策略 + ETag/If-Modified-Since + 主动失效。

- 如何实现网络层可测试？
  关键：接口抽象 + MockWebServer + 错误注入测试。
