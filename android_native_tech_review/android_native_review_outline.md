# Android 原生技术复习大纲

## 核心组件
- **[Activity](components/activity.md)**
  - 生命周期
  - 启动模式
  - 状态保存与恢复
  - Intent 与 Intent Filter

- **[Service](components/service.md)**
  - 生命周期
  - 前台服务
  - 绑定服务
  - IntentService

- **[BroadcastReceiver](components/broadcast_receiver.md)**
  - 静态注册与动态注册
  - 有序广播与无序广播
  - 本地广播
  - 系统广播

- **[ContentProvider](components/content_provider.md)**
  - URI 结构
  - CRUD 操作
  - ContentObserver
  - 权限管理

## UI 系统
- **布局系统**
  - XML 布局
  - Jetpack Compose
  - [布局管理器](ui/layout_managers.md)
  - 自定义 View

- **[控件系统](ui/control_system.md)**
  - 基础控件
  - 自定义控件
  - 适配器模式
  - 列表与网格

## 数据存储
- **[SharedPreferences](storage/shared_preferences.md)**
- **[文件存储](storage/file_storage.md)**
- **[SQLite 数据库](storage/sqlite.md)**
- **[Room 持久化库](storage/room.md)**

## [网络编程](network/network_programming.md)
- **OkHttp**
- **Retrofit**
- **Volley**
- **WebSocket**

## [多线程与异步](multithreading/multithreading.md)
- **Thread**
- **Handler**
- **AsyncTask**
- **ThreadPoolExecutor**
- **Coroutine**

## [系统服务](system_services/system_services.md)
- **PackageManager**
- **ActivityManager**
- **NotificationManager**
- **LocationManager**

## [安全性](security/security.md)
- **权限管理**
- **数据加密**
- **网络安全**
- **应用签名**

## [性能优化](performance/performance_optimization.md)
- **内存优化**
- **UI 优化**
- **网络优化**
- **电池优化**

## 架构设计
- **[MVC](architecture/mvc.md)**
- **[MVP](architecture/mvp.md)**
- **[MVVM](architecture/mvvm.md)**
- **[Clean Architecture](architecture/clean_architecture.md)**

## 测试
- **单元测试**
- **集成测试**
- **UI 测试**

## 版本兼容性
- **API 级别**
- **权限变化**
- **行为变化**
- **兼容性方案**

## 跨平台开发
- **[Kotlin Multiplatform (KMP)](ui/xml_vs_compose.md#kotlin-multiplatform-kmp)**
  - 核心概念
  - 共享模块
  - 平台特定实现
  - 预期声明 (expect/actual)
- **[Compose Multiplatform (CMP)](ui/xml_vs_compose.md#compose-multiplatform-cmp)**
  - 跨平台UI
  - 平台适配
  - 与KMP集成
  - 开发工具与生态
## 架构落地与工程实践
- **[Clean Architecture](architecture/clean_architecture.md)**
  - 分层职责与依赖方向
  - UseCase 与 Repository 边界
  - 错误模型与可恢复性
  - 渐进式迁移策略

## 深度专题（工程化）
- **[Activity 深度实践](components/activity.md)**
- **[网络编程（工程实战）](network/network_programming.md)**
- **[多线程与异步（现代实践）](multithreading/multithreading.md)**
- **[MVVM 架构（进阶落地）](architecture/mvvm.md)**
- **[Clean Architecture](architecture/clean_architecture.md)**
- **[安全工程实践](security/security.md)**
- **[性能优化（工程化实战）](performance/performance_optimization.md)**

## 遗留架构迁移专题
- **[MVC（历史模式与迁移指南）](architecture/mvc.md)**
- **[MVP（遗留架构维护与升级）](architecture/mvp.md)**

## 数据层进阶
- **[Room（一致性与迁移实践）](storage/room.md)**
- **[SQLite（底层能力与遗留治理）](storage/sqlite.md)**

## 学习路线与决策
- **[Android 学习路线图](android_learning_roadmap.md)**
- **[布局体系（性能与可维护性）](ui/layout_managers.md)**
- **[控件体系（状态与交互）](ui/control_system.md)**

## 面试冲刺
- **[Android 面试冲刺索引](android_interview_sprint.md)**

## 冲刺速记
- **[Android 1 页速记卡](android_quick_cards.md)**

## 高频原理专题
- **[Binder / IPC 面试专题](system_services/binder_ipc.md)**
- **[Retrofit 进阶专题](network/retrofit_advanced.md)**
- **[ANR 与消息机制专题](multithreading/anr_looper_handler.md)**
- **[Activity 启动流程专题](components/activity_startup_pipeline.md)**
- **[Handler/MessageQueue/Choreographer 专题](multithreading/handler_messagequeue_choreographer.md)**
- **[Compose 性能与重组优化专题](ui/compose_performance.md)**
