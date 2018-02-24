标记
===

标记声明
---
标记是给代码附加元数据的手段。如果要定义标记，只需在类前面放置一个 `annotation` 修饰符：

```kotlin
annotation class Fancy
```

使用元标记（meta-annotation）可以给标记指定额外属性：

- `@Target` 用于指定了可作用其上的元素（类、函数、属性和表达式等）
- `@Retention` 用于指定标记的生命周期是到 class 文件还是运行时
- `@Repeatable` 允许同样的标记在一个元素上可使用多次
- `@MustBeDocumented` 用于指定标记是 public API 的一部分，API 文档的类或者方法签名需要包含这个标记。

```kotlin
@Target(AnnotationTarget.CLASS, AnnotationTarget.FUNCTION,
        AnnotationTarget.VALUE_PARAMETER, AnnotationTarget.EXPRESSION)
@Retention(AnnotationRetention.SOURCE)
@MustBeDocumented
annotation class Fancy
```

### 用法

```kotlin
@Fancy class Foo {
    @Fancy fun baz(@Fancy foo: Int): Int {
        return (@Fancy 1)
    }
}
```

如果需要标记首要构造函数，构造函数的声明中需要添加 `constructor` 关键字，并且把标记加在它前面：

```kotlin
class Foo @Inject constructor(dependency: MyDependency) {
    // ...
}
```

也可以标记属性访问器：

```kotlin
class Foo {
    var x: MyDependency? = null
        @Inject set
}
```

### 构造器
标记可以有带参数的构造器。

```kotlin
annotation class Special(val why: String)

@Special("example") class Foo {}
```

允许的类型参数有：

- Java 基础类型（Int, Long 等）所对应的类型
- 字符串
- 类（`Foo::class`）
- 枚举
- 其他标记
- 以上类型的数组

表及参数不可以是空类型，因为 JVM 不支持把 `null` 作为标记属性的值来存储。

如果一个标记用作其他标记的参数，名字前面没有 `@` 符号：

```kotlin
annotation class ReplaceWith(val expression: String)

annotation class Deprecated(
        val message: String
        val replaceWith: ReplaceWidth = ReplaceWith(""))

@Deprecated("This function is deprecated, use === instead", ReplaceWith("this == other"))
```

如果要把类作为标记的实参，可以使用 Kotlin class（`KClass`）。Kotlin 编译器会自动把它转化成 Java 类，这样 Java 代码就能够正常识别出标记和实参。

```kotlin
import kotlin.reflect.KClass

annotation class Ann(val arg1: KClass<*>, val arg2: KClass<out Any?>)

@Ann(String::class, Int::class) class MyClass
```

### lambda

标记还可以用在 lambda 上。他们作用执行 lambda 体的 `invoke()` 方法。对于 Quasar 这样的框架会比较有用，因为它把标记作为并发控制器。

```kotlin
annotation class Suspendable

val f = @Suspendable { Fiber.sleep(10) }
```

标记使用点目标（Use-site Target）
---
当标记一个属性或者首要构造函数的参数时，对应的一个 Kotlin 元素会生成多个 Java 元素，因此标记在 Java 字节码中会有多个可能的位置。为了明确指明标记的位置，可以使用如下语法：

```kotlin
class Example(@field:Ann val foo,   // annotate Java field
              @get:Ann val bar,     // annotate Java getter
              @param:Ann val quux)  // annotate Java constructor parameter
```

同样的语法可用于标记整个文件。在文件顶部，package 指令之前（如果是文件是默认 package，则在 import 之前）放置一个目标是 `file` 的标记：

```kotlin
@file:JvmName("Foo")

package org.jetbrains.demo
```

如果一个目标（target）有多个标记，可以把所有的标记放进作用目标之后的方括号内，这样可以避免重复：

```kotlin
class Example {
    @set:[Inject VisibleForTesting]
    var collaborator: Collaborator
}
```

使用点目标的完整清单：

* `file`
* `property` (annotations with this target are not visible to Java);
* `field`
* `get` (property getter)
* `set` (property setter)
* `receiver` (receiver parameter of an extension function or property);
* `param` (constructor parameter);
* `setparam` (property setter parameter);
* `delegate` (the filed storing the delegate instance for a delegated property)


