# 范围
范围表达式的形式是基于 `rangeTo` 函数，它的操作符形式是 `..`，同时由 `in` 和 `!in` 来配合使用。范围的定义适用于任何可比较的类型，但是对于整数类型的基础类型，它的实现是优化过的。下面是一些用法的例子：

```kotlin
if (i in 1..10) { // equivalent of 1 <= i && i <= 10
    println(i)
}
```

整数类型的范围（`IntRange`，`LongRange`，`CharRange`）有一个额外的特性：他们可被遍历。编译器会将其类似地转换成 Java 的索引 `for` 循环，并且没有额外的开销。

```kotlin
for (i in 1..4) print(i) // prints "1234"

for (i in 4..1) print(i) // prints nothing
```

如果想逆序遍历数字呢？很简单，使用标准库中定义的 `downTo()` 函数即可：

```kotlin
for (i in 4 downTo 1) print(i) // prints "4321"
```

能是用 1 之外的任意步幅来遍历数字吗？当然，使用 `step()` 函数即可：

```kotlin
for (i in 1..4 step 2) print(i) // prints "13"

for (i in 4 downTo 1 step 2) print(i) // prints "42"
```

如果要创建一个不包含最后一个元素的范围，可以使用 `until` 函数：

```kotlin
for (i in 1 until 10) { // i in [1, 10), 10 is exclued
    println(i)
}
```

## 工作原理

范围实现了一个通用的接口：`ClosedRange<T>`。

`ClosedRange<T>` 表示一个数学上的闭区间（closed interval），适用于可比较类型。它有两个端点：`start` 和 `endInclusive`（包含在范围之内）。主要的操作是 `contains`，通常用于 `in/!in` 的操作形式中。

整数类型的渐进（`IntProgression`，`LongProgression`，`CharProgression`）表示一个数学渐进。渐进的定义包含 第一个（`first`）元素，最后（`last`）一个元素和一个非零的步幅（`step`）。第一个元素是 `first`，后续的元素是前一个元素加上 `step`。除非渐进为空，否则最后（`last`）一个元素总是会被遍历命中。

一个渐进是 `Iterable<N>` 的子类型，`N` 对应着 `Int`，`Long` 或者 `Char`，所以它可以用于 `for` 循环以及 `map`，`filter` 之类的函数中。`Proression` 的遍历等价于 Java/JavaScript 的索引 `for` 循环。

```kotlin
for (int i = first; i != last; i += step) {
  // ...
}
```

对于整数类型，`..` 操作符会创建一个实现了 `ClosedRange<T>` 和 `*Progression` 的对象。例如 `IntRange` 实现了 `ClosedRange<Int>` 并且继承了 `IntProgression`，因此 `IntProgression` 定义的所有操作在 `IntRange` 中也都可用。`downTo()` 和 `step()` 函数的结果永远是一个 `*Progression`。

渐进由定义在伴生对象的 `fromClosedRange` 函数所构建：

```kotlin
IntProgression.fromClosedRange(start, end, step)
```

渐进的最后（`last`）一个元素的计算方式如下，最后的结果是 `(last - first) % step === 0`：
* 如果 `step` 是正数，那么 `last` 不会大于 `end`；
* 如果 `step` 是负数，那么 `last` 不会小于 `end`。

## 实用函数

### `rangeTo()`

整数类型的 `rangeTo()` 操作符仅仅只是调用 `*Range` 类的构造器，例如：

```kotlin
class Int {
    // ...
    operator fun rangeTo(other: Long): LongRange = LongRange(this, other)
    // ...
    operator fun rangeTo(other: Int): IntRange = IntRange(this, other)
    // ...
}
```

浮点数（`Double`，`Float`）并没有定义他们的 `rangeTo` 操作符，标准库为泛型 `Comparable` 类型提供了替代方案：

```kotlin
public operator fun <T: Comparable<T>> T.rangeTo(that: T): ClosedRange<T>
```

这个函数返回的范围不能用于迭代。

### `downTo()`
`downTo()` 扩展函数是为任意的整数类型对而定义，这里有两个例子：

```kotlin
fun Long.downTo(other: Int): LongProgression {
    return LongProgression.fromClosedRange(this, other.toLong(), -1L)
}

fun Byte.downTo(other: Int): IntProgression {
    return IntProgression.fromClosedRange(this.toInt(), other, -1)
}
```

### `reversed()`
`reversed()` 扩展函数会作用于每一个 `*Progression` 类，所有的都返回一个逆向的渐进：

```kotlin
fun IntProgression.reversed(): IntProgress {
    return IntProgression.fromClosedRange(last, first, -step)
}
```

### `step()`
`steop()` 扩展函数会作用于每一个 `*Progression` 类，所有的都返回一个修改了 `step` 值（函数实参）的渐进。步幅值要求必须是正值，因此这个函数不会改变迭代的方向：

```kotlin
fun IntProgression.step(step: Int): IntProgression {
    if (step <= 0) throw IllegalArgumentException("Step must be positive, was: $step")
    return IntProgression.fromClosedRange(first, last, if (this.step > 0) step else -step)
}

fun CharProgression.step(step: Int): CharProgression {
    if (step <= 0) throw IllegalArgumentException("Step must be positive, was: $step")
    return CharProgression.fromClosedRange(first, last, if (this.step > 0) step else -step)
}
```

注意，作为返回值的渐进，它的 `last` 值可能与原始表达式的 `last` 值不一样，原因是为了保证公式 `(last - first) % step == 0` 的不变性。例如：

```kotlin
(1..12 step 2).last == 11   // progression with values [1, 3, 5, 7, 9, 11]
(1..12 step 3).last == 10   // progression with values [1, 4, 7, 10]
(1..12 step 4).last == 9    // progression with values [1, 5, 9]
```


