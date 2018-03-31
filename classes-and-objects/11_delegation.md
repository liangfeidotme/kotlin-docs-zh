# 代理
## 属性代理
有一个专门的章节来讲述[代理属性](12_delegated-properties.md)。

## 用代理实现
[代理模式](https://en.wikipedia.org/wiki/Delegation_pattern)已经被证明是一个继承实现（implementation inheritance）的好方法，Kotlin 无需其他代码，原生支持。`Derived` 类可以继承 `Base` 接口，然后把它的 public 方法都代理到一个指定对象上：

```kotlin
interface Base {
    fun print()
}

class BaseImpl(val x: Int) : Base {
    override fun print() { print(x) }
}

class Derived(b: Base) : Base by b

fun main(args: Array<String>) {
    val b = BaseImpl(10)
    Deribed(b).print() // prints 10
}
```

`Derived` 父类列表中的 `by` 子句表示 `b` 会被存储在所有的 `Derived` 对象的内部，编译器会生成 `Base` 的所有方法，这些方法会跳向 `b`。

### 覆写用代理实现的接口成员
覆写的工作方式与我们预期的一致：编译器会使用我们 `override` 的实现而不是代理对象中的实现。如果我们给 `Derived` 增加覆写的函数 `override fun print() { print("abc") }`，程序执行后会打印出 “abc” 而不是 “10”。

```kotlin
interface Base {
    fun printMessage()
    fun printMessageLine()
}

class BaseImpl(val x: Int) : Base {
    override fun printMessage() { print(x) }
    override fun printMessageLine() { println(x) }
}

class Derived(b: Base) : Base by b {
    override fun printMessage() { print("abc") }
}

fun main(args: Array<String>) {
    val b = BaseImpl()
    Derived(b).printMessage()
    Derived(b).printMessageLine()
}
```

但是，使用这种方式覆写的的成员不会被代理对象的成员调用，因为它们只能访问自己实现的接口成员：

```kotlin
interface Base {
    val message: String
    fun print()
}

class BaseImpl(val x: Int) : Base {
    override val message = "BaseImpl: x = $x"
    override fun print() { println(message) }
}

class Derived(b: Base) : Base by b {
    // Thie property is not accessed from b's implementation of `print`
    override val message = "Message of Derived"
}

fun main(args: Array<String>) {
    val b = BaseImpl(10)
    val derived = Derived(b)
    derived.print()
    println(derived.message)
}
```

---

https://kotlinlang.org/docs/reference/delegation.html
