# 代理

## 类代理
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

`Derived` 父类列中的 `by` 子句表示 `b` 会被存储在 `Derived` 的所有对象中，编译器会生成 `Base` 的所有方法，这些方法会跳向 `b`。

注意，覆写的工作方式与我们预期的一致：编译器会使用我们 `override` 的实现而不是代理对象中的实现。如果我们给 `Derived` 增加覆写的函数 `override fun print() { print("abc") }`，程序执行后会打印出 “abc” 而不是 “10”。
