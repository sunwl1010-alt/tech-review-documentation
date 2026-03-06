# MVC（历史模式与迁移指南）

## 1. 在 Android 中的真实形态

Android 里的 MVC 常退化为 `Activity = Controller + View`，最终出现 Massive Activity：
- UI、状态、网络、数据库逻辑耦合
- 测试困难，需求变更成本高
- 生命周期问题频发

## 2. MVC 的可取之处

- 上手快，适合小型 PoC
- 结构直观，学习门槛低
- 对超小页面改动成本低

## 3. 主要问题

- 职责边界不清
- 控制器臃肿
- 可复用性差
- 难以并行开发

## 4. 遗留项目迁移策略（MVC -> MVVM/Clean）

### 阶段 1：提取数据边界
- 把网络/数据库代码从 Activity 提取到 Repository
- 引入统一错误模型

### 阶段 2：提取状态边界
- 增加 ViewModel 管理 UIState
- Activity 只做渲染与事件上报

### 阶段 3：提取业务边界
- 引入 UseCase
- 将复杂业务判断从 UI 层迁走

### 阶段 4：治理规范
- 代码评审加入边界检查
- 新需求禁止回流到 Activity 业务逻辑

## 5. 风险控制

- 不做全量重写，按 feature 渐进迁移
- 每次迁移配套回归用例
- 指标驱动（崩溃率、迭代效率、缺陷率）验证收益

## 6. 面试深问

- 为什么 Android MVC 容易变成 Massive Activity？
- 如何在不影响业务节奏下完成 MVC 迁移？
