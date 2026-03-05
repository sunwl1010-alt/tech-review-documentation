# 安全性

## 权限管理

### 权限类型

- **正常权限**：不会直接威胁用户隐私的权限，系统自动授予
- **危险权限**：可能涉及用户隐私或设备安全的权限，需要用户明确授予
- **特殊权限**：需要特殊申请流程的权限，如 SYSTEM_ALERT_WINDOW、WRITE_SETTINGS

### 权限申请流程

```kotlin
// 检查权限
if (ContextCompat.checkSelfPermission(context, Manifest.permission.CAMERA) != PackageManager.PERMISSION_GRANTED) {
    // 申请权限
    ActivityCompat.requestPermissions(activity, arrayOf(Manifest.permission.CAMERA), REQUEST_CAMERA_PERMISSION)
}

// 处理权限申请结果
override fun onRequestPermissionsResult(requestCode: Int, permissions: Array<String>, grantResults: IntArray) {
    when (requestCode) {
        REQUEST_CAMERA_PERMISSION -> {
            if (grantResults.isNotEmpty() && grantResults[0] == PackageManager.PERMISSION_GRANTED) {
                // 权限授予成功
                openCamera()
            } else {
                // 权限授予失败
                Toast.makeText(this, "Camera permission denied", Toast.LENGTH_SHORT).show()
            }
            return
        }
    }
}
```

### 权限说明

```kotlin
// 检查是否需要显示权限说明
if (ActivityCompat.shouldShowRequestPermissionRationale(activity, Manifest.permission.CAMERA)) {
    // 显示权限说明
    AlertDialog.Builder(activity)
        .setTitle("Permission Needed")
        .setMessage("Camera permission is needed to take photos")
        .setPositiveButton("OK") { _, _ ->
            ActivityCompat.requestPermissions(activity, arrayOf(Manifest.permission.CAMERA), REQUEST_CAMERA_PERMISSION)
        }
        .setNegativeButton("Cancel") { _, _ -> }
        .show()
}
```

### Android 13+ 权限变更

- **通知权限**：需要单独申请 `POST_NOTIFICATIONS` 权限
- **媒体权限**：细化为 `READ_MEDIA_IMAGES`、`READ_MEDIA_VIDEO`、`READ_MEDIA_AUDIO`
- **运行时权限**：增加了更多运行时权限，如 `NEARBY_WIFI_DEVICES`

## 数据加密

### 对称加密

```kotlin
// 使用 AES 加密
fun encryptData(data: String, key: String): String {
    val keySpec = SecretKeySpec(key.toByteArray(), "AES")
    val cipher = Cipher.getInstance("AES/CBC/PKCS5Padding")
    val iv = ByteArray(16)
    SecureRandom().nextBytes(iv)
    cipher.init(Cipher.ENCRYPT_MODE, keySpec, IvParameterSpec(iv))
    val encrypted = cipher.doFinal(data.toByteArray())
    return Base64.encodeToString(iv + encrypted, Base64.DEFAULT)
}

fun decryptData(encryptedData: String, key: String): String {
    val keySpec = SecretKeySpec(key.toByteArray(), "AES")
    val cipher = Cipher.getInstance("AES/CBC/PKCS5Padding")
    val data = Base64.decode(encryptedData, Base64.DEFAULT)
    val iv = data.copyOfRange(0, 16)
    val encrypted = data.copyOfRange(16, data.size)
    cipher.init(Cipher.DECRYPT_MODE, keySpec, IvParameterSpec(iv))
    val decrypted = cipher.doFinal(encrypted)
    return String(decrypted)
}
```

### 非对称加密

```kotlin
// 生成 RSA 密钥对
fun generateRSAKeyPair(): KeyPair {
    val keyPairGenerator = KeyPairGenerator.getInstance("RSA")
    keyPairGenerator.initialize(2048)
    return keyPairGenerator.generateKeyPair()
}

// 使用 RSA 加密
fun encryptWithRSA(data: String, publicKey: PublicKey): String {
    val cipher = Cipher.getInstance("RSA/ECB/PKCS1Padding")
    cipher.init(Cipher.ENCRYPT_MODE, publicKey)
    val encrypted = cipher.doFinal(data.toByteArray())
    return Base64.encodeToString(encrypted, Base64.DEFAULT)
}

// 使用 RSA 解密
fun decryptWithRSA(encryptedData: String, privateKey: PrivateKey): String {
    val cipher = Cipher.getInstance("RSA/ECB/PKCS1Padding")
    cipher.init(Cipher.DECRYPT_MODE, privateKey)
    val encrypted = Base64.decode(encryptedData, Base64.DEFAULT)
    val decrypted = cipher.doFinal(encrypted)
    return String(decrypted)
}
```

### Android KeyStore

```kotlin
// 生成密钥
fun generateKey(keyAlias: String) {
    val keyGenerator = KeyGenerator.getInstance(KeyProperties.KEY_ALGORITHM_AES, "AndroidKeyStore")
    val spec = KeyGenParameterSpec.Builder(
        keyAlias,
        KeyProperties.PURPOSE_ENCRYPT or KeyProperties.PURPOSE_DECRYPT
    )
        .setBlockModes(KeyProperties.BLOCK_MODE_CBC)
        .setEncryptionPaddings(KeyProperties.ENCRYPTION_PADDING_PKCS7)
        .setUserAuthenticationRequired(false)
        .build()
    keyGenerator.init(spec)
    keyGenerator.generateKey()
}

// 使用 KeyStore 加密
fun encryptWithKeyStore(data: String, keyAlias: String): String {
    val keyStore = KeyStore.getInstance("AndroidKeyStore")
    keyStore.load(null)
    val secretKey = keyStore.getKey(keyAlias, null) as SecretKey
    
    val cipher = Cipher.getInstance("AES/CBC/PKCS7Padding")
    cipher.init(Cipher.ENCRYPT_MODE, secretKey)
    val iv = cipher.iv
    val encrypted = cipher.doFinal(data.toByteArray())
    return Base64.encodeToString(iv + encrypted, Base64.DEFAULT)
}

// 使用 KeyStore 解密
fun decryptWithKeyStore(encryptedData: String, keyAlias: String): String {
    val keyStore = KeyStore.getInstance("AndroidKeyStore")
    keyStore.load(null)
    val secretKey = keyStore.getKey(keyAlias, null) as SecretKey
    
    val data = Base64.decode(encryptedData, Base64.DEFAULT)
    val iv = data.copyOfRange(0, 16)
    val encrypted = data.copyOfRange(16, data.size)
    
    val cipher = Cipher.getInstance("AES/CBC/PKCS7Padding")
    cipher.init(Cipher.DECRYPT_MODE, secretKey, IvParameterSpec(iv))
    val decrypted = cipher.doFinal(encrypted)
    return String(decrypted)
}
```

## 网络安全

### HTTPS

```kotlin
// 使用 OkHttp 配置 HTTPS
val client = OkHttpClient.Builder()
    .sslSocketFactory(sslSocketFactory, trustManager)
    .build()

// 自定义 SSL 配置
val trustManager = object : X509TrustManager {
    override fun checkClientTrusted(chain: Array<out X509Certificate>?, authType: String?) {}
    override fun checkServerTrusted(chain: Array<out X509Certificate>?, authType: String?) {
        // 验证服务器证书
    }
    override fun getAcceptedIssuers(): Array<X509Certificate> = arrayOf()
}

val sslContext = SSLContext.getInstance("TLS")
sslContext.init(null, arrayOf(trustManager), SecureRandom())
val sslSocketFactory = sslContext.socketFactory
```

### 网络安全配置

在 `res/xml/network_security_config.xml` 中配置：

```xml
<?xml version="1.0" encoding="utf-8"?>
<network-security-config>
    <base-config cleartextTrafficPermitted="false">
        <trust-anchors>
            <certificates src="system"/>
            <certificates src="user"/>
        </trust-anchors>
    </base-config>
    <domain-config cleartextTrafficPermitted="true">
        <domain includeSubdomains="true">localhost</domain>
        <domain includeSubdomains="true">10.0.2.2</domain>
    </domain-config>
</network-security-config>
```

在 `AndroidManifest.xml` 中引用：

```xml
<application
    android:networkSecurityConfig="@xml/network_security_config"
    ...>
</application>
```

### 防止网络攻击

1. **防止 SQL 注入**：使用参数化查询
2. **防止 XSS 攻击**：对输入进行验证和转义
3. **防止 CSRF 攻击**：使用 CSRF Token
4. **防止中间人攻击**：使用 HTTPS，验证证书
5. **防止 DDoS 攻击**：限制请求频率

## 应用签名

### 签名的作用

- **应用标识**：确保应用的唯一性
- **应用完整性**：防止应用被篡改
- **权限授予**：系统基于签名授予权限
- **应用更新**：只有相同签名的应用才能更新

### 生成签名密钥

```bash
# 使用 keytool 生成密钥
keytool -genkey -v -keystore my-release-key.keystore -alias my-key-alias -keyalg RSA -keysize 2048 -validity 10000
```

### 配置签名

在 `build.gradle` 中配置：

```gradle
signingConfigs {
    release {
        storeFile file("my-release-key.keystore")
        storePassword "password"
        keyAlias "my-key-alias"
        keyPassword "password"
    }
}

buildTypes {
    release {
        signingConfig signingConfigs.release
        // 其他配置
    }
}
```

### 签名验证

```kotlin
// 获取应用签名
fun getAppSignature(context: Context): String {
    val packageName = context.packageName
    val packageInfo = context.packageManager.getPackageInfo(packageName, PackageManager.GET_SIGNATURES)
    val signatures = packageInfo.signatures
    val md = MessageDigest.getInstance("SHA1")
    for (signature in signatures) {
        md.update(signature.toByteArray())
    }
    return Base64.encodeToString(md.digest(), Base64.DEFAULT)
}

// 验证签名
fun verifyAppSignature(context: Context, expectedSignature: String): Boolean {
    val actualSignature = getAppSignature(context)
    return actualSignature == expectedSignature
}
```

### Android App Bundle

- **优势**：减小应用体积，支持动态功能模块
- **签名**：使用 Google Play App Signing 管理签名密钥
- **配置**：在 `build.gradle` 中启用 App Bundle

## 最佳实践

1. **最小权限原则**：只申请必要的权限
2. **加密敏感数据**：使用 Android KeyStore 存储密钥
3. **使用 HTTPS**：所有网络请求使用 HTTPS
4. **验证服务器证书**：防止中间人攻击
5. **安全的密码存储**：使用 bcrypt 或 Argon2 等算法
6. **定期更新依赖**：修复已知安全漏洞
7. **代码混淆**：使用 ProGuard 或 R8 混淆代码
8. **安全测试**：定期进行安全测试和漏洞扫描

## 常见问题

1. **权限被拒绝**：用户拒绝授予权限，需要优雅处理
2. **证书验证失败**：服务器证书无效或过期
3. **密钥管理**：密钥丢失或泄露
4. **应用被篡改**：签名验证失败
5. **网络攻击**：SQL 注入、XSS 等攻击

## 面试题

1. **Q**: Android 权限分为哪几类？
   **A**: 正常权限、危险权限、特殊权限

2. **Q**: 如何安全存储敏感数据？
   **A**: 使用 Android KeyStore 存储密钥，对敏感数据进行加密

3. **Q**: 如何防止中间人攻击？
   **A**: 使用 HTTPS，验证服务器证书，配置网络安全策略

4. **Q**: 应用签名的作用是什么？
   **A**: 确保应用唯一性、完整性，防止篡改，支持应用更新

5. **Q**: 如何处理权限被拒绝的情况？
   **A**: 显示权限说明，引导用户在设置中开启权限，提供替代功能