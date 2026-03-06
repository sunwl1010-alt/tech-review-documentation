# Android 1 页速记卡（冲刺版）

## 1. Activity
关键结论：Activity 只做生命周期边界与 UI 容器，不承载业务编排。  
易错点：`onCreate` 做重初始化、任务栈 flag 混用、状态恢复缺失。  
30 秒答题：职责收敛到渲染和导航；状态分层用 `savedState + ViewModel + 持久层`；复杂业务下沉 UseCase。

## 2. Service / WorkManager
关键结论：能被系统调度就不用常驻 Service。  
易错点：前台服务滥用、通知不合规、`onStartCommand` 阻塞主线程。  
30 秒答题：实时可感知任务用 Foreground Service；可靠延迟任务用 WorkManager；短任务用协程。

## 3. BroadcastReceiver
关键结论：Receiver 是轻入口，不是任务执行器。  
易错点：`onReceive` 做耗时任务、忘记反注册、广播暴露过宽。  
30 秒答题：`onReceive` 只路由事件，重任务转 WorkManager；动态注册优先，敏感广播加权限与校验。

## 4. ContentProvider
关键结论：Provider 主要用于跨进程共享协议，不是应用内 DAO 替代。  
易错点：错误导出、URI 粒度过粗、selection 字符串拼接注入。  
30 秒答题：先定义 URI 与权限模型，再做事务与 `notifyChange`，并用参数化查询防注入。

## 5. MVVM / Clean Architecture
关键结论：稳定依赖方向比“模式名词”更重要。  
易错点：ViewModel 透出可变状态、UI 直接依赖 DTO/Entity、Repository 只是转发层。  
30 秒答题：View 仅渲染，ViewModel 管状态机，UseCase 表达业务语义，Repository 负责数据策略与错误归一化。

## 6. 协程与并发
关键结论：结构化并发是稳定性的基础。  
易错点：吞 `CancellationException`、无生命周期收集、共享可变状态竞态。  
30 秒答题：按任务类型选 Dispatcher；`repeatOnLifecycle` 收集流；并发场景用单写原则与互斥保护。

## 7. 网络层
关键结论：网络层核心是可恢复与可观测，不是请求能发通。  
易错点：错误模型分散、重试无退避、token 刷新并发风暴。  
30 秒答题：统一 `ApiResult`，幂等接口指数退避重试，401 单飞刷新，打通 traceId 与耗时埋点。

## 8. Room / SQLite
关键结论：数据一致性优先于“写起来快”。  
易错点：线上 destructive migration、事务边界错误、索引随意加。  
30 秒答题：迁移脚本必须可测试；领域一致性操作放同事务；索引靠查询计划验证。

## 9. 性能优化
关键结论：先基线再优化，指标驱动而非体感驱动。  
易错点：不做 trace 就改代码、把所有初始化塞启动路径、只看均值不看分位数。  
30 秒答题：围绕启动/渲染/内存/ANR 建预算；定位瓶颈后单变量优化；benchmark + 线上监控双验证。

## 10. 安全
关键结论：安全是工程体系，不是单点“加密/混淆”。  
易错点：组件默认导出、明文敏感数据、WebView bridge 暴露。  
30 秒答题：权限最小化、存储分级加密、网络禁明文+证书策略、发布设置安全门禁。

## 11. 高频追问速答模板
- 先结论：明确给出取舍方案。
- 再依据：给 2~3 个技术理由。
- 再落地：说明上线做法与指标收益。
- 再边界：说明风险与替代方案。

## 12. 复习节奏建议（7天）
- Day1-2：组件 + 架构
- Day3：并发 + 网络
- Day4：数据层 + 迁移
- Day5：性能 + 安全
- Day6：面试深问模拟（60-90秒）
- Day7：错题回看 + 速记卡复述
