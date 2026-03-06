# Flutter 原生交互

## 为什么需要原生交互
Flutter 自带了完整的 UI 渲染体系，但它不能直接覆盖所有平台能力。常见需要下沉到原生层的场景包括：

- 调用系统能力：相机、蓝牙、定位、推送、通知、后台任务
- 复用现有原生 SDK：地图、支付、广告、埋点、音视频
- 接入已有原生模块：公司内部 Android / iOS 基建、账号体系、风控组件
- 嵌入复杂原生视图：地图、WebView、播放器、AR 视图
- 调用本地高性能库：C / C++ / Rust 算法库，此时更适合 FFI

原生交互的核心不是“能不能调通”，而是要明确边界：

- UI 能否继续由 Flutter 渲染
- 调用是一次性命令还是持续事件流
- 是否需要跨语言高频数据交换
- 是否必须嵌入原生 View
- 是否已经有现成插件，是否值得自研

## 技术选型图

### Platform Channel
适合 Flutter 与 Android / iOS 之间的业务能力调用，特点是：

- 面向平台能力和业务 API
- 调用链路清晰，易于封装成插件
- 支持双向通信
- 数据通过消息编解码传递
- 适合中低频调用，不适合超高频大数据量交换

### PlatformView
适合在 Flutter 页面中直接嵌入原生视图，特点是：

- 解决“必须使用原生 View”的问题
- 常见于地图、WebView、视频播放器、广告组件
- 会带来渲染、手势、层级和性能成本
- 应该作为必要时使用的方案，而不是默认方案

### FFI
适合 Dart 直接调用 C / C++ / Rust 动态库，特点是：

- 绕过 Platform Channel，减少平台桥接层
- 适合算法、音视频处理、图像处理、加解密等高性能场景
- 不适合直接调用 Android / iOS SDK 业务接口
- 更偏底层能力复用，不是常规业务 API 调用手段

### 简单决策原则

- 调系统 API、原生 SDK：优先 Platform Channel
- 嵌原生控件：使用 PlatformView
- 调 C / C++ / Rust 库：优先 FFI
- 已有成熟插件：优先用插件，不要重复造轮子

## Platform Channel 原理
Platform Channel 本质上是 Flutter Engine 提供的一套跨端消息通道机制。Dart 侧通过 `BinaryMessenger` 把消息发给平台层，平台层处理后再把结果返回给 Dart。

整体链路可以理解为：

1. Dart 侧把方法名和参数编码成二进制消息
2. Engine 把消息转发给对应平台的 channel handler
3. Android / iOS 侧解析消息并执行原生逻辑
4. 平台层把结果或错误编码后返回 Dart

几个关键点：

- 通信是异步的，Dart 侧通常拿到 `Future`
- 通道名称必须全局唯一且两端一致
- 编解码默认依赖 `StandardMessageCodec` / `StandardMethodCodec`
- 数据不是共享内存，而是序列化后跨边界传递
- 高频调用会产生明显桥接成本

## 三种 Channel 的区别

### MethodChannel
最常用，适合“发起一次请求，得到一次结果”的场景。

典型场景：

- 获取设备信息
- 调起支付
- 打开系统设置页
- 请求原生能力执行后返回结果

特点：

- 语义清晰，最适合 RPC 风格调用
- Dart 侧使用 `invokeMethod`
- 原生侧按方法名分发处理
- 支持成功、失败、未实现三类结果

### EventChannel
适合平台层持续向 Flutter 推送事件流。

典型场景：

- 传感器数据
- 定位变化
- 下载进度
- 音频播放状态变化
- 原生 SDK 事件回调流

特点：

- Dart 侧对应 `Stream`
- 原生侧需要管理订阅与取消订阅
- 重点不在“能收到”，而在生命周期管理是否正确

### BasicMessageChannel
适合更通用的消息传递，不强调方法调用语义。

典型场景：

- 双向发送结构化消息
- 自定义协议通信
- 少量但较灵活的数据交换

特点：

- 双方都可以主动发消息
- 可使用自定义 codec
- 在普通业务里用得少，更多是特殊协议或框架级通信

## 数据类型与编解码
默认 `StandardMessageCodec` 支持常见类型映射。

Dart 常见类型：

- `bool`
- `int`
- `double`
- `String`
- `Uint8List`
- `List`
- `Map`
- `null`

工程上要注意：

- `Map<String, dynamic>` 在平台侧不一定天然强类型，解析时要做好判空和类型检查
- 大对象、深层嵌套对象会增加序列化成本
- 二进制数据可以传，但不要把桥接层当作大文件传输通道
- 平台侧返回错误时要带稳定的 `code`，不要只返回字符串

更稳妥的做法：

- 为每个方法定义明确的 request / response 结构
- 统一错误码枚举
- 把桥接层视为 API 层，而不是随意透传 `dynamic`

## MethodChannel 实战结构

### Dart 侧建议封装
不要在页面里直接写 `invokeMethod`，而是做一层 gateway / service 抽象。

```dart
import 'package:flutter/services.dart';

class NativeDeviceService {
  static const MethodChannel _channel = MethodChannel(
    'com.example.device/methods',
  );

  Future<DeviceInfo> getDeviceInfo() async {
    final result = await _channel.invokeMapMethod<String, dynamic>('getDeviceInfo');
    if (result == null) {
      throw const PlatformException(
        code: 'empty_result',
        message: 'Native returned empty device info.',
      );
    }
    return DeviceInfo.fromMap(result);
  }
}

class DeviceInfo {
  final String brand;
  final String model;
  final String systemVersion;

  const DeviceInfo({
    required this.brand,
    required this.model,
    required this.systemVersion,
  });

  factory DeviceInfo.fromMap(Map<String, dynamic> map) {
    return DeviceInfo(
      brand: map['brand'] as String? ?? '',
      model: map['model'] as String? ?? '',
      systemVersion: map['systemVersion'] as String? ?? '',
    );
  }
}
```

### Android 侧示意
```kotlin
class MainActivity : FlutterActivity() {
    override fun configureFlutterEngine(flutterEngine: FlutterEngine) {
        super.configureFlutterEngine(flutterEngine)

        MethodChannel(
            flutterEngine.dartExecutor.binaryMessenger,
            "com.example.device/methods"
        ).setMethodCallHandler { call, result ->
            when (call.method) {
                "getDeviceInfo" -> {
                    val data = mapOf(
                        "brand" to Build.BRAND,
                        "model" to Build.MODEL,
                        "systemVersion" to Build.VERSION.RELEASE,
                    )
                    result.success(data)
                }
                else -> result.notImplemented()
            }
        }
    }
}
```

### 错误处理原则

- 业务失败：使用 `result.error(code, message, details)`
- 未实现：使用 `notImplemented`
- 不要把平台异常直接原样丢给上层页面
- Dart 侧统一把 `PlatformException` 转为领域错误

## EventChannel 实战要点
EventChannel 真正难的点在资源管理，而不是 API 本身。

### 典型生命周期

1. Flutter 开始订阅 `Stream`
2. 原生侧开始注册监听器
3. 原生持续推送事件
4. Flutter 取消订阅
5. 原生侧必须释放监听器、回调、线程资源

### 常见问题

- `onListen` 注册了监听，但 `onCancel` 没释放，导致内存泄漏
- 页面销毁后订阅还在，继续往已销毁页面推数据
- 高频事件不做节流，桥接层被打爆
- 原生线程直接频繁回调 UI，造成卡顿

### 工程建议

- 只在真正需要时开启订阅
- 在 Dart 侧及时 `cancel`
- 在原生侧做采样、节流、去抖
- 把高频原始数据预处理后再发给 Flutter

## BasicMessageChannel 什么时候用
多数业务直接用 MethodChannel 就够了。只有以下情况才优先考虑 BasicMessageChannel：

- 你不想把通信抽象成“方法调用”
- 双端都需要主动发送消息
- 你需要自定义消息协议或 codec
- 你在做插件框架、容器化、动态化等基础设施

如果只是普通的“拿数据、执行操作、拿结果”，继续用 MethodChannel 更清晰。

## PlatformView

### PlatformView 解决什么问题
Flutter 自己绘制 UI，所以很多原生 View 不能直接塞进 Widget 树。PlatformView 就是为了解决“把原生视图嵌进 Flutter 页面”这个问题。

常见场景：

- 地图
- WebView
- 视频播放器
- 广告组件
- 系统级复杂原生控件

### 代价是什么
PlatformView 不是零成本方案，常见代价包括：

- 渲染性能下降
- 手势冲突更复杂
- 页面层级混合更难排查
- 平台差异变多
- 滚动场景下更容易卡顿

所以设计原则是：

- 能用 Flutter 自绘的，不要上 PlatformView
- 必须复用原生控件时再用
- 避免在长列表里大量嵌入 PlatformView

### 性能理解
PlatformView 会引入 Flutter 渲染层和平台视图层的协作成本。尤其在 Android 上，大量平台视图、频繁刷新、复杂手势叠加时，更容易出现掉帧。

高频面试里常见追问：为什么地图、WebView、视频播放器更容易使用 PlatformView？
核心回答是：这些控件已经有成熟的原生 View 与生态，重写成本极高，而且通常依赖平台层能力与渲染体系。

## 线程与生命周期

### 线程边界
需要明确三个层面的线程问题：

- Flutter UI isolate 负责 Dart 代码和 Widget 构建
- 平台主线程负责 Android / iOS 的 UI 相关操作
- 原生耗时任务不应阻塞平台主线程，也不应把结果同步卡在 Flutter 页面上

实践原则：

- 调平台 UI API 时，回到平台主线程
- 耗时 I/O、加密、文件、网络操作放后台线程
- 不要在 channel handler 里做重 CPU 工作
- 不要误以为用了 Platform Channel 就天然异步安全

### 生命周期边界
原生交互很容易踩生命周期坑：

- Activity / ViewController 已销毁，但异步结果还在回调
- EventChannel 页面退出后未取消订阅
- PlatformView dispose 不完整，导致资源未释放
- 插件持有 `Context` / `Activity` 强引用，导致泄漏

工程上至少做到：

- 明确 attach / detach 时机
- 所有监听器都可释放
- 页面级订阅跟随 Widget 生命周期管理
- 原生层避免长生命周期持有短生命周期对象

## Platform Channel 与 FFI 的边界
这是面试和工程里都很高频的点。

### Platform Channel 适合

- 调 Android / iOS SDK
- 调系统能力
- 调原生业务模块
- 与原生页面、原生组件协作

### FFI 适合

- 调 C / C++ / Rust 动态库
- 高频数据处理
- 图像、音视频、编解码、算法库
- 需要减少桥接层开销的底层计算能力

### 一句话区分

- Platform Channel 是 Dart 和平台层 API 之间通信
- FFI 是 Dart 和本地动态库之间直接绑定

不要把它们对立起来。很多真实项目是混用的：

- 用 Platform Channel 调系统能力
- 用 FFI 调算法库
- 再把两者封装成统一仓储或 service 给页面层使用

## 插件化设计建议
原生交互一旦进入中大型项目，最好插件化，而不是散落在业务页面里。

推荐分层：

1. Dart facade：对上层暴露稳定 API
2. Channel gateway：负责桥接和序列化
3. Platform implementation：Android / iOS 各自实现
4. Domain mapping：把平台数据转换成业务模型

这样做的收益：

- 页面层不依赖平台细节
- 错误码和数据模型可统一
- 便于 mock 和测试
- 后续可替换为三方插件或 FFI 实现

## 常见问题与解决思路

### 1. `MissingPluginException`
常见原因：

- channel 名称不一致
- 插件未正确注册
- 热重载后原生注册状态异常
- 调用时机过早

解决：

- 核对 Dart / Android / iOS 的 channel 名
- 尝试完整重启应用而不是只热重载
- 检查插件注册和 engine 配置

### 2. 数据类型不匹配
常见原因：

- Dart 传 `int`，平台按 `String` 取
- `Map` 结构变化但两端未同步
- `null` 值未做判空

解决：

- 给接口定义稳定 schema
- 两端都做显式类型校验
- 不要依赖隐式转换

### 3. 回调成功了但页面没更新
常见原因：

- 结果回来了，但状态没写入正确的状态管理层
- 页面已销毁
- 订阅已取消

解决：

- 把 channel 返回结果收敛到 service / notifier / bloc
- 检查页面生命周期和 mounted 状态

### 4. PlatformView 滚动卡顿
常见原因：

- 长列表中嵌入多个平台视图
- 视图频繁重建
- 与复杂动画、透明层叠加

解决：

- 减少数量
- 避免频繁销毁重建
- 把原生视图放到必要的关键位置，而不是全页面铺满

## 测试思路

### Dart 侧测试

- 对 facade / service 做单元测试
- mock `MethodChannel` 返回值和异常
- 验证错误映射是否正确

### 插件集成测试

- 在 Android / iOS 真机或模拟器上验证关键路径
- 至少覆盖权限、失败场景、生命周期切换
- 对 EventChannel 验证订阅与取消逻辑

### PlatformView 测试

- 重点测创建、销毁、页面切后台再恢复
- 重点测滚动、手势、旋转、重建
- 不要只测“能显示出来”

## 面试高频题

### 1. Flutter 和原生通信有哪些方式
标准回答：

- 常规业务通信主要使用 Platform Channel
- 其中包括 MethodChannel、EventChannel、BasicMessageChannel
- 如果要嵌入原生视图，用 PlatformView
- 如果要直接调用 C / C++ / Rust 动态库，用 FFI

### 2. MethodChannel、EventChannel、BasicMessageChannel 的区别
标准回答：

- MethodChannel 适合一次请求一次响应
- EventChannel 适合持续事件流
- BasicMessageChannel 适合更通用的双向消息协议
- 实战里 MethodChannel 最常用，EventChannel 次之，BasicMessageChannel 较少

### 3. Platform Channel 和 FFI 怎么选
标准回答：

- 如果目标是 Android / iOS 系统 API 或原生 SDK，就用 Platform Channel
- 如果目标是本地动态库或高性能算法库，就用 FFI
- 如果项目同时涉及平台能力和算法库，两者可以组合使用

### 4. PlatformView 为什么容易有性能问题
标准回答：

- 因为它涉及 Flutter 渲染体系和平台原生视图体系协同
- 会增加混合渲染、手势分发、层级管理成本
- 在长列表、复杂动画、高刷新场景下更容易掉帧
- 所以 PlatformView 是必要时使用，而不是默认方案

### 5. 原生交互设计时最该注意什么
标准回答：

- 明确边界：到底是系统能力、原生视图还是底层动态库
- 做好生命周期管理和资源释放
- 控制桥接频率，避免高频大数据传输
- 用统一错误码和数据模型封装通道层
- 尽量插件化，不让页面直接依赖平台细节

## 面试答题模板

### 题目：Flutter 如何调用原生能力
可以按这个结构回答：

1. 先说明主方案是 Platform Channel
2. 再区分 MethodChannel、EventChannel、BasicMessageChannel 的适用场景
3. 如果涉及原生 View，补充 PlatformView
4. 如果涉及 C / C++ / Rust 库，补充 FFI
5. 最后加一句工程化建议：封装、错误处理、生命周期管理

### 题目：你在项目里怎么做原生桥接设计
可以按这个结构回答：

1. 页面层不直接调用 channel
2. service / repository 层做统一封装
3. request / response 模型固定化
4. 平台异常统一转换为业务异常
5. 高频数据场景尽量减少桥接次数，必要时改为 FFI 或原生预处理

## 复习重点

- 理解 Platform Channel 的消息桥接本质
- 记住三种 Channel 的适用场景
- 说清楚 PlatformView 的代价
- 能区分 Platform Channel 和 FFI
- 能从线程、生命周期、性能、错误处理四个角度回答工程问题
