# 网络请求

## http 包

### 基本使用

```dart
// 添加依赖
// dependencies:
//   http: ^0.13.5

import 'dart:convert';
import 'package:http/http.dart' as http;

// GET 请求
Future<void> fetchData() async {
  final response = await http.get(Uri.parse('https://api.example.com/data'));
  
  if (response.statusCode == 200) {
    // 成功
    final data = jsonDecode(response.body);
    print(data);
  } else {
    // 失败
    print('Request failed with status: ${response.statusCode}');
  }
}

// POST 请求
Future<void> postData() async {
  final response = await http.post(
    Uri.parse('https://api.example.com/data'),
    headers: <String, String>{
      'Content-Type': 'application/json; charset=UTF-8',
    },
    body: jsonEncode(<String, String>{
      'name': 'Flutter',
      'version': '3.0.0',
    }),
  );
  
  if (response.statusCode == 201) {
    // 成功
    final data = jsonDecode(response.body);
    print(data);
  } else {
    // 失败
    print('Request failed with status: ${response.statusCode}');
  }
}

// PUT 请求
Future<void> putData() async {
  final response = await http.put(
    Uri.parse('https://api.example.com/data/1'),
    headers: <String, String>{
      'Content-Type': 'application/json; charset=UTF-8',
    },
    body: jsonEncode(<String, String>{
      'name': 'Updated Flutter',
    }),
  );
  
  if (response.statusCode == 200) {
    // 成功
    final data = jsonDecode(response.body);
    print(data);
  } else {
    // 失败
    print('Request failed with status: ${response.statusCode}');
  }
}

// DELETE 请求
Future<void> deleteData() async {
  final response = await http.delete(
    Uri.parse('https://api.example.com/data/1'),
  );
  
  if (response.statusCode == 200) {
    // 成功
    print('Delete successful');
  } else {
    // 失败
    print('Request failed with status: ${response.statusCode}');
  }
}
```

### 错误处理

```dart
Future<void> fetchDataWithErrorHandling() async {
  try {
    final response = await http.get(Uri.parse('https://api.example.com/data'));
    
    if (response.statusCode == 200) {
      final data = jsonDecode(response.body);
      print(data);
    } else {
      throw Exception('Failed with status code: ${response.statusCode}');
    }
  } catch (e) {
    print('Error: $e');
  }
}
```

## Dio

### 基本使用

```dart
// 添加依赖
// dependencies:
//   dio: ^4.0.6

import 'dart:convert';
import 'package:dio/dio.dart';

// 创建 Dio 实例
final dio = Dio();

// GET 请求
Future<void> fetchData() async {
  try {
    final response = await dio.get('https://api.example.com/data');
    print(response.data);
  } catch (e) {
    print('Error: $e');
  }
}

// POST 请求
Future<void> postData() async {
  try {
    final response = await dio.post('https://api.example.com/data',
      data: {
        'name': 'Flutter',
        'version': '3.0.0',
      },
    );
    print(response.data);
  } catch (e) {
    print('Error: $e');
  }
}

// PUT 请求
Future<void> putData() async {
  try {
    final response = await dio.put('https://api.example.com/data/1',
      data: {
        'name': 'Updated Flutter',
      },
    );
    print(response.data);
  } catch (e) {
    print('Error: $e');
  }
}

// DELETE 请求
Future<void> deleteData() async {
  try {
    final response = await dio.delete('https://api.example.com/data/1');
    print(response.data);
  } catch (e) {
    print('Error: $e');
  }
}
```

### 高级功能

#### 拦截器

```dart
// 添加请求拦截器
dio.interceptors.add(InterceptorsWrapper(
  onRequest: (options, handler) {
    // 在发送请求前做些什么
    options.headers['Authorization'] = 'Bearer token';
    return handler.next(options);
  },
  onResponse: (response, handler) {
    // 在收到响应后做些什么
    return handler.next(response);
  },
  onError: (DioError e, handler) {
    // 在请求失败时做些什么
    return handler.next(e);
  },
));
```

#### 请求取消

```dart
// 创建取消令牌
final cancelToken = CancelToken();

// 发送请求
Future<void> fetchData() async {
  try {
    final response = await dio.get(
      'https://api.example.com/data',
      cancelToken: cancelToken,
    );
    print(response.data);
  } catch (e) {
    if (CancelToken.isCancel(e)) {
      print('Request canceled');
    } else {
      print('Error: $e');
    }
  }
}

// 取消请求
void cancelRequest() {
  cancelToken.cancel('Canceled by user');
}
```

#### 超时设置

```dart
// 全局超时设置
dio.options.connectTimeout = Duration(seconds: 5);
dio.options.receiveTimeout = Duration(seconds: 3);

// 单个请求超时设置
final response = await dio.get(
  'https://api.example.com/data',
  options: Options(
    connectTimeout: Duration(seconds: 10),
    receiveTimeout: Duration(seconds: 5),
  ),
);
```

## Retrofit

### 基本使用

```dart
// 添加依赖
// dependencies:
//   retrofit: ^3.0.1
//   json_annotation: ^4.6.0
//   
// dev_dependencies:
//   retrofit_generator: ^4.0.1
//   build_runner: ^2.3.3
//   json_serializable: ^6.3.1

// 定义 API 接口
import 'package:retrofit/retrofit.dart';
import 'package:dio/dio.dart';
import 'package:json_annotation/json_annotation.dart';

part 'api_service.g.dart';

@RestApi(baseUrl: 'https://api.example.com')
abstract class ApiService {
  factory ApiService(Dio dio, {String baseUrl}) = _ApiService;

  @GET('/data')
  Future<List<Item>> getItems();

  @GET('/data/{id}')
  Future<Item> getItem(@Path('id') int id);

  @POST('/data')
  Future<Item> createItem(@Body() Item item);

  @PUT('/data/{id}')
  Future<Item> updateItem(@Path('id') int id, @Body() Item item);

  @DELETE('/data/{id}')
  Future<void> deleteItem(@Path('id') int id);
}

// 定义数据模型
@JsonSerializable()
class Item {
  final int id;
  final String name;
  final String description;

  Item(this.id, this.name, this.description);

  factory Item.fromJson(Map<String, dynamic> json) => _$ItemFromJson(json);
  Map<String, dynamic> toJson() => _$ItemToJson(this);
}

// 生成代码
// 运行命令: flutter pub run build_runner build

// 使用
final dio = Dio();
final apiService = ApiService(dio);

// 调用 API
Future<void> fetchItems() async {
  try {
    final items = await apiService.getItems();
    print(items);
  } catch (e) {
    print('Error: $e');
  }
}
```

## WebSocket

### 基本使用

```dart
import 'dart:io';

Future<void> connectWebSocket() async {
  final socket = await WebSocket.connect('wss://echo.websocket.org');
  
  // 发送消息
  socket.add('Hello WebSocket!');
  
  // 监听消息
  socket.listen(
    (data) {
      print('Received: $data');
    },
    onError: (error) {
      print('Error: $error');
    },
    onDone: () {
      print('WebSocket closed');
    },
  );
  
  // 关闭连接
  // socket.close();
}
```

### 使用 web_socket_channel 库

```dart
// 添加依赖
// dependencies:
//   web_socket_channel: ^2.2.0

import 'package:web_socket_channel/io.dart';
import 'package:web_socket_channel/web_socket_channel.dart';

void connectWebSocket() {
  final channel = IOWebSocketChannel.connect('wss://echo.websocket.org');
  
  // 发送消息
  channel.sink.add('Hello WebSocket!');
  
  // 监听消息
  channel.stream.listen(
    (message) {
      print('Received: $message');
    },
    onError: (error) {
      print('Error: $error');
    },
    onDone: () {
      print('WebSocket closed');
    },
  );
  
  // 关闭连接
  // channel.sink.close();
}
```

## 最佳实践

1. **错误处理**：
   - 使用 try-catch 捕获异常
   - 处理网络错误和服务器错误
   - 提供用户友好的错误提示

2. **请求管理**：
   - 使用拦截器统一处理认证信息
   - 实现请求取消机制
   - 设置合理的超时时间

3. **缓存策略**：
   - 实现请求缓存
   - 处理离线情况
   - 合理使用本地存储

4. **性能优化**：
   - 批量请求
   - 合理使用连接池
   - 压缩请求和响应数据

5. **安全性**：
   - 使用 HTTPS
   - 保护敏感信息
   - 实现请求签名

## 常见问题

1. **网络连接问题**：
   - 无网络连接
   - 网络不稳定
   - 超时问题

2. **认证问题**：
   - Token 过期
   - 权限不足
   - 认证失败

3. **数据处理问题**：
   - JSON 解析错误
   - 数据类型不匹配
   - 数据为空

4. **性能问题**：
   - 请求过多
   - 响应数据过大
   - 并发请求管理

5. **跨域问题**：
   - CORS 错误
   - 跨域请求被阻止

## 面试题

1. **Q**: Flutter 中常用的网络请求库有哪些？
   **A**: http 包、Dio、Retrofit

2. **Q**: Dio 相比 http 包有哪些优势？
   **A**: 支持拦截器、请求取消、超时设置、FormData、文件上传等高级功能

3. **Q**: 如何处理网络请求中的错误？
   **A**: 使用 try-catch 捕获异常，处理不同类型的错误（网络错误、服务器错误、超时等）

4. **Q**: 如何实现 WebSocket 连接？
   **A**: 使用 dart:io 中的 WebSocket 类或 web_socket_channel 库

5. **Q**: 如何优化网络请求性能？
   **A**: 实现请求缓存、批量请求、合理使用连接池、压缩数据