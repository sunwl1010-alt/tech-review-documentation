# Kotlin Multiplatform (KMP)

## 核心概念

### 定义与目标
- **定义**：Kotlin Multiplatform是Kotlin官方推出的跨平台开发技术，允许开发者使用同一套Kotlin代码库构建运行在不同平台上的应用
- **目标**：实现"一次编写，多处运行"，减少平台特定代码的重复
- **核心理念**：将业务逻辑与平台特定代码分离，最大化代码共享

### 支持平台
- **移动平台**：Android、iOS
- **桌面平台**：Windows、macOS、Linux
- **Web平台**：浏览器
- **服务器平台**：JVM、Node.js
- **其他平台**：嵌入式设备

### 核心组件
- **共享模块**：包含平台无关的业务逻辑代码
- **平台特定模块**：包含针对特定平台的实现代码
- **预期声明**：使用`expect`和`actual`关键字处理平台差异
- **构建系统**：使用Gradle构建工具管理依赖和构建过程

### 工作原理
- **代码共享**：共享模块中的代码可以被所有平台使用
- **平台适配**：通过`expect`和`actual`机制处理平台特定差异
- **构建过程**：为每个平台生成特定的编译产物
- **运行时**：在不同平台上运行生成的平台特定代码

## 实现原理

### 代码结构
- **共享模块**：位于`shared`目录，包含平台无关的代码
- **平台模块**：位于各平台目录（如`androidApp`、`iosApp`），包含平台特定代码
- **依赖关系**：平台模块依赖共享模块

### 预期声明机制
- **`expect`关键字**：在共享模块中声明预期的函数、类或属性
- **`actual`关键字**：在平台特定模块中提供具体实现
- **编译检查**：编译器确保每个`expect`都有对应的`actual`实现

### 构建流程
1. **配置项目**：在`settings.gradle.kts`中配置项目结构
2. **定义共享模块**：在`shared/build.gradle.kts`中配置共享模块
3. **定义平台模块**：在各平台模块中配置依赖
4. **编写代码**：在共享模块中编写业务逻辑，在平台模块中处理平台特定功能
5. **构建项目**：运行Gradle构建命令，为各平台生成编译产物

### 跨平台数据传输
- **数据模型**：在共享模块中定义数据模型
- **序列化**：使用`kotlinx.serialization`进行跨平台数据序列化
- **网络请求**：使用Ktor进行跨平台网络请求
- **数据存储**：使用SQLDelight进行跨平台数据库操作

## 使用场景

### 跨平台业务逻辑
- **数据模型**：在多个平台上共享数据结构
- **网络请求**：统一网络请求逻辑
- **业务规则**：共享业务逻辑和规则
- **数据存储**：跨平台数据存储方案

### 多平台应用开发
- **移动应用**：同时开发Android和iOS应用
- **全平台应用**：开发覆盖移动、桌面和Web的应用
- **代码复用**：减少平台特定代码的重复
- **一致性**：确保不同平台上的业务逻辑一致

### 库开发
- **跨平台库**：开发可在多个平台使用的库
- **代码共享**：最大化库代码的共享
- **平台适配**：为不同平台提供特定实现
- **统一API**：为所有平台提供统一的API

### 现有项目迁移
- **渐进式采用**：逐步将现有代码迁移到KMP
- **模块重构**：将业务逻辑提取到共享模块
- **平台适配**：为现有平台代码提供适配层
- **测试策略**：确保迁移后的代码质量

## 常见问题及解决方案

### 平台差异处理
- **问题**：不同平台的API差异
  **解决方案**：使用`expect`和`actual`机制处理平台差异

- **问题**：平台特定功能无法共享
  **解决方案**：在平台特定模块中实现，通过接口在共享模块中调用

- **问题**：第三方库平台支持不一致
  **解决方案**：选择跨平台支持的库，或为不同平台提供不同的实现

### 构建问题
- **问题**：构建配置复杂
  **解决方案**：使用官方模板和插件，参考官方文档

- **问题**：构建时间长
  **解决方案**：使用增量构建，优化依赖管理

- **问题**：平台特定构建错误
  **解决方案**：分别构建各平台，定位具体错误

### 性能问题
- **问题**：跨平台代码性能劣于原生代码
  **解决方案**：优化共享代码，对性能敏感部分使用平台特定实现

- **问题**：启动时间慢
  **解决方案**：优化初始化过程，延迟加载非必要功能

- **问题**：内存使用高
  **解决方案**：优化内存管理，避免内存泄漏

### 工具支持问题
- **问题**：IDE支持不完善
  **解决方案**：使用最新版本的Android Studio或IntelliJ IDEA

- **问题**：调试困难
  **解决方案**：使用平台特定的调试工具，结合日志

- **问题**：测试覆盖不足
  **解决方案**：为共享代码和平台特定代码分别编写测试

## 代码示例

### 共享模块示例
```kotlin
// 共享模块中的预期声明
expect class Platform() {
    val name: String
}

// 共享业务逻辑
class SharedRepository {
    fun getPlatformName(): String {
        return Platform().name
    }
    
    fun calculateResult(a: Int, b: Int): Int {
        return a + b
    }
}

// 数据模型
@Serializable
data class User(
    val id: Int,
    val name: String,
    val email: String
)

// 网络请求
class ApiService {
    suspend fun fetchUsers(): List<User> {
        // 使用Ktor进行网络请求
        val client = HttpClient {
            install(ContentNegotiation) {
                json()
            }
        }
        
        return client.get("https://api.example.com/users") {
            contentType(ContentType.Application.Json)
        }
    }
}
```

### 平台特定实现示例
```kotlin
// Android平台实现
actual class Platform actual constructor() {
    actual val name: String = "Android"
}

// iOS平台实现
actual class Platform actual constructor() {
    actual val name: String = "iOS"
}

// Web平台实现
actual class Platform actual constructor() {
    actual val name: String = "Web"
}

// Desktop平台实现
actual class Platform actual constructor() {
    actual val name: String = "Desktop"
}
```

### 平台集成示例
```kotlin
// 共享模块中的接口
expect class NotificationService {
    fun sendNotification(title: String, message: String)
}

// Android平台实现
actual class NotificationService actual constructor() {
    actual fun sendNotification(title: String, message: String) {
        // 使用Android Notification API
        val notificationManager = context.getSystemService(Context.NOTIFICATION_SERVICE) as NotificationManager
        // 实现通知逻辑
    }
}

// iOS平台实现
actual class NotificationService actual constructor() {
    actual fun sendNotification(title: String, message: String) {
        // 使用iOS Notification API
        // 实现通知逻辑
    }
}
```

### 构建配置示例
```kotlin
// settings.gradle.kts
rootProject.name = "MultiplatformApp"

include(":shared")
include(":androidApp")
include(":iosApp")
include(":webApp")
include(":desktopApp")

// shared/build.gradle.kts
plugins {
    kotlin("multiplatform")
    kotlin("plugin.serialization")
}

kotlin {
    jvm()
    iosX64()
    iosArm64()
    iosSimulatorArm64()
    js(IR) {
        browser()
    }
    macosX64()
    macosArm64()
    linuxX64()
    windowsX64()

    sourceSets {
        val commonMain by getting {
            dependencies {
                implementation("org.jetbrains.kotlinx:kotlinx-coroutines-core")
                implementation("org.jetbrains.kotlinx:kotlinx-serialization-json")
                implementation("io.ktor:ktor-client-core")
                implementation("io.ktor:ktor-client-content-negotiation")
                implementation("io.ktor:ktor-serialization-json")
            }
        }
        val commonTest by getting {
            dependencies {
                implementation(kotlin("test"))
            }
        }
        val jvmMain by getting
        val jvmTest by getting
        val iosMain by creating {
            dependsOn(commonMain)
        }
        val iosTest by creating {
            dependsOn(commonTest)
        }
        val jsMain by getting {
            dependsOn(commonMain)
        }
        val jsTest by getting {
            dependsOn(commonTest)
        }
        val macosMain by creating {
            dependsOn(commonMain)
        }
        val macosTest by creating {
            dependsOn(commonTest)
        }
        val linuxMain by creating {
            dependsOn(commonMain)
        }
        val linuxTest by creating {
            dependsOn(commonTest)
        }
        val windowsMain by creating {
            dependsOn(commonMain)
        }
        val windowsTest by creating {
            dependsOn(commonTest)
        }
    }
}
```

## 面试常考问题及参考答案

### 基础理论

**1. 什么是Kotlin Multiplatform (KMP)？**

**答案**：Kotlin Multiplatform是Kotlin官方推出的跨平台开发技术，允许开发者使用同一套Kotlin代码库构建运行在不同平台上的应用，实现"一次编写，多处运行"，减少平台特定代码的重复。

**2. KMP的核心组件有哪些？**

**答案**：
- **共享模块**：包含平台无关的业务逻辑代码
- **平台特定模块**：包含针对特定平台的实现代码
- **预期声明**：使用`expect`和`actual`关键字处理平台差异
- **构建系统**：使用Gradle构建工具管理依赖和构建过程

**3. 如何处理KMP中的平台差异？**

**答案**：使用`expect`和`actual`关键字处理平台差异。在共享模块中使用`expect`声明预期的功能，在各平台特定模块中使用`actual`提供具体实现。

### 实际应用

**4. KMP适合什么场景？**

**答案**：
- **跨平台应用开发**：同时开发Android、iOS、Web和Desktop应用
- **代码复用需求高的项目**：需要在多个平台上共享业务逻辑
- **库开发**：开发可在多个平台使用的库
- **现有项目迁移**：逐步将现有代码迁移到KMP

**5. 如何开始使用KMP？**

**答案**：
- 安装最新版本的Android Studio或IntelliJ IDEA
- 使用KMP项目模板创建新项目
- 配置共享模块和平台特定模块
- 编写共享的业务逻辑和平台特定代码
- 构建并运行各平台应用

**6. KMP与其他跨平台框架的区别是什么？**

**答案**：
- **与Flutter相比**：KMP与现有Kotlin/Java代码集成更无缝，可以与平台原生代码和库深度集成
- **与React Native相比**：KMP使用Kotlin语言，类型安全，性能更接近原生
- **与Xamarin相比**：KMP使用Kotlin语言，构建系统更现代化

### 性能优化

**7. 如何优化KMP应用的性能？**

**答案**：
- **代码优化**：优化共享代码，避免不必要的计算
- **平台特定优化**：对性能敏感部分使用平台特定实现
- **构建优化**：使用增量构建，优化依赖管理
- **内存管理**：优化内存使用，避免内存泄漏

**8. KMP应用的性能与原生应用相比如何？**

**答案**：KMP应用的性能接近原生应用，因为共享代码会被编译为各平台的原生代码。对于性能敏感的部分，可以使用平台特定实现来达到与原生应用相同的性能水平。

### 架构设计

**9. 如何设计一个KMP项目的架构？**

**答案**：
- **模块化设计**：将应用分为共享模块和平台特定模块
- **分层架构**：在共享模块中使用分层架构，如数据层、业务逻辑层和UI层
- **依赖注入**：使用依赖注入管理依赖
- **状态管理**：使用适合的状态管理方案
- **测试策略**：为共享代码和平台特定代码分别编写测试

**10. 如何处理KMP项目中的依赖管理？**

**答案**：
- **共享依赖**：在共享模块中添加跨平台支持的依赖
- **平台特定依赖**：在平台特定模块中添加平台特定的依赖
- **版本管理**：统一管理依赖版本，确保兼容性
- **依赖解析**：处理依赖冲突，确保构建成功

**11. 如何测试KMP应用？**

**答案**：
- **共享代码测试**：在共享模块中编写单元测试
- **平台特定测试**：在各平台模块中编写集成测试
- **UI测试**：为各平台编写UI测试
- **自动化测试**：集成CI/CD流程，自动运行测试

**12. KMP的未来发展趋势是什么？**

**答案**：
- **生态系统完善**：更多的官方和第三方库支持
- **工具链改进**：更好的IDE支持和构建工具
- **平台覆盖扩展**：支持更多平台
- **性能优化**：持续优化跨平台代码的性能
- **采用率提高**：越来越多的项目采用KMP技术

Kotlin Multiplatform代表了跨平台开发的未来方向，它提供了一种更高效、更一致的跨平台开发方式。随着生态系统的不断完善，KMP将成为构建跨平台应用的主流技术。