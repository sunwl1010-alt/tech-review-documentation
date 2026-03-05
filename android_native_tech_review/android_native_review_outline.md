# Android 原生技术复习大纲

## 核心组件
- **[Activity](components/activity.md)**
  - 生命周期
  - 启动模式
  - 状态保存与恢复
  - Intent 与 Intent Filter

- **[Service](components/service.md)**
  - 生命周期
  - 前台服务
  - 绑定服务
  - IntentService

- **[BroadcastReceiver](components/broadcast_receiver.md)**
  - 静态注册与动态注册
  - 有序广播与无序广播
  - 本地广播
  - 系统广播

- **[ContentProvider](components/content_provider.md)**
  - URI 结构
  - CRUD 操作
  - ContentObserver
  - 权限管理

## UI 系统
- **布局系统**
  - XML 布局
  - Jetpack Compose
  - [布局管理器](ui/layout_managers.md)
  - 自定义 View

- **控件系统**
  - 基础控件
  - 自定义控件
  - 适配器模式
  - 列表与网格

## 数据存储
- **[SharedPreferences](storage/shared_preferences.md)**
- **[文件存储](storage/file_storage.md)**
- **[SQLite 数据库](storage/sqlite.md)**
- **Room 持久化库**

## 网络编程
- **OkHttp**
- **Retrofit**
- **Volley**
- **WebSocket**

## 多线程与异步
- **Thread**
- **Handler**
- **AsyncTask**
- **ThreadPoolExecutor**
- **Coroutine**

## 系统服务
- **PackageManager**
- **ActivityManager**
- **NotificationManager**
- **LocationManager**

## 安全性
- **权限管理**
- **数据加密**
- **网络安全**
- **应用签名**

## 性能优化
- **内存优化**
- **UI 优化**
- **网络优化**
- **电池优化**

## 架构设计
- **[MVC](architecture/mvc.md)**
- **[MVP](architecture/mvp.md)**
- **[MVVM](architecture/mvvm.md)**
- **Clean Architecture**

## 测试
- **单元测试**
- **集成测试**
- **UI 测试**

## 版本兼容性
- **API 级别**
- **权限变化**
- **行为变化**
- **兼容性方案**