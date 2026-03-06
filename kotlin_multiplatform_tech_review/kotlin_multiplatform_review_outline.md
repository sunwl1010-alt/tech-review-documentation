# Kotlin Multiplatform 技术复习大纲

## 核心概念
- **[Kotlin Multiplatform (KMP)](core_concepts/kotlin_multiplatform.md)**
  - 定义与目标
  - 支持平台
  - 核心组件
  - 工作原理

- **[Compose Multiplatform (CMP)](core_concepts/compose_multiplatform.md)**
  - 定义与目标
  - 支持平台
  - 核心特性
  - 与Jetpack Compose的关系

## 项目结构
- **[共享模块](project_structure/shared_module.md)**
  - 预期声明 (expect)
  - 平台实现 (actual)
  - 共享业务逻辑
  - 依赖管理

- **[平台特定模块](project_structure/platform_specific.md)**
  - Android模块
  - iOS模块
  - Web模块
  - Desktop模块

## 核心功能
- **[跨平台业务逻辑](core_features/business_logic.md)**
  - 数据模型
  - 网络请求
  - 数据存储
  - 业务规则

- **[跨平台UI](core_features/cross_platform_ui.md)**
  - Composable函数
  - 状态管理
  - 布局系统
  - 平台适配

- **[平台集成](core_features/platform_integration.md)**
  - 原生API调用
  - 平台特定功能
  - 第三方库集成
  - 权限管理

## 工具与生态
- **[开发工具](tools/development_tools.md)**
  - Android Studio
  - IntelliJ IDEA
  - Xcode
  - 构建工具

- **[生态系统](tools/ecosystem.md)**
  - 官方库
  - 第三方库
  - 社区支持
  - 学习资源

## 最佳实践
- **[项目架构](best_practices/architecture.md)**
  - 模块化设计
  - 依赖注入
  - 状态管理
  - 测试策略

- **[性能优化](best_practices/performance.md)**
  - 代码优化
  - 构建优化
  - 运行时优化
  - 平台特定优化

- **[迁移策略](best_practices/migration.md)**
  - 现有项目迁移
  - 渐进式采用
  - 代码组织
  - 测试覆盖

## 部署与发布
- **[多平台部署](deployment/multiplatform_deployment.md)**
  - Android发布
  - iOS发布
  - Web发布
  - Desktop发布

- **[CI/CD](deployment/ci_cd.md)**
  - 持续集成
  - 持续部署
  - 自动化测试
  - 版本管理

## 案例研究
- **[示例应用](case_studies/sample_apps.md)**
  - 跨平台计数器
  -  todo应用
  - 天气应用
  - 社交应用

- **[生产案例](case_studies/production_cases.md)**
  - 大型应用案例
  - 性能指标
  - 开发效率
  - 维护成本
