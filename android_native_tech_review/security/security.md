# Android 安全工程实践（进阶）

## 为什么需要“安全工程化”
Android 安全不是“加个权限校验”或“开启混淆”就结束，而是一个覆盖设计、开发、测试、上线、监控的持续过程。建议从以下四层建立安全基线：

- 应用层：权限、输入校验、鉴权、反重放、防注入
- 传输层：TLS、证书校验、接口签名、重试与幂等
- 存储层：密钥管理、数据分级加密、备份策略
- 供应链层：依赖安全、构建产物、发布与签名链路

## 一、权限治理：最小化 + 分层授权

### 1.1 权限分级策略
- 核心权限：摄像头、麦克风、定位、通知、媒体读取
- 敏感能力：后台定位、无障碍、悬浮窗、安装未知来源
- 高风险接口：导出组件、深链、文件共享、WebView bridge

实践建议：
- 先做“功能-权限映射表”，每个权限必须对应明确业务功能。
- 对可选能力改为“按需申请”，不要冷启动集中申请。
- 对拒绝权限的路径做完整降级（而不是直接退出）。

### 1.2 运行时权限（Activity Result API）
```kotlin
private val requestCameraPermission =
    registerForActivityResult(ActivityResultContracts.RequestPermission()) { granted ->
        if (granted) {
            openCamera()
        } else {
            showCameraPermissionFallback()
        }
    }

private fun ensureCameraPermission() {
    when {
        ContextCompat.checkSelfPermission(
            this,
            Manifest.permission.CAMERA
        ) == PackageManager.PERMISSION_GRANTED -> openCamera()

        shouldShowRequestPermissionRationale(Manifest.permission.CAMERA) -> {
            showRationaleDialog(
                title = "需要相机权限",
                message = "用于拍照上传头像，可在系统设置中随时关闭"
            ) { requestCameraPermission.launch(Manifest.permission.CAMERA) }
        }

        else -> requestCameraPermission.launch(Manifest.permission.CAMERA)
    }
}
```

## 二、组件暴露面收敛：先不暴露，再精确暴露

### 2.1 Manifest 防御清单
- 所有 `activity/service/receiver/provider` 显式声明 `android:exported`
- 非必要组件设为 `android:exported="false"`
- `intent-filter` 仅保留必要 action/category/data
- `provider` 使用 `grantUriPermissions` + 最小 URI 范围

### 2.2 PendingIntent 风险控制
Android 12+ 必须明确可变性。默认使用不可变：
```kotlin
val pending = PendingIntent.getActivity(
    context,
    1001,
    Intent(context, DetailActivity::class.java),
    PendingIntent.FLAG_UPDATE_CURRENT or PendingIntent.FLAG_IMMUTABLE
)
```

## 三、网络安全：默认不信任 + 可观测

### 3.1 Network Security Config
- 禁止明文流量：`cleartextTrafficPermitted="false"`
- 仅对内网调试域名开例外
- 区分 debug/release 证书策略

`res/xml/network_security_config.xml` 示例：
```xml
<network-security-config>
    <base-config cleartextTrafficPermitted="false">
        <trust-anchors>
            <certificates src="system"/>
        </trust-anchors>
    </base-config>
    <domain-config cleartextTrafficPermitted="true">
        <domain includeSubdomains="true">10.0.2.2</domain>
    </domain-config>
</network-security-config>
```

### 3.2 证书锁定（Pinning）
```kotlin
val pinner = CertificatePinner.Builder()
    .add("api.example.com", "sha256/AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA=")
    .build()

val client = OkHttpClient.Builder()
    .certificatePinner(pinner)
    .build()
```

注意：Pinning 配置要有轮转预案（至少两把有效 pin，支持灰度切换）。

## 四、数据安全：分级存储 + 密钥托管

### 4.1 数据分级
- L1（公开）：无敏感信息，可明文缓存
- L2（业务敏感）：用户偏好、业务配置，建议加密
- L3（高敏）：token、私钥、身份证件，必须加密 + 最短驻留

### 4.2 Keystore + EncryptedSharedPreferences
```kotlin
val masterKey = MasterKey.Builder(context)
    .setKeyScheme(MasterKey.KeyScheme.AES256_GCM)
    .build()

val securePrefs = EncryptedSharedPreferences.create(
    context,
    "secure_prefs",
    masterKey,
    EncryptedSharedPreferences.PrefKeyEncryptionScheme.AES256_SIV,
    EncryptedSharedPreferences.PrefValueEncryptionScheme.AES256_GCM
)

securePrefs.edit().putString("access_token", token).apply()
```

实践建议：
- token 只存刷新所需最小信息，避免长期持久化 access token。
- 高敏数据增加“设备绑定”与“过期策略”。

## 五、WebView 安全：当成外部输入执行器

### 5.1 最小能力原则
```kotlin
webView.settings.apply {
    javaScriptEnabled = false
    allowFileAccess = false
    allowContentAccess = false
    mixedContentMode = WebSettings.MIXED_CONTENT_NEVER_ALLOW
}
```

### 5.2 JavaScript Bridge 防护
- 非必要不暴露 `addJavascriptInterface`
- 必须暴露时：仅在受信域名页面开启；接口不执行敏感操作；参数强校验
- 页面来源必须可验证，禁止加载不受控 URL

## 六、签名与发布链路安全

### 6.1 构建与签名
- 生产 keystore 与开发 keystore 彻底隔离
- CI 通过密钥管理系统注入，不落盘明文
- 发布包做 hash 归档，支持追溯

### 6.2 供应链治理
- 依赖锁版本（避免动态版本如 `1.2.+`）
- 周期性执行 SCA（依赖漏洞扫描）
- 对关键依赖记录 SBOM（软件物料清单）

## 七、反欺诈与风控（可选增强）

- 风险设备识别：root、hook、模拟器、调试器
- 行为风控：登录、支付、提现链路增加风控策略
- 接口签名：时间戳 + nonce + HMAC，防重放攻击

## 八、安全测试与验收标准

### 8.1 测试矩阵
- 静态检测：Manifest、导出组件、明文日志、硬编码密钥
- 动态检测：抓包、MITM、重放、深链注入、越权访问
- 回归测试：高风险路径（登录、支付、隐私数据访问）

### 8.2 可执行验收门禁
- 发现明文 token 存储：阻断发布
- 发现导出组件无权限保护：阻断发布
- 发现 debug 配置进入 release 包：阻断发布

## 九、常见误区

- 误区 1：混淆 = 安全
  说明：混淆只提升逆向成本，无法替代鉴权和数据保护。

- 误区 2：只做客户端校验
  说明：所有关键校验必须服务端再验证，客户端只能提升体验。

- 误区 3：只关注代码，不关注流程
  说明：很多高风险问题发生在配置、发布和依赖环节。

## 十、面试深问与答题框架

- 问：如何落地 Android 安全体系？
  答题框架：威胁建模 -> 资产分级 -> 控制措施 -> 验收门禁 -> 线上监控。

- 问：你如何处理证书 Pinning 失效？
  答题框架：双 pin 轮转 + 灰度发布 + 紧急降级策略（受控开关）+ 审计日志。

- 问：客户端存储 token 的最佳实践？
  答题框架：最小化存储、Keystore 托管、短生命周期、失效机制、服务端撤销能力。
