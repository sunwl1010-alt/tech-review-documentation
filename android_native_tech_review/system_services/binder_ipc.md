# Binder / IPC 面试专题（Android）

## 1. Binder 是什么，为什么它是 Android IPC 主力

Binder 是 Android 的跨进程通信机制，核心由四部分组成：
- Client：发起调用的一方
- Server：提供服务的一方
- ServiceManager：服务注册与查询中心
- Binder Driver（内核驱动）：负责跨进程事务传递

为什么不用 Socket 作为系统主 IPC：
- Binder 有明确的服务管理模型（按服务名查找）
- Binder 支持对象语义与权限边界
- Binder 在 Android 系统服务体系中有更低的调用成本和更好的一致性

## 2. 调用链路（面试高频）

典型链路：
1. Server 通过 `addService` 注册到 ServiceManager
2. Client 通过 `getService` 获取远程 Binder 代理（Proxy）
3. Client 调用代理方法
4. 代理把参数写入 `Parcel`，通过 Binder Driver 发起事务
5. Server 端 `Stub` 解包并执行
6. 结果回写 `Parcel` 返回 Client

关键词：`Proxy/Stub/Parcel/transact/onTransact`。

## 3. AIDL 核心点

### 3.1 方向标记
- `in`：仅客户端 -> 服务端
- `out`：仅服务端 -> 客户端
- `inout`：双向（有性能成本）

实践建议：
- 默认优先 `in`
- 谨慎使用 `inout`，会增加拷贝与语义复杂度

### 3.2 oneway
`oneway` 表示异步单向调用，客户端不等待返回。适合通知类调用，不适合需要强一致结果的场景。

### 3.3 版本兼容
- 新增接口方法时保持向后兼容
- 尽量避免修改已有方法语义
- 使用稳定的数据结构并预留字段

## 4. Binder 线程模型与并发风险

服务端方法默认跑在 Binder 线程池，不是主线程。常见风险：
- 线程安全问题（共享状态无保护）
- 死锁（Binder 回调链互相等待）
- 阻塞 Binder 线程导致系统调用堆积

建议：
- Binder 方法只做轻逻辑，重任务切到业务线程池
- 严禁在 Binder 线程做长时间 I/O
- 对共享状态加锁或单写模型

## 5. linkToDeath / DeathRecipient

远程进程死亡后，客户端可以通过 `linkToDeath` 感知并重连。

用途：
- 重建连接
- 清理失效资源
- 快速失败而不是长时间等待

面试答点：这属于“进程级可用性治理”，不是业务异常处理替代。

## 6. 传输限制与 TransactionTooLargeException

Binder 不适合传大对象，典型问题是 `TransactionTooLargeException`。

规避策略：
- 传 ID，不传大对象内容
- 分页 / 分块传输
- 大数据走文件、数据库或共享内存
- Activity 状态保存不要塞大 Bundle

## 7. Binder 与其他 IPC 取舍

- Binder/AIDL：强类型接口、系统服务式调用、双向能力
- Messenger：基于消息，简单但能力受限
- ContentProvider：结构化数据共享
- Socket：跨平台或长连接协议场景

一句话：Android 系统内“服务调用型 IPC”优先 Binder。

## 8. 安全与权限

- 导出服务默认关闭
- 必要导出时加 `signature` 级权限
- 服务端做调用方 UID/包名校验
- 入参严格校验，防止越权与注入

## 9. 常见面试追问

### Q1：Binder 为什么比 Socket 更适合 Android 系统服务？
答：Binder 有 ServiceManager、对象语义、权限边界和成熟的系统调用链；Socket 更偏通用传输层，不直接提供服务治理语义。

### Q2：AIDL 的 `in/out/inout` 有什么区别？
答：本质是参数拷贝方向和可见性。`in` 最常用、语义最清晰；`inout` 成本高，应尽量避免。

### Q3：如何避免 Binder 调用导致 ANR？
答：避免主线程同步等待远程重调用；限制单次调用耗时；服务端重任务下沉线程池；客户端设置超时和降级策略。

## 10. 可复用答题模板（60秒）

- 结论：Binder 是 Android 服务调用型 IPC 主力。
- 依据：ServiceManager + Driver + Proxy/Stub 架构；强类型接口；权限边界。
- 落地：AIDL 只传轻量参数，重任务异步化，挂 `linkToDeath` 做重连。
- 边界：大对象传输和复杂流式数据不用 Binder，改文件/数据库/Socket。
