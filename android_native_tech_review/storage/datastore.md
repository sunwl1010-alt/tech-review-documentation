# DataStore 面试专题（替代 SharedPreferences）

## 1. 结论先说

DataStore 相比 SharedPreferences 的核心优势是：异步、安全、可观察、事务一致性更好。  
SharedPreferences 适合存量兼容；新项目配置存储优先 DataStore。

## 2. 两种 DataStore

- Preferences DataStore：键值对，轻量、无 schema
- Proto DataStore：强类型 schema（protobuf），适合结构化配置

## 3. Preferences DataStore 示例

```kotlin
private val Context.dataStore by preferencesDataStore("settings")

object Keys {
    val DARK_MODE = booleanPreferencesKey("dark_mode")
}

val darkModeFlow: Flow<Boolean> = context.dataStore.data
    .map { it[Keys.DARK_MODE] ?: false }

suspend fun updateDarkMode(enabled: Boolean) {
    context.dataStore.edit { prefs ->
        prefs[Keys.DARK_MODE] = enabled
    }
}
```

## 4. 为什么比 SharedPreferences 稳

- 写入是挂起函数，避免主线程阻塞
- 读取是 Flow，可直接与 UI 状态绑定
- 更新具备原子性，减少并发覆盖问题

## 5. 迁移策略（面试加分）

- 存量项目先做双读（优先 DataStore，没有再读 SharedPreferences）
- 灰度后执行一次迁移写入 DataStore
- 最后下线旧存储路径

## 6. 常见追问

### Q1：DataStore 能替代 Room 吗？
答：不能。DataStore 适合配置/偏好，不适合复杂结构化查询。

### Q2：什么时候选 Proto DataStore？
答：当配置结构复杂且需要 schema 管理、版本演进时。

## 7. 60秒答题模板

- 结论：配置存储新项目优先 DataStore。
- 依据：异步、安全、Flow 可观察、事务一致性更好。
- 落地：我会先接入 Preferences 版本，结构复杂再升级 Proto。
- 边界：业务主数据仍应使用 Room/SQLite。
