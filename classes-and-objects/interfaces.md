# 接口

Kotlin 的接口非常类似于 Java8。可以包含抽象方法的声明以及方法的实现。接口与抽象类的区别是不能存储状态。接口可以有属性，但是必须声明为抽象类型或者实现了访问器。

接口通过关键字 `interface` 来定义：

```kotlin
interface MyInterface {
    fun bar()
    fun foo() {
        // optional body
    }
}
```

## 实现接口

类或者对象可以实现一个或多个接口：

```kotlin
class Child : MyInterface {
    override fun bar() {
        // body
    }
}
```

## 类的属性

在接口中可以声明属性。属性可以是抽象的，或者提供了访问器的实现。属性没有备份字段，因此访问器不能引用他们。

```kotlin
interface MyInterface {
    val prop: Int // abstract

    val propertyWithImplementation: String
        get() = "foo"

    fun foo() {
        print(prop)
    }
}

class Child : MyInterface {
    override val prop: Int = 29
}
```

## 解决覆写冲突

如果父类的列表中有多个类型，我们可能会集成了同一个方法的多个实现。例如：

```kotlin
interface A {
    fun foo() { print("A") }
    fun bar()
}

interface B {
    fun foo() { print("B") }
    fun bar() { print("bar") }
}

class C : A {
    override fun bar() { print("bar") }
}

class D : A, B {
    override fun foo() {
        super<A>.foo()
        super<B>.foo()
    }

    override fun bar() {
        super<B>.bar()
    }
}
```

接口 A 和 B 都声明了函数 `foo()` 和 `bar()`，他们都实现了 `foo()`，但是只有 B 实现了 `bar()`（`bar()` 在 A 中没有用 abstract 标记，因为这是在接口中，如果函数没有函数默认就是抽象的）。如果我们现在要继承 A 来实现一个具体的类 C，很明显，必须重载 `bar()` 并且提供一个实现。

但是，如果我们从 A 和 B 继承一个 D，我们就需要实现从多个接口继承下来的方法，并且要明确地指明 D 应该如何实现它们。这两条同样适用于只继承了单个实现（`bar()`）以及继承了多个实现（`foo()`）的方法。
