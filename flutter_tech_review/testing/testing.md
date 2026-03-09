# 测试体系

## 为什么 Flutter 测试不能只停留在“会写 testWidgets”
Flutter 项目里最常见的测试误区有两个：

- 只写少量单元测试，UI 和交互完全不测
- 把大量问题都丢到集成测试，导致测试慢、脆弱、维护成本高

真正有效的测试体系，目标不是“覆盖率数字好看”，而是降低回归风险、提高重构信心、让核心流程可验证。

Flutter 测试要解决的核心问题：

- 业务逻辑是否正确
- Widget 在不同状态下是否正确渲染
- 用户交互后页面状态是否正确变化
- 多模块协作时是否有回归
- 发布前关键流程是否仍然可用

## Flutter 测试的三层结构
Flutter 官方常见测试分层可以理解为三层：

1. 单元测试
2. Widget 测试
3. 集成测试

这三层不是谁替代谁，而是职责不同。

## 单元测试

### 测什么
单元测试主要验证纯逻辑或小范围对象行为。

常见对象：

- 工具函数
- formatter / parser
- repository 里的纯逻辑部分
- use case
- 状态管理器的业务规则
- DTO / mapper / validator

### 适合的内容

- 输入输出明确
- 不依赖 UI
- 不依赖真实网络
- 不依赖真实平台能力
- 逻辑边界清晰

### 优点

- 运行快
- 失败定位清晰
- 最适合覆盖边界条件
- 最适合支撑重构

### 不适合测什么

- Widget 渲染
- 复杂交互流程
- 多层真实协作
- 平台集成问题

### 示例
```dart
import 'package:flutter_test/flutter_test.dart';

int sum(int a, int b) => a + b;

void main() {
  test('sum adds two integers', () {
    expect(sum(1, 2), 3);
    expect(sum(-1, 1), 0);
  });
}
```

## Widget 测试

### 本质是什么
Widget 测试介于单元测试和集成测试之间。它会在测试环境里构建 Flutter Widget 树，验证 UI 渲染和交互逻辑。

可以理解为：

- 它测的是真实 Widget
- 但运行在受控的测试环境里
- 比集成测试快得多
- 比单元测试更贴近真实页面行为

### 常见用途

- 页面初始渲染
- 点击按钮后的 UI 变化
- 表单输入与校验
- loading / success / error 三态展示
- 路由跳转后的页面结构验证
- 状态变化是否驱动正确的 Widget 更新

### 为什么 Widget 测试很重要
Flutter 是声明式 UI 框架，很多 Bug 不是纯逻辑错，而是状态变化没有正确映射到界面上。这个层级正好用来验证这一点。

### 示例
```dart
import 'package:flutter/material.dart';
import 'package:flutter_test/flutter_test.dart';

void main() {
  testWidgets('counter increments after tap', (tester) async {
    await tester.pumpWidget(
      const MaterialApp(
        home: CounterPage(),
      ),
    );

    expect(find.text('0'), findsOneWidget);

    await tester.tap(find.byType(ElevatedButton));
    await tester.pump();

    expect(find.text('1'), findsOneWidget);
  });
}

class CounterPage extends StatefulWidget {
  const CounterPage({super.key});

  @override
  State<CounterPage> createState() => _CounterPageState();
}

class _CounterPageState extends State<CounterPage> {
  int count = 0;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: Column(
        children: [
          Text('$count'),
          ElevatedButton(
            onPressed: () => setState(() => count++),
            child: const Text('Add'),
          ),
        ],
      ),
    );
  }
}
```

## 集成测试

### 测什么
集成测试验证应用多个模块协作是否正常，通常会跑在真实设备或模拟器上。

典型场景：

- 登录流程
- 下单 / 支付前关键链路
- 启动到首页主流程
- 搜索、列表、详情、返回这一整段链路
- 与平台插件、权限、原生桥接有关的关键流程

### 特点

- 最接近真实运行环境
- 覆盖面最广
- 运行最慢
- 最容易不稳定
- 定位成本最高

### 工程原则

- 集成测试只覆盖最关键、最高风险流程
- 不要把所有细节都丢给集成测试
- 能在单元或 Widget 测试解决的，尽量前置解决

## 测试金字塔
这是最核心的测试设计思想之一。

### 理想比例

- 底层是大量单元测试
- 中层是适量 Widget 测试
- 顶层是少量关键集成测试

原因很简单：

- 越底层越快、越稳定、越容易定位
- 越顶层越贵、越慢、越脆弱

如果一个项目主要依赖集成测试，通常说明测试分层设计有问题。

## 如何决定一个功能该写哪种测试

### 写单元测试，如果它是

- 纯逻辑
- 输入输出清晰
- 不需要 Widget 树
- 可以通过 mock/fake 隔离依赖

### 写 Widget 测试，如果它是

- 页面展示逻辑
- 组件交互逻辑
- 状态变化后的渲染验证
- 表单、弹窗、列表、路由等 UI 相关行为

### 写集成测试，如果它是

- 关键用户路径
- 多模块联动
- 依赖真实平台能力
- 权限、插件、原生交互相关流程

## `pump`、`pumpAndSettle`、异步测试
这是 Flutter 测试里的高频基础点。

### `pump`
推动一帧渲染，用于让 Widget 树更新一次。

### `pumpAndSettle`
持续推动帧，直到没有更多待处理动画或异步调度。

### 使用原则

- 普通状态更新后，优先 `pump`
- 依赖动画、路由、异步任务完成时，再考虑 `pumpAndSettle`
- 不要无脑全用 `pumpAndSettle`，否则测试可能变慢甚至挂住

### 异步场景要注意

- Future 完成时机
- 动画和过渡
- debounce / throttle
- 定时器
- Stream 推送

这些如果控制不好，很容易造成测试不稳定。

## mock、fake、stub 的区别

### mock
模拟依赖并验证调用行为。

适合：

- 需要验证某个方法是否被调用
- 需要精细控制返回值和异常

### fake
用一个轻量的、可运行的替身实现依赖。

适合：

- 依赖逻辑不复杂
- 比 mock 更自然
- 需要构造可运行测试环境

### stub
重点是提供固定返回值，不强调行为验证。

### 实战建议

- 能用 fake 的地方优先 fake，通常更稳
- 需要验证交互行为时用 mock
- 不要过度 mock，尤其不要把内部实现调用细节都锁死

## 可测试性设计
很多“测试难写”不是测试问题，而是代码结构问题。

### 不可测试代码的典型特征

- 页面直接 new 网络请求对象
- 页面直接解析 JSON
- 大量静态方法和全局单例
- 状态、网络、缓存、导航全部耦合在一个类里
- 平台能力直接写死在业务逻辑中

### 更可测试的设计

- 依赖注入
- 抽象接口隔离外部依赖
- repository / service 分层
- 页面层只依赖状态与事件
- 平台层、网络层、缓存层都可替换

一句话概括：先把代码写成可替换、可组合，测试自然就能写。

## 状态管理怎么测
不同状态管理库写法不同，但测试思路基本一致。

### 测状态逻辑
优先单元测试：

- 初始状态是否正确
- 事件触发后状态是否变化正确
- 失败场景是否产生正确错误状态
- 加载态、成功态、失败态是否完整

### 测 UI 响应
用 Widget 测试：

- 注入 fake repository 或 fake notifier
- 驱动状态变化
- 验证界面是否切换到对应状态

重点不在状态管理库本身，而在“状态是否正确驱动 UI”。

## 网络请求怎么测
不要在测试里直接打真实接口，除非你明确在做端到端验收。

### 推荐策略

- 单元测试：mock/fake repository 或 data source
- Widget 测试：验证 loading / data / error UI
- 集成测试：只验证极少数关键链路

### 不建议

- 大量测试直连真实后端
- 让网络波动决定测试结果
- 页面测试里顺带测网络库本身

## Platform Channel / 插件怎么测
这类问题往往不是“有没有测试”，而是测试层次放错了。

### 推荐分层

- Dart 封装层：单元测试错误映射、参数转换
- Widget 层：测 UI 对桥接结果的响应
- 集成测试：测真实平台能力关键路径

如果把所有平台能力都留到集成测试验证，维护成本会很高。

## Golden Test 的定位
Golden Test 用于验证 UI 视觉输出是否发生非预期变化。

适合：

- 设计稳定的组件库
- 主题切换
- 多状态静态页面
- 复杂自定义绘制组件

不适合：

- 高频变动页面
- 强依赖动态数据页面
- 把 Golden Test 当作全部 UI 测试手段

Golden Test 能发现“看起来变了”，但不能替代交互和业务逻辑测试。

## 常见问题与解决思路

### 1. 测试很慢
常见原因：

- 太依赖集成测试
- 测试里等待过多
- `pumpAndSettle` 滥用
- 用了太多真实依赖

解决：

- 尽量前移到单元和 Widget 测试
- 去掉不必要等待
- 用 fake 替代真实网络和存储

### 2. 测试很脆
常见原因：

- 断言过度依赖实现细节
- 依赖真实时间、真实网络、真实环境
- 选择器不稳定

解决：

- 断言行为和可见结果，不断言无关细节
- 固定外部输入
- 给关键节点稳定 key 或稳定文本

### 3. Widget 测试经常卡住
常见原因：

- 有未结束动画
- 无限轮询或 Stream 未收敛
- `pumpAndSettle` 等不到稳定状态

解决：

- 精确 `pump` 所需时长
- 检查动画、定时器、轮询逻辑
- 控制异步源头，而不是盲等

### 4. 覆盖率高但 Bug 还很多
原因：

- 覆盖率只代表“跑到过”，不代表“断言有效”
- 关键业务路径没被测到
- 错误场景和边界条件没覆盖

解决：

- 不迷信覆盖率数字
- 优先补高风险路径和失败场景
- 看关键断言是否真正有价值

## CI 中怎么接测试
Flutter 工程里常见的质量门禁流程：

1. `flutter pub get`
2. `dart format --set-exit-if-changed .`
3. `flutter analyze`
4. `flutter test`
5. 关键项目再补 `integration_test`

工程建议：

- PR 阶段至少跑 analyze + unit/widget test
- 集成测试可以按主分支、夜间或关键发布分支执行
- 覆盖率可以作为参考指标，不建议单独作为唯一门禁

## 什么时候适合 TDD
TDD 不是所有页面都要用，但在这些场景很有效：

- 复杂业务规则
- 纯逻辑计算
- 容易反复变化的规则系统
- 想先固化输入输出边界再写实现

对大量纯 UI 搭建页面，TDD 的收益通常没有业务逻辑那么高。

## 面试高频题

### 1. Flutter 有哪些测试类型
标准回答：

- 单元测试验证纯逻辑和小范围对象行为
- Widget 测试验证 Widget 渲染和交互
- 集成测试验证真实环境下的关键业务流程
- 三者应该形成测试金字塔，而不是互相替代

### 2. Widget 测试和集成测试的区别是什么
标准回答：

- Widget 测试运行在受控测试环境，速度更快，适合验证页面和交互
- 集成测试运行在更真实的设备环境，覆盖范围更广，但更慢、更脆
- 所以大部分 UI 行为应该在 Widget 测试解决，关键链路再用集成测试兜底

### 3. 为什么很多项目测试难推进
标准回答：

- 不是测试框架不行，而是代码不可测试
- 依赖注入、分层、抽象接口不到位时，测试会非常痛苦
- 页面直接依赖网络、数据库、平台能力，会导致测试难写、难维护

### 4. mock 和 fake 怎么选
标准回答：

- 需要验证调用行为时用 mock
- 需要一个轻量可运行替身时优先 fake
- 过度 mock 会让测试和实现细节耦合得太深

### 5. 覆盖率高是不是就说明测试做得好
标准回答：

- 不是
- 覆盖率只能说明代码被执行过，不代表断言有效
- 更重要的是关键路径、错误场景、边界条件有没有覆盖到

## 面试答题模板

### 题目：你在 Flutter 项目里怎么搭建测试体系
可以按这个结构回答：

1. 先按单元、Widget、集成测试分层
2. 业务逻辑尽量前置到单元测试
3. 页面交互和状态渲染放在 Widget 测试
4. 关键主流程用少量集成测试兜底
5. 配合 CI 做 analyze、test 和关键场景验证

### 题目：怎么让 Flutter 代码更容易测试
可以按这个结构回答：

1. 做依赖注入
2. 隔离网络、存储、平台能力
3. 页面层只处理状态和交互
4. repository / service 层可替换
5. 用 fake 和 mock 控制外部依赖

## 复习重点

- 记住单元、Widget、集成测试的职责边界
- 理解测试金字塔为什么重要
- 说清 `pump` 和 `pumpAndSettle` 的区别
- 知道 mock、fake、stub 的适用场景
- 能把测试体系和代码可测试性设计联系起来
