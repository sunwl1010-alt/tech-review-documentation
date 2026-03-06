# Android 学习路线图（初中高）

## 初级（能独立开发页面）
- 组件基础：Activity/Fragment/Service
- UI 基础：布局、RecyclerView、状态管理
- 数据基础：Room/网络请求
- 工程基础：日志、异常处理、基础测试

## 中级（能负责模块）
- 架构：MVVM + Repository + UseCase
- 并发：协程、Flow、生命周期收集
- 性能：启动、卡顿、内存、ANR
- 安全：权限、网络、存储、组件暴露

## 高级（能负责系统设计）
- 架构演进：遗留迁移与模块化治理
- 稳定性：发布门禁、回归体系、线上可观测
- 平台策略：多端协同、长期演进、技术债治理
- 团队能力：规范、评审、最佳实践沉淀

## 技术决策树（简版）

- 后台任务是否要求实时用户感知？
  - 是：Foreground Service
  - 否：WorkManager

- 页面状态复杂且多异步来源？
  - 是：MVVM + StateFlow（必要时 MVI）
  - 否：轻量 MVVM

- 本地存储是结构化业务数据？
  - 是：Room
  - 否：键值配置可用 DataStore/SharedPreferences

- 列表性能瓶颈在 bind/绘制？
  - bind：减少计算、DiffUtil、稳定 ID
  - 绘制：降层级、减少过度绘制

## 建议学习顺序
1. 先打牢组件 + 网络 + 数据基础
2. 再掌握并发 + 架构 + 测试
3. 最后做性能、安全、架构演进实战
