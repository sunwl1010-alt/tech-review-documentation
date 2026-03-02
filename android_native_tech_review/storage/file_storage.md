# 文件存储

## 核心概念

### 文件存储
- **定义**：Android系统中用于存储文件的机制
- **作用**：存储大文件、二进制数据、缓存等
- **类型**：
  - 内部存储：应用私有目录，无需权限
  - 外部存储：公共目录，需要权限
  - 缓存存储：临时文件，可能被系统清理

### 内部存储
- **路径**：/data/data/包名/files/
- **特点**：
  - 应用私有，其他应用无法访问
  - 应用卸载时自动删除
  - 无需权限
  - 存储空间有限

### 外部存储
- **路径**：/storage/emulated/0/Android/data/包名/files/（应用私有）
- **路径**：/storage/emulated/0/（公共目录）
- **特点**：
  - 应用私有目录：应用卸载时自动删除
  - 公共目录：应用卸载时不会删除
  - 需要存储权限
  - 存储空间较大

### 缓存存储
- **内部缓存**：/data/data/包名/cache/
- **外部缓存**：/storage/emulated/0/Android/data/包名/cache/
- **特点**：
  - 临时存储，可能被系统清理
  - 应用卸载时自动删除
  - 内部缓存无需权限，外部缓存需要权限

## 实现原理

### 文件操作
- **文件创建**：使用File类创建文件
- **文件读写**：使用FileInputStream、FileOutputStream、BufferedReader、BufferedWriter等
- **文件删除**：使用File.delete()
- **文件重命名**：使用File.renameTo()

### 权限管理
- **存储权限**：
  - Android 6.0以下：在AndroidManifest.xml中声明
  - Android 6.0及以上：需要运行时权限请求
- **权限类型**：
  - READ_EXTERNAL_STORAGE：读取外部存储
  - WRITE_EXTERNAL_STORAGE：写入外部存储
  - MANAGE_EXTERNAL_STORAGE：管理外部存储（Android 10+）

### 存储适配
- **Android 4.4**：引入getExternalFilesDir()和getExternalCacheDir()
- **Android 6.0**：引入运行时权限
- **Android 10**：引入分区存储，限制对外部存储的访问
- **Android 11**：强化分区存储，默认启用

### 文件访问模式
- **MODE_PRIVATE**：默认模式，只有当前应用可以访问
- **MODE_WORLD_READABLE**：已废弃，其他应用可以读取
- **MODE_WORLD_WRITEABLE**：已废弃，其他应用可以写入

## 使用场景

### 内部存储
- **应用配置文件**：存储应用的配置信息
- **用户数据**：存储用户的个人数据
- **敏感数据**：存储需要保密的数据
- **小文件**：存储体积较小的文件

### 外部存储
- **大文件**：存储体积较大的文件，如视频、音频等
- **共享文件**：需要与其他应用共享的文件
- **媒体文件**：图片、视频、音频等
- **下载文件**：从网络下载的文件

### 缓存存储
- **临时文件**：临时使用的文件
- **网络缓存**：网络请求的缓存
- **图片缓存**：加载的图片缓存
- **计算结果**：临时计算结果

### 特定目录
- **文档目录**：存储文档文件
- **图片目录**：存储图片文件
- **音频目录**：存储音频文件
- **视频目录**：存储视频文件

## 常见问题及解决方案

### 权限问题
- **问题**：Android 6.0及以上版本权限被拒绝
  **解决方案**：使用运行时权限请求

- **问题**：Android 10+无法访问外部存储
  **解决方案**：使用分区存储API，或请求MANAGE_EXTERNAL_STORAGE权限

- **问题**：应用崩溃，提示权限被拒绝
  **解决方案**：确保正确请求和处理权限

### 存储适配问题
- **问题**：不同Android版本的存储API不同
  **解决方案**：适配不同版本的存储API，使用兼容库

- **问题**：分区存储导致文件访问失败
  **解决方案**：使用MediaStore或Storage Access Framework

- **问题**：外部存储不可用
  **解决方案**：检查外部存储状态，使用内部存储作为备选

### 性能问题
- **问题**：文件读写速度慢
  **解决方案**：使用缓冲流，异步读写，避免主线程操作

- **问题**：文件过大导致内存不足
  **解决方案**：使用流式读写，避免一次性加载整个文件

- **问题**：频繁读写文件影响性能
  **解决方案**：批量操作，使用缓存

### 安全问题
- **问题**：敏感数据存储不安全
  **解决方案**：加密存储，使用内部存储

- **问题**：外部存储文件被其他应用访问
  **解决方案**：使用内部存储，或加密文件

- **问题**：应用卸载后数据泄露
  **解决方案**：使用内部存储，或在卸载时清理外部存储数据

## 代码示例

### 内部存储
```java
// 写入文件
public void writeToInternalStorage(Context context, String fileName, String content) {
    try {
        FileOutputStream fos = context.openFileOutput(fileName, Context.MODE_PRIVATE);
        fos.write(content.getBytes());
        fos.close();
    } catch (IOException e) {
        e.printStackTrace();
    }
}

// 读取文件
public String readFromInternalStorage(Context context, String fileName) {
    StringBuilder content = new StringBuilder();
    try {
        FileInputStream fis = context.openFileInput(fileName);
        BufferedReader br = new BufferedReader(new InputStreamReader(fis));
        String line;
        while ((line = br.readLine()) != null) {
            content.append(line).append("\n");
        }
        br.close();
        fis.close();
    } catch (IOException e) {
        e.printStackTrace();
    }
    return content.toString();
}

// 列出内部存储文件
public void listInternalFiles(Context context) {
    String[] files = context.fileList();
    for (String file : files) {
        Log.d("FileStorage", "File: " + file);
    }
}

// 删除内部存储文件
public void deleteInternalFile(Context context, String fileName) {
    context.deleteFile(fileName);
}
```

### 外部存储
```java
// 检查外部存储是否可用
public boolean isExternalStorageWritable() {
    String state = Environment.getExternalStorageState();
    return Environment.MEDIA_MOUNTED.equals(state);
}

public boolean isExternalStorageReadable() {
    String state = Environment.getExternalStorageState();
    return Environment.MEDIA_MOUNTED.equals(state) || Environment.MEDIA_MOUNTED_READ_ONLY.equals(state);
}

// 获取外部存储应用私有目录
public File getExternalFilesDir(Context context, String type) {
    return context.getExternalFilesDir(type);
}

// 写入外部存储
public void writeToExternalStorage(Context context, String fileName, String content) {
    if (isExternalStorageWritable()) {
        File file = new File(context.getExternalFilesDir(null), fileName);
        try {
            FileOutputStream fos = new FileOutputStream(file);
            fos.write(content.getBytes());
            fos.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}

// 读取外部存储
public String readFromExternalStorage(Context context, String fileName) {
    if (isExternalStorageReadable()) {
        File file = new File(context.getExternalFilesDir(null), fileName);
        StringBuilder content = new StringBuilder();
        try {
            FileInputStream fis = new FileInputStream(file);
            BufferedReader br = new BufferedReader(new InputStreamReader(fis));
            String line;
            while ((line = br.readLine()) != null) {
                content.append(line).append("\n");
            }
            br.close();
            fis.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
        return content.toString();
    }
    return "";
}
```

### 缓存存储
```java
// 获取内部缓存目录
public File getInternalCacheDir(Context context) {
    return context.getCacheDir();
}

// 获取外部缓存目录
public File getExternalCacheDir(Context context) {
    return context.getExternalCacheDir();
}

// 写入缓存
public void writeToCache(Context context, String fileName, String content) {
    File cacheDir = context.getCacheDir();
    File file = new File(cacheDir, fileName);
    try {
        FileOutputStream fos = new FileOutputStream(file);
        fos.write(content.getBytes());
        fos.close();
    } catch (IOException e) {
        e.printStackTrace();
    }
}

// 清理缓存
public void clearCache(Context context) {
    File cacheDir = context.getCacheDir();
    if (cacheDir != null && cacheDir.isDirectory()) {
        for (File file : cacheDir.listFiles()) {
            file.delete();
        }
    }
    
    File externalCacheDir = context.getExternalCacheDir();
    if (externalCacheDir != null && externalCacheDir.isDirectory()) {
        for (File file : externalCacheDir.listFiles()) {
            file.delete();
        }
    }
}
```

### 权限请求
```java
// 检查权限
public boolean checkStoragePermission(Context context) {
    if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.M) {
        return ContextCompat.checkSelfPermission(context, Manifest.permission.WRITE_EXTERNAL_STORAGE) == PackageManager.PERMISSION_GRANTED;
    }
    return true;
}

// 请求权限
public void requestStoragePermission(Activity activity) {
    if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.M) {
        ActivityCompat.requestPermissions(activity, new String[]{Manifest.permission.WRITE_EXTERNAL_STORAGE}, 1);
    }
}

// 处理权限回调
@Override
public void onRequestPermissionsResult(int requestCode, @NonNull String[] permissions, @NonNull int[] grantResults) {
    super.onRequestPermissionsResult(requestCode, permissions, grantResults);
    if (requestCode == 1) {
        if (grantResults.length > 0 && grantResults[0] == PackageManager.PERMISSION_GRANTED) {
            // 权限授予
            Log.d("Permission", "Storage permission granted");
        } else {
            // 权限拒绝
            Log.d("Permission", "Storage permission denied");
        }
    }
}
```

### 分区存储适配
```java
// Android 10+ 存储适配
public void saveFile(Context context, String fileName, byte[] data) {
    if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.Q) {
        // 使用MediaStore
        ContentResolver resolver = context.getContentResolver();
        ContentValues values = new ContentValues();
        values.put(MediaStore.MediaColumns.DISPLAY_NAME, fileName);
        values.put(MediaStore.MediaColumns.MIME_TYPE, "application/octet-stream");
        values.put(MediaStore.MediaColumns.RELATIVE_PATH, Environment.DIRECTORY_DOCUMENTS);
        
        Uri uri = resolver.insert(MediaStore.Files.getContentUri(MediaStore.VOLUME_EXTERNAL_PRIMARY), values);
        if (uri != null) {
            try {
                OutputStream outputStream = resolver.openOutputStream(uri);
                if (outputStream != null) {
                    outputStream.write(data);
                    outputStream.close();
                }
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    } else {
        // 使用传统方式
        File file = new File(context.getExternalFilesDir(Environment.DIRECTORY_DOCUMENTS), fileName);
        try {
            FileOutputStream fos = new FileOutputStream(file);
            fos.write(data);
            fos.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

## 面试常考问题及参考答案

### 基础理论

**1. Android中的文件存储有哪些类型？它们的区别是什么？**

**答案**：
- **内部存储**：应用私有目录，无需权限，应用卸载时自动删除，存储空间有限
- **外部存储**：公共目录，需要权限，应用私有目录在应用卸载时自动删除，公共目录不会，存储空间较大
- **缓存存储**：临时文件，可能被系统清理，应用卸载时自动删除

**2. 内部存储和外部存储的区别是什么？**

**答案**：
- **内部存储**：
  - 路径：/data/data/包名/files/
  - 特点：应用私有，无需权限，应用卸载时自动删除，存储空间有限
  - 适用场景：存储敏感数据、应用配置文件

- **外部存储**：
  - 路径：/storage/emulated/0/Android/data/包名/files/（应用私有）或/storage/emulated/0/（公共目录）
  - 特点：需要权限，存储空间较大，应用私有目录在应用卸载时自动删除
  - 适用场景：存储大文件、共享文件、媒体文件

**3. 如何检查外部存储是否可用？**

**答案**：使用Environment.getExternalStorageState()方法检查外部存储状态：
- Environment.MEDIA_MOUNTED：外部存储可用且可读写
- Environment.MEDIA_MOUNTED_READ_ONLY：外部存储可用但只读
- 其他状态：外部存储不可用

### 实际应用

**4. 如何在Android 6.0及以上版本请求存储权限？**

**答案**：
1. 在AndroidManifest.xml中声明权限
2. 使用ContextCompat.checkSelfPermission()检查权限
3. 使用ActivityCompat.requestPermissions()请求权限
4. 在onRequestPermissionsResult()中处理权限回调

**5. 如何处理Android 10+的分区存储？**

**答案**：
- 使用MediaStore API存储媒体文件
- 使用Storage Access Framework存储非媒体文件
- 使用应用私有目录存储应用数据
- 对于特殊需求，请求MANAGE_EXTERNAL_STORAGE权限

**6. 如何实现文件的读写操作？**

**答案**：
- 写入文件：使用FileOutputStream或BufferedWriter
- 读取文件：使用FileInputStream或BufferedReader
- 对于大文件，使用流式读写避免内存溢出

### 性能优化

**7. 如何优化文件读写性能？**

**答案**：
- 使用缓冲流（BufferedReader、BufferedWriter）
- 异步读写，避免阻塞主线程
- 批量操作，减少I/O次数
- 使用内存映射文件（MappedByteBuffer）处理大文件
- 合理使用缓存

**8. 如何处理大文件的读写？**

**答案**：
- 使用流式读写，避免一次性加载整个文件到内存
- 分块读取和写入
- 使用内存映射文件
- 异步操作，避免阻塞主线程
- 监控内存使用，避免OOM

### 安全问题

**9. 如何安全存储敏感数据？**

**答案**：
- 使用内部存储，避免外部存储
- 加密存储数据
- 使用KeyStore存储加密密钥
- 避免存储明文密码等敏感信息
- 应用卸载时清理数据

**10. 如何确保应用卸载后数据被删除？**

**答案**：
- 使用内部存储
- 使用外部存储的应用私有目录（/storage/emulated/0/Android/data/包名/）
- 在应用退出时清理外部存储的公共目录数据
- 使用File.delete()删除文件

### 架构设计

**11. 如何设计一个可扩展的文件存储架构？**

**答案**：
- 封装文件操作工具类
- 支持不同存储类型（内部、外部、缓存）
- 适配不同Android版本
- 提供异步操作接口
- 支持文件加密
- 提供缓存管理

**12. 文件存储与其他存储方式的对比？**

**答案**：
- **文件存储**：适合存储大文件、二进制数据，灵活但查询效率低
- **SharedPreferences**：适合存储轻量级键值对，简单易用但不适合大量数据
- **SQLite**：适合存储结构化数据，支持复杂查询但操作复杂
- **Room**：SQLite的封装，提供更友好的API
- **ContentProvider**：适合跨应用数据共享

根据数据类型和使用场景选择合适的存储方式。对于大文件和二进制数据，文件存储是最佳选择。