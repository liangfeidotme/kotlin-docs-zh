# 协程

> Kotlin 1.1+ 的协程处在实验阶段

一些 API 会发起耗时较长的操作（例如网络 IO、文件IO，CPU 或 GPU 密集型任务等），这些操作会阻塞调用者，直到他们执行结束。协程可以避免阻塞线程，以一种成本更低、更可控的方式：协程的挂起。

协程把这个难题封装到了库里，简化了异步编程。程序逻辑在协程里可以表现为**顺序式**，底层库会帮我们处理好异步性。库可以把用户代码的相关部分封装成回调，订阅相关事件，把执行规划到不同线程上（甚至是不同机器上！），代码简单到如同顺序执行。

很多其他语言的异步机制可以基于协程实现成库。包括 C# 和 ECMAScript 的 `async/await`，Go 的 `channels` 和 `select`，C# 和 Python 的 `generators/yield`。下面的内容描述了如何构造这些库。

## 阻塞和挂起
从根本上来说，协程是不阻塞线程、能够被挂起的计算。阻塞线程的代价会比较高，特别是在高负荷的情况下，因为保持相对较小数量的线程会比较实用，所以如果阻塞了其中之一会导致一些重要任务的延迟。

另一方面来说，协程的挂起几乎是“免费的”。没有上下文切换，也不需要任何 OS 的参与。在此之上，挂起可以通过自定义库来控制，带来了更大的扩展：作为库作者，我们可以决定挂起会触发什么，并且可以根据需求来优化/记录/拦截它。

还有一个不同，协程并不能在随机的指令处挂起，而是只发生在所谓的*挂起点*处，也就对特定标记过的函数发起的调用。

## 挂起函数
挂起发生在调用 `suspend` 修饰符标记过的函数时：

```kotlin
suspend fun doSomething(foo: Foo): Bar {
    ...
}
```

这些函数称为**挂起函数**，因为对他们的调用会挂起一个协程（如果调用结果已经准备就绪，库本身可以决定继续执行，无需挂起）。挂起函数可以像普通函数一样接收参数、返回结果，但是他们只能从协程或者其他挂起函数中发起调用，内联的函数字面量也是如此。

实际上，如果启动一个协程，至少要有一个挂起函数，并且通常是一个挂起的 lambda。我们来看一个例子，一个简化的 `async()` 函数（来自 `kotlin.coroutines` 库）：

```kotlin
fun <T> async(block: suspend () -> T)
```

这里的 `async()` 是一个常规函数（不是挂起函数），但是 `block` 参数用 `suspend` 修饰的函数类型：`suspend() -> T`。所以，当我们给 `async()` 传入 lambda 时，它就变成了一个*挂起的 lambda*，我们可以从中调用挂起函数：

```kotlin
async {
    doSomething(foo)
    ...
}
```

> 注意：目前，挂起函数的类型不能用作超类型，而且匿名挂起函数暂不支持。

以此类推，`await()` 也是一个挂起函数（因为也能从 `async {}` 区块中发起调用），它会挂起一个协程，直到完成计算、返回结果：

```kotlin
async {
    ...
    val result = computation.await()
    ...
}
```

需要注意的是，没有内联到挂起函数中的函数字面量和类似 `main()` 的常规函数都无法调用挂起函数 `await()` 和 `doSomething()`：

```kotlin
fun main(args: ArrayList<String>) {
    doSomething() // ERROR: Suspending function called from a non-coroutine context

    async {
        ...
        computations.forEach {  // `forEach` is an inline function, the lambda is inlined
            it.await() // OK
        }

        thread {  // `thread` is not an inline function, so the lambda is not inlined
            doSomething()   // ERROR
        }
    }
}
```

还有一点需要注意，挂起函数可以是虚函数，覆写时需要指定 `suspend` 修饰符：

```kotlin
interface Base {
    suspend fun foo()
}

class Derived: Base {
    override suspend fun foo() { ... }
}
```

## `@RestrictsSuspension` 注解
扩展函数（以及 lambda）也可以使用 `suspend`，跟常规函数一样。这样就允许创建 DSL 以及其他可扩展的 API。一些场景下，库作者需要阻止用户为挂起协程添加*新的方式*。

为了实现这一点，可以使用 `@RestrictsSuspension` 注解。当接收器的类或者接口 `R` 用了这个注解，那么所有挂起的扩展都需要代理到 `R` 的成员或者扩展上去。因为扩展之间无法无限互相代理（程序不会终结），这样能够保证所有的协程都会经由 `R` 的成员，而库作者对成员具有完全的控制权。

这种应用场景很*少见*，例如每一个挂起在库中都需要以一种特殊的方式来处理。例如，当为下面的 `buildSequence()` 函数实现生成器时，我们需要保证，协程中的任何挂起调用都要以 `yeild()` 或者 `yieldAll()` 而不是其他函数来结尾。这就是为什么 `SequenceBuilder` 要用 `@RestrictsSuspention` 来标注：

```kotlin
@RestrictsSuspension
public abstract class SequenceBuilder<in T> {
    ...
}
```

Github 源码：https://github.com/JetBrains/kotlin/blob/master/libraries/stdlib/src/kotlin/coroutines/experimental/SequenceBuilder.kt

## 协程的内部工作机制
这里不会给出关于协程底层如何工作的完整解释，但是一个粗略的认识会比较重要。

协程的实现完全依靠编译器技术（无需 VM 或 OS 的支持），而且挂起通过代码转换实现。基本上，每个挂起函数（可能有优化，这里不赘述）都会转换成一个状态机，它的状态对应着挂起的调用。正好的挂起之前，下一个状态连带相关的局部变量等会存储在由编译器生成的类的某个字段中。在协程恢复时，局部变量会复原，状态机会从正好位于挂起之后的那个状态处继续往下处理。

一个挂起的协程可以作为一个对象被存储以及来回传递，这个对象能够保持挂起的状态和局部环境。这类对象的类型是 `Continuation`，并且这里所描述的所有代码转换都对应着经典的 [CPS(Continuation-Passing Style)](https://en.wikipedia.org/wiki/Continuation-passing_style)。因此，挂起函数的底层其实携带了一个类型是 `Continuation` 的额外参数。

更多关于协程的详细内容可以在在[设计文档](https://github.com/Kotlin/kotlin-coroutines/blob/master/kotlin-coroutines-informal.md)中找到。有关其他语言（例如 C# 或者 ECMAScript 2016）中 async/await 的描述在这里也是相关的，即使它们实现的语言特性不一定像 Kotlin 的协程那样通用。

## 协程的实验性状态
协程的设计是实验性的，未来可能会有变化。Kotlin 1.1+ 的协程在编译时默认会报警告：*“协程”这个特性是实验性的*。如果要移除这个警告，需要指定一个 opt-in 的标志。

基于它实验性质的状态，标准库中协程相关的 API 都放在 `kotlin.coroutines.experimental` 包下面。等到设计完成并且实验状态升级之后，最终会移到 `kotlin.coroutines` 包下，实验性的包也会保留着（可能在其他的产物中），以兼容老版本。

**注意（重要）**：我们建议库作者遵守同样的约定：给暴露协程 API 的包名加上 “experimental” 后缀（例如 `com.example.experimental`），这样能够保证库的二进制兼容性。最终 API 发布以后，按照以下步骤修改：

- 把所有 API 拷贝到 `com.example` 包下（去掉了 `experimental` 后缀），
- 保留实验性的包，做好向下兼容。

这样可以最小化用户迁移的成本。

## 标准 API
协程有三个主要组成部分：

- 语言层面支持，上述的挂起函数是其一；
- 低级核心 API，由标准库中提供；
- 高级 API，可直接用于用户代码中。

### 底层 API：kotlin.coroutines
低级 API 的数量相对较小，除了创建高级的库，不应用作他用。由两个主要的库组成：

- `kotlin.coroutines.experimental`，包含主要类型和基础类型，例如：
    - `createCoroutine()`
    - `startCoroutnie()`
    - `suspendCoroutine()`
- `kotlin.coroutines.experimental.intrinsics` ，包含像 `suspendCoroutineOrReturn` 的低级别固有内容。

### kotlin.coroutines 中的生成器 API

`Kotlin.coroutines.experimental` 中唯一的“应用级”函数有：
- `buildSequence()`
- `buildIterator()`

这些函数由 `kotlin-stdlib` 提供，因为他们与序列有关。实际上，这些函数（可以只看 `buildSequence()`）实现了迭代器，也就是提供了一个低成本创建懒队列的方式：

```kotlin
val fibonacciSeq = buildSequence {
    var a = 0
    var b = 1

    yield(1)

    while (true) {
        yield(a + b)

        val tmp = a + b
        a = b
        b = tmp
    }
}
```

以上代码通过协程创建了一个可懒加载的潜在无限斐波那契数列，而协程则是通过调用 `yield()` 函数生产出连续的菲波那切数字。当迭代这么一个序列时，迭代器的每一步会执行协程的另一部分，产生下一个数字。所以，我们可以从这个队列中取出任意有限数量的数字，例如 `fibonacciSeq.take(8).toList()` 会产生 `[1, 1, 2, 3, 5, 8, 13, 21]`。协程以足够低成本的方式使其变得可操作。

为了展示这种队列真正的懒加载性，我们可以在 `buildSequence(0` 的调用中打印调试信息：

```kotlin
val lazySeq = buildSequence {
    print("START")
    for (i in 1..5) {
        yield(i)
        print("STEP ")
    }
    print("END")
}
```

以上代码执行后会打印出前三个元素。这些数字在循环中以 `STEP` 为间隔，这就证明了计算是懒执行的。如果要打印 `1`，我们只需要执行到第一个 `yield(i)`，并在路径中打印出 `START`。然后要打印 `2` 的话，我们需要继续执行下一个 `yield(i)`，然后打印出 `STEP`。`3` 也是一样。下一个 `STEP` 绝对不会打印出来（`END` 也不会），因为我们没有请求序列中更多的元素。

如果要一次性产生所有元素的集合（或者序列），使用 `yieldAll()` 函数：

```kotlin
val lazySeq = buildSequence {
    yield(0)
    yieldAll(1..10)
}

lazySeq.forEach { print("$it") }
```

`buildIterator()` 的工作方式类似于 `buildSequence()`，只不过返回值是一个懒迭代器。

可以通过为 `SequenceBuilder` 添加挂起扩展的方式来自定义产生（yielding）逻辑（支撑了上述的 `@RestrictsSuspension` 注解）。

```kotlin
suspend fun SequenceBuilder<Int>.yieldIfOdd(x: Int) {
    if (x % 2 != 0) yield(x)
}

val lazySeq = buildSequence {
    for (i in 1..10) yieldIfOdd(i)
}
```

### 其他高级 API：kotlinx.coroutines
Kotlin 标准库中只有协程相关的核心 API。主要包括核心基础类型和接口，所有基于协程的库都可能会用到。

大部分基于协程的应用级 API 会以单独的库发布：`kotlinx.coroutine`。这个库包含：

- 平台无关的异步编程，位于 `kotlinx-coroutines-core`：
    - 这个模块包含类Go的通道，支持 `select` 和其他遍历的基础类型，
    - 这个库的源码位于 Github —— kotlinx.coroutines
- 基于 `CompletableFuture` （JDK8）的 API：`kotlinx-coroutines-jdk8`；
- 基于 NIO（非阻塞 IO，JDK7 及以上）的 API：`kotlinx-coroutines-nio`；
- 支持 Swing（`kotlinx-coroutines-swing`）和 JavaFx（`kotlinx-coroutines-javafx`）；
- 支持 RxJava：`kotlinx-coroutines-rx`。

这些库既提供了便利的 API，使得普通任务变得简单，而且提供了端到端的例子，展示了如何构建一个基于携程的库。
