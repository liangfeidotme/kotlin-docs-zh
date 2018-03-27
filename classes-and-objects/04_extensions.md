# 扩展
Kotlin 可以无需依靠继承或某种设计模式（例如装饰器）来扩展一个类的功能，这点类似 C# 和 Gosu。这个能力通过一个叫 `extensions` 的特殊声明来实现。 Kotlin 支持扩展函数（extension functions）和扩展属性（extension properties）。

## 扩展函数
扩展函数的名字需要把接收者类型（被扩展的类型）作为前缀。下面的代码展示了如何给 `MutableList<Int>` 增加一个 `swap` 函数：

```kotlin
fun MutableList<Int>.swap(index1: Int, index2: Int) {
    val tmp = this[index1]  // 'this' corresponds to the list
    this[index1] = this[index2]
    this[index2] = tmp
}
```

扩展函数的 `this` 关键字指向接收者对象（位于点号之前）。这样就可以调用任何 `MutableList<Int>` 的这个函数。

```kotlin
val l = mutableListOf(1, 2, 3)
l.swap(0, 2) // 'this' inside 'swap()' will hold the value of 'l'
```

当然，这个函数对于任何 `MutableList<T>` 也是有意义的，所以可以让其更通用：

```kotlin
fun <T> MutableList<T>.swap(index1: Int, index2: Int) {
    val tmp = this[index1] // 'this' corresponds to the list
    this[index1] = this[index2]
    this[index2] = tmp
}
```

泛型参数的声明位于函数名之前，这样做的目的是为了让它在接收者类型表达式中可用。具体可见泛型函数。

## 扩展会被静态解析
扩展没有实际修改他们所扩展的类。定义扩展并没有给原有的类插入新的成员，只是通过点操作使得新的函数在这种类型的变量上变得可被调用。

需要强调的是，扩展函数的分发方式是静态的，也就是说，他们不是接收器类型的虚函数。这就意味着，被调用的扩展函数是由调用这个函数的表达式类型所决定，而不是由运行时得到的表达式的结果类型所决定。例如：

```kotlin
open class C

class D: C()

fun C.foo() = "c"

fun D.foo() = "d"

fun printFoo(c: C) {
    println(c.foo())
}

printFoo(D())
```

这个例子会打印出“c”，因为被调用的扩展函数只依赖于参数`c`所声明的类型，即 `C` 这个类。

如果成员函数的类和扩展函数的接收者类型一样，并且他们的名字和参数也都一样，那么优先级高的是成员函数。例如：

```kotlin
class C {
    fun foo() { println("memeber") }
}

fun C.foo() { println("extension") }
```

如果调用 `c.foo()`（任意 `C` 类型的变量 `c`），会打印出 “member”，而不是 “extension”。

但是，利用扩展函数去重载成员函数是完全OK的（函数名相同，但是函数签名不同）。

```kotlin
class C {
    fun foo() { println("member") }
}

fun C.foo(i: Int) { println("extension") }
```

调用 `C().foo(1)` 会打印出 “extension”。


## 可空的接收者
注意扩展定义在一个可空的接收者类型上。这种扩展可在对象变量上被调用，即使它的值为空，并且在函数体内可以检查 `this == null`。这也是为什么不做判空也可以调用 `toString()` 方法，因为检查已经在扩展函数中做掉了。

```kotlin
fun Any?.toString(): String {
    if (this == null) return "null"
    // after the null check, 'this' is autocast to a non-null type, 
    // so the toString() below resolves to the member function of 
    // the Any class
    return toString()
}
```

## 扩展属性
与函数类似，Kotlin 支持扩展属性：

```kotlin
val <T> List<T>.lastIndex: Int
    get() = size - 1
```

注意，因为扩展并没有在类中插入成员，所有并没有一个高效的方式让一个扩展属性拥有幕后字段。这也是为什么扩展属性没有初始化器。它们的行为只能通过显示地提供 getter/setter 来定义。

例子：

```kotlin
val Foo.bar = 1 // error: initializers are not allowed for extension properties
```

## 伴生对象扩展
如果一个类有伴生对象，那么也可以给这个伴生对象定义扩展函数和扩展属性。

```kotlin
class MyClass {
    companion object { } // will be called "Companion"
}

fun MyClass.Companion.foo() {
    // ...
}
```

就像伴生对象的常规成员，它们也可以利用类名作为限定符来调用。

```kotlin
MyClass.foo()
```

## 扩展的域

很多时候，我们在最顶层定义扩展，也就是说，直接位于包的定义之下：

```kotlin
package foo.bar

fun Baz.goo() { ... }
```

如果要在扩展所在的包之外使用它，我们首先需要在调用侧导入：

```kotlin
package com.example.usage

import foo.bar.goo  // importing all extensions by name "goo"

                    // or
import foo.bar.*    // importing everthing from "foo.bar"

fun usage(baz: Baz) {
    baz.goo()
}
```

## 把扩展声明为成员
在一个类内部可以声明其他类的扩展。这样的扩展内部会有多个隐式接收器（implicit receivers）—— 无需限定符就可以访问它们的对象成员。扩展声明所在的类的实例称为分发接收者（dispatch receiver），扩展方法接收器类型的实例称为扩展接收者（extension receiver）。

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

当分发接收者（dispatch receiver）和扩展接收者（extension receiver）的成员名字有冲突时，扩展接收着（extension receiver）的优先级更高。可以利用**限定 this 语法**来引用分发接收者的成员。

```kotlin
class C {
    fun D.foo() {
        toString()          // calls D.toString()
        this@C.toString()   // calls C.toString()
    }
}
```

声明为成员的扩展可以 是 `open`，子类可以覆写。这就意味着，这种扩展函数对应于分发接收者（dispatch receiver）是虚函数，但是对于扩展接收者来说，它是静态的。

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
C().caller(D1())    // prints "D.foo in C" - extension receiver is resolved statically
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

但是我们并不想在 `List` 类中实现所有可能的方法，不是吗？这就是扩展能够发挥作用的地方。
