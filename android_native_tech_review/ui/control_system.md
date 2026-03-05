# 控件系统

## 基础控件

### TextView
- **功能**：显示文本内容
- **常用属性**：`text`、`textSize`、`textColor`、`gravity`
- **代码示例**：
  ```kotlin
  val textView = findViewById<TextView>(R.id.textView)
  textView.text = "Hello World"
  textView.textSize = 18f
  textView.setTextColor(Color.BLUE)
  ```

### Button
- **功能**：响应用户点击事件
- **常用属性**：`text`、`background`、`onClick`
- **代码示例**：
  ```kotlin
  val button = findViewById<Button>(R.id.button)
  button.setOnClickListener {
      Toast.makeText(this, "Button clicked", Toast.LENGTH_SHORT).show()
  }
  ```

### EditText
- **功能**：接收用户输入
- **常用属性**：`hint`、`inputType`、`maxLength`
- **代码示例**：
  ```kotlin
  val editText = findViewById<EditText>(R.id.editText)
  val input = editText.text.toString()
  ```

### ImageView
- **功能**：显示图片
- **常用属性**：`src`、`scaleType`、`adjustViewBounds`
- **代码示例**：
  ```kotlin
  val imageView = findViewById<ImageView>(R.id.imageView)
  imageView.setImageResource(R.drawable.ic_launcher)
  ```

## 自定义控件

### 自定义 View
- **创建方式**：继承 View 或 ViewGroup
- **核心方法**：
  - `onMeasure()`：测量控件大小
  - `onLayout()`：布局子控件
  - `onDraw()`：绘制控件内容
- **代码示例**：
  ```kotlin
  class CustomView @JvmOverloads constructor(
      context: Context,
      attrs: AttributeSet? = null,
      defStyleAttr: Int = 0
  ) : View(context, attrs, defStyleAttr) {
      
      override fun onDraw(canvas: Canvas?) {
          super.onDraw(canvas)
          // 绘制逻辑
      }
  }
  ```

### 自定义复合控件
- **创建方式**：组合多个基础控件
- **优点**：提高代码复用性，简化布局
- **示例**：创建一个带图标的按钮控件

## 适配器模式

### 适配器的作用
- 连接数据与 UI 控件
- 处理数据的展示逻辑
- 提高代码的可维护性

### 常用适配器
- **ArrayAdapter**：适用于简单数据列表
- **BaseAdapter**：基础适配器，需要重写多个方法
- **RecyclerView.Adapter**：RecyclerView 的专用适配器

### RecyclerView 适配器示例
```kotlin
class MyAdapter(private val data: List<String>) : RecyclerView.Adapter<MyAdapter.ViewHolder>() {
    
    class ViewHolder(itemView: View) : RecyclerView.ViewHolder(itemView)
    
    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): ViewHolder {
        val view = LayoutInflater.from(parent.context)
            .inflate(R.layout.item_layout, parent, false)
        return ViewHolder(view)
    }
    
    override fun onBindViewHolder(holder: ViewHolder, position: Int) {
        // 绑定数据
    }
    
    override fun getItemCount(): Int = data.size
}
```

## 列表与网格

### ListView
- **特点**：传统列表控件
- **优缺点**：简单易用，但性能不如 RecyclerView
- **使用场景**：数据量较小的列表

### RecyclerView
- **特点**：更灵活、性能更好的列表控件
- **优势**：
  - 支持多种布局管理器
  - 内置视图复用机制
  - 支持动画效果
- **布局管理器**：
  - `LinearLayoutManager`：线性布局
  - `GridLayoutManager`：网格布局
  - `StaggeredGridLayoutManager`：瀑布流布局

### GridView
- **特点**：专门用于显示网格布局
- **使用场景**：图片画廊、应用列表等

## 最佳实践

1. **控件复用**：使用 RecyclerView 提高性能
2. **布局优化**：减少嵌套层级，使用 ConstraintLayout
3. **触摸反馈**：为可点击控件添加合适的触摸反馈
4. **响应式设计**：适配不同屏幕尺寸
5. **无障碍支持**：为控件添加内容描述

## 常见问题

1. **控件不显示**：检查布局参数、可见性设置
2. **点击事件不响应**：检查是否设置了点击监听器，是否被其他控件遮挡
3. **性能问题**：避免在 `onDraw` 中进行耗时操作
4. **内存泄漏**：避免在适配器中持有 Activity 引用

## 面试题

1. **Q**: RecyclerView 相比 ListView 有哪些优势？
   **A**: 支持多种布局管理器、内置视图复用机制、性能更好、支持动画效果

2. **Q**: 如何自定义一个 View？
   **A**: 继承 View 或 ViewGroup，重写 onMeasure()、onLayout()、onDraw() 方法

3. **Q**: 适配器模式的作用是什么？
   **A**: 连接数据与 UI 控件，处理数据展示逻辑，提高代码可维护性

4. **Q**: 如何优化列表的滚动性能？
   **A**: 使用 RecyclerView、避免在绑定数据时进行耗时操作、使用视图持有者模式、优化图片加载