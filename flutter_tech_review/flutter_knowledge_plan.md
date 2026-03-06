# Flutter Knowledge Plan

## 当前扫描结果

- 已有主题：核心概念、状态管理、导航、网络、存储、性能、原生交互、三方库、测试、部署
- 空目录：`native_interop`
- 已覆盖较多“概览型”内容，但还缺少一批进阶主题和实战型内容
- 新增深度方向应重点补齐：渲染原理、Skia、线程模型、渲染流水线、面试高频题

## 优先补充的知识

### P0：先补齐主干

1. `native_interop`
   - `dart:ffi`
   - 动态库加载
   - 类型映射
   - 内存管理
   - Rust/C/C++ 桥接
2. 渲染底层
   - Skia 渲染引擎
   - Flutter Engine 分层
   - Widget / Element / RenderObject
   - Layer / Scene / Raster
   - UI / Raster / Platform 线程
2. 状态管理缺口
   - `Riverpod`
   - `Redux`
   - 状态管理选型对比
3. 路由进阶
   - Navigator 1.0 vs 2.0
   - `go_router`
   - 深链路与 URL 同步

### P1：补全工程化能力

1. 项目结构设计
   - feature-first
   - layer-first
   - clean architecture 在 Flutter 的落地
2. 异常与日志
   - 全局错误捕获
   - Crash 上报
   - 网络错误分层
3. 依赖注入
   - `get_it`
   - `injectable`
   - 模块边界设计

### P2：补全性能与渲染认知

1. 渲染流水线
   - build
   - layout
   - paint
   - compositing
   - raster
2. 性能排查工具
   - DevTools
   - Timeline
   - Rebuild 分析
   - 内存快照
3. 大列表与复杂页面优化
   - Sliver 体系
   - 图片缓存
   - isolate
4. 面试表达能力
   - 高频问题标准回答
   - 追问点拆解
   - 性能分析答题框架

### P3：补全平台化与发布能力

1. 多平台适配
   - Android
   - iOS
   - Web
   - Windows/macOS/Linux
2. CI/CD
   - 自动化构建
   - 签名管理
   - 渠道发布
3. 包与插件开发
   - package
   - plugin
   - federated plugin

## 建议新增文档清单

1. `native_interop/native_interop.md`
2. `core_concepts/rendering_engine.md`
3. `performance/render_pipeline.md`
4. `interview/flutter_interview_keypoints.md`
5. `state_management/riverpod.md`
6. `state_management/redux.md`
7. `navigation/router_advanced.md`
8. `best_practices/project_structure.md`
9. `best_practices/error_handling.md`
10. `best_practices/dependency_injection.md`
11. `deployment/ci_cd.md`
12. `third_party/plugin_development.md`

## 推荐学习顺序

1. Flutter 架构 + Widget 体系
2. 渲染引擎 + Skia + 线程模型
3. 渲染流水线与性能排查
4. 状态管理对比与选型
5. 路由与页面组织
6. 网络、存储、错误处理
7. Platform Channel
8. FFI / native interop
9. 测试、发布、CI/CD

## 写作模板建议

每篇文档尽量统一为以下结构：

1. 主题定义
2. 核心概念
3. 实现原理
4. 使用场景
5. 常见问题与解决方案
6. 代码示例
7. 面试高频问题
8. 最佳实践

## 下一步建议

1. 继续补 `Riverpod`、`Redux` 和状态管理选型文档
2. 增加 `best_practices` 目录，承接项目结构、错误处理、依赖注入
3. 逐篇把现有概览文档升级成“原理 + 场景 + 高频问答”结构
