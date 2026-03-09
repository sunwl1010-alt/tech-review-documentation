# 网络请求

## 为什么网络层不能只会发请求
Flutter 里“能发 GET / POST”只是起点，真正决定工程质量的是网络层设计是否稳定、可观测、可扩展。

很多项目的网络问题不是不会调用 API，而是这些问题没处理好：

- token 过期后请求雪崩
- 页面退出后请求还在跑，结果回写错页面
- 错误全都变成一个 `Exception`
- JSON 解析散落在页面层
- 重试、超时、缓存、降级没有统一策略
- WebSocket 和普通 HTTP 逻辑混在一起

所以网络请求这一块，应该从“调用库”提升到“客户端架构”。

## Flutter 网络层常见技术栈

### `http`
Dart 官方生态里最基础的 HTTP 库。

适合：

- 简单请求
- 小型项目
- 脚本或工具型代码
- 对拦截器、取消、复杂配置要求不高的场景

特点：

- 轻量
- 易上手
- 功能相对基础
- 工程化能力不如 Dio

### Dio
Flutter 项目里最常见的网络请求库之一。

适合：

- 中大型项目
- 需要拦截器、超时、取消、上传下载进度、统一错误处理
- 需要把认证、日志、重试、刷新 token 等能力收敛在网络层

特点：

- 配置能力强
- 拦截器完善
- 支持 `CancelToken`
- 更适合工程化网络架构

### Retrofit
Retrofit 不是替代 Dio，而是基于 Dio 的接口声明式封装。

适合：

- API 数量多
- 希望通过注解声明接口
- 愿意接受代码生成成本

注意：

- Retrofit 解决的是接口定义样板问题
- 它不替代错误处理、缓存策略、认证策略这些架构问题

### WebSocket
适合长连接、服务端主动推送场景。

适合：

- 聊天
- 实时行情
- 推送型业务状态
- 协同编辑或实时事件系统

不要把 WebSocket 和普通 HTTP 请求看成一个问题，它们的连接管理、重连策略、状态同步方式都不同。

## 一个合理的网络层应该长什么样
推荐最小分层：

1. API client：封装 Dio / http 实例
2. data source：按业务域发起请求
3. repository：组合远端数据、本地缓存和错误映射
4. domain / state layer：把结果变成页面可消费状态

简化理解：

- client 负责“怎么发”
- data source 负责“调哪个接口”
- repository 负责“怎么组合和兜底”
- 页面层只关心“结果是什么”

页面不应该直接操作 Dio，也不应该直接解析 JSON。

## `http` 的定位
如果项目很轻，`http` 完全够用。

```dart
import 'dart:convert';
import 'package:http/http.dart' as http;

Future<UserDto> fetchUser() async {
  final response = await http.get(
    Uri.parse('https://api.example.com/users/me'),
  );

  if (response.statusCode != 200) {
    throw HttpException('Unexpected status: ${response.statusCode}');
  }

  return UserDto.fromJson(jsonDecode(response.body) as Map<String, dynamic>);
}
```

但要明确：

- 它更偏“发请求工具”
- 很多工程能力需要自己补
- 一旦项目变复杂，通常会转向 Dio

## Dio 为什么更适合实战
Dio 在 Flutter 项目里常用，不是因为它“更高级”，而是因为它更适合收敛横切能力。

### 基础初始化
```dart
import 'package:dio/dio.dart';

class ApiClient {
  late final Dio dio;

  ApiClient() {
    dio = Dio(
      BaseOptions(
        baseUrl: 'https://api.example.com',
        connectTimeout: const Duration(seconds: 10),
        receiveTimeout: const Duration(seconds: 10),
        sendTimeout: const Duration(seconds: 10),
        headers: {
          'Accept': 'application/json',
        },
      ),
    );
  }
}
```

### 为什么要统一初始化
统一初始化能把这些能力集中在一处：

- `baseUrl`
- 超时
- 公共 header
- 拦截器
- 日志
- token 注入
- 统一错误转换

这样页面和业务层不会感知这些细节。

## 拦截器
拦截器是 Dio 最大的工程价值之一。

### 常见用途

- 请求前注入 token
- 统一打印日志
- 响应后处理业务码
- 错误转换
- 401 刷新 token
- 请求埋点和耗时统计

### 示例
```dart
dio.interceptors.add(
  InterceptorsWrapper(
    onRequest: (options, handler) async {
      options.headers['Authorization'] = 'Bearer token';
      handler.next(options);
    },
    onResponse: (response, handler) {
      handler.next(response);
    },
    onError: (error, handler) {
      handler.next(error);
    },
  ),
);
```

### 工程注意点

- 不要在拦截器里堆太多业务逻辑
- 认证、日志、错误转换分开更清晰
- 拦截器是横切层，不是业务编排层

## 错误分层
网络层最常见的问题之一，是所有失败都被写成 `catch (e) {}`。

更合理的做法是按层次处理错误。

### 常见错误类别

- 连接失败：无网、DNS、握手失败
- 超时：连接超时、接收超时、发送超时
- 取消：页面退出、用户主动取消
- HTTP 错误：401、403、404、500
- 业务错误：HTTP 200，但业务码失败
- 解析错误：JSON 结构不符合预期

### 建议做法
定义统一异常模型，例如：

```dart
sealed class AppException implements Exception {
  final String message;
  const AppException(this.message);
}

class NetworkException extends AppException {
  const NetworkException(super.message);
}

class UnauthorizedException extends AppException {
  const UnauthorizedException(super.message);
}

class ServerException extends AppException {
  const ServerException(super.message);
}

class ParseException extends AppException {
  const ParseException(super.message);
}
```

然后把 Dio 的错误集中映射成自己的错误体系，而不是让页面直接理解 `DioException`。

## token 刷新
这是面试和项目里都非常高频的点。

### 典型流程

1. 请求发出时带 access token
2. 服务端返回 401
3. 客户端尝试用 refresh token 换新 access token
4. 刷新成功后重放原请求
5. 刷新失败则清理登录态并跳登录页

### 真正难点

- 多个请求同时 401，不能每个都去刷新一次
- 刷新期间后续请求要排队或等待
- 刷新失败要避免死循环
- 登录态清理要统一，不要多个页面各自处理

### 面试标准回答点

- token 刷新不能简单写在每个请求里
- 应该在拦截器或认证层统一控制
- 需要并发保护，避免重复刷新
- 刷新成功后要重放原请求

## 超时、取消与重试

### 超时
超时是基础保护，不是可选项。

至少要考虑：

- connect timeout
- receive timeout
- send timeout

不要无限等待，否则网络问题会直接拖垮用户体验。

### 取消请求
在 Flutter 页面里，取消能力非常重要。

典型场景：

- 页面销毁后取消未完成请求
- 搜索联想时取消旧请求
- 用户主动终止上传下载

```dart
final cancelToken = CancelToken();

await dio.get(
  '/search',
  cancelToken: cancelToken,
);

cancelToken.cancel('page disposed');
```

### 重试
重试要有边界，不能“失败就一直重试”。

适合重试的场景：

- 短暂网络波动
- 幂等请求失败，例如 GET
- 某些可恢复的 5xx 错误

不适合盲目重试的场景：

- 非幂等写操作
- 权限错误
- 参数错误
- 明确业务失败

工程建议：

- 限制次数
- 做退避策略
- 只对明确可重试错误生效

## JSON 解析与模型转换
页面层不应该直接 `jsonDecode` 后到处取 key。

推荐做法：

1. 网络层拿到原始 JSON
2. DTO 层负责解析
3. repository 或 mapper 转成业务模型
4. 页面层只消费强类型对象

```dart
class UserDto {
  final int id;
  final String name;

  const UserDto({required this.id, required this.name});

  factory UserDto.fromJson(Map<String, dynamic> json) {
    return UserDto(
      id: json['id'] as int? ?? 0,
      name: json['name'] as String? ?? '',
    );
  }
}
```

这比在页面里写 `map['data']['user']['name']` 稳定得多。

## Retrofit 的价值和边界

### 价值

- 用注解声明接口
- 减少样板代码
- 和 Dio 配合自然
- API 数量多时可维护性更好

### 边界

- 不能代替错误分层
- 不能自动解决缓存策略
- 不能代替 repository
- 会引入代码生成和维护成本

适合团队协作明确、接口数量多的项目，不一定适合所有场景。

## 上传下载
Dio 对上传下载支持比较好。

常见场景：

- 图片上传
- 文件下载
- 下载进度展示
- 断点续传扩展

关键点：

- 文件上传注意超时和取消
- 下载大文件注意保存路径和权限
- 进度回调不要直接把高频事件全量打到 UI 层

## WebSocket
WebSocket 解决的是“长连接下服务端主动推送”的问题。

### 使用场景

- 聊天消息
- 实时推送
- 交易行情
- 在线协同状态

### 需要考虑的问题

- 连接建立与关闭时机
- 前后台切换策略
- 心跳保活
- 自动重连
- 去重与顺序问题
- 与本地状态同步

### 工程边界

- WebSocket 连接管理最好独立成专门 service
- 不要让页面直接管理连接生命周期
- HTTP 负责请求响应，WebSocket 负责实时推送，两者职责不同

## 缓存与离线
网络层不该只想着请求成功时怎么展示，还要考虑请求失败时怎么兜底。

常见策略：

- memory cache：短期内快速复用
- disk cache：接口数据落地缓存
- stale-while-revalidate：先给旧数据，再后台刷新
- 离线兜底：无网时读本地缓存

缓存不是网络库自动帮你完成的，往往要和 repository、本地存储一起设计。

## 安全性
网络层至少要关注这些点：

- 使用 HTTPS
- token 和敏感信息安全存储
- 不在日志里打印敏感 header
- 对关键接口考虑签名或设备校验
- WebView / H5 混合场景注意 Cookie 和鉴权一致性

如果是高安全场景，还会涉及证书校验、抓包防护、风控参数等。

## 常见问题与解决思路

### 1. 页面退出后请求结果还回来了
原因：

- 请求没有取消
- 状态回写没有判断页面是否仍然有效

解决：

- 页面销毁时取消请求
- 结果不要直接写页面，统一回写到状态层

### 2. token 过期后大量接口同时失败
原因：

- 每个请求各自刷新 token
- 没有并发保护

解决：

- 统一在认证层刷新
- 做串行化或队列等待
- 刷新失败直接统一登出

### 3. 明明 HTTP 200，但业务还是失败
原因：

- 把 HTTP 成功当成业务成功

解决：

- 区分 HTTP 状态码和业务码
- 统一做响应体业务码判断

### 4. 页面里到处是 `try/catch`
原因：

- 错误没有在网络层和 repository 层收敛

解决：

- 底层统一转异常模型
- 页面层只处理少量用户可见状态

## 面试高频题

### 1. Flutter 项目里为什么很多人用 Dio 而不是 `http`
标准回答：

- `http` 更轻，但功能偏基础
- Dio 支持拦截器、取消、超时、上传下载、统一配置
- 中大型项目更需要这些工程化能力
- 所以 Dio 更适合作为主网络客户端

### 2. 网络层设计时最重要的点有哪些
标准回答：

- 统一 client 初始化
- 错误分层和异常映射
- token 注入与刷新
- 请求取消、超时、重试边界
- JSON 解析和模型转换不要落到页面层

### 3. token 刷新为什么容易出问题
标准回答：

- 因为它涉及并发请求、登录态一致性和原请求重放
- 多个请求同时 401 时，不能重复刷新
- 刷新失败还要避免死循环
- 所以应该统一在认证层或拦截器层处理

### 4. Retrofit 的价值是什么
标准回答：

- 它能让接口定义更声明式
- 减少模板代码
- 提升 API 数量多时的可维护性
- 但它只是接口封装工具，不解决网络架构本身的问题

### 5. WebSocket 和 HTTP 的区别是什么
标准回答：

- HTTP 更适合请求响应模型
- WebSocket 适合长连接和服务端主动推送
- WebSocket 需要额外管理连接状态、心跳、重连和同步问题
- 两者往往会在一个项目里并存

## 面试答题模板

### 题目：你在 Flutter 项目里怎么设计网络层
可以按这个结构回答：

1. 用 Dio 作为统一 client
2. 通过拦截器处理 token、日志、错误转换
3. data source 专注接口调用
4. repository 负责缓存、容错和模型转换
5. 页面层只消费结果状态，不直接依赖网络库

### 题目：网络请求这块你做过哪些工程化处理
可以按这个结构回答：

1. 统一超时和 header
2. 支持取消请求
3. 做 token 刷新和并发保护
4. 把异常映射成业务可理解错误
5. 对重要接口补缓存、重试和降级策略

## 复习重点

- 说清 `http`、Dio、Retrofit、WebSocket 的边界
- 理解拦截器的价值在于横切能力收敛
- 区分 HTTP 成功和业务成功
- 理解 token 刷新、取消请求、重试策略的工程难点
- 能把网络层、repository、页面层职责说清楚
