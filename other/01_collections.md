# 集合：List / Set / Map
与很多其他语言不同的是，Kotlin 会区分可变（mutable）和不可变（immutable）的集合（例如 list、set、map 等）。精确地控制集合何时可以被编辑不但可以排除 bug，还有助于设计良好的 API。

理解清楚可变集合的*视图*和真正的可变集合之间的区别是非常重要的。两者都很容易创建，但是类型系统并不能表明两者的不同，如果有必要的话，需要开发者自己来跟踪。

`List<out T>` 是一个接口，它提供了只读的操作，例如 `size`，`get` 等。与 Java 一致，它继承了 `Collection<T>`，进而也继承了 `Iterable<T>`。能够改动 list 的方法由 `MutableList<T>` 提供。这种模式同样适用于 `Set<out T>/MutableSet<T>` 和 `Map<K, out V>/MutableMap<K, V>`。

先来看一下 list 和 set 类型的基本用法：

```kotlin
val numbers: MutableList<Int> = mutableListOf(1, 2, 3)
val readOnlyView: List<Int> = numbers
println(numbers)        // prints "[1, 2, 3]"
numbers.add(4)
println(readOnlyView)   // prints "[1, 2, 3, 4]"
readOnlyView.clear()    // -> does not compile

val strings = hashSetOf("a", "b", "c", "c")
assert(strings.size == 3)
```

Kotlin 没有专用的（dedicated）语法来创建 list 和 set。只能使用标准库提供的 `listOf()`、`mutableListOf()`、`setOf()`。对性能要求不高的代码可以通过习语 - `mapOf(a to b, c to d)` 来创建 map。

注意，变量 `readOnlyView` 指向了同一个 list 并且会随这个底层的 list 发生变化。如果一个 list 唯一的引用是一个只读的变体，那么可以认为这个集合是完全不可变的。这种集合的创建方式很简单：

```kotlin
val items = listOf(1, 2, 3)
```

当前，`listOf` 方法的实现利用了数组列表，但是将来可以基于“他们知道他们不可变”这个事实直接返回一个内存更高效的完全不可变集合。

注意，只读类型是协变的。也就是说，`List<Rectangle>` 可以赋给 `List<Shap>`（假设 Rectangle 继承自 Shape）。这个操作无法用于可变的集合类型，因为可能会导致运行时出错。

有时，我们希望给调用方返回一个集合的快照，并且这个快照保证不会改变：

```kotlin
class Controller {
    private val _items = mutableListOf<String>()
    val items: List<String> get() = _items.toList()
}
```

`toList` 这个扩展方法只是拷贝了一份 list 的元素，如此一来，返回的 list 绝对不会改变。

list 和 set 有很多有用的扩展方法：

```kotlin
val items = listOf(1, 2, 3, 4)
items.first() == 1
items.last() == 4
items.filter { it % 2 == 0} // returns [2, 4]

val rwList = mutableListOf(1, 2, 3)
rwList.requireNoNulls()     // returns [1, 2, 3]
if (rwList.none { it > 6 }) println("No items above 6") // prints "No items above 6"
val item = rwList.firstOrNull()
```

除此之外，还有 `sort`、`zip`、`fold`、`reduce` 等等。

Map 遵守同样的模式 。创建和访问的方式如下：

```kotlin
val readWriteMap = hashMapOf("foo" to 1, "bar" to 2)
println(readWriteMap["foo"])
val snapshot: Map<String, Int> = HashMap(readWriteMap)
```
