# 内联函数
使用高阶函数需要付出运行时代价：每个函数都是一个对象，并且拥有一个闭包，即函数体内可访问的那些变量。内存分配（包括函数对象还有类）以及虚调用（virtual call）都会带来运行时开销。

但是，貌似很多情况下，内联 lambda 表达式可以消除这类开销。下面的函数很好低展示了这种情况。`lock` 函数可以简单地内联到调用处。考虑如下 case：

```kotlin
lock(l) { foo() }
```

它不会为参数创建一个函数对象然后生成一个调用，相反，编译器会生成如下代码：

```kotlin
l.lock()
try {
    foo()
}
finally {
    l.unlock()
}
```

这不就是我们一开始就想要得到的吗？

为了让编译器实现这个，我们需要用 `inline` 修饰符来标记 `lock()` 函数：

```kotlin
inline fun <T> lock(lock: Lock, body: () -> T): T {
    // ...
}
```
`inline` 修饰符会影响到函数资深以及传入到的 lambda：他们都会内联到调用处。

内联可能会导致代码体积变大；但是，如果以一种比较合理的方式使用（例如，避免内联大函数），它会打来性能上的回报，特别是循环中大量的调用。

## 非内联
如果只想让传入到内联函数的某些 lambda 成为内联，可以用 `noinline` 修饰符来标记函数参数：

```kotlin
inline fun foo(inlined: () -> Unit, noinline notInlined: () -> Unit) {
    // ...
}
```

可内联的 lambda 只能在内联函数中调用，或者作为可内联的参数来传递，但是 `noinline` 可以用任何我们喜欢的方式来操作：存储在字段中，传递分发等。

如果一个内联函数没有可内联的函数参数以及没有具体化类型的参数，编译器会提出警告，因为内联这种函数很可能会带来益处（如果确信需要这种内联，可以使用注解 `@Suppress("NOTHING_TO_INLINE")` 注解来抑制这种警告）。

## 非局部返回
我们只能使用一个普通的，无限定的 `return` 来离开命名函数或者匿名函数。这就意味着，如果要离开一个 lambda，我们必须使用标签，lambda 不允许使用裸 `return`。因为 lambda 并不能使包围函数返回：

```kotlin
fun foo() {
    ordinaryFunction {
        return // ERROR: can not make `foo` return here
    }
}
```

但是如果传入 lambda 的函数是内联的，返回也可以被内联，因此如下是可以的：

```kotlin
fun foo() {
    inlineFunction {
        return // OK: the lambda is inlined
    }
}
```
这类返回（位于 lambda 内，但是离开的是包围函数）被称作*非局部*返回。我们比较熟悉循环中的这类构造，其包围在内联函数之内：

```kotlin
fun hasZeros(ints: List<Int>): Boolean {
    ints.forEach {
        if (it == ) return true // returns from hasZeros
    }
    return false
}
```

注意，有的内联函数不会直接在函数体内调用传入的 lambda，而是在其他的执行环境中调用，例如局部对象或者嵌套函数。这种情况下，非局部的控制流也是不允许的。如果要指明这点，可以使用 `crossinline` 修饰符来标记 lambda 参数：

```kotlin
inline fun f(crossinline body: () -> Unit) {
    val f = object: Runnable {
        override fun run() = body()
    }
}
```

> `break` 和 `continue` 在内联 lambda 中还不可用，但是已经排上日程。

## 具体化的类型参数
有事我们需要访问作为参数传递过来的类型：

```kotlin
fun <T> TreeNode.findParentOfType(clazz: Class<T>): T? {
    var p = parent
    while (p != null && !clazz.isInstance(p)) {
        p = p.parent
    }
    @Suppress("UNCHCKED_CAST")
    return p as T?
}
```

以上代码会往上遍历一个树，利用反射来检查某个节点是否是某个类型。这样没有任何问题，但是调用处的代码不算美观：

```kotlin
treeNode.findParentOfType(MyTreeNode::class.java)
```

我们实际需要的只是简单地给函数传入一个类型，例如下面这种调用方式：

```kotlin
treeNode.findParentOfType<MyTreeNode>()
```

为了达到这个目标，内联函数支持*具体化的类型参数*，所以我们可以写成如下形式：

```kotlin
inline fun <reified T> TreeNode.findParentOfType():
T? {
    var p = parent
    while (p != null && p !is T) {
        p = p.parent
    }
    return p as T?
}
```
我们使用 `reified` 修饰符来限定这个类型参数，这样函数内部就可以直接访问了，就好像一个普通的类。因为函数是内联的，所以无需反射，也可以使用 `!is` 和 `as` 这类普通操作符。除此之外，可以像上面提到的一样发起调用：`myTree.findParentOfType<MyTreeNodeType>()`。

即使多数场景不需要反射，但是我们依然可以利用具体化的类型参数来使用它：

```kotlin
inline fun <reified T> memberof() = T::class.members

fun main(s: Array<String>) {
    println(membersOf<StringBuilder>().joinToString("\n")
}
```

普通函数（没有标记为内联）不拥有具体化的参数。一个参数如果没有运行时的表现形式（例如，一个非具体化的类型参数，或者是像 `Nothing` 这种虚拟类型），不能用作具体化类型参数的实参。

更底层的描述，可查看：https://github.com/JetBrains/kotlin/blob/master/spec-docs/reified-type-parameters.md。

## 内联属性（1.1 开始支持）

`inline` 修饰符可用于没有备份字段的属性访问器上。可以标注单独的属性访问器：

```kotlin
val foo: Foo
    inline get() = Foo()

var bar: Bar
    get() = ...
    inline set(v) { ... }
```

也可以标注整个属性，这样使得两个访问器都成为内联：

```kotlin
inline var bar: Bar
    get() = ...
    set(v) { ... }
```

在调用处，内联访问器的内联方式与常规内联函数一致。

## 公共 API 内联函数的限制
当一个内联函数是 `public` 或者 `protected` 并且不是 `private` 或者 `internal` 声明的一部分时，它会被当做是模块的公共 API。它可以在其他模块中调用，并且同时会内联到调用处。

这样会带来某些二进制兼容性的风险，这类风险是由声明了内联函数的模块改动所引发的，因为改动之后，发起调用的模块可能会编译不过。

为了避免这种由于模块非公开 API 的改动所带来的不兼容性的风险，公共 API 内联函数不允许使用非公开 API 的声明，即 `private` 和 `internal` 的声明和位于其中的某一部分。

`internal` 声明可以用 `@PublishedApi` 标记，这样会允许我们在公共 API 内联函数中使用。如果 `internal` 内联函数被标记为 `@PublishApi`，那么函数体也会被检查，就好像它是一个公共函数。


