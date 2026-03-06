# Flutter 技术复习大纲

## 核心概念
- **[Flutter 架构](core_concepts/flutter_architecture.md)**
  - Framework / Engine / Embedder
  - 自渲染架构
  - Platform Channel 与 FFI
- **[渲染引擎与 Skia](core_concepts/rendering_engine.md)**
  - Skia 渲染引擎
  - Widget / Element / RenderObject
  - Layer 与 Scene
  - UI / Raster / Platform 线程
  - 掉帧原理
- **[Dart 语言](core_concepts/dart_language.md)**
  - 语法基础
  - 异步编程
  - 空安全
  - 泛型
- **[Dart 进阶](core_concepts/dart_advanced.md)**
  - event loop
  - microtask / event queue
  - Future / Stream
  - isolate
  - 并发与内存模型
- **[Widget 体系](core_concepts/widget_system.md)**
  - StatelessWidget
  - StatefulWidget
  - InheritedWidget
  - RenderObjectWidget
  - BuildContext / Key / 约束系统

## 状态管理
- **[Provider](state_management/provider.md)**
- **[Bloc](state_management/bloc.md)**
- **[GetX](state_management/getx.md)**
- **[Riverpod](state_management/riverpod.md)**
- **[Redux](state_management/redux.md)**
- **[状态管理选型](state_management/state_management_selection.md)**
  - 局部状态 vs 全局状态
  - Riverpod / Bloc / GetX / Redux 对比
  - 选型原则

## 路由与导航
- **[基础导航](navigation/routing_navigation.md)**
  - `Navigator.push` / `pop`
  - 命名路由
  - 参数传递
- **[路由进阶](navigation/router_advanced.md)**
  - Navigator 1.0 vs 2.0
  - Router 体系
  - `go_router`
  - 深链路
  - Web URL 同步
  - 嵌套路由与重定向

## 网络请求
- **[http 与 Dio](network/network_request.md)**
- **[Retrofit](network/network_request.md)**
- **[WebSocket](network/network_request.md)**
- **[网络层进阶](network/network_advanced.md)**
  - Dio 拦截器
  - token 刷新
  - 超时 / 取消 / 重试
  - 错误分层
- **[数据层架构](network/data_layer_architecture.md)**
  - repository
  - remote / local datasource
  - 缓存策略
  - 数据编排

## 本地存储
- **[SharedPreferences](storage/local_storage.md)**
- **[SQLite](storage/local_storage.md)**
- **[Hive](storage/local_storage.md)**
- **[Isar](storage/local_storage.md)**

## 性能优化
- **[性能优化总览](performance/performance_optimization.md)**
  - Widget 优化
  - 状态管理优化
  - 网络优化
  - 内存优化
- **[渲染流水线](performance/render_pipeline.md)**
  - build / layout / paint
  - compositing / raster
  - UI 卡顿 vs Raster 卡顿
  - DevTools 排查路径

## 原生交互
- **[Platform Channel](native_interaction/native_interaction.md)**
  - MethodChannel
  - EventChannel
  - BasicMessageChannel
  - PlatformView
- **[Native Interop / FFI](native_interop/native_interop.md)**
  - `dart:ffi`
  - 动态库加载
  - 类型映射
  - 内存管理
  - Rust / C / C++ 桥接

## 第三方库
- **[第三方库总览](third_party/third_party_libraries.md)**
  - 状态管理库
  - 网络库
  - UI 库
  - 工具库
- **[三方生态选型](third_party/ecosystem_selection.md)**
  - 选型维度
  - 抽象层
  - 版本与安全
- **[插件开发](third_party/plugin_development.md)**
  - package / plugin / federated plugin
  - Platform Channel
  - FFI
  - 插件设计边界

## 测试
- **[测试体系](testing/testing.md)**
  - 单元测试
  - Widget 测试
  - 集成测试
- **[测试进阶](testing/testing_advanced.md)**
  - 测试金字塔
  - mock / fake / stub
  - golden test
  - 稳定性问题
- **[质量保障](testing/quality_pipeline.md)**
  - CI
  - 覆盖率
  - 质量门禁
  - 回归策略

## 面试高频
- **[Flutter 面试高频考点](interview/flutter_interview_keypoints.md)**
  - 底层渲染原理
  - Skia 与线程模型
  - 性能优化追问
  - 状态管理选型
  - Platform Channel 与 FFI

## 部署与发布
- **[部署发布](deployment/deployment.md)**
  - Android 发布
  - iOS 发布
  - Web 发布
  - 桌面发布
- **[多平台发布进阶](deployment/multiplatform_release.md)**
  - Android / iOS / Web / Desktop 差异
  - 签名与审核
  - 包体积与渠道
- **[发布流水线](deployment/release_pipeline.md)**
  - CI/CD
  - 签名管理
  - 产物归档
  - 测试分发与正式发布

## 最佳实践
- **[项目结构](best_practices/project_structure.md)**
  - feature-first
  - core / shared / features
  - 分层与模块边界
- **[错误处理](best_practices/error_handling.md)**
  - 错误分层
  - 全局捕获
  - 崩溃上报
  - 重试与降级
- **[依赖注入](best_practices/dependency_injection.md)**
  - 构造函数注入
  - `get_it`
  - `injectable`
  - 生命周期管理
