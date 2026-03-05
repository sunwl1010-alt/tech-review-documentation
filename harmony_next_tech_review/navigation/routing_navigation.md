# 路由与导航

## 核心概念

路由与导航是 HarmonyOS Next 应用中实现页面切换和数据传递的重要机制。它允许用户在不同页面之间进行导航，是构建多页面应用的基础。

## 实现原理

HarmonyOS Next 的路由与导航基于以下核心概念：

1. **Router**: 负责管理页面栈，处理页面的跳转、返回等操作
2. **Page**: 应用中的页面，是导航的基本单位
3. **RouteOptions**: 定义路由跳转的参数和配置
4. **NavDestination**: 导航目标，包含页面信息和参数

## 基本使用

### 1. 页面跳转

```typescript
import router from '@ohos.router';

// 基本跳转
router.push({
  url: 'pages/DetailPage',
  params: {
    id: '123',
    name: 'Product'
  }
});

// 带动画的跳转
router.push({
  url: 'pages/DetailPage',
  params: {
    id: '123'
  },
  animated: true
});
```

### 2. 页面返回

```typescript
import router from '@ohos.router';

// 返回上一页
router.back();

// 返回到指定页面
router.back({
  url: 'pages/HomePage'
});

// 带参数返回
router.back({
  params: {
    result: 'success'
  }
});
```

### 3. 页面替换

```typescript
import router from '@ohos.router';

// 替换当前页面
router.replace({
  url: 'pages/LoginPage'
});

// 清空页面栈并跳转
router.replace({
  url: 'pages/HomePage',
  params: {
    reset: true
  }
});
```

## 高级特性

### 1. 参数传递

#### 基本参数传递

```typescript
// 跳转时传递参数
router.push({
  url: 'pages/DetailPage',
  params: {
    id: '123',
    name: 'Product',
    price: 99.9
  }
});

// 接收参数
import router from '@ohos.router';

@Entry
@Component
struct DetailPage {
  @State id: string = '';
  @State name: string = '';
  @State price: number = 0;

  aboutToAppear() {
    const params = router.getParams();
    if (params) {
      this.id = params.id as string;
      this.name = params.name as string;
      this.price = params.price as number;
    }
  }

  build() {
    Column() {
      Text(`ID: ${this.id}`)
      Text(`Name: ${this.name}`)
      Text(`Price: ${this.price}`)
    }
  }
}
```

#### 复杂参数传递

```typescript
// 传递对象
router.push({
  url: 'pages/DetailPage',
  params: {
    user: JSON.stringify({
      id: 1,
      name: 'John',
      age: 25
    })
  }
});

// 接收对象
aboutToAppear() {
  const params = router.getParams();
  if (params) {
    const userStr = params.user as string;
    const user = JSON.parse(userStr);
    console.log(`User: ${user.name}, Age: ${user.age}`);
  }
}
```

### 2. 路由拦截

#### 全局路由拦截

```typescript
import router from '@ohos.router';

// 设置路由拦截器
router.setRouterInterceptor({
  // 跳转前拦截
  intercept: (options: RouterOptions) => {
    console.log(`Navigating to: ${options.url}`);
    
    // 检查是否需要登录
    if (options.url.includes('profile') && !isLoggedIn()) {
      // 重定向到登录页
      router.replace({
        url: 'pages/LoginPage',
        params: {
          redirect: options.url
        }
      });
      return false; // 阻止原跳转
    }
    return true; // 允许跳转
  },
  // 跳转后回调
  callback: (options: RouterOptions) => {
    console.log(`Navigated to: ${options.url}`);
  }
});
```

#### 页面级拦截

```typescript
@Entry
@Component
struct ProfilePage {
  aboutToAppear() {
    // 页面加载前检查
    if (!isLoggedIn()) {
      router.replace({
        url: 'pages/LoginPage'
      });
    }
  }

  build() {
    // 页面内容
  }
}
```

### 3. 导航动画

#### 内置动画

```typescript
// 淡入淡出动画
router.push({
  url: 'pages/DetailPage',
  animated: true,
  animationType: router.RouteAnimationType.Fade
});

// 滑动动画
router.push({
  url: 'pages/DetailPage',
  animated: true,
  animationType: router.RouteAnimationType.Slide
});

// 缩放动画
router.push({
  url: 'pages/DetailPage',
  animated: true,
  animationType: router.RouteAnimationType.Scale
});
```

#### 自定义动画

```typescript
router.push({
  url: 'pages/DetailPage',
  animated: true,
  animationDuration: 500, // 动画持续时间（毫秒）
  animationType: router.RouteAnimationType.Custom,
  customAnimation: {
    // 自定义动画配置
    enterFrom: {
      opacity: 0,
      transform: {
        translateX: 300
      }
    },
    exitTo: {
      opacity: 0,
      transform: {
        translateX: -100
      }
    }
  }
});
```

### 4. 页面栈管理

```typescript
import router from '@ohos.router';

// 获取当前页面栈
const stack = router.getRouteStack();
console.log(`Current stack size: ${stack.length}`);

// 清空页面栈
router.clear();

// 返回到根页面
router.back({
  url: 'pages/HomePage'
});

// 跳转到新页面并清空之前的栈
router.replace({
  url: 'pages/HomePage'
});
```

### 5. 导航状态管理

```typescript
import router from '@ohos.router';

// 监听路由变化
router.on('routeChange', (event) => {
  console.log(`Route changed: ${event.url}`);
});

// 监听页面显示
router.on('pageShow', (event) => {
  console.log(`Page shown: ${event.url}`);
});

// 监听页面隐藏
router.on('pageHide', (event) => {
  console.log(`Page hidden: ${event.url}`);
});
```

## 最佳实践

1. **路由配置集中管理**:
   - 创建路由配置文件，集中管理所有页面路由
   - 使用常量定义路由路径，避免硬编码

2. **参数传递规范**:
   - 对于简单数据，直接传递基本类型
   - 对于复杂数据，使用 JSON.stringify/parse
   - 避免传递过大的数据，考虑使用全局状态管理

3. **导航动画使用**:
   - 为不同类型的页面切换选择合适的动画
   - 避免过度使用动画，影响性能

4. **路由拦截合理使用**:
   - 用于权限检查、登录验证等场景
   - 避免过度拦截，影响用户体验

5. **页面栈管理**:
   - 及时清理不需要的页面，避免内存泄漏
   - 合理使用 replace 方法处理登录、退出等场景

## 常见问题及解决方案

### 1. 参数传递失败

**问题**: 页面跳转时参数传递失败或接收不到

**解决方案**:
- 检查参数类型是否正确
- 对于复杂对象，使用 JSON.stringify/parse
- 检查参数大小，避免传递过大的数据
- 使用全局状态管理传递复杂数据

### 2. 页面跳转后白屏

**问题**: 页面跳转后出现白屏

**解决方案**:
- 检查目标页面是否存在
- 检查页面代码是否有错误
- 检查页面加载是否有耗时操作，考虑使用异步加载
- 检查路由配置是否正确

### 3. 路由循环跳转

**问题**: 页面之间循环跳转，导致栈溢出

**解决方案**:
- 检查路由拦截逻辑，避免无限重定向
- 使用 replace 方法替代 push 方法
- 在页面加载前检查跳转条件

### 4. 导航动画不生效

**问题**: 设置了动画但不生效

**解决方案**:
- 确保 animated 参数设置为 true
- 检查动画类型是否正确
- 检查动画持续时间是否合理
- 对于自定义动画，检查配置是否正确

### 5. 页面返回后数据丢失

**问题**: 页面返回后之前的数据丢失

**解决方案**:
- 使用 @StorageProp 或全局状态管理保存数据
- 在页面 onPageHide 时保存状态
- 在页面 aboutToAppear 时恢复状态

## 面试问题及答案

### 1. HarmonyOS Next 中的路由与导航是如何实现的？

**答案**:
HarmonyOS Next 的路由与导航基于 Router 模块实现，主要包含以下核心概念：
- **Router**: 负责管理页面栈，处理页面的跳转、返回等操作
- **Page**: 应用中的页面，是导航的基本单位
- **RouteOptions**: 定义路由跳转的参数和配置
- **NavDestination**: 导航目标，包含页面信息和参数

通过 router.push()、router.back()、router.replace() 等方法实现页面的跳转、返回和替换操作。

### 2. 如何在 HarmonyOS Next 中传递参数？

**答案**:
在 HarmonyOS Next 中传递参数有以下几种方式：

1. **基本参数传递**:
   ```typescript
   router.push({
     url: 'pages/DetailPage',
     params: {
       id: '123',
       name: 'Product'
     }
   });
   ```

2. **接收参数**:
   ```typescript
   const params = router.getParams();
   if (params) {
     const id = params.id as string;
   }
   ```

3. **复杂参数传递**:
   - 使用 JSON.stringify() 序列化对象
   - 在接收端使用 JSON.parse() 反序列化

4. **全局状态管理**:
   - 使用 AppStorage 或 Preferences 存储全局数据
   - 适用于需要在多个页面共享的数据

### 3. 如何实现路由拦截？

**答案**:
在 HarmonyOS Next 中实现路由拦截主要有两种方式：

1. **全局路由拦截**:
   ```typescript
   router.setRouterInterceptor({
     intercept: (options) => {
       // 检查是否需要登录
       if (options.url.includes('profile') && !isLoggedIn()) {
         router.replace({ url: 'pages/LoginPage' });
         return false; // 阻止原跳转
       }
       return true; // 允许跳转
     }
   });
   ```

2. **页面级拦截**:
   ```typescript
   @Entry
   @Component
   struct ProfilePage {
     aboutToAppear() {
       if (!isLoggedIn()) {
         router.replace({ url: 'pages/LoginPage' });
       }
     }
     // 页面内容
   }
   ```

### 4. 如何优化导航性能？

**答案**:
优化导航性能的方法包括：

1. **合理使用页面栈**:
   - 及时清理不需要的页面，避免栈过深
   - 使用 replace 方法处理登录、退出等场景

2. **优化页面加载**:
   - 避免在 aboutToAppear 中执行耗时操作
   - 使用异步加载和懒加载
   - 优化页面渲染性能

3. **合理使用动画**:
   - 为不同类型的页面切换选择合适的动画
   - 避免过度使用动画，影响性能
   - 合理设置动画持续时间

4. **预加载**:
   - 对于常用页面，考虑预加载
   - 使用缓存机制减少重复加载

### 5. 如何处理页面返回的数据？

**答案**:
处理页面返回的数据有以下几种方式：

1. **使用 router.back() 传递参数**:
   ```typescript
   // 返回时传递数据
   router.back({
     params: {
       result: 'success',
       data: { id: 1, name: 'Product' }
     }
   });
   ```

2. **使用全局状态管理**:
   - 在返回前更新全局状态
   - 在目标页面监听状态变化

3. **使用回调函数**:
   - 定义全局回调函数
   - 在返回时调用回调函数

4. **使用事件总线**:
   - 发布返回事件
   - 在目标页面订阅事件

## 总结

路由与导航是 HarmonyOS Next 应用开发中的重要组成部分，它不仅实现了页面之间的切换，还支持参数传递、路由拦截、导航动画等高级功能。

通过合理使用路由与导航机制，可以构建出流畅、用户友好的多页面应用。在实际开发中，应根据应用的具体需求选择合适的导航方式，并遵循最佳实践，确保应用的性能和用户体验。

掌握路由与导航的核心概念和使用方法，对于构建高质量的 HarmonyOS Next 应用至关重要。