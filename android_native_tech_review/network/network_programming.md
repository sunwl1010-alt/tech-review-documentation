# 网络编程

## 核心概念

网络编程是 Android 应用中与服务器通信的重要技术，用于获取数据、提交数据、实时通信等场景。Android 提供了多种网络编程方案，包括 HttpURLConnection、OkHttp、Retrofit、Volley 等。

## 实现原理

Android 网络编程的基本原理是通过 HTTP/HTTPS 协议与服务器进行通信，主要包括以下步骤：

1. **建立连接**: 与服务器建立 TCP 连接
2. **发送请求**: 发送 HTTP 请求（GET、POST、PUT、DELETE 等）
3. **接收响应**: 接收服务器返回的响应数据
4. **解析数据**: 解析响应数据（JSON、XML 等格式）
5. **处理结果**: 根据响应结果进行相应的处理

## 基本使用

### 1. HttpURLConnection

```java
import java.io.BufferedReader;
import java.io.InputStream;
import java.io.InputStreamReader;
import java.net.HttpURLConnection;
import java.net.URL;

public class HttpURLConnectionExample {
    public static void sendGetRequest() {
        new Thread(new Runnable() {
            @Override
            public void run() {
                try {
                    URL url = new URL("https://api.example.com/data");
                    HttpURLConnection connection = (HttpURLConnection) url.openConnection();
                    connection.setRequestMethod("GET");
                    connection.setConnectTimeout(5000);
                    connection.setReadTimeout(5000);
                    
                    int responseCode = connection.getResponseCode();
                    if (responseCode == HttpURLConnection.HTTP_OK) {
                        InputStream inputStream = connection.getInputStream();
                        BufferedReader reader = new BufferedReader(new InputStreamReader(inputStream));
                        StringBuilder response = new StringBuilder();
                        String line;
                        while ((line = reader.readLine()) != null) {
                            response.append(line);
                        }
                        reader.close();
                        inputStream.close();
                        
                        // 处理响应数据
                        String responseData = response.toString();
                        System.out.println("Response: " + responseData);
                    }
                    connection.disconnect();
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
        }).start();
    }
    
    public static void sendPostRequest() {
        new Thread(new Runnable() {
            @Override
            public void run() {
                try {
                    URL url = new URL("https://api.example.com/data");
                    HttpURLConnection connection = (HttpURLConnection) url.openConnection();
                    connection.setRequestMethod("POST");
                    connection.setConnectTimeout(5000);
                    connection.setReadTimeout(5000);
                    connection.setDoOutput(true);
                    connection.setRequestProperty("Content-Type", "application/json");
                    
                    String jsonInputString = "{\"name\": \"John\", \"email\": \"john@example.com\"}";
                    connection.getOutputStream().write(jsonInputString.getBytes());
                    
                    int responseCode = connection.getResponseCode();
                    if (responseCode == HttpURLConnection.HTTP_OK) {
                        InputStream inputStream = connection.getInputStream();
                        BufferedReader reader = new BufferedReader(new InputStreamReader(inputStream));
                        StringBuilder response = new StringBuilder();
                        String line;
                        while ((line = reader.readLine()) != null) {
                            response.append(line);
                        }
                        reader.close();
                        inputStream.close();
                        
                        // 处理响应数据
                        String responseData = response.toString();
                        System.out.println("Response: " + responseData);
                    }
                    connection.disconnect();
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
        }).start();
    }
}
```

### 2. OkHttp

```java
import okhttp3.Call;
import okhttp3.Callback;
import okhttp3.MediaType;
import okhttp3.OkHttpClient;
import okhttp3.Request;
import okhttp3.RequestBody;
import okhttp3.Response;

public class OkHttpExample {
    private static final OkHttpClient client = new OkHttpClient();
    private static final MediaType JSON = MediaType.parse("application/json;
 charset=utf-8");
    
    public static void sendGetRequest() {
        Request request = new Request.Builder()
            .url("https://api.example.com/data")
            .build();
        
        client.newCall(request).enqueue(new Callback() {
            @Override
            public void onFailure(Call call, IOException e) {
                e.printStackTrace();
            }
            
            @Override
            public void onResponse(Call call, Response response) throws IOException {
                if (response.isSuccessful()) {
                    String responseData = response.body().string();
                    System.out.println("Response: " + responseData);
                }
            }
        });
    }
    
    public static void sendPostRequest() {
        String json = "{\"name\": \"John\", \"email\": \"john@example.com\"}";
        RequestBody body = RequestBody.create(JSON, json);
        
        Request request = new Request.Builder()
            .url("https://api.example.com/data")
            .post(body)
            .build();
        
        client.newCall(request).enqueue(new Callback() {
            @Override
            public void onFailure(Call call, IOException e) {
                e.printStackTrace();
            }
            
            @Override
            public void onResponse(Call call, Response response) throws IOException {
                if (response.isSuccessful()) {
                    String responseData = response.body().string();
                    System.out.println("Response: " + responseData);
                }
            }
        });
    }
}
```

### 3. Retrofit

```java
// API 接口定义
public interface ApiService {
    @GET("/data")
    Call<User> getUser();
    
    @POST("/data")
    Call<User> createUser(@Body User user);
}

// 数据模型
public class User {
    private String name;
    private String email;
    
    // 构造方法、getter、setter 省略
}

// Retrofit 配置
public class RetrofitClient {
    private static final String BASE_URL = "https://api.example.com";
    private static Retrofit retrofit;
    
    public static Retrofit getRetrofitInstance() {
        if (retrofit == null) {
            retrofit = new Retrofit.Builder()
                .baseUrl(BASE_URL)
                .addConverterFactory(GsonConverterFactory.create())
                .build();
        }
        return retrofit;
    }
    
    public static ApiService getApiService() {
        return getRetrofitInstance().create(ApiService.class);
    }
}

// 使用示例
public class RetrofitExample {
    public static void getUser() {
        ApiService apiService = RetrofitClient.getApiService();
        Call<User> call = apiService.getUser();
        
        call.enqueue(new Callback<User>() {
            @Override
            public void onResponse(Call<User> call, Response<User> response) {
                if (response.isSuccessful()) {
                    User user = response.body();
                    System.out.println("User: " + user.getName());
                }
            }
            
            @Override
            public void onFailure(Call<User> call, Throwable t) {
                t.printStackTrace();
            }
        });
    }
    
    public static void createUser() {
        ApiService apiService = RetrofitClient.getApiService();
        User user = new User("John", "john@example.com");
        Call<User> call = apiService.createUser(user);
        
        call.enqueue(new Callback<User>() {
            @Override
            public void onResponse(Call<User> call, Response<User> response) {
                if (response.isSuccessful()) {
                    User createdUser = response.body();
                    System.out.println("Created User: " + createdUser.getName());
                }
            }
            
            @Override
            public void onFailure(Call<User> call, Throwable t) {
                t.printStackTrace();
            }
        });
    }
}
```

### 4. Volley

```java
import com.android.volley.Request;
import com.android.volley.RequestQueue;
import com.android.volley.Response;
import com.android.volley.VolleyError;
import com.android.volley.toolbox.JsonObjectRequest;
import com.android.volley.toolbox.StringRequest;
import com.android.volley.toolbox.Volley;

import org.json.JSONObject;

public class VolleyExample {
    private static RequestQueue requestQueue;
    
    public static void init(Context context) {
        requestQueue = Volley.newRequestQueue(context);
    }
    
    public static void sendGetRequest() {
        String url = "https://api.example.com/data";
        
        StringRequest stringRequest = new StringRequest(Request.Method.GET, url, 
            new Response.Listener<String>() {
                @Override
                public void onResponse(String response) {
                    System.out.println("Response: " + response);
                }
            }, 
            new Response.ErrorListener() {
                @Override
                public void onErrorResponse(VolleyError error) {
                    error.printStackTrace();
                }
            });
        
        requestQueue.add(stringRequest);
    }
    
    public static void sendPostRequest() {
        String url = "https://api.example.com/data";
        
        try {
            JSONObject jsonObject = new JSONObject();
            jsonObject.put("name", "John");
            jsonObject.put("email", "john@example.com");
            
            JsonObjectRequest jsonObjectRequest = new JsonObjectRequest(Request.Method.POST, url, jsonObject, 
                new Response.Listener<JSONObject>() {
                    @Override
                    public void onResponse(JSONObject response) {
                        System.out.println("Response: " + response.toString());
                    }
                }, 
                new Response.ErrorListener() {
                    @Override
                    public void onErrorResponse(VolleyError error) {
                        error.printStackTrace();
                    }
                });
            
            requestQueue.add(jsonObjectRequest);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

## 高级特性

### 1. 网络拦截器

```java
// OkHttp 拦截器
class LoggingInterceptor implements Interceptor {
    @Override
    public Response intercept(Chain chain) throws IOException {
        Request request = chain.request();
        
        long t1 = System.nanoTime();
        System.out.println(String.format("Sending request %s on %s%n%s",
            request.url(), chain.connection(), request.headers()));
        
        Response response = chain.proceed(request);
        
        long t2 = System.nanoTime();
        System.out.println(String.format("Received response for %s in %.1fms%n%s",
            response.request().url(), (t2 - t1) / 1e6d, response.headers()));
        
        return response;
    }
}

// 添加拦截器
OkHttpClient client = new OkHttpClient.Builder()
    .addInterceptor(new LoggingInterceptor())
    .build();
```

### 2. 网络缓存

```java
// OkHttp 缓存
int cacheSize = 10 * 1024 * 1024; // 10 MiB
Cache cache = new Cache(context.getCacheDir(), cacheSize);

OkHttpClient client = new OkHttpClient.Builder()
    .cache(cache)
    .build();

// 缓存控制
Request request = new Request.Builder()
    .url("https://api.example.com/data")
    .header("Cache-Control", "max-age=60") // 缓存 60 秒
    .build();
```

### 3. WebSocket

```java
import okhttp3.OkHttpClient;
import okhttp3.Request;
import okhttp3.WebSocket;
import okhttp3.WebSocketListener;
import okio.ByteString;

public class WebSocketExample {
    private WebSocket webSocket;
    
    public void connect() {
        OkHttpClient client = new OkHttpClient();
        Request request = new Request.Builder()
            .url("wss://echo.websocket.org")
            .build();
        
        webSocket = client.newWebSocket(request, new WebSocketListener() {
            @Override
            public void onOpen(WebSocket webSocket, okhttp3.Response response) {
                // 连接成功
                webSocket.send("Hello WebSocket!");
            }
            
            @Override
            public void onMessage(WebSocket webSocket, String text) {
                // 接收文本消息
                System.out.println("Received: " + text);
            }
            
            @Override
            public void onMessage(WebSocket webSocket, ByteString bytes) {
                // 接收二进制消息
                System.out.println("Received bytes: " + bytes);
            }
            
            @Override
            public void onClosing(WebSocket webSocket, int code, String reason) {
                // 连接关闭中
                webSocket.close(1000, null);
            }
            
            @Override
            public void onFailure(WebSocket webSocket, Throwable t, okhttp3.Response response) {
                // 连接失败
                t.printStackTrace();
            }
        });
    }
    
    public void sendMessage(String message) {
        if (webSocket != null) {
            webSocket.send(message);
        }
    }
    
    public void close() {
        if (webSocket != null) {
            webSocket.close(1000, "Closing");
        }
    }
}
```

### 4. 网络安全

```java
// HTTPS 证书固定
public class CertificatePinning {
    public static OkHttpClient getPinnedClient() {
        try {
            CertificatePinner certificatePinner = new CertificatePinner.Builder()
                .add("api.example.com", "sha256/AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA=")
                .build();
            
            return new OkHttpClient.Builder()
                .certificatePinner(certificatePinner)
                .build();
        } catch (Exception e) {
            e.printStackTrace();
            return new OkHttpClient();
        }
    }
}

// 网络安全配置
// 在 res/xml/network_security_config.xml 中
/*
<?xml version="1.0" encoding="utf-8"?>
<network-security-config>
    <domain-config>
        <domain includeSubdomains="true">example.com</domain>
        <pin-set>
            <pin digest="SHA-256">AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA=</pin>
        </pin-set>
    </domain-config>
</network-security-config>
*/

// 在 AndroidManifest.xml 中
/*
<application
    android:networkSecurityConfig="@xml/network_security_config"
    ...>
*/
```

## 最佳实践

1. **选择合适的网络库**:
   - 简单请求: HttpURLConnection
   - 复杂请求: OkHttp
   - RESTful API: Retrofit
   - 频繁小请求: Volley

2. **网络操作在后台线程**:
   - 使用 AsyncTask、线程池或 Kotlin Coroutine
   - 避免在主线程执行网络操作

3. **添加网络状态检查**:
   - 检查网络连接状态
   - 处理网络中断情况

4. **实现请求重试机制**:
   - 处理临时网络故障
   - 避免请求失败导致用户体验差

5. **使用缓存**:
   - 减少网络请求
   - 提高应用响应速度
   - 支持离线访问

6. **添加超时设置**:
   - 避免请求无限期等待
   - 提高用户体验

7. **错误处理**:
   - 处理网络错误
   - 处理服务器错误
   - 提供友好的错误提示

8. **数据解析**:
   - 使用 Gson、Jackson 等库解析 JSON
   - 避免手动解析，减少错误

9. **网络安全**:
   - 使用 HTTPS
   - 实现证书固定
   - 避免明文传输敏感信息

10. **监控和分析**:
    - 监控网络请求性能
    - 分析网络错误率
    - 优化网络请求

## 常见问题及解决方案

### 1. 网络请求失败

**问题**: 网络请求失败，返回错误

**解决方案**:
- 检查网络连接状态
- 检查 URL 是否正确
- 检查服务器是否正常
- 检查请求参数是否正确
- 检查权限是否配置

### 2. 主线程网络操作

**问题**: 在主线程执行网络操作，导致应用崩溃

**解决方案**:
- 使用 AsyncTask
- 使用线程池
- 使用 Kotlin Coroutine
- 使用 OkHttp 的 enqueue 方法

### 3. 内存泄漏

**问题**: 网络请求导致内存泄漏

**解决方案**:
- 取消未完成的请求
- 使用弱引用持有 Context
- 在 Activity/Fragment 销毁时取消请求

### 4. 证书验证错误

**问题**: HTTPS 证书验证失败

**解决方案**:
- 确保服务器证书有效
- 实现正确的证书固定
- 不要禁用证书验证（不安全）

### 5. 网络请求超时

**问题**: 网络请求超时

**解决方案**:
- 设置合理的超时时间
- 实现请求重试机制
- 优化服务器响应速度

### 6. 大量网络请求

**问题**: 同时发送大量网络请求，导致性能问题

**解决方案**:
- 使用请求队列
- 合并相似请求
- 实现请求节流
- 使用批量 API

### 7. 数据解析错误

**问题**: JSON/XML 解析错误

**解决方案**:
- 检查服务器返回的数据格式
- 使用健壮的解析库
- 添加错误处理
- 实现数据模型验证

### 8. 网络状态变化

**问题**: 网络状态变化导致请求失败

**解决方案**:
- 监听网络状态变化
- 在网络恢复时重试失败的请求
- 实现离线模式

## 面试问题及答案

### 1. Android 中有哪些网络编程方案？各有什么优缺点？

**答案**:
- **HttpURLConnection**: 
  - 优点: 系统内置，无需额外依赖
  - 缺点: 代码繁琐，功能有限

- **OkHttp**: 
  - 优点: 功能强大，支持拦截器、缓存、WebSocket 等
  - 缺点: 需要添加依赖

- **Retrofit**: 
  - 优点: 基于 OkHttp，使用注解，代码简洁，支持多种数据格式
  - 缺点: 需要添加多个依赖

- **Volley**: 
  - 优点: 适合频繁小请求，支持缓存，请求队列
  - 缺点: 不适合大文件传输

### 2. 如何在 Android 中实现网络请求的缓存？

**答案**:
- **OkHttp 缓存**: 
  ```java
  int cacheSize = 10 * 1024 * 1024; // 10 MiB
  Cache cache = new Cache(context.getCacheDir(), cacheSize);
  OkHttpClient client = new OkHttpClient.Builder()
      .cache(cache)
      .build();
  ```

- **缓存控制头**: 
  ```java
  Request request = new Request.Builder()
      .url(url)
      .header("Cache-Control", "max-age=60")
      .build();
  ```

- **Volley 缓存**: Volley 内置缓存机制，可通过 Cache 接口自定义

### 3. 如何处理网络请求的错误？

**答案**:
- **异常处理**: 捕获网络异常
- **错误回调**: 使用回调接口处理错误
- **错误码处理**: 根据 HTTP 状态码处理不同错误
- **重试机制**: 对临时错误实现重试
- **用户提示**: 向用户显示友好的错误信息

### 4. 如何实现 WebSocket 通信？

**答案**:
- **使用 OkHttp WebSocket**:
  ```java
  OkHttpClient client = new OkHttpClient();
  Request request = new Request.Builder()
      .url("wss://echo.websocket.org")
      .build();
  WebSocket webSocket = client.newWebSocket(request, new WebSocketListener() {
      // 实现回调方法
  });
  ```

- **主要回调方法**:
  - onOpen: 连接成功
  - onMessage: 接收消息
  - onClosing: 连接关闭中
  - onFailure: 连接失败

### 5. 如何保证网络请求的安全性？

**答案**:
- **使用 HTTPS**: 加密传输
- **证书固定**: 防止中间人攻击
- **参数加密**: 加密敏感参数
- **Token 验证**: 使用 token 进行身份验证
- **网络安全配置**: 配置网络安全策略
- **避免明文存储**: 不在本地存储敏感信息

### 6. 如何优化网络请求性能？

**答案**:
- **减少请求次数**: 合并请求，使用批量 API
- **压缩数据**: 使用 gzip 压缩
- **缓存策略**: 合理使用缓存
- **请求优先级**: 优先处理重要请求
- **连接复用**: 使用连接池
- **超时设置**: 合理设置超时时间
- **预加载**: 预加载可能需要的数据

### 7. 如何处理网络状态变化？

**答案**:
- **注册网络状态监听器**:
  ```java
  ConnectivityManager connectivityManager = (ConnectivityManager) context.getSystemService(Context.CONNECTIVITY_SERVICE);
  NetworkRequest.Builder builder = new NetworkRequest.Builder();
  connectivityManager.registerNetworkCallback(builder.build(), new ConnectivityManager.NetworkCallback() {
      @Override
      public void onAvailable(Network network) {
          // 网络可用
      }
      
      @Override
      public void onLost(Network network) {
          // 网络丢失
      }
  });
  ```

- **检查网络状态**:
  ```java
  public static boolean isNetworkAvailable(Context context) {
      ConnectivityManager cm = (ConnectivityManager) context.getSystemService(Context.CONNECTIVITY_SERVICE);
      NetworkInfo activeNetwork = cm.getActiveNetworkInfo();
      return activeNetwork != null && activeNetwork.isConnectedOrConnecting();
  }
  ```

### 8. 如何实现文件上传和下载？

**答案**:
- **文件上传**:
  ```java
  // OkHttp 文件上传
  MultipartBody.Builder builder = new MultipartBody.Builder()
      .setType(MultipartBody.FORM)
      .addFormDataPart("name", "John")
      .addFormDataPart("file", "file.txt",
          RequestBody.create(MediaType.parse("text/plain"), new File("path/to/file.txt"));
  
  RequestBody requestBody = builder.build();
  Request request = new Request.Builder()
      .url("https://api.example.com/upload")
      .post(requestBody)
      .build();
  ```

- **文件下载**:
  ```java
  // OkHttp 文件下载
  Request request = new Request.Builder()
      .url("https://api.example.com/file")
      .build();
  
  client.newCall(request).enqueue(new Callback() {
      @Override
      public void onResponse(Call call, Response response) throws IOException {
          if (response.isSuccessful()) {
              InputStream inputStream = response.body().byteStream();
              FileOutputStream outputStream = new FileOutputStream("path/to/file");
              byte[] buffer = new byte[1024];
              int len;
              while ((len = inputStream.read(buffer)) != -1) {
                  outputStream.write(buffer, 0, len);
              }
              outputStream.close();
              inputStream.close();
          }
      }
      
      @Override
      public void onFailure(Call call, IOException e) {
          e.printStackTrace();
      }
  });
  ```

## 总结

网络编程是 Android 应用开发中的重要组成部分，选择合适的网络库和实现方式对于应用的性能和用户体验至关重要。

在实际开发中，应根据应用的具体需求选择合适的网络方案，实现合理的错误处理、缓存策略和安全措施，以构建高性能、可靠的网络通信功能。

掌握网络编程的核心概念和最佳实践，对于构建高质量的 Android 应用至关重要。