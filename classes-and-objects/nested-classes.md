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

关于内部类中 `this` 在歧义可参考 [Qualified this expressions](this-expression.md)。

## 匿名内部类

匿名内部类的实例可通过对象声明来创建：

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

如果对象是一个函数式 Java 接口的实例（也就是，一个带有单个抽象函数的 Java 接口），创建预发可以简化为给 lambda 表达式前缀一个接口的类型：

```kotlin
val listener = ActionListener { println("clicked") }
```

