# 反射
反射是一组语言特性或者库特性，它允许在运行时“反省”我们自己的程序的结构。Kotlin 把函数和属性作为语言的一等公民，“反省”他们（例如，在运行时获取属性或者函数的名字或类型）可以用一种函数式或者响应式的方式交织在（closely interwined）一起。

> 在 Java 平台上，反射由一个运行时组件来支持，这个组件分发为为一个额外的 JAR 文件（kotlin-reflect.jar）。对于不使用反射的应用，这样可以减少运行时库的大小。如果确实要用到反射，务必确保 .jar 文件添加到了工程的 classpath 中。

## 类引用
反射最基本的特征是获取 Kotlin 类运行时的引用，获取一个静态已知类的引用可以使用*类字面量*语法：

```kotlin
val c = MyClass::class
```

这个引用是 KClass 类型的值。

注意，Kotlin 的类引用不同于 Java。如果要获取 Java 的类引用，请使用 `KClass` 实例的 `.java` 属性。

## 绑定的类引用（从 1.1 开始支持）
如果要获取一个具体对象的类引用，可以把对象作为接收者，使用同样的 `::class` 语法：

```kotlin
val widget: Widget = ...
assert(widget is GoodWidget) { "Bad widget: ${widget::class.qualifiedName}" }
```

我们获取到一个对象的真实类，例如 `GoodWidget` 或 `BadWidget`，即使接收者的类型是 `Widget`。

## 函数引用
如果我们把一个命名函数声明为：

```kotlin
fun isOdd(x: Int) = x % 2 != 0
```

我们可以简单地直接调用它（`isOdd(5)`），但是我们也可以将其作为一个值来传递（例如，传递给其它函数）。我们可以使用 `::` 操作符来做到这一点：

```kotlin
val numbers = listOf(1, 2, 3)
println(numbers.filter(::isOdd)) // prints [1, 3]
```

这里的 `::isOdd` 是函数类型 `(Int) -> Boolean` 的一个值。

如果根据上下文可以知道期望的类型，`::` 也能用于重载的函数。例如：

```kotlin
fun isOdd(x: Int) = x % 2 != 0
fun isOdd(s: String) = s == "brillig" || s == "slithy" || s == "tove"

val numbers = listOf(1, 2, 3)
println(numbers.filter(::isOdd) // refers to isOdd(x: Int)
```

或者还有一种方式，我们可以把方法的引用存储在一个变量中并且指明类型，这样就提供了必要的上下文：

```kotlin
val predicate: (String) -> Boolean = ::isOdd    // refers to isOdd(x: String)
```

如果我们需要使用类的成员，或者扩展函数，需要加以限定。例如，`String::toCharArray` 是一个类型为 `String: String.() -> CharArray` 的扩展函数。

## 例子：函数组合
考虑如下函数：

```kotlin
fun <A, B, C> compose(f: (B) -> C, g: (A) -> B): (A) -> C {
    return { x -> f(g(x)) }
}
```

它的返回值是两个函数的组合：`compose(f, g) = f(g(*))`。然后就可以将其应用到可调用的引用上：

```kotlin
fun length(s: String) = s.length

val oddLength = compose(::isOdd, ::length)
val strings = listOf("a", "ab", "abc")

println(strings.filter(oddLength)) // Prints "[a, abc]"
```

## 属性引用
访问作为一等公民的属性也可以使用 `::` 操作符：

```kotlin
val x = 1

fun main(args: Array<String>) {
    println(::x.get())  // prints "1"
    println(::x.name)   // prints "x"
}
```

`::x` 表达的计算结果是 `KProperty<Int>` 类型的属性对象，它允许我们使用 `get()` 读取它的值，或者利用 `name` 属性取得属性名。更多信息可前往 `Kproperty` 类的文档。

对于可变属性，例如 `var y = 1`，`::y` 返回一个 `KMutableProperty<Int>` 类型的值，它有 `get()` 方法：

```kotlin
var y = 1

fun main(args: Array<String>) {
    ::y.set(2)
    println(y)  // prints "2"
}
```

在使用无参函数的地方也可以使用类型引用：

```kotlin
val strs = listOf("a", "bc", "def")
println(strs.map(String::length)) // prints [1, 2, 3]
```

如果要访问作为类成员的属性，我们需要加以限定：

```kotlin
class A(val p: Int)

fun main(args: Array<String>) {
    val prop = A::p
    println(prop.get(A(1))) // prints "1"
}
```

对于扩展属性：

```kotlin
val String.lastChar: Char
    get() = this[length - 1]

fun main(args: Array<String>) {
    println(String::lastChar.get("abc")) // prints "c"
}
```

### 与 Java 反射的互操作性

针对 Java 平台的标准库中包含了反射类的扩展，它们提供了与 Java 反射对象之间的一个映射（见 `kotlin.reflect.jvm` 包）。例如，如果要找出 Kotlin 属性的幕后字段或者是等价于 getter 的 Java 方法，可以写如下代码：

```kotlin
import kotlin.reflect.jvm.*

class A(val p: Int)

fun main(args: Array<String>) {
    println(A::p.javaGetter)  // prints "public final int A.getP()"
    println(A::p.javaField) // prints "private final int A.p"
}
```

如果要获取与 Java 类对应的 Kotlin 类，可以使用 `.kotlin` 扩展属性：

```kotlin
fun getKClass(o: Any): KClass<Any> = o.javaClass.kotlin
```

## 构造器引用

构造器可以像方法和属性那样被引用。如果函数类型的对象所使用的参数和构造器一样，并且返回合适类型的对象，那么使用这个函数的地方也可以使用构造器。构造器的引用方式是 `::` 操作符加上类名。考虑下满这个函数，它的参数是一个没有参数的函数类型，返回值类型是 `Foo`：

```kotlin
class Foo

fun function(factory: () -> Foo) {
    val x: Foo = factory()
}
```

使用 `::Foo`（Foo 类的无参构造器），我们可以简单地像下面这样调用它：

```kotlin
function(::Foo)
```

## 绑定的函数和属性引用（从 1.1 开始支持）

我们可以引用一个特定对象的实例方法：

```kotlin
val numberRegex = "\\d+".toRegex()
println(numberRegex.matches("29")) // prints "true"

val isNumber = numberRegex::matches
println(isNumber("29"))
```

上面的代码中并没有直接调用 `matches` 方法，而是把它的引用存了下来。这个引用会跟它的接收者绑定。它可以被直接调用（上例），也可以用在使用函数类型表达式的地方：

```kotlin
val strings = listOf("abc", "124", "a70")
println(strings.filter(numberRegex::matches)) // prints "[124]"
```

对比一下绑定的和对应的未绑定的引用的类型。绑定的可调用引用会把接收者“附加”给自己，所以接收者的类型就再也不是一个参数：

```kotlin
val isNumber: (CharSequence) -> Boolean = numberRegex::matches

val matches: (Regex, CharSequence) -> Boolean = Regex::matches
```

属性引用也可以绑定：

```kotlin
val prop = "abc"::length
println(prop.get()) // prints "3"
```

从 Kotlin 1.2 开始，明确把 `this` 指明为接收者变成非必要了：`this::foo` 和 `::foo` 是等价的。
