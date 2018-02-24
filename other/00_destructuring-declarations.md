解构声明
===

如果能够把一个对象解构成多个变量，用起来会很方便，例如：

```kotlin
val (name, age) = person
```

这种语法叫 _解构声明_（destructuring declaration）。一个解构声明可以同时创建多个变量。

上例中，我们声明了两个变量，而且可以独立使用它们。

```kotlin
println(name)
println(age)
```

一个解构声明会编译成如下代码：

```kotlin
val name = person.component1()
val age = person.component2()
```

`component1()` 和 `component2()` 函数是*约定原则（the principle of conventions）*的另一个例子，*约定原则*在 Kotlin 中有广泛的使用，例如，`+` 和 `*` 以及 `for` 循环。只要提供了对应数量的组件函数（component function）就可以作为解构声明的右侧变量。所以，可以有 `component3`、`component4` 等等。

需要注意的是，`componentN` 函数需要用 `operator` 关键词修饰，这样才能在解构声明中使用。

结构声明可以用在 `for` 循环中：

```kotlin
for ((a, b) in collection) { ... }
```

变量 `a` 和 `b` 的赋值来自于 collection 元素的 `component1` 和 `component2`。

栗子：一个函数返回两个变量
---
假如说我们需要一个函数返回两个值。例如，一个结果对象和一个状态码。一个比较简洁（compact）的方式是声明一个[数据类](classes-and-objects/data-classes.md)，然后返回它的实例。

```kotlin
data class Result(val result: Int, val status: Status)

fun function(...): Result {
    // computations

    return Result(result, status)
}

// Now, to use this function
val (result, status) = function(...)
```

因为数据类会自动生成 `componentN` 函数，可以直接用于解构声明。

栗子：解构声明和 Map
---

如下可能是遍历 map 最优雅的方式：

```kotlin
for ((key, value) in map) {
    // do something with the key and the value
}
```

为了满足以上操作，我们需要：

- 通过提供 `iterator()` 函数让 map 表示一系列值（a sequence of values）
- 通过提供 `component1()` 和 `component2()` 函数让 map 的每个元素表示一个键值对（a pair）

标准库确实提供了如下扩展：

```kotlin
operator fun <K, V> Map<K, V>.iterator(): Iterator<Map.Entry<K, V>> = entrySet().iterator()
operator fun <K, V> Map.Entry<K, V>.component1() = getKey()
operator fun <K, V> Map.Entry<K, V>.component2() = getValue()
```

所以，你可以自由地在 for 循环中使用解构声明来操作 map（当然也包含数据类的集合）。

下划线表示无用变量（1.1 开始支持）
---

解构声明中不需要的变量可以用下划线表示：

```kotlin
val (_, status) = getResult()
```

下划线对应的 `componentN` 函数会被直接跳过，并不会被执行。

Lambda 中的解构（1.1 开始支持）
---
结构声明也可以用于 lambda 参数。如果 labmda 的某个参数是 `Pair` 类型（或者 `Map.Entry`，或者任何实现了 `componentN` 函数的类型），可以把这个参数结构成括号内的多个参数：

```kotlin
map.mapValues { entry -> "${entry.value}!" }
map.mapValues { (key, value) -> "$value!" }
```

直接声明两个参数和把一个参数结构成 pair 是不同的：

```kotlin
{ a -> ... }            // one parameter
{ a, b -> ... }         // two parameters
{ (a, b) -> ... }       // a destructured pair
{ (a, b), c -> ... }    // a desctructed pair and another parameter
```

如果参数结构之后的某个元素用不到，可以用下划线表示，这样就不用取名了。

```kotlin
map.mapValues { (_, value) -> "$value!" }
```

可以指定整个结构参数的类型或者单独指定某个参数：

```kotlin
map.mapValues { (_, value): Map.Entry<Int, String> -> "$value!" }
map.mapValues { (_, value: String) -> "$value!" }
```
