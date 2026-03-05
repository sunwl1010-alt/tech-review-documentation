# 路由与导航

## 基本导航

### Navigator.push 和 Navigator.pop

```dart
// 基本导航
Navigator.push(
  context,
  MaterialPageRoute(builder: (context) => SecondScreen()),
);

// 返回上一页
Navigator.pop(context);

// 带返回值的导航
// 第一个页面
final result = await Navigator.push(
  context,
  MaterialPageRoute(builder: (context) => SecondScreen()),
);
print('Result: $result');

// 第二个页面
Navigator.pop(context, 'Hello from Second Screen');
```

### 命名路由

#### 定义路由

```dart
MaterialApp(
  initialRoute: '/',
  routes: {
    '/': (context) => HomeScreen(),
    '/second': (context) => SecondScreen(),
    '/third': (context) => ThirdScreen(),
  },
);
```

#### 使用命名路由

```dart
// 导航到命名路由
Navigator.pushNamed(context, '/second');

// 带参数的命名路由
Navigator.pushNamed(
  context,
  '/third',
  arguments: {'id': 123, 'name': 'Flutter'},
);

// 从命名路由返回
Navigator.pop(context);
```

## 路由参数

### 传递参数

```dart
// 通过构造函数传递参数
Navigator.push(
  context,
  MaterialPageRoute(
    builder: (context) => DetailScreen(itemId: 123),
  ),
);

// 通过 arguments 传递参数
Navigator.pushNamed(
  context,
  '/detail',
  arguments: Item(id: 123, name: 'Product'),
);
```

### 接收参数

```dart
// 从构造函数接收
class DetailScreen extends StatelessWidget {
  final int itemId;

  const DetailScreen({Key? key, required this.itemId}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('Detail')),
      body: Center(child: Text('Item ID: $itemId')),
    );
  }
}

// 从 arguments 接收
class DetailScreen extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    final Item item = ModalRoute.of(context)!.settings.arguments as Item;
    
    return Scaffold(
      appBar: AppBar(title: Text('Detail')),
      body: Center(child: Text('Item: ${item.name}, ID: ${item.id}')),
    );
  }
}
```

## 路由动画

### 自定义页面过渡动画

```dart
Navigator.push(
  context,
  PageRouteBuilder(
    pageBuilder: (context, animation, secondaryAnimation) => SecondScreen(),
    transitionsBuilder: (context, animation, secondaryAnimation, child) {
      const begin = Offset(1.0, 0.0);
      const end = Offset.zero;
      const curve = Curves.ease;

      var tween = Tween(begin: begin, end: end).chain(CurveTween(curve: curve));

      return SlideTransition(
        position: animation.drive(tween),
        child: child,
      );
    },
  ),
);
```

### 内置过渡动画

```dart
// 淡入淡出
Navigator.push(
  context,
  PageRouteBuilder(
    pageBuilder: (context, animation, secondaryAnimation) => SecondScreen(),
    transitionsBuilder: (context, animation, secondaryAnimation, child) {
      return FadeTransition(
        opacity: animation,
        child: child,
      );
    },
  ),
);

// 缩放
Navigator.push(
  context,
  PageRouteBuilder(
    pageBuilder: (context, animation, secondaryAnimation) => SecondScreen(),
    transitionsBuilder: (context, animation, secondaryAnimation, child) {
      return ScaleTransition(
        scale: animation,
        child: child,
      );
    },
  ),
);
```

## 深层链接

### 配置深层链接

#### Android 配置

在 `AndroidManifest.xml` 中添加：

```xml
<activity
  android:name=".MainActivity"
  android:exported="true">
  <intent-filter>
    <action android:name="android.intent.action.MAIN" />
    <category android:name="android.intent.category.LAUNCHER" />
  </intent-filter>
  <intent-filter android:autoVerify="true">
    <action android:name="android.intent.action.VIEW" />
    <category android:name="android.intent.category.DEFAULT" />
    <category android:name="android.intent.category.BROWSABLE" />
    <data
      android:scheme="https"
      android:host="example.com"
      android:pathPrefix="/product"
    />
  </intent-filter>
</activity>
```

#### iOS 配置

在 `Info.plist` 中添加：

```xml
<key>CFBundleURLTypes</key>
<array>
  <dict>
    <key>CFBundleURLSchemes</key>
    <array>
      <string>example</string>
    </array>
    <key>CFBundleURLName</key>
    <string>com.example.app</string>
  </dict>
</array>
```

### 处理深层链接

```dart
class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      initialRoute: '/',
      routes: {
        '/': (context) => HomeScreen(),
        '/product': (context) => ProductScreen(),
      },
      onGenerateRoute: (settings) {
        if (settings.name == '/product') {
          final uri = Uri.parse(settings.name!);
          final productId = uri.queryParameters['id'];
          return MaterialPageRoute(
            builder: (context) => ProductScreen(id: productId),
          );
        }
        return null;
      },
    );
  }
}

// 使用 uni_links 库处理深层链接
// 添加依赖：uni_links: ^0.5.1

class HomeScreen extends StatefulWidget {
  @override
  _HomeScreenState createState() => _HomeScreenState();
}

class _HomeScreenState extends State<HomeScreen> {
  Uri? _initialLink;
  Uri? _latestLink;

  @override
  void initState() {
    super.initState();
    _initDeepLinks();
  }

  void _initDeepLinks() async {
    // 获取初始链接
    final initialLink = await getInitialUri();
    setState(() {
      _initialLink = initialLink;
    });

    // 监听链接变化
    linkStream.listen((uri) {
      setState(() {
        _latestLink = uri;
      });
    });
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('Home')),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            Text('Initial Link: $_initialLink'),
            Text('Latest Link: $_latestLink'),
          ],
        ),
      ),
    );
  }
}
```

## 导航器键

### 全局导航

```dart
final GlobalKey<NavigatorState> navigatorKey = GlobalKey<NavigatorState>();

MaterialApp(
  navigatorKey: navigatorKey,
  // ...
);

// 从任何地方导航
navigatorKey.currentState?.push(
  MaterialPageRoute(builder: (context) => SecondScreen()),
);

// 从任何地方返回
navigatorKey.currentState?.pop();
```

## 嵌套导航

```dart
class HomeScreen extends StatefulWidget {
  @override
  _HomeScreenState createState() => _HomeScreenState();
}

class _HomeScreenState extends State<HomeScreen> {
  final _navigatorKey = GlobalKey<NavigatorState>();

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('Home')),
      body: Navigator(
        key: _navigatorKey,
        initialRoute: 'home',
        onGenerateRoute: (settings) {
          WidgetBuilder builder;
          switch (settings.name) {
            case 'home':
              builder = (context) => HomeContent();
              break;
            case 'profile':
              builder = (context) => ProfileScreen();
              break;
            default:
              builder = (context) => HomeContent();
          }
          return MaterialPageRoute(builder: builder, settings: settings);
        },
      ),
      bottomNavigationBar: BottomNavigationBar(
        items: [
          BottomNavigationBarItem(icon: Icon(Icons.home), label: 'Home'),
          BottomNavigationBarItem(icon: Icon(Icons.person), label: 'Profile'),
        ],
        onTap: (index) {
          if (index == 0) {
            _navigatorKey.currentState?.pushReplacementNamed('home');
          } else {
            _navigatorKey.currentState?.pushReplacementNamed('profile');
          }
        },
      ),
    );
  }
}
```

## 最佳实践

1. **路由管理**：
   - 使用命名路由提高代码可读性
   - 集中管理路由配置
   - 使用路由生成器处理动态路由

2. **参数传递**：
   - 简单参数使用构造函数
   - 复杂参数使用 arguments
   - 深层链接使用 URI 参数

3. **动画效果**：
   - 为不同场景选择合适的过渡动画
   - 避免过度使用复杂动画影响性能

4. **深层链接**：
   - 正确配置 Android 和 iOS 的深层链接
   - 处理初始链接和动态链接

5. **错误处理**：
   - 处理不存在的路由
   - 处理参数解析错误

## 常见问题

1. **路由冲突**：
   - 命名路由与动态路由冲突
   - 嵌套导航器的路由管理

2. **参数传递错误**：
   - 参数类型不匹配
   - 参数为 null 时的处理

3. **深层链接配置错误**：
   - Android 或 iOS 配置不正确
   - 链接格式与处理逻辑不匹配

4. **导航状态管理**：
   - 导航栈管理不当
   - 多次导航导致的状态混乱

5. **性能问题**：
   - 复杂动画导致的卡顿
   - 路由切换时的内存占用

## 面试题

1. **Q**: Flutter 中如何实现页面导航？
   **A**: 使用 Navigator.push 和 Navigator.pop 进行基本导航，使用命名路由进行更有组织的导航

2. **Q**: 如何在 Flutter 中传递参数给下一个页面？
   **A**: 通过构造函数传递简单参数，通过 arguments 传递复杂参数，通过 URI 参数处理深层链接

3. **Q**: 如何实现自定义页面过渡动画？
   **A**: 使用 PageRouteBuilder 自定义 transitionsBuilder 方法

4. **Q**: 什么是深层链接？如何在 Flutter 中实现？
   **A**: 深层链接是一种从外部应用或网页直接打开应用内特定页面的机制，通过配置 Android 和 iOS 的 URL  schemes 并在应用中处理链接

5. **Q**: 如何在 Flutter 中实现底部导航栏与页面导航的结合？
   **A**: 使用嵌套导航器，每个底部导航项对应一个导航栈