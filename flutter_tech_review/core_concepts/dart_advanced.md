# Dart 进阶：异步、事件循环与并发

## 一、为什么 Dart 进阶值得单独学

Flutter 的很多中高级问题，表面看是 UI 或状态管理问题，底层其实都绕不开 Dart 运行模型。

高频面试题通常会落在这些点：

1. `Future` 和 `Stream` 的区别
2. event loop 是怎么工作的
3. microtask 和 event queue 的区别
4. `async/await` 的本质是什么
5. isolate 和线程是什么关系
6. 为什么重计算会卡 UI

如果这些点答不清，Flutter 性能、异步、状态管理很多题都会答浅。

## 二、Dart 是单线程吗

这是一个非常高频但很容易答错的问题。

更准确的回答是：

- 一个 isolate 内部是单线程执行模型
- 但 Dart 可以通过多个 isolate 实现并发
- isolate 之间不共享堆内存，通过消息通信

所以不要简单说“Dart 就是单线程语言”，这不严谨。

## 三、event loop 是什么

Dart 运行异步任务依赖 event loop。

你可以把它理解成：

- 同步代码先执行
- 异步任务被放入不同队列
- 当前调用栈清空后，event loop 按规则取任务执行

这个模型决定了很多执行顺序问题。

## 四、event queue 和 microtask queue

这是面试非常爱问的点。

### event queue

通常放这些任务：

- I/O
- Timer
- UI 事件
- 外部事件回调

### microtask queue

优先级高于 event queue。

常见来源：

- `scheduleMicrotask`
- 某些 `Future` 回调调度

### 核心规则

当前同步代码执行完后，会先清空 microtask queue，再处理 event queue。

## 五、为什么 microtask 容易被追问

因为它直接影响执行顺序。

例如很多人只知道“Future 是异步”，但不知道：

- 为什么有的回调比 `Timer` 更早执行
- 为什么滥用 microtask 会影响 UI 响应

### 面试里的稳妥回答

microtask 优先级高于普通事件队列，适合非常短、需要尽快执行的后续逻辑，但不适合堆积大量任务，否则会阻塞后续事件处理。

## 六、`Future` 的本质

`Future` 代表“一次性异步结果”。

它最终只有三种状态：

1. 未完成
2. 完成并返回值
3. 完成并返回错误

### 适合场景

- 网络请求
- 读文件
- 一次性计算结果
- 初始化任务

### 高频误区

不要把 `Future` 理解成“新线程”。

`Future` 只是异步结果抽象，不代表底层一定开线程。

## 七、`async/await` 的本质

`async/await` 本质是语法糖。

它让异步代码写起来像同步流程，但底层仍然建立在 `Future` 之上。

### 好处

- 可读性高
- 错误处理更自然
- 逻辑链更容易维护

### 面试不要答成

“用了 await 就同步了”

正确说法是：

`await` 只是暂停当前 async 函数的后续执行，把控制权还给 event loop，并在 Future 完成后恢复执行。

## 八、`Future` 和 `Stream` 的区别

### `Future`

- 一次返回一个结果
- 只完成一次

### `Stream`

- 可以持续返回多个结果
- 支持订阅、取消订阅、完成、错误

### 常见场景

`Future`：

- 拉一次接口
- 读一次配置

`Stream`：

- WebSocket 消息
- 位置更新
- 输入事件流
- 数据库监听

## 九、单订阅 Stream 和广播 Stream

### 单订阅 Stream

- 默认更常见
- 一个 Stream 只能有一个 listener
- 更适合线性消费数据流

### 广播 Stream

- 支持多个 listener
- 更适合事件分发场景

### 高频坑点

很多异常都来自：

- 重复监听单订阅 Stream
- 没有取消订阅
- 页面销毁后仍保留监听

## 十、错误处理为什么重要

Dart 异步错误如果不处理，很容易变成难排查问题。

### `Future` 错误

可以通过：

- `try/catch`
- `catchError`

### `Stream` 错误

可以通过：

- `listen(onError: ...)`
- 流内部错误处理

### 面试关注点

不是“会不会 catch”，而是你是否知道：

- 同步异常和异步异常的边界不同
- 不同异步源的错误处理方式不同

## 十一、isolate 是什么

isolate 是 Dart 的并发执行单元。

### 核心特点

1. 每个 isolate 有独立内存
2. 不共享堆对象
3. 通过消息传递通信

### 为什么这样设计

因为这样可以避免传统共享内存并发里的锁竞争和数据竞态问题。

### 代价

- 通信成本更高
- 数据传输有拷贝或受限传递成本
- 不是所有任务都值得起 isolate

## 十二、什么时候该用 isolate

适合：

- 大 JSON 解析
- 图片处理
- 加密压缩
- 大量排序、聚合、计算

不适合：

- 很轻的小任务
- 高频小颗粒任务
- 强依赖 UI 上下文的逻辑

### 高频判断标准

如果任务明显占用主 isolate，导致掉帧或交互卡顿，就应该考虑搬离主 isolate。

## 十三、为什么重计算会卡 Flutter

因为 Flutter UI 的 Dart 逻辑通常运行在主 isolate 上。

如果你在主 isolate 上做：

- 大对象 JSON decode
- 复杂排序
- 图片像素处理
- 大量同步循环

就会占用 UI Thread 对应的执行时机，导致：

- 动画掉帧
- 页面卡顿
- 手势响应延迟

## 十四、`compute` 的定位

在 Flutter 中，`compute` 是一个方便把函数丢到后台 isolate 处理的简化工具。

适合：

- 单次、纯函数型、数据可序列化的重任务

不适合：

- 频繁调用的小任务
- 复杂双向长期通信

## 十五、内存模型要点

Dart 垃圾回收会自动管理内存，但这不代表你不用关心内存问题。

### 常见内存风险

1. Stream 未取消订阅
2. Controller 未释放
3. 大对象长期持有
4. 缓存策略失控
5. isolate 通信传递大对象成本高

### 高频误区

“有 GC 就不会内存泄漏”是错误的。

只要对象仍被引用，GC 就不会回收。

## 十六、面试高频问答

### 1. Dart 是单线程吗？

单个 isolate 内是单线程执行模型，但 Dart 可以通过多个 isolate 实现并发，所以不能简单粗暴地说 Dart 只是单线程。

### 2. microtask 和 event queue 的区别是什么？

microtask 优先级更高，当前同步代码执行完后会先清空 microtask，再处理 event queue。

### 3. `Future` 和 `Stream` 的区别是什么？

`Future` 表示一次性异步结果，`Stream` 表示持续多个异步事件。

### 4. `async/await` 的本质是什么？

它是基于 `Future` 的语法糖，让异步流程更接近同步写法，但不会把异步变成真正同步阻塞。

### 5. isolate 和线程的关系是什么？

isolate 是 Dart 的并发模型单位，不等同于传统线程；它有独立内存，通过消息通信。

### 6. 什么时候应该把任务放到 isolate？

当任务明显重 CPU、会阻塞主 isolate，影响 UI 流畅度时。

## 十七、推荐回答模板

如果面试官问“Dart 异步和并发模型怎么理解”，更完整的回答方式是：

1. 先说明 Dart 在单个 isolate 内是事件循环驱动的单线程模型
2. 再解释同步代码、microtask、event queue 的执行顺序
3. 说明 `Future` 是一次性结果，`Stream` 是持续事件流
4. 最后说明重 CPU 任务要用 isolate 隔离，避免阻塞 Flutter 主 UI 执行

这样回答，能把 Dart 基础、Flutter 性能和工程实践串起来。
