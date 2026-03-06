# Clean Architecture 实战落地（Android）

## 1. 为什么是 Clean Architecture
在中大型 Android 项目里，主要问题通常不是“功能不会写”，而是：
- 业务逻辑散落在 Activity/Fragment
- 数据层和 UI 强耦合，改需求牵一发而动全身
- 难测试、难重构、难并行开发

Clean Architecture 的核心价值是“依赖方向稳定”：
- 外层依赖内层
- 内层不感知外层实现细节
- 业务规则独立于框架与平台

## 2. 分层与职责边界

### 2.1 推荐分层
- presentation：UI + 状态管理（ViewModel / MVI）
- domain：UseCase + Entity + Repository 抽象
- data：Repository 实现 + Local/Remote DataSource + Mapper

### 2.2 依赖规则
- presentation -> domain
- data -> domain
- domain 不依赖任何 Android SDK（纯 Kotlin 优先）

## 3. 工程目录示例
```text
app/
  feature/
    user/
      presentation/
      domain/
      data/
  core/
    network/
    database/
    common/
```

## 4. 数据流设计（单向）

建议采用单向数据流（UDF）：
- UI 发送 Intent
- ViewModel 处理 Intent，调用 UseCase
- UseCase 与 Repository 交互
- 结果回到 UIState

优点：状态演进可追踪、易测试、可回放。

## 5. UseCase 设计原则

- 一个 UseCase 只做一件事
- 输入输出明确（参数对象 + 结果对象）
- 对外暴露稳定语义，不泄露数据层细节

```kotlin
class FetchUserProfileUseCase(
    private val repository: UserRepository
) {
    suspend operator fun invoke(userId: String): Result<UserProfile> {
        return repository.fetchUserProfile(userId)
    }
}
```

## 6. Repository 边界与反模式

### 6.1 正确边界
Repository 负责“数据聚合策略”，而不是简单转发。

示例职责：
- 远端/本地数据源协调
- 缓存策略（cache aside / stale-while-revalidate）
- 错误归一化（网络错误、业务错误、解析错误）

### 6.2 常见反模式
- Repository 仅调用一个 API 方法，等于无抽象
- Domain 直接依赖 Retrofit DTO
- UI 感知数据库表结构

## 7. 错误模型与可恢复性

建议统一错误模型，避免 UI 到处 `try-catch`：

```kotlin
sealed interface AppError {
    data object Network : AppError
    data object Unauthorized : AppError
    data class Business(val code: Int, val message: String) : AppError
    data object Unknown : AppError
}
```

搭配 `Result<T, AppError>`（或 Kotlin Result + 扩展），让错误处理可组合。

## 8. 并发与协程约束

- ViewModelScope 负责 UI 生命周期范围任务
- IO 密集放 `Dispatchers.IO`，CPU 密集放 `Dispatchers.Default`
- 避免在 UseCase 中直接切主线程
- 对并行请求显式声明结构化并发边界（`coroutineScope`）

## 9. 测试策略（分层）

- domain：纯单元测试（最快，覆盖业务规则）
- data：仓储测试（fake server + in-memory db）
- presentation：ViewModel 状态流测试（Turbine）

示例：UseCase 单测关注输入输出，不依赖 Android 环境。

## 10. 演进策略：从 MVC/MVP/MVVM 平滑迁移

1. 先抽 domain 接口（UseCase、Repository）
2. 再把数据访问从 UI 层迁移到 data 层
3. 最后统一状态模型与事件流
4. 每个 feature 小步迁移，避免大爆炸重构

## 11. 性能与架构的平衡

- 不要过度抽象：小项目可减少层数
- 热路径避免多层无意义 mapper
- 可观测性优先：关键链路保留 trace 与日志

## 12. 面试深问

- 如何判断一个项目是否适合引入 Clean Architecture？
  关键看团队规模、需求变更频率、模块复杂度、测试诉求。

- 如何避免“分层过度导致开发变慢”？
  通过模板化约束、代码生成、统一错误模型、减少样板代码。
