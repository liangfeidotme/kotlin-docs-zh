扩展
===

Kotlin 可以无需依靠继承或设计模式来扩展一个类的功能，类似 C# 和 Gosu。这个能力通过一个叫 `extensions` 的特殊声明来实现。 Kotlin 支持扩展函数（extension functions）和扩展属性（extension properties）。

扩展函数
---
扩展函数的名字需要把_receiver type_（例如，被扩展的类型）作为前缀。下面的代码展示了如何给 `MutableList<Int>` 增加一个 `swap` 函数：

```kotlin
fun MutableList<Int>.swap(index1: Int, index2: Int) {
    val tmp = this[index1]  // 'this' corresponds to the list
    this[index1] = this[index2]
    this[index2] = tmp
}
```

扩展函数的 `this` 关键字指向 receiver object（dot 前面）。调用方式如下：

```kotlin
val l = mutableListOf(1, 2, 3)
l.swap(0, 2) // 'this' inside 'swap()' will hold the value of 'l'
```

也可以作用于泛型：

```kotlin
fun <T> MutableList<T>.swap(index1: Int, index2: Int) {
    val tmp = this[index1] // 'this' corresponds to the list
    this[index1] = this[index2]
    this[index2] = tmp
}
```

把扩展声明为成员
---
在一个类内部可以声明其他类的扩展。这样的扩展内部会有多个隐式的接收器（implicit receivers）—— 无需修饰符就可以访问对象的成员。扩展声明所在类的实例称为分发接收器（dispatch receiver），扩展方法接收器类型的实例成为扩展接收器（extension receiver）。

```kotlin
class D {
    fun bar() { /* ... */ }
}

class C {
    fun baz() { /* ... */ }

    fun D.foo() {
        bar()   // calls D.bar
        baz()   // calls C.baz
    }

    fun caller(d: D) {
        d.foo() // call the extension function
    }
}
```

当分发接收器（dispatch receiver）和扩展接收器（extension receiver）的成员名字有冲突时，扩展接收器（extension receiver）获胜。可以使用修饰过的 this 来访问分发接收器的成员。

```kotlin
class C {
    fun D.foo() {
        toString()          // calls D.toString()
        this@C.toString()   // calls C.toString()
    }
}
```

声明为成员的扩展可以声明为 `open`，这样子类可以重写。这种扩展函数的调用在分发接收器（dispatch receiver）上是多态的（虚函数），但是扩展接收器的类型是固定的。

```kotlin
open class D {
}

class D1 : D() {
}

open class C {
    open fun D.foo() {
        println("D.foo in C")
    }

    open fun D1.foo() {
        println("D1.foo in C")
    }

    fun caller(d: D) {
        d.foo()     // call the extension function
    }
}

class C1 : C() {
    override fun D.foo() {
        println("D.foo in C1")
    }

    override fun D1.foo() {
        println("D1.foo in C1")
    }
}

C().caller(D())     // prints "D.foo in C"
C1().caller(D())    // prints "D.foo in C1" - dispatch receiver is resolved virtually
C().caller(D1())    // prints "D.foo in C" - extension receiver is resolved virtually
```

动力
---
Java 的世界中，我们习惯了以 “*Utils” 来命名工具类：`FileUtils`、`StringUtils` 等等。`java.util.Collections` 是一个典型，代码很容易变成下面的样子，让人不爽：

```java
// Java
Collections.swap(list, Collections.binarySearch(list,
Collections.max(otherList)), Collections.max(list));
```

可以使用 static import 来简化一下：

```java
// Java
swap(list, binarySearch(list, max(otherList)), max(list));
```

稍微好一些，但是我们几乎无法使用 IDE 强大的代码自动补全功能。如果改成如下形式可能会更好：

```java
// Java
list.swap(list.binarySearch(otherList.max()), list.max());
```

但是我们并不想实现一个 `List` 类（需要实现它所有的方法）。这就是扩展能够发挥作用的地方。
