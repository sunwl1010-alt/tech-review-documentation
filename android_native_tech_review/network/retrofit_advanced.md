# Retrofit 进阶面试专题（配合 OkHttp）

## 1. Retrofit 的定位

Retrofit 不是“网络库本体”，它是“声明式 API + 适配层”：
- 底层请求执行：OkHttp
- 接口声明：Retrofit interface
- 数据转换：Converter
- 调用风格适配：CallAdapter（协程/Rx/自定义结果）

## 2. 核心执行链路

1. 调用接口方法
2. Retrofit 解析注解构建 Request
3. 交给 OkHttp 执行
4. 响应经过 Converter 转为目标类型
5. CallAdapter 把结果包装成 `suspend` / `Flow` / `Result`

面试高频关键词：`ConverterFactory`、`CallAdapterFactory`、`Invocation`。

## 3. OkHttp 拦截器顺序与职责

- Application Interceptor：重试、鉴权、统一 header、日志脱敏
- Network Interceptor：与网络层细节相关（重定向、缓存观察）

实践建议：
- 鉴权、trace-id、幂等 key 放 Application Interceptor
- release 禁止完整 body 日志

## 4. 统一错误模型（重点）

问题：很多项目把异常散落在 ViewModel/Repository，导致处理不一致。  
解法：统一 `ApiResult` + 错误映射层。

常见映射：
- `IOException` -> 网络错误
- HTTP 401/403 -> 鉴权错误
- HTTP 5xx -> 服务端错误
- 业务码非0 -> 业务错误
- 解析异常 -> 数据协议错误

## 5. 错误体解析

面试常问：`response.errorBody()` 怎么处理？

建议：
- 使用同一 JSON 解析器反序列化错误体
- 错误体解析失败时兜底为通用错误
- 不要在 UI 层解析错误体

## 6. Token 刷新与并发 401

推荐“单飞刷新”策略：
- 首个 401 请求触发刷新 token
- 其他 401 请求等待刷新结果
- 刷新成功重放请求，失败统一登出

避免：每个请求各自刷新，造成风暴和竞态。

## 7. 超时、取消与重试

- connect/read/write/callTimeout 分层配置
- 协程取消应透传到底层 Call.cancel
- 重试只对幂等请求启用，并使用指数退避 + 抖动

## 8. Converter / CallAdapter 扩展

### Converter
- Gson/Moshi/Kotlinx Serialization
- 建议统一一个序列化标准，避免行为差异

### CallAdapter
- 可把 `Call<T>` 统一转成 `Result<T>`
- 统一成功/失败语义，减少样板代码

## 9. 可测试性（面试加分）

- 单元测试：接口层 mock + 错误映射测试
- 集成测试：MockWebServer 验证超时、重试、401 刷新
- 契约测试：确保字段变化能被及时发现

## 10. 高频追问

### Q1：Retrofit 和 OkHttp 的关系？
答：Retrofit 负责“声明与适配”，OkHttp 负责“真实请求执行”。

### Q2：为什么要统一错误模型？
答：提高一致性和可观测性，避免每层重复处理异常。

### Q3：如何防止 401 刷新雪崩？
答：单飞刷新 + 队列等待 + 失败快速收敛。

### Q4：为什么重试不能全开？
答：非幂等请求重试会放大副作用，必须按业务语义选择。

## 11. 60秒答题模板

- 结论：Retrofit 是 API 声明与适配层，稳定性关键在 OkHttp 配置与错误治理。
- 依据：拦截器分层、统一错误模型、单飞 token 刷新、幂等重试。
- 落地：MockWebServer 覆盖 401/超时/弱网场景，发布前做回归门禁。
- 边界：大文件传输和长连接场景需配套下载器/WebSocket，不靠 Retrofit 单独解决。
