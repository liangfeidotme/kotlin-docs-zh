# 类型别名
类型别名可以为现有类型提供替代名称。如果一个变量的名字比较长，我们可以用一个短名字来代替。

我们经常会希望能够缩短比较长的泛型类型。例如，缩短集合类型通常会比较有吸引力：

```kotlin
typealias NodeSet = Set<Network.Node>
typealias FileTable<K> = MutableMap<K, MutableList<File>>
```

也可以为函数类型提供不同别名：

```kotlin
typealias MyHandler = (Int, String, Any) -> Unit
typealias Predicate<T> = (T) -> Boolean
```

可以给内部和嵌套类提供新名字：

```kotlin
class A {
    inner class Inner
}
class B {
    inner class Inner
}

typealias AInner = A.Inner
typealias BInner = B.Inner
```

类型别名并不会引入新的类型。它们等价于其所对应的底层类型。当新增 `typealias Predicate<T>` 并且在代码中使用 `Predicate<Int>` 时，Kotlin 编译器会把它展开为 `(Int) -> Boolean`。所以在需要通用函数类型的地方，我们可以传入一个别名类型的变量，反之亦然：

```kotlin
typealias Predicate<T> = (T) -> Boolean

fun foo(p: Predicate<Int>) = p(42)

fun main(args: Array<String>) {
    val f: (Int) -> Boolean = { it > 0 }
    println(foo(f))  // prints "true"

    val p: Predicate<Int> = { it > 0 }
    println(listOf(1, -2).filter(p)) // prints "[1]"
}
```
