# 可见性修饰符

类、对象、接口、构造器、函数、属性以及它们的 setter 都有 *可见性修饰符*。（getter 的可见性与属性一致）。Kotlin 中有四个可见性修饰符：`private`、`protected`、`internal` 和 `public`。默认的可见性（如果没有显示指明修饰符）是 `public`。

下面会解释修饰符如何作用于不同类型的声明域（different types of declaring scopes）。

## 包
函数、属性，类、对象以及接口可以声明在“最顶层”，也就是说，直接在包内。

```kotlin
// file name: example.kt
package foo

fun baz() {}
class Bar {}
```

- 如果没有指明任何可见性修饰符，默认为 `public`，意味着你的定义在任何地方都是可见的。
- 如果用 `private` 来标记声明，那么只在声明所在的文件内可见。
- 如果标记为 `internal`，只要是同一个模块，都可见。
- `protected` 对于顶层声明不可用。

注意：如果要使用其他包的可见顶层声明，任然需要导入。

例子：

```kotlin
// file name: example.kt
package foo

private fun foo() {} // visible inside example.kt

public var bar: Int = 5 // property is visible everywhere
    private set         // setter is visible only in example.kt

internal val baz = 6    // visible inside the same module
```

## 类和接口
对于声明在类中的成员：

- `private` 意味着只在这个类中可见（包含类所有的成员）；
- `protected` —— 与 `private` 一致并且在子类中也可见；
- `internal` —— 与类的可见性一致，但是仅限于当前模块内；
- `public` —— 可见性与类一致。

Java 用户要注意：内部类的私有成员对外部类不可见。

如果覆写了一个 `protected` 成员并且没有显示指明可见性的话，覆写的成员的可见性也会是 `protected`。

例子：

```kotlin
open class Outer {
    private val a = 1
    protected open val b = 2
    internal val c = 3
    val d = 4 // public by default

    protected class Nested {
        public val e: Int = 5
    }
}

class Subclass : Outer() {
    // a is not visible
    // b, c, and d are visible
    // Nested and e are visible

    override val d = 5  // 'b' is protected
}

class Unrelated(o: Outer) {
    // o.a, o.b are not visible
    // o.c and o.d are visible (same module)
    // Outer.Nested is not visible, and Nested::e is not visible either
}
```

### 构造器
如果要指明类构造器的可见性，可以使用如下语法（注意，需要写明 `constructor` 关键字）：

```kotlin
class C private constructor(a: Int) { ... }
```

这里的构造器是私有。默认情况下，所有的构造器都是 `public`，只要是类可见的地方，构造器都可见（例如，`internal` 类的构造器只在同样的模块内可见）。

### 局部声明
局部变量、函数和类没有可见性修饰符。

## 模块
可见性修饰符 `internal` 表示成员在同样的模块中是可见的。更具体一点来说，一个模块就是编译在一起的 Kotlin 文件集合：

- 一个 IntelliJ IDEA 模块；
- 一个 Maven 工程；
- 一个 Gradle source set（有一个例外，`test` 代码集合可以访问 `main` 的内部声明）；
- 通过调用一次 Ant 任务编译的文件集合
