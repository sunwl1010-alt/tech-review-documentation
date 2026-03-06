# Flutter Native Interop

## 主题定位

`native_interop` 关注的是 Flutter 与原生库的直接互操作，重点是 `dart:ffi`、C ABI、动态库加载、内存管理，以及与 Rust/C/C++ 的桥接。

这部分和 `native_interaction/native_interaction.md` 的区别：

- `native_interaction`：偏 Platform Channel，与 Android/iOS 平台代码通信。
- `native_interop`：偏 FFI，直接调用 `.so`、`.dll`、`.dylib` 等原生动态库。

## 核心概念

### FFI 是什么
- `FFI`（Foreign Function Interface）用于让 Dart 直接调用原生语言导出的函数。
- 常见目标语言：`C`、`C++`（通常通过 C ABI 暴露）、`Rust`（通常通过 `extern "C"` 暴露）。
- 适用场景：高性能计算、复用已有原生库、音视频编解码、加密、图像处理、设备 SDK 接入。

### FFI 与 Platform Channel 的区别

| 维度 | FFI | Platform Channel |
| --- | --- | --- |
| 调用对象 | 动态库函数 | Android/iOS 平台代码 |
| 性能 | 更高，少一层消息传递 | 较灵活，但有序列化开销 |
| 适用场景 | 算法库、跨平台原生核心 | 平台能力、系统 API、生命周期 |
| 开发复杂度 | 更高，需要处理 ABI/内存 | 相对更易上手 |

### 动态库与 ABI
- Windows 常见：`.dll`
- Android 常见：`.so`
- iOS/macOS 常见：`.dylib` / Framework
- 关键前提：Dart 侧声明必须和原生导出函数签名严格匹配。

### 内存管理
- Dart 管 Dart 堆内存，FFI 调用时常需要手动申请和释放原生内存。
- 常用 API：`calloc`、`malloc`、`free`
- 风险点：内存泄漏、悬垂指针、越界访问、重复释放。

## 基础使用流程

1. 准备原生动态库，并导出 C ABI 接口。
2. 在 Flutter 中通过 `DynamicLibrary.open()` 或 `DynamicLibrary.process()` 加载库。
3. 使用 `lookup<NativeFunction<...>>()` 找到符号。
4. 用 `asFunction()` 转成 Dart 可调用函数。
5. 做参数转换、错误处理、资源释放。

## Dart 侧示例

```dart
import 'dart:ffi';
import 'dart:io';

typedef NativeAddFunc = Int32 Function(Int32 a, Int32 b);
typedef DartAddFunc = int Function(int a, int b);

class NativeLib {
  late final DynamicLibrary _lib;
  late final DartAddFunc add;

  NativeLib() {
    _lib = Platform.isWindows
        ? DynamicLibrary.open('native_lib.dll')
        : DynamicLibrary.open('libnative_lib.so');

    add = _lib
        .lookup<NativeFunction<NativeAddFunc>>('add')
        .asFunction();
  }
}
```

## 原生侧导出示例

### C

```c
#include <stdint.h>

int32_t add(int32_t a, int32_t b) {
    return a + b;
}
```

### Rust

```rust
#[no_mangle]
pub extern "C" fn add(a: i32, b: i32) -> i32 {
    a + b
}
```

## 典型场景

### 接算法库
- 图像处理
- 音视频编解码
- 压缩与解压
- 加密签名

### 复用现有原生 SDK
- 已有 C/C++ 业务库
- 设备厂商提供的底层 SDK
- 跨平台共享一套核心逻辑

### 性能敏感模块
- 大量数值计算
- 实时音视频处理
- 大文件处理

## 工程要点

### 类型映射
- `Int32` <-> `int`
- `Double` <-> `double`
- `Pointer<Utf8>` <-> 字符串指针
- 结构体需要 `Struct` 映射

### 字符串处理
- Dart 字符串通常要转 `Utf8` 指针再传给原生。
- 原生返回字符串时，需要明确谁负责释放内存。

### 回调
- 原生调用 Dart 回调可以用 `Pointer<NativeFunction<...>>`
- 需要注意线程限制与生命周期。

### 错误处理
- FFI 本身不提供异常边界保护。
- 原生崩溃会直接导致应用崩溃。
- 建议统一设计错误码、日志和降级策略。

## 常见问题

### 崩溃
- 原因：函数签名不匹配、空指针、非法内存访问。
- 排查：先核对 ABI、参数类型、返回值类型、结构体对齐。

### 库加载失败
- 原因：动态库未打包、路径错误、架构不匹配。
- 排查：检查 `armeabi-v7a`、`arm64-v8a`、`x86_64` 等产物是否齐全。

### 内存泄漏
- 原因：申请后未释放、回调长期持有、字符串返回策略不清晰。
- 排查：梳理分配与释放责任边界。

## 面试高频问题

1. FFI 和 Platform Channel 分别适合什么场景？
2. Dart 调用 C 库的基本步骤是什么？
3. 为什么 FFI 签名不匹配会直接崩溃？
4. 如何处理 FFI 中的字符串和结构体？
5. Flutter 中接 Rust 动态库时要注意什么？

## 建议扩展

- 增加 `Struct`、`Union`、回调函数示例
- 增加 Android / iOS 动态库打包说明
- 增加 Rust + Flutter 实战案例
- 增加 `ffigen` 自动生成绑定说明
