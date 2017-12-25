控制流：if, when, for, white
===

if 表达式
---

Kotlin 的 `if` 是一个表达式，也就是说，它有返回值。因此三元操作符（condition ? then : else）就失去用武之地了，因为普通 `if` 就可以担当此任。


```kotlin
// Traditional usage
var max = a
if (a < b) max = b

// With else
var max: Int
if (a > b) {
    max = a
} else {
    max = b
}

// As expression
val max = if (a > b) a else b
```

`if` 分支也可以是 block，最后一个表达式是 block 的值：

```kotlin
val max = if (a > b) {
    print("Choose a")
    a
} else {
    print("Choose b")
    b
}
```

如果把 `if` 用作表达式而不是声明的话（例如，返回它的值或者把它赋给其他变量），这个表达式需要一个 `else` 分支。

when 表达式
---
`when` 取代了 switch 操作符（比如 C 语言）。最简单的形式：

```kotlin
when (x) {
    1 -> print("x == 1")
    2 -> print("x == 2")
    else -> { // Note the block
        print("x is neither 1 nor 2")
    }
}
```

`when` 会把参数按顺序去匹配所有分支，直到某个满足某个分支条件。`when` 可以是表达式，也可以是声明。如果是表达式，满足条件的分支的值就成为整个表达式的值。如果是声明，单个分支的值会被忽略。（类似 `if`，每个分支都可以是 block，block 中最后一个表达式的值才是它的值）。

`else` 在其他分支条件无法成立时会被执行。如果 `when` 用作表达式，那么 `else` 是强制要求，除非编译器能够验证所有的分支条件都被覆盖到了。

如果多个分支的处理方式一致，可以用逗号把分支条件联合起来：

```kotlin
when (x) {
    0, 1 -> print("x == 0 or x == 1")
    else -> print("otherwise")
}
```

不只是常量，任何表达式都可以作为分支条件：

```kotlin
when (x) {
    parseInt(s) -> print("s encodes x")
    else -> print("s does not encode x")
}
```

也可以用 `in` 或者 `!in` 来判断一个值是否在 range 或 collection 内：

```kotlin
when (x) {
    in 1..10 -> print("x is in the range")
    in validNumbers -> print("x is valid")
    !in 10..20 -> print("x is outside the range")
    else -> print("none of the above"
}
```

还可以用 `is` 或者 `!is` 来做类型判断。因为有 smart cast，所以无需显示的类型转换：


```kotlin
fun hasPrefix(x: Any) = when(x) {
    is String -> x.startsWith("prefix")
    else -> false
}
```

`when` 也可以取代 `if-else-if` 链。如果 `when` 没有参数，那么分支条件只是布尔表达式，当条件满足时，分支就会被执行。

```kotlin
when {
    x.isOdd() -> print("x is odd")
    x.isEven() -> print("x is even")
    else -> print("x is funny")
}
```

for 循环
---
只要提供了迭代器，`for` 循环就可以遍历。类似 C# 的 `foreach`。语法如下：

```kotlin
for (item in collection) print(item)
```

body 可以是 block：

```kotlin
for (item: Int in ints) {
    // ...
}
```

怎样才算是“提供迭代器”呢？

* 拥有一个 `iterator()` 的成员或扩展函数，并且返回值要满足如下条件：
    * 有一个成员或扩展函数 `next()`
    * 有一个成员或扩展函数 `hasNext()`，并且返回值类型是 `Boolean`

以上三个函数需要使用 `operator` 来标记。

作用域数组的 `for` 循环会编译成一个基于索引（index-based）的循环，所以不会创建迭代器。

利用索引遍历数组的方式如下：

```kotlin
for (i in array.indices) {
    print(array[i])
}
```

注意，“区域的遍历（iteration through a range）”会做最佳优化，并不会借助额外对象。

另外，`withIndex` 是一个库函数：

```kotlin
for ((index, value) in array.withIndex()) {
    println("the element at $index is $value")
}
```

while 循环
---

`while` 和 `do..while` 的用法没有不同：

```kotlin
while (x > ) {
    x--
}

do {
    val y = retrieveData();
} while (y != null) // y is visible here!
```

循环中的 break 和 continue
---

Kotlin 也支持传统的 `break` 和 `continue`。具体可见[返回和跳转](returns-and-jumps.md)
