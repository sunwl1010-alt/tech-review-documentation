# DataStore vs SharedPreferences 对照与迁移清单

## 1. 一句话结论

- 新项目配置存储优先 DataStore
- 存量项目可阶段性保留 SharedPreferences 并迁移
- 结构化配置优先 Proto DataStore

## 2. 能力对照

| 维度 | SharedPreferences | DataStore |
|---|---|---|
| 读写模型 | 同步 API（易阻塞主线程） | 异步 + Flow |
| 一致性 | `apply/commit` 容易并发覆盖 | `edit` 事务更新 |
| 可观察性 | 依赖监听器，能力有限 | 天然 `Flow` |
| 错误处理 | 异常处理薄弱 | 明确异常链路 |
| 结构化能力 | 键值散乱 | Preferences/Proto 可选 |
| 迁移成本 | 低（已广泛使用） | 中（需改读写模型） |

## 3. 选型建议

- 轻量开关、主题、引导页标记：Preferences DataStore
- 多字段结构化配置：Proto DataStore
- 高并发配置读写：DataStore 优先
- 历史遗留模块短期不可动：保留 SharedPreferences，先做双读迁移

## 4. 迁移策略（可落地）

### 阶段 1：双读
- 读路径优先 DataStore
- 未命中时回退 SharedPreferences

### 阶段 2：回填
- 读到旧值后写入 DataStore
- 记录迁移版本标识，避免重复回填

### 阶段 3：收敛
- 全量完成后移除 SharedPreferences 读写

## 5. 常见坑

- 在主线程频繁收集 DataStore Flow
- 把业务主数据放 DataStore（应放 Room）
- 忘记处理异常导致流中断

## 6. 面试高频追问

### Q1：DataStore 能完全替代 SharedPreferences 吗？
答：配置类场景基本可以，但迁移要分阶段，避免一次性重构风险。

### Q2：为什么 DataStore 更安全？
答：异步事务更新和明确的异常处理链路，减少并发覆盖和主线程阻塞风险。

## 7. 30 秒答题模板

- 结论：新项目用 DataStore，存量项目双读迁移。
- 依据：异步、事务一致性、Flow 可观察。
- 落地：先双读回填，再逐步下线 SharedPreferences。
