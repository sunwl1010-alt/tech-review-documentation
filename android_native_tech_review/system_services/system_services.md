# 系统服务

## PackageManager

### 基本概念

PackageManager 是 Android 系统中负责管理应用包的服务，提供了应用信息查询、安装、卸载等功能。

### 常用方法

```kotlin
val packageManager = context.packageManager

// 获取应用信息
val packageInfo = packageManager.getPackageInfo(packageName, 0)
val versionName = packageInfo.versionName
val versionCode = packageInfo.versionCode

// 获取应用图标
val appIcon = packageManager.getApplicationIcon(packageName)

// 获取应用名称
val appName = packageManager.getApplicationLabel(packageInfo.applicationInfo).toString()

// 检查应用是否已安装
fun isAppInstalled(packageName: String): Boolean {
    try {
        packageManager.getPackageInfo(packageName, 0)
        return true
    } catch (e: PackageManager.NameNotFoundException) {
        return false
    }
}

// 启动其他应用
fun launchApp(packageName: String) {
    val intent = packageManager.getLaunchIntentForPackage(packageName)
    if (intent != null) {
        context.startActivity(intent)
    }
}

// 获取已安装的应用列表
val installedApps = packageManager.getInstalledApplications(0)
for (appInfo in installedApps) {
    val appName = packageManager.getApplicationLabel(appInfo).toString()
    val packageName = appInfo.packageName
}
```

### 权限相关

```kotlin
// 检查权限是否已授予
fun checkPermission(permission: String): Boolean {
    return packageManager.checkPermission(permission, context.packageName) == PackageManager.PERMISSION_GRANTED
}

// 获取应用请求的权限
val packageInfo = packageManager.getPackageInfo(packageName, PackageManager.GET_PERMISSIONS)
val requestedPermissions = packageInfo.requestedPermissions
```

## ActivityManager

### 基本概念

ActivityManager 是 Android 系统中负责管理 Activity、Service 和任务栈的服务。

### 常用方法

```kotlin
val activityManager = context.getSystemService(Context.ACTIVITY_SERVICE) as ActivityManager

// 获取正在运行的应用进程
val runningAppProcesses = activityManager.runningAppProcesses
for (processInfo in runningAppProcesses) {
    val processName = processInfo.processName
    val pid = processInfo.pid
}

// 获取正在运行的服务
val runningServices = activityManager.getRunningServices(100)
for (serviceInfo in runningServices) {
    val serviceName = serviceInfo.service.className
    val pid = serviceInfo.pid
}

// 获取内存信息
val memoryInfo = ActivityManager.MemoryInfo()
activityManager.getMemoryInfo(memoryInfo)
val isLowMemory = memoryInfo.lowMemory
val availableMemory = memoryInfo.availMem

// 强制停止应用
if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.KITKAT) {
    activityManager.forceStopPackage(packageName)
}
```

## NotificationManager

### 基本概念

NotificationManager 是 Android 系统中负责发送和管理通知的服务。

### 发送通知

```kotlin
val notificationManager = context.getSystemService(Context.NOTIFICATION_SERVICE) as NotificationManager

// 创建通知渠道（Android 8.0+）
if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {
    val channelId = "my_channel"
    val channelName = "My Channel"
    val channelDescription = "Channel description"
    val importance = NotificationManager.IMPORTANCE_DEFAULT
    val channel = NotificationChannel(channelId, channelName, importance).apply {
        description = channelDescription
    }
    notificationManager.createNotificationChannel(channel)
}

// 构建通知
val notificationBuilder = NotificationCompat.Builder(context, "my_channel")
    .setSmallIcon(R.drawable.ic_notification)
    .setContentTitle("Notification Title")
    .setContentText("Notification content")
    .setPriority(NotificationCompat.PRIORITY_DEFAULT)
    .setContentIntent(pendingIntent) // 点击通知后跳转的意图
    .setAutoCancel(true) // 点击后自动取消

// 发送通知
val notificationId = 1
notificationManager.notify(notificationId, notificationBuilder.build())

// 取消通知
notificationManager.cancel(notificationId)

// 取消所有通知
notificationManager.cancelAll()
```

### 通知样式

```kotlin
// 大文本样式
val bigTextStyle = NotificationCompat.BigTextStyle()
    .bigText("Long notification content goes here...")
    .setBigContentTitle("Big Title")
    .setSummaryText("Summary")

val notificationBuilder = NotificationCompat.Builder(context, "my_channel")
    .setSmallIcon(R.drawable.ic_notification)
    .setStyle(bigTextStyle)
    .setContentTitle("Notification Title")
    .setContentText("Notification content")

// 图片样式
val bigPictureStyle = NotificationCompat.BigPictureStyle()
    .bigPicture(BitmapFactory.decodeResource(resources, R.drawable.big_image))
    .setBigContentTitle("Big Picture Title")
    .setSummaryText("Summary")

// 收件箱样式
val inboxStyle = NotificationCompat.InboxStyle()
    .addLine("Line 1")
    .addLine("Line 2")
    .addLine("Line 3")
    .setBigContentTitle("Inbox Title")
    .setSummaryText("3 new messages")
```

## LocationManager

### 基本概念

LocationManager 是 Android 系统中负责获取设备位置信息的服务。

### 获取位置

```kotlin
val locationManager = context.getSystemService(Context.LOCATION_SERVICE) as LocationManager

// 检查位置权限
if (ContextCompat.checkSelfPermission(context, Manifest.permission.ACCESS_FINE_LOCATION) != PackageManager.PERMISSION_GRANTED) {
    // 请求权限
    ActivityCompat.requestPermissions(activity, arrayOf(Manifest.permission.ACCESS_FINE_LOCATION), REQUEST_LOCATION_PERMISSION)
    return
}

// 获取最后已知位置
val lastKnownLocation = locationManager.getLastKnownLocation(LocationManager.GPS_PROVIDER)
if (lastKnownLocation != null) {
    val latitude = lastKnownLocation.latitude
    val longitude = lastKnownLocation.longitude
}

// 注册位置监听器
val locationListener = object : LocationListener {
    override fun onLocationChanged(location: Location) {
        val latitude = location.latitude
        val longitude = location.longitude
    }
    
    override fun onStatusChanged(provider: String?, status: Int, extras: Bundle?) {}
    
    override fun onProviderEnabled(provider: String) {}
    
    override fun onProviderDisabled(provider: String) {}
}

// 注册监听器
locationManager.requestLocationUpdates(LocationManager.GPS_PROVIDER, 1000, 1f, locationListener)

// 取消注册
locationManager.removeUpdates(locationListener)
```

### 位置提供者

- **GPS_PROVIDER**：通过 GPS 卫星获取位置，精度高但耗电大
- **NETWORK_PROVIDER**：通过网络（WiFi、移动网络）获取位置，精度较低但耗电小
- **PASSIVE_PROVIDER**：被动接收其他应用的位置更新

###  fusedLocationProviderClient（推荐）

```kotlin
// 添加依赖
// implementation "com.google.android.gms:play-services-location:19.0.1"

val fusedLocationClient = LocationServices.getFusedLocationProviderClient(context)

// 检查权限
if (ContextCompat.checkSelfPermission(context, Manifest.permission.ACCESS_FINE_LOCATION) == PackageManager.PERMISSION_GRANTED) {
    fusedLocationClient.lastLocation
        .addOnSuccessListener { location ->
            if (location != null) {
                val latitude = location.latitude
                val longitude = location.longitude
            }
        }
        .addOnFailureListener {
            // 处理错误
        }
}
```

## 其他系统服务

### TelephonyManager

```kotlin
val telephonyManager = context.getSystemService(Context.TELEPHONY_SERVICE) as TelephonyManager

// 获取设备 ID
val deviceId = telephonyManager.deviceId

// 获取手机号码
val phoneNumber = telephonyManager.line1Number

// 获取网络运营商名称
val carrierName = telephonyManager.networkOperatorName

// 获取 SIM 卡状态
val simState = telephonyManager.simState
```

### WifiManager

```kotlin
val wifiManager = context.getSystemService(Context.WIFI_SERVICE) as WifiManager

// 检查 WiFi 是否开启
val isWifiEnabled = wifiManager.isWifiEnabled

// 开启/关闭 WiFi
if (Build.VERSION.SDK_INT < Build.VERSION_CODES.Q) {
    wifiManager.isWifiEnabled = true // 开启
    wifiManager.isWifiEnabled = false // 关闭
}

// 获取当前连接的 WiFi 信息
val wifiInfo = wifiManager.connectionInfo
val ssid = wifiInfo.ssid
val bssid = wifiInfo.bssid
val ipAddress = wifiInfo.ipAddress
```

### BatteryManager

```kotlin
val batteryManager = context.getSystemService(Context.BATTERY_SERVICE) as BatteryManager

// 获取电池电量
val batteryLevel = batteryManager.getIntProperty(BatteryManager.BATTERY_PROPERTY_CAPACITY)

// 检查电池是否正在充电
val isCharging = batteryManager.getIntProperty(BatteryManager.BATTERY_PROPERTY_STATUS) == BatteryManager.BATTERY_STATUS_CHARGING

// 通过 Intent 获取电池信息
val batteryIntent = context.registerReceiver(null, IntentFilter(Intent.ACTION_BATTERY_CHANGED))
val level = batteryIntent?.getIntExtra(BatteryManager.EXTRA_LEVEL, -1) ?: -1
val scale = batteryIntent?.getIntExtra(BatteryManager.EXTRA_SCALE, -1) ?: -1
val batteryPercentage = level * 100 / scale
```

## 最佳实践

1. **权限处理**：获取系统服务前确保已获得必要的权限
2. **服务生命周期**：及时注册和取消注册监听器，避免内存泄漏
3. **异常处理**：处理可能的异常，如权限被拒绝、服务不可用等
4. **性能考虑**：避免频繁调用系统服务，特别是耗电的服务如 GPS
5. **版本兼容性**：注意不同 Android 版本的 API 差异

## 常见问题

1. **权限被拒绝**：用户拒绝授予必要的权限
2. **服务不可用**：某些系统服务在特定设备或状态下可能不可用
3. **耗电问题**：过度使用 GPS 等服务会导致电池消耗过快
4. **内存泄漏**：未正确取消注册监听器
5. **版本兼容性**：不同 Android 版本的 API 行为可能不同

## 面试题

1. **Q**: 如何获取应用的版本信息？
   **A**: 使用 PackageManager 的 getPackageInfo() 方法获取应用信息，然后访问 versionName 和 versionCode 属性

2. **Q**: 如何发送通知？
   **A**: 使用 NotificationManager，在 Android 8.0+ 上需要先创建通知渠道，然后构建并发送通知

3. **Q**: 如何获取设备位置？
   **A**: 使用 LocationManager 或 FusedLocationProviderClient，需要先获取位置权限

4. **Q**: 如何检查应用是否已安装？
   **A**: 使用 PackageManager 的 getPackageInfo() 方法，捕获 NameNotFoundException 异常

5. **Q**: 如何获取正在运行的应用进程？
   **A**: 使用 ActivityManager 的 getRunningAppProcesses() 方法