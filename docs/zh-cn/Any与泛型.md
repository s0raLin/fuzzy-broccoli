泛型让我们在定义类（或接口）和方法时，可以对类型进行参数化。参数化类型使得我们能够重用同一段代码来处理不同的具体类型。

## 用泛型重用代码

考虑一个泛型类 `GenericType`，它用来存储某种“类型”的值：
```kotlin
class GenericType<T>(val t: T) {
    fun get(): T {
        return t
    }
}
```
这样，我们就可以用具体的类型创建这个类的对象，例如：
```kotlin
val stringInstance: GenericType<String> = GenericType<String>("abc")
val stringField = stringInstance.get()
```

同理，我们也可以用其他类型（如 `Int`、`Char`）创建实例，并调用 `get()` 方法来访问存储的字段。这样，泛型允许我们用同一份类和方法处理不同的类型。

---
## 用 Any 类型重用代码

还有另一种重用代码的方法，就是声明字段为 `Any` 类型，这样可以赋值任意引用类型。

示例：
```kotlin
class NonGenericClass(val value: Any) {    
    fun get(): Any {
        return value
    }
}
```
我们也可以用和之前泛型示例中一样的字符串来创建实例：
```kotlin
val anyInstance: NonGenericClass = NonGenericClass("abc")
```
同样，也可以用 `Int`、`Char` 或其他引用类型来创建实例。使用 `Any` 类型可以重用同一个类来存储不同类型的数据。

---

## 泛型的优势：从运行时转到编译时

调用 `get()` 方法后，我们得到的是 `Any` 类型，而不是具体的 `String` 或 `Int`：
```kotlin
val nonGenericInstance: NonGenericClass = NonGenericClass("abc")
val str: String = nonGenericInstance.get() // 编译错误：类型不匹配
```
要得到字符串，我们必须显式地进行类型转换：
```kotlin
val str: String = nonGenericInstance.get() as String // "abc"
```
这没问题，因为实际存的是字符串，但如果存的不是字符串呢？比如：
```kotlin
val instance: NonGenericClass = NonGenericClass(123)
val string: String = instance.get() as String // 抛出java.lang.ClassCastException
```
这时就会发生运行时异常。

这就是泛型相比 `Any` 的主要优势：因为泛型在编译时就进行类型检查，不需要显式类型转换，也避免了运行时异常。

例如：
```kotlin
val stringInstance: GenericType<String> = GenericType<String>("abc")

val str: String = stringInstance.get() // 不需要类型转换
val num: Int = stringInstance.get() // 编译失败
```
当程序员写错时，会在编译时就发现错误，而不是运行时才报错。泛型让编译器帮我们做类型转换检查，既安全又灵活。

---

## 多个类型参数

很多情况下，类需要多个类型参数，Kotlin 支持定义多个类型参数，用逗号分隔：
```kotlin
class MultipleGenerics<T, P>(var valueT: T, var valueP: P)
```
这比将所有参数都声明为 `Any` 更好也更安全。

---

## 举例说明泛型与 Any 的区别

定义两个类：泛型类 `PairGeneric` 和非泛型类 `PairNonGeneric`，它们都有两个任意类型的字段 `first` 和 `second`：
```kotlin
class PairGeneric<T, P>(var first: T, var second: P) {
    fun changeFirst(newValue: T) {
        first = newValue
    }

    fun changeSecond(newValue: P) {
        second = newValue
    }
    fun printData() {
        println("Value:")
        println("first = $first")
        println("second = $second")
    }
}

class PairNonGeneric(var first: Any, var second: Any) {
    fun changeFirst(newValue: Any) {
        first = newValue
    }

    fun changeSecond(newValue: Any) {
        second = newValue
    }
    fun printData() {
        println("Value:")
        println("first = $first")
        println("second = $second")
    }
}
```

测试代码：

```kotlin
fun main() {
    val genericPair: PairGeneric<String, Int> = PairGeneric("John", 8)
    val nonGenericPair: PairNonGeneric = PairNonGeneric("Kate", 18)

    genericPair.changeFirst(8) // 编译失败，类型不匹配
    nonGenericPair.changeSecond("Smith") // 允许，运行时不报错

    nonGenericPair.printData()
}
```
输出：
```makefile
Value:
first = Kate
second = Smith
```
由于 `PairNonGeneric` 中的字段是 `Any` 类型，可以轻易地赋值不同类型，这在小程序中可能没问题，但写大型程序时容易导致难以发现的错误。使用泛型的话，如果赋值类型不匹配，编译时就会报错，方便发现和修复问题。

----
## 总结

泛型和 `Any` 都能帮助我们写通用代码，但使用 `Any` 需要显式类型转换，容易出错；而泛型将类型检查推迟到编译时，保证类型安全，更灵活也更安全。

---

