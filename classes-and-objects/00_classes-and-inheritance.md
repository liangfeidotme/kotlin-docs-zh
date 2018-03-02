# 类和继承
## 类
Kotlin 的类使用关键字 `class` 来定义：

```kotlin
class Invoice {
}
```

类的声明包含：类名、类头（指明类型参数、首要构造器等）、类体（由花括号包裹）。类头和类体都是可选的；如果没有类体，花括号也可以省略。

```kotlin
class Empty
```

### 构造器
Kotlin 的一个类可以有一个**首要构造器**和多个**次要构造器**。首要构造器是类头的一部分：位于类名（以及可选类型参数）之后。

```kotlin
class Person constructor(firstName: String) {
}
```

如果首要构造器没有任何标注或者可视化修饰符，`constructor` 关键字可以省略。

```kotlin
class Person(firstName: String) {
}
```
首要构造器不能包含任何代码。初始化代码可位于用 `init` 关键字作为前缀的**初始化区块（initializer block）**中。

一个实例在初始化的过程中，初始化区块会按照他们声明的顺序执行，中间可以穿插属性的初始化。

```kotlin
class InitOrderDemo(name: String) {
    val firstProperty = "First property: $name".also(::println)

    init {
        println("First initializer block that prints ${name}")
    }

    val secondProperty = "Second property: ${name.length}".also(::println)

    init {
        println("Second initializer block that prints ${name.length}")
    }
}
```

注意：首要构造器的参数可以直接用于初始化区块。当然也可以用于属性初始化。

```kotlin
class Customer(name: String) {
    val customer = name.toUpperCase();
}
```

实际上，有简洁的语法支持在首要构造器中声明属性并对他们进行初始化：

```kotlin
class Person(val firstName: String, val lastName: String, val age: Int) {
    // ...
}
```

和其他常规的属性一样，首要构造器中的属性可以被声明为可变（`var`）或者只读（`val`）。

如果构造器中有注解或者可见性修饰符，必须要有 `constructor` 关键字，修饰符位于它前面：

```kotlin
class Customer public @Inject constructor(name: String) { ... }
```

**次要构造器**

一个类也可以用 `constructor` 作为前缀来声明次要构造器（secondary constructor）：

```kotlin
class Person {
    constructor(parent: Person) {
        parent.children.add(this);
    }
}
```

如果一个类有首要构造器，那么每个次要构造器都要代理到首要构造器，可以直接或者通过其他次要构造器间接实现。使用 `this` 关键字可以把调用代理到同一个类的其他构造器：

```kotlin
class Person(val name: String) {
    constructor(name: String, parent: Person) : this(name) {
        parent.children.add(this)
    }
}
```

注意：初始化区块里的代码会成为首要构造器的一部分。次要构造器的第一条语句会代理到首要构造器，因此初始化区块中的代码会优先执行。即使没有首要构造器，代理也会静默发生，所以初始化区块仍然会被执行到。

```kotlin
class Constructors {
    init {
        println("Init block")
    }

    constructor(i: Int) {
        println("Constructor")
    }
}
```

如果一个非抽象的类没有声明任何构造器（首要或次要），那么它会自动生成一个 public 的无参首要构造器。如果不想要 public 构造器，可以声明一个非默认可见性并且空的首要构造器。

```kotlin
class DontCreateMe private constructor () {
}
```

> 注意：在 JVM 上，如果首要构造器的所有参数都有默认值，编译器额外会生成一个使用默认值的无参构造器。这样可以方便 Jackson 或者 JPA 这样的库通过无参构造器来创建类的实例。
```kotlin
class Customer(val customerName: String = "")
```

### 创建类的实例
创建类实例时，可以像调用普通函数那样调用构造器：

```kotlin
val invoice = Invoice()
val customer = Customer("Joe Smith")
```

注意，Kotlin 没有 `new` 关键字。

创建嵌套、内部和匿名类的实例在“嵌套类”那一章节。

### 类的成员
一个类可以包含：

* 构造器和初始化区块
* 函数
* 属性
* 嵌套和内部类
* 对象声明

## 继承
Kotlin 中所有的类都有一个共同的父类 - `Any`，对于没有声明父类，那么`Any` 就是默认父类。

```kotlin
class Example // Implicitly inherits from Any
```

> 注意： `Any` 不是 `java.lang.Object`，特别的是，它除了 `equals()`、`hashCode()` 和 `toString()` 之外，没有任何其他成员。

如果要显示声明一个父类，需要把父类的类型放置在冒号之后：

```kotlin
open class Base(p: Int)

class Derived(p: Int) : Base(p)
```

> 这里的 `open` 注解与 Java 的 `final` 正好相反：它允许这个类被继承。默认情况下，Kotlin 中所有的类都是 final，正好符合 Effective Java 的第 17 条：要么为继承而设计，并提供文档说明，要么就禁止继承。

如果继承的类有首要构造器，那么它的基类可以（并且必须）使用首要构造器的参数在它出现的地方进行初始化。

如果这个类没有首要构造器，那么每一个次要构造器必须使用 `super` 关键字来初始化基类，或者调用另一个实现了这个要求的构造器。注意，在这种情况下，不同的次要构造器可以调用不同的基类构造器。

```Kotlin
class MyView : View {
    constructor(ctx: Context) : super(ctx)

    constructor(ctx: Context, attrs: AttributeSet) : super(ctx, attrs)
}
```

### 方法覆写
就像前面提到的，Kotlin 倾向于让事物变得明确。与 Java 不同的是，对于可覆写的成员（可被称之为 open）和覆写本身，Kotlin 需要用注解明确地标记：

```kotlin
open class Base {
    open fun v() {}
    fun nv() {}
}

class Derived() : Base() {
    override fun v() {}
}
```

`override` 必须标记在 `Derived.v()` 上，否则会导致编译出错。如果函数没有 `open` 注解，像 `Base.nv()`，在子类中声明一个同样签名的方法是非法的，不管有没有用 `override`。final 类（没有 `open` 注解的类）不允许有 open 成员。

用 `override` 标注的成员自身是 open，也就是说，它可以在子类中被覆写。如果想禁止再被重载，可以使用 `final`：

```kotlin
open class AnotherDerived() : Base() {
    final override fun v() {}
}
```

### 属性覆写
属性覆写的工作方式和方法覆写一样，在父类中声明的属性，如果想要在子类中重新声明，必须使用 `override` 标注，并且类型必须兼容。每个声明的属性都可以被覆写为自带初始化或者带有 getter 方法的属性。

```kotlin
open class Foo {
    open val x: Int get() { ... }
}

class Bar1 : Foo() {
    override val x: Int = ...
}
```

也可以用 `var` 属性来覆写 `val` 属性，但是反过来不可以。原因是：`val` 属性本质上声明了一个 `getter` 方法，用 `var` 覆写就等价于在继承的类中额外定义了一个 setter 方法。

注意，`override` 可以用在首要构造器的属性声明中。

```kotlin
interface Foo {
    val count: Int
}

class Bar1(override val count: Int) : Foo

class Bar2 : Foo {
    override var count: Int = 0
}
```

### 继承类的初始化顺序
在构造一个继承类的实例时，完成基类的初始化是第一步（只在基类构造器参数的评估之后），因此要早于继承类执行它的初始化逻辑。

```kotlin
open class Base(val name: String) {
    init { println("Initializing Base") }

    open val size: Int = 
    name.length.also { println("Initializing size in Base: $it") }
}

class Derived(
    name: String,
    val lastName: String
) : Base(name.capitalize().also { println("Argument for Base: $it") }) {
    init { println("Initializing Derived") }

    override val size: Int = 
    (super.size + lstName.length).also { println("Initializing size in Derived: $it") }
}
```

这就意味着，基类构造器在执行时，继承类所声明或覆写的属性还没有被初始化。如果这些属性被用到了基类的初始化逻辑中（无论是直接还是间接地通过另一个覆写的 `open` 成员），可能会导致不正确的行为或者运行时失败。所以设计基类时，在构造器、属性初始化以及 `init` 块中要避免使用 `open` 成员。

### 调用父类实现
使用 `super` 关键字，子类中的代码可以调用父类的函数和属性访问器。

```kotlin
open class Foo {
    open fun f() { println("Foo.f()") }
    open val x: Int get() = 1
}

class Bar : Foo() {
    override fun f() {
        super.f();
        println("Bar.f()")
    }

    override val x: Int get() = super.x + 1
}
```

内部类如果要访问外部类的父类，访问方式是 `super` 再加上外部类类名：

```kotlin
class Bar : Foo() {
    override fun f() { /* ... */ }
    override val x: Int get() = 0

    inner class Baz {
        fun g() {
            super@Bar.f()  // Calls Foo's implementation of f()
            println(super@Bar.x)
        }
    }
}
```

### 覆写规则
在 Kotlin 中，实现继承要遵守如下规则：如果一个类间接地从多个父类中继承了同一个成员的多个实现，那么它必须覆写这个成员并且提供自己的实现（也可以直接使用继承中的某一个）。我们可以采用用 `super` 加上尖括号包裹父类名字的方式来标记继承实现是来自哪个父类，例如，`super<Base>`。

```kotlin
open class A {
    open fun f() { print("A") }
    fun a() { print("a") }
}

interface B {
    fun f() { print("B") } // interface members are 'open' by default
    fun b() { print("b") }
}

class C() : A(), B {
    // the compile requires f() to be overridden:
    override fun f() {
        super<A>.f() // call to A.f()
        super<B>.f() // call to B.f()
    }
}
```

可以同时继承 `A` 和 `B` ，`a()` 和 `b()` 也没有问题，因为 `C` 只继承了这些方法当中的一个实现。但是 `C` 继承了两个 `f()` 的实现，因此我们必须要在 `C` 中重写 `f()` 来提供我们自己的实现，这样才能排除二义性。

## 抽象类
类和它的某些成员可以声明为 `abstract`。类的抽象成员没有实现。我们没必要用 open 来标注一个抽象的类或函数，因此这是显而易见（It goes without saying）。

我们把非抽象的 open 成员覆写为抽象的：

```kotlin
open class Base {
    open fun f() {}
}

abstract class Derived : Base() {
    override abstract fun f()
}
```

## 联合对象
与 Java 或者 C# 不同的是，Kotlin 的类没有静态方法。大部分情况下，推荐使用包级别的函数来替代。

如果我们需要写一个函数，这个函数可以不通过类的实例来调用，但是它需要访问到类本身的内部（例如，一个工厂方法），那么在那个类中将其写成一个对象声明类型的成员。

更具体一点，如果在类中声明了一个联合对象，就可以使用像 Java/C# 静态方法那样的语法来调用它的成员——只使用类名作为限定符。
