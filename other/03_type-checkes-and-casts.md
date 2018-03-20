# 类型检查和转换：'is' 和 'as'

## `is` 和 `!is` 操作符
在运行时可以通过 `is` 或者它的否定形 `!is` 来判断一个对象是否是某个类型：

```kotlin
if (obj is String) {
    print(obj.length)
}

if (obj !is String) { // same as !(obj is String)
    print("Not a String")
} else {
    print(obj.length)
}
```

## 智能转换
大多数情况下，我们不会用到显示转换操作符，因为编译器会追踪 `is` 检查以及不可变值的显示转换，然后在需要时自动插入（安全）转换代码：

```kotlin
fun demo(x: Any) {
    if (x is String) {
        print(x.length) // x is automatically cast to String
    }
}
```

编译器对于类型转换是否安全是足够智能的，例如否定检查导致的返回：

```kotlin
if (x !is String) return
print(x.length) // x is automatically cast to String
```

或者位于 `&&` 和 `||` 的右侧：

```kotlin
// x is automatically cast to string on the right-hand side of `||`
if (x !is String || x.length == ) return

// x is automatically cast to string on the right-hand side of `&&`
if (x is String && x.length > 0) {
    print(x.length) // x is automatically cast to String
}
```

这种智能转换也适用于 `when` 表达式和 `while` 循环：

```kotlin
when (x) {
    is Int -> print(x + 1)
    is String -> print(x.length + 1)
    is IntArray -> print(x.sum())
}
```

当编译器无法保证变量在检查和使用之间不能改变时，智能转换并不会工作。更具体一点，只有满足如下条件时，只能转换才可用：

* `val` 局部变量 - 永远不会包含局部代理属性。
* `val` 属性 - 如果属性是私有属性或内部属性，或者检查发生在属性声明所在的模块。智能转换对于开放的属性或者有自定义 getter 的属性并不可用。
* `var` 局部变量 - 这个局部变量不能在检查和使用之间被修改，不能在 lambda 中被修改，也不是局部代理属性。
* `var` 属性 - 永远不会（因为这个变量随时都会被其他代码修改）。

## “不安全”的转换操作符
通常情况下，如果转换是不可能的，转换操作符会抛出一个异常。我们称之为*不安全*的。这个不安全的转换在 Kotlin 中是通过中缀操作符 `as` 完成的：

```kotlin
val x: String = y as String
```

注意，`null` 不能转换成 `String`，因为它是非空类型，例如，如果 `y` 是 `null`，上面的代码会抛出一个异常。为了匹配 Java 的转换语义，我们必须在转换操作的右侧使用可空类型：

```kotlin
val x: String? = y as String?
```

## “安全”（可空）的转换操作
为了避免异常抛出，可以使用**安全**的转换操作符 `as?`，它在失败会返回 `null`：

```kotlin
val x: String? = y as? String
```

注意，即使 `as?` 右侧是一个非空类型 `String`，转换的结果依然是可空的。

## 类型擦除和泛型类型检查
Kotlin 只能在编译时保证涉及到泛型的操作的类型安全，然而在运行时，泛型实例并没有携带关于真实类型实参的任何信息。例如，`List<Foo>` 会擦除为 `List<*>`。一般情况下，无法在运行时去检查一个实例是否属于某个具体类型实参的泛型类型。

基于以上，编译器会禁止 `is` 检查，因为类型擦除的原因，这个检查无法在运行时执行，例如 `ints is List<Int>` 或者 `list is T`（类型形参）。但是，我们可以检查一个实例是否是星映射类型。

```kotlin
if (something is List<*>) {
    something.forEach { println(it) } // The items are typed as `Any?`
}
```

同样的，当一个实例的类型参数（在编译时）已经被检查过之后，可以对类型的非泛型部分做 `is` 检查或者转换。这种情况下会去掉尖括号：

```kotlin
fun handleStrings(list: List<String>) {
    if (list is ArrayList) {
        // `list` is smart-cast to `ArrayList<String>`
    }
}
```

同样的去掉类型实参的语法也适用于不考虑类型实参的转换：`list as ArrayList`。

带具体化类型参数的内联函数会把实际的类型实参内联到调用处，这也使得 `arg is T` 对类型参数是可行的，但是，如果 `arg` 本身是一个泛型的实例，那么它的类型参数依然会被擦除。例如：

```kotlin
inline fun <reified A, reified B> Pair<*, *>.asPairOf(): Pair<A, B>? {
    if (first !is A || second !is B) return null
    return first as A to second as B
}

val somePair: Pair<Any?, Any?> = "items" to listOf(1, 2, 3)

val stringToSomething = somePair.asPairOf<String, Any>()
val stringToInt = somePair.asPairOf<String, Int>()
val stringToList = somePair.asPairOf<String, List<*>>()
val stringToStringList = somePair.asPairOf<String, List<String>>()
```

## 未检查的转换
如上所述，类型擦除使得在运行时检查泛型实例的实际类型形参变得不可能，而且代码中的泛型类型由于彼此连接得不太紧密，编译器也无法确保类型安全。

即便如此，我们高级的程序逻辑来应用
