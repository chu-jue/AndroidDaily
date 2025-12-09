---
title: "Kotlin 高阶函数：用法与工程实践"
date: 2025-12-10 08:00:00
categories: ["Kotlin","高阶函数"]
tags: ["Kotlin","高阶函数"]
---

# Kotlin 高阶函数：用法与工程实践

## 一、什么是高阶函数

高阶函数是指可以接收函数作为参数或返回函数作为结果的函数。这是函数式编程的核心概念之一。

基本示例

```kotlin
// 接收函数作为参数
fun calculate(x: Int, y: Int, operation: (Int, Int) -> Int): Int {
    return operation(x, y)
}

// 返回函数作为结果
fun getMultiplier(factor: Int): (Int) -> Int {
    return { value -> value * factor }
}
```

## 二、核心用法

### 1. Lambda表达式

```kotlin
val sum = { a: Int, b: Int -> a + b }
val result = calculate(10, 5, sum) // 15

// 更简洁的写法
calculate(10, 5) { a, b -> a * b }
```

### 2. 带接收者的Lambda (DSL构建)

```kotlin
fun buildString(action: StringBuilder.() -> Unit): String {
    val sb = StringBuilder()
    sb.action()
    return sb.toString()
}

val result = buildString {
    append("Hello")
    append(" ")
    append("Kotlin")
}
```

### 3. 函数类型别名 (提高可读性)

```kotlin
typealias OnClick = (View) -> Unit
typealias DataTransformer<T> = (T) -> T

class Button {
    private var onClickListener: OnClick? = null
    
    fun setOnClickListener(listener: OnClick) {
        this.onClickListener = listener
    }
}
```

### 4. 内联函数 (性能优化)

```kotlin
inline fun <T> measureTimeMillis(block: () -> T): Pair<T, Long> {
    val start = System.currentTimeMillis()
    val result = block()
    val end = System.currentTimeMillis()
    return result to (end - start)
}
```

## 三、工程最佳实践

### 1. 自定义高阶函数

```kotlin
// 带默认参数的函数类型
fun executeWithRetry(
    maxRetries: Int = 3,
    delay: Long = 100,
    action: () -> Boolean
): Boolean {
    repeat(maxRetries) { attempt ->
        if (action()) return true
        if (attempt < maxRetries - 1) Thread.sleep(delay)
    }
    return false
}

// 使用
val success = executeWithRetry(maxRetries = 5) {
    apiCall().isSuccess
}
```

### 2. 领域特定语言 (DSL)

```kotlin
class DatabaseConfig {
    var url: String = ""
    var timeout: Int = 5000
    var poolSize: Int = 10
}

fun database(block: DatabaseConfig.() -> Unit): DatabaseConfig {
    return DatabaseConfig().apply(block)
}

// 使用
val config = database {
    url = "jdbc:mysql://localhost:3306/mydb"
    timeout = 10000
    poolSize = 20
}
```

### 3. 资源管理

```kotlin
inline fun <T : AutoCloseable, R> T.use(block: (T) -> R): R {
    try {
        return block(this)
    } finally {
        this.close()
    }
}

// 扩展函数版本
inline fun <T, R> withResource(resource: T, block: T.() -> R): R 
        where T : AutoCloseable {
    try {
        return resource.block()
    } finally {
        resource.close()
    }
}
```

### 4. 组合函数 (Function Composition)

```kotlin
infix fun <A, B, C> ((B) -> C).compose(g: (A) -> B): (A) -> C {
    return { x -> this(g(x)) }
}

// 使用
val addTwo = { x: Int -> x + 2 }
val multiplyByThree = { x: Int -> x * 3 }

val composed = multiplyByThree compose addTwo
println(composed(5)) // (5 + 2) * 3 = 21
```

### 5. 柯里化 (Currying)

```kotlin
fun <A, B, C> curry(f: (A, B) -> C): (A) -> (B) -> C {
    return { a -> { b -> f(a, b) } }
}

// 使用
val curriedAdd = curry { a: Int, b: Int -> a + b }
val addFive = curriedAdd(5)
println(addFive(3)) // 8
```

## 四、架构设计中的应用

### 1. 策略模式替代

```kotlin
// 传统策略模式
interface SortStrategy {
    fun sort(list: List<Int>): List<Int>
}

// 使用高阶函数
class Sorter(private val strategy: (List<Int>) -> List<Int>) {
    fun sort(list: List<Int>) = strategy(list)
}

// 使用
val quickSorter = Sorter { it.sorted() }
val bubbleSorter = Sorter { list -> 
    // 冒泡排序实现
    list.toMutableList().apply { /* 排序逻辑 */ }
}
```

### 2. 回调处理

```kotlin
class Repository {
    // 避免回调地狱
    suspend fun fetchData(
        onSuccess: (Data) -> Unit = {},
        onError: (Throwable) -> Unit = {},
        onFinally: () -> Unit = {}
    ) = try {
        val data = api.fetch()
        onSuccess(data)
        data
    } catch (e: Exception) {
        onError(e)
        throw e
    } finally {
        onFinally()
    }
}
```

### 3. 依赖注入

```kotlin
class UseCase(
    private val validate: (Request) -> Boolean,
    private val fetchData: (Request) -> Data,
    private val transform: (Data) -> Result
) {
    fun execute(request: Request): Result {
        if (!validate(request)) throw ValidationException()
        val data = fetchData(request)
        return transform(data)
    }
}
```

## 五、性能注意事项

### 1. 适当使用 inline

```kotlin
// 适合内联的情况
inline fun <T> T.applyIf(condition: Boolean, block: T.() -> Unit): T {
    if (condition) block()
    return this
}

// 不适合内联的情况（函数类型参数被存储）
fun storeCallback(callback: () -> Unit) {
    // callback被存储，不能内联
}
```

### 2. 避免重复创建Lambda

```kotlin
// 不好：每次调用都创建新的Lambda
fun processList(list: List<Int>) {
    list.filter { it > 0 }
        .map { it * 2 }
        .forEach { println(it) }
}

// 更好：复用函数引用
val isPositive: (Int) -> Boolean = { it > 0 }
val double: (Int) -> Int = { it * 2 }

fun processListOptimized(list: List<Int>) {
    list.filter(isPositive)
        .map(double)
        .forEach(::println)
}
```

## 六、测试中的使用

```kotlin
class TestHighOrderFunctions {
    
    @Test
    fun `test with mock functions`() {
        var callbackInvoked = false
        val mockCallback: () -> Unit = { callbackInvoked = true }
        
        val processor = Processor(mockCallback)
        processor.execute()
        
        assertTrue(callbackInvoked)
    }
    
    @Test
    fun `test function composition`() {
        val addOne = { x: Int -> x + 1 }
        val multiplyByTwo = { x: Int -> x * 2 }
        
        val composed = multiplyByTwo compose addOne
        
        assertEquals(6, composed(2)) // (2 + 1) * 2 = 6
    }
}
```

## 七、常见陷阱与解决方案

### 1. 空函数类型参数

```kotlin
// 使用可空函数类型
var onClick: ((View) -> Unit)? = null

// 安全调用
onClick?.invoke(view)

// 或者提供默认值
fun setOnClickListener(listener: (View) -> Unit = {}) {
    this.onClickListener = listener
}
```

### 2. 避免过度复杂

```kotlin
// 避免：过于复杂的函数签名
fun <A, B, C, D, E> superComplex(
    f: (A) -> B,
    g: (B) -> C,
    h: (C) -> D,
    i: (D) -> E
): (A) -> E {
    return { a -> i(h(g(f(a)))) }
}

// 保持简洁：使用标准库的 let、run、apply等
data.transform()
    .validate()
    .process()
```

### 3. 明确泛型类型

```kotlin
// 清晰明确的泛型参数
fun <T, R> List<T>.mapCustom(
    transform: (T) -> R
): List<R> {
    return this.map(transform)
}

// 避免过度泛化
fun <T> T.doEverything(action: (T) -> Unit): T {
    action(this)
    return this
}
```

## 八、实际工程案例

### 1. Android中的使用

```kotlin
class MainActivity : AppCompatActivity() {
    
    // View绑定扩展
    private inline fun <T : View> T.onClick(crossinline action: () -> Unit) {
        setOnClickListener { action() }
    }
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        
        findViewById<Button>(R.id.button).onClick {
            // 处理点击
            viewModel.loadData()
        }
    }
}
```

### 2. 后端API处理

```kotlin
class RequestHandler {
    
    fun handleRequest(
        request: Request,
        validate: (Request) -> ValidationResult = { ValidationResult.Valid },
        process: (Request) -> Response,
        log: (Request, Response) -> Unit = { _, _ -> }
    ): Response {
        val validation = validate(request)
        if (!validation.isValid) {
            return ErrorResponse(validation.message)
        }
        
        return process(request).also { response ->
            log(request, response)
        }
    }
}
```

总结要点

1. 保持简洁：高阶函数应该让代码更清晰，而不是更复杂
2. 类型安全：充分利用Kotlin的类型系统，使用类型别名提高可读性
3. 性能意识：合理使用inline，避免不必要的对象创建
4. 可测试性：高阶函数更容易进行单元测试和模拟
5. DSL设计：为特定领域创建直观的API
6. 错误处理：考虑函数参数可能抛出的异常
7. 文档化：复杂的高阶函数需要清晰的文档说明

高阶函数是Kotlin的强大特性，合理使用可以显著提高代码的表达力、复用性和可维护性。