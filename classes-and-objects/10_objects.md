# 对象表达式和对象声明
有时我们创建对象的时候，需要对一个类做出微小的修改，没有必要显示地声明一个新的子类。Java 使用匿名内部类来处理这种 case。Kotlin 稍微概括了这个概念，称作 *对象表达式*和*对象声明*。

## 对象表达式
我们可以用如下代码来创建一个匿名类的对象，这个类继承了某个类型（或多个类型）：

```kotlin
window.addMouseListener(object : MouseAdapter() {
    override fun mouseClicked(e: MouseEvent) {
        // ...
    }

    override fun mouseEntered(e: MouseEvent) {
        // ...
    }
})
```

如果父类有构造函数，恰当的构造参数也要传递给它。如果有多个父类，可以在冒号之后用逗号隔开：

```kotlin
open class A(x: Int) {
    public open val y: Int = x
}

interface B {...}

val ab: A = object : A(1), B {
    override val y = 15
}
```

如果万一我们需要的“只是一个对象”，没有无关紧要的父类，可以简写：

```kotlin
fun foo() {
    val adHoc = object {
        var x: Int = 0
        var y: Int = 0
    }

    print(adHoc.x + adHoc.y)
}
```

注意，匿名对象只能在局部和私有声明中用作类型。如果把匿名对象作为公共函数的返回值或者作为公共属性的类型，函数或者属性的实际类型会变成匿名对象的父类，如果没有父类的话是 `Any`。匿名对象中添加的成员是不可访问的。 

```kotlin
class C {
    // Private function, so the return type is the annoymouse object type
    private fun foo() = object {
        val x: String "x"
    }

    // Public function, so the return type is Any
    fun publicFoo() = object {
        val x: String = "X"
    }

    fun bar() {
        val x1 = foo().x        // works
        val x2 = publicFoo().x  // ERROR: Unresolved reference 'x'
    }
}
```

与 Java 的匿名内部类一样，对象表达式中的代码只能访问它的包围区域（enclosing scope）。（与 Java 不同，Kotlin 不会限定为 final 变量）。

```kotlin
fun coutClicks(window: JComponent) {
    var clickCount = 0
    var enterCount = 0

    window.addMouseListener(object : MouseAdapter() {
        override fun mouseClicked(e: MouseEvent) {
            clickCount++
        }

        override fun mouseEntered(e: MouseEvent) {
            enterCount++
        }
    })
    // ...
}
```

## 对象声明
单例模式是一个非常实用的设计模式，Kotlin 继 Scale 之后让声明一个单例变得很容易：

```kotlin
object DataProviderManager {
    fun registerDataProvider(provider: DataProvider) {
        // ...
    }

    val allDataProviders: Collection<DataProvider>
        get() = // ...
}
```

这种写法称为*对象声明*，它永远会有一个位于 `object` 关键字之后的名字。就像变量声明一样，对象声明不是一个表达式，所以不能用于赋值声明的右侧。

对象声明的初始化是类型安全的。

直接通过名字就可以使用这个对象：

```kotlin
DataProviderManager.registerDataProvider(...)
```

这种对象可以有父类：

```kotlin
object DefaultListener : MouseAdapter() {
    override fun mouseClicked(e: MouseEvent) {
        // ...
    }

    override fun mouseEntered(e: MouseEvent) {
        // ...
    }
}
```

**注意**：对象声明无法是局部的（也就是说，直接嵌套在函数内），但是他们可以嵌套在其他的对象声明或者非内部类中。

## 伴生对象
位于类内部的对象声明可以用 `companion` 关键字标记：

```kotlin
class MyClass {
    companion object Factory {
        fun create(): MyClass = MyClass()
    }
}
```

调用伴生对象的成员比较简单，把类名作为限定符就可以了：

```kotlin
val instance = MyClass.create()
```

伴生对象的名字可以省略，这种情况下可以使用名字 `Companion`：

```kotlin
class MyClass {
    companion object {
    }
}

val x = MyClass.Companion
```

注意，即使伴生对象的成员看起来很像其他语言的静态成员，但是，在运行时他们依然是真实对象的实例成员，并且有其他能力，例如，实现接口：

```kotlin
interface Factory<T> {
    fun create(): T
}

class MyClass {
    companion object : Factory<MyClass> {
        override fun create(): MyClass = MyClass()
    }
}
```

但是，在 JVM 上，你可以使用 `@JvmStatic` 注解来让伴生对象的成员生成为真正的静态方法和字段。具体可参考**Java互操作性**章节。

## 对象声明和对象表达式在语义上的不同
对象表达式和对象声明在语义上有一个重要的不同点：

* 对象表达式在使用处会**立即**执行（以及初始化）；
* 对象声明只有在第一次访问时才会**延迟**初始化；
* 伴生对象在相应的类被加载之后会初始化，与 Java 静态初始化器的语义相匹配。
