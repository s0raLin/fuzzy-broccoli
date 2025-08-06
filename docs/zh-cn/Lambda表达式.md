## 1. 函数基础

在 Kotlin 中，函数声明形式如下：
```kotlin
fun functionName(param1: Type1, param2: Type2): ReturnType {
    // 函数体
    return someValue
}
```
示例：
```kotlin
fun sum(a: Int, b: Int): Int {
    return a + b
}
```
调用：
```kotlin
val result = sum(2, 3)  // result = 5
```

----
## 2. 函数类型与高阶函数

Kotlin 中函数也是“值”，它们有自己的类型。例如：
```kotlin
val f: (Int, Int) -> Int = ::sum
```
这里 `f` 是一个函数类型的变量，代表一个接收两个 `Int` 返回 `Int` 的函数。`::sum` 是函数引用。

高阶函数指的是**以函数作为参数或返回值的函数**，例如：
```kotlin
fun operateOnTwoNumbers(a: Int, b: Int, op: (Int, Int) -> Int): Int {
    return op(a, b)
}

val result = operateOnTwoNumbers(2, 3, ::sum)  // result = 5
```

## 3. Lambda 表达式基础

Lambda 是匿名函数的简写形式，语法为：
```kotlin
val lambdaName: (参数类型) -> 返回类型 = { 参数列表 -> 函数体 }
```
示例：
```kotlin
val add: (Int, Int) -> Int = { x, y -> x + y }
println(add(2, 3))  // 输出 5
```
在函数参数中使用 lambda：
```kotlin
listOf(1, 2, 3).filter { it > 1 }  // 返回 List(2, 3)
```
这里的 `{ it > 1 }` 是一个 lambda 表达式，`it` 是隐式参数，代表列表中的每个元素。

---

## 4. Lambda 返回值

- Lambda 的最后一行表达式是返回值，不需要显式写 `return`。
    
- 如果是多行代码，最后一行决定返回值。
    

示例：
```kotlin
val result = listOf(1, 2, 3).map {
    val doubled = it * 2
    doubled + 1  // 最后一行是返回值
}
println(result)  // [3, 5, 7]
```
---

## 5. `return` 的用法区别

- 在普通函数中，`return` 用来结束函数并返回结果。
    
- 在 lambda 中，如果用不带标签的 `return`，默认返回的是外层函数，可能导致编译错误。
    
- 用带标签的限定返回 `return@label`，可以只退出 lambda，而不中断外层函数。
    

---
## 6. 变量捕获（闭包）

Lambda 可以访问和修改外部变量：
```kotlin
var count = 0
val increment = {
    count += 1
    println(count)
}
increment()  // 1
increment()  // 2
println(count)  // 2，外部变量也变了
```
这是闭包的特性，lambda 捕获外部变量的引用，而不是复制值。

---

## 7. Kotlin 标准库中常用的高阶函数

- `filter` — 过滤满足条件的元素。
    
- `map` — 对集合中的元素做映射转换。
    
- `forEach` — 对每个元素执行操作。
    
- `fold` / `reduce` — 聚合函数。
    
- `any` / `all` — 判断是否存在满足条件的元素。
    

这些都需要传入 lambda 作为参数。

----
# 总结

理解这些基础后：

- 你能更好地写 lambda 表达式。
    
- 理解高级用法中的“提前返回”和“变量捕获”。
    
- 明白为什么函数可以像变量一样被传递和操作。

-----
# 复杂的 lambda 表达式

有时，lambda 表达式中的代码不够简短，不能写成一行，这时你需要将代码分成多行。在这种情况下，lambda 中的最后一行会被当作 lambda 的返回值。
```kotlin
fun main() {
    val originalText = "I don't know... what to say...123456"
    val textWithoutSmallDigits = originalText.filter {
        val isNotDigit = !it.isDigit()
        val stringRepresentation = it.toString()

        isNotDigit || stringRepresentation.toInt() >= 5
    }
    println(textWithoutSmallDigits)
}
```
**代码说明**  
这个程序会输出：
```cpp
// I don't know... what to say...56
```

---

# 提前返回 (Earlier Returns)

此外，lambda 还可以包含“提前返回”。在 Kotlin 中，“提前返回”是指使用 `return` 关键字提前结束 lambda 表达式或函数的执行，且不必等到代码块末尾。

提前返回必须使用限定返回（qualified return）语法，也就是说，`return` 后面需要加上 `@` 和标签名。标签名通常是传入 lambda 的函数名。我们来把上面那个例子改写成使用提前返回的写法，结果保持不变：
```kotlin
fun main() {
    val originalText = "I don't know... what to say...123456"
    val textWithoutSmallDigits = originalText.filter {
        if (!it.isDigit()) {
            return@filter true
        }

        it.toString().toInt() >= 5
    }
    println(textWithoutSmallDigits)
}
```
**代码说明**  
这个 lambda 表达式中，如果当前字符不是数字，马上提前返回 `true`，否则判断该数字是否大于等于 5。输出结果与之前相同：
```cpp
// I don't know... what to say...56
```

---

# 捕获变量（Capturing Variables）

捕获变量，也叫闭包（closure），指的是 lambda 表达式或匿名函数把外部作用域中的变量包裹进来，使得这些变量可以在 lambda 内部使用。

当你捕获一个变量时，lambda 会“记住”那个变量的值，即使外部变量已经不在作用域内，lambda 依然可以访问它。这让我们写依赖于外部状态的函数变得更加方便，而不需要通过参数传递。

捕获变量在事件驱动编程或回调函数中尤其有用。比如，你有一个异步操作，回调函数里需要访问外部的 UI 元素状态，lambda 通过捕获这些变量就可以做到。

在 Kotlin 中，如果 lambda 使用了外部变量，我们说这个变量被 lambda 捕获了。捕获的变量可以被 lambda 读取，也可以修改，且修改对外部变量是可见的。

来看一个例子：
```kotlin
var count = 0

val changeAndPrint = {
    ++count
    println(count)
}

println(count)    // 0
changeAndPrint()  // 1
count += 10
changeAndPrint()  // 12
println(count)    // 12
```
**代码说明**  
这里我们定义了一个 lambda `changeAndPrint`，它会将 `count` 加一并打印。注意打印的数字会反映外部变量 `count` 的变化。lambda 内外对 `count` 的修改是互相影响的。

---

# 高阶函数示例：运行时创建函数

下面的函数将一个接受两个参数的函数 `f` 转换为一个只接受一个参数的函数。实现方式是创建一个新的 lambda，捕获了 `value` 和 `f`：
```kotlin
fun placeArgument(value: Int, f: (Int, Int) -> Int): (Int) -> Int {
    return { i -> f(value, i) }
}
```
比如：
```kotlin
fun sum(a: Int, b: Int): Int = a + b
val mul2 = { a: Int, b: Int -> a * b }
```
我们可以这样调用：
```kotlin
val increment = placeArgument(1, ::sum)
val triple = placeArgument(3, mul2)

println(increment(4))   // 5
println(increment(40))  // 41
println(triple(4))      // 12
println(triple(40))     // 120
```
这里 `increment` 和 `triple` 都是新的函数，由原函数和捕获的参数组合而成。

---

# 总结

- Kotlin 支持复杂的多行 lambda，最后一行是返回值。
    
- 可以用限定返回语法实现 lambda 内部的提前返回。
    
- lambda 可以捕获并操作外部变量，形成闭包。
    
- 通过捕获变量，可以在运行时动态创建新的函数，写出更简洁优雅的函数式代码。
    
- 函数在 Kotlin 中是“一等公民”，可以像变量一样传递和操作。
    

---
