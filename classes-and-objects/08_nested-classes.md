# 嵌套和内部类

类可以嵌套在其他类中：

```kotlin
class Outer {
    private val bar: Int = 1
    class Nested {
        fun foo() = 2
    }
}

val demo = Outer.Nested().foo() // == 2
```

## 内部类

一个类可以标记为 `inner`，这样它就可以访问外部类的成员。内部类持有外部类对象的引用：

```kotlin
class Outer {
    private val bar: Int = 1
    inner class Inner {
        fun foo() = bar
    }
}


val demo = Outer().Inner().foo() // == 1
```

关于如何消除 `this` 在内部类中的歧义可参考*限定的 this 表达式*。

## 匿名内部类

匿名内部类的实例可通过**对象表达式**来创建：

```kotlin
window.addMouseListener(object: MouseAdapter() {
    override fun mouseClicked(e: MouseEvent) {
        // ...
    }

    override fun mouseEntered(e: MouseEvent) {
        // ...
    }
})
```

如果这个对象是一个函数式的 Java 接口的实例（例如，一个带有单个抽象函数的 Java 接口），可以使用 lambda 表达式来创建，但是要把这个接口类型作为表达式的前缀：

```kotlin
val listener = ActionListener { println("clicked") }
```

