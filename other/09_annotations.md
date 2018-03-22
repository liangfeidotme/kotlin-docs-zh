# 注解
## 标记声明
注解是给代码附加元数据的手段。如果要定义一个注解，只需在类前面放置一个 `annotation` 修饰符：

```kotlin
annotation class Fancy
```

使用元注解（meta-annotation）可以给注解指定额外属性：

- `@Target` 用于指定可作用其上的元素（类、函数、属性和表达式等）
- `@Retention` 用于指定注解要存活到 class 还是运行时
- `@Repeatable` 允许同样的注解在一个元素上使用多次
- `@MustBeDocumented` 用于指定注解是 public API 的一部分，API 文档的类或者方法签名需要包含这个注解

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

如果要注解首要构造器，在构造器的声明中需要添加 `constructor` 关键字，并且把注解加在它前面：

```kotlin
class Foo @Inject constructor(dependency: MyDependency) {
    // ...
}
```

也可以注解属性访问器：

```kotlin
class Foo {
    var x: MyDependency? = null
        @Inject set
}
```

### 构造器
注解可以有带参数的构造器。

```kotlin
annotation class Special(val why: String)

@Special("example") class Foo {}
```

可用的类型参数有：

- 对应于 Java 基础类型（Int, Long 等）的类型
- 字符串
- 类（`Foo::class`）
- 枚举
- 其他标记
- 以上类型的数组

注解的参数不可以是空类型，因为 JVM 不支持把 `null` 作为注解属性的值来存储。

如果一个注解用作其他注解的参数，名字前面没有 `@` 符号：

```kotlin
annotation class ReplaceWith(val expression: String)

annotation class Deprecated(
        val message: String
        val replaceWith: ReplaceWidth = ReplaceWith(""))

@Deprecated("This function is deprecated, use === instead", ReplaceWith("this == other"))
```

如果要把类作为注解的实参，可以使用 Kotlin 类（`KClass`）。Kotlin 编译器会自动把它转化成 Java 类，这样 Java 代码就能够正常识别出注解和实参。

```kotlin
import kotlin.reflect.KClass

annotation class Ann(val arg1: KClass<*>, val arg2: KClass<out Any?>)

@Ann(String::class, Int::class) class MyClass
```

### lambda

注解也可以用于 lambda。它们会应用到的 `invoke()` 方法（lambda 体会生成于内）。对于 Quasar 这样的框架会比较有用，因为它把注解用作并发控制器。

```kotlin
annotation class Suspendable

val f = @Suspendable { Fiber.sleep(10) }
```

## 注解使用处目标（Use-site Target）
当注解一个属性或者首要构造器的参数时，对应的一个 Kotlin 元素会生成多个 Java 元素，因此注解在 Java 字节码中会有多个可能的位置。为了明确指明注解的位置，可以使用如下语法：

```kotlin
class Example(@field:Ann val foo,   // annotate Java field
              @get:Ann val bar,     // annotate Java getter
              @param:Ann val quux)  // annotate Java constructor parameter
```

同样的语法可用于注解整个文件。在文件顶部，package 指令之前（如果是文件是默认 package，则在 import 之前）放置一个目标是 `file` 的标记：

```kotlin
@file:JvmName("Foo")

package org.jetbrains.demo
```

如果一个目标（target）有多个注解，可以把所有的注解放进作用目标之后的方括号内，这样可以避免重复：

```kotlin
class Example {
    @set:[Inject VisibleForTesting]
    var collaborator: Collaborator
}
```

使用处目标的完整清单：

* `file`
* `property` (目标是它的注解对 Java 不可见);
* `field`
* `get` (属性 getter)
* `set` (属性  setter)
* `receiver` (扩展函数或扩展属性的接收者参数);
* `param` (构造器参数);
* `setparam` (属性 setter 参数);
* `delegate` (存储代理属性的代理实例的字段)

如果要注解扩展函数的接收者参数，可使用如下语法：

```kotlin
fun @receiver:Fancy String.myExtension() { }
```

如果不指明使用处目标，那么选定的目标的就会变成使用注解的 `@Target` 注解。如果有多个可用目标，下面列表中出现的第一个可用目标会被使用：

- `param`
- `property`
- `field`

## Java 注解

Java 的注解和 Kotlin 100% 兼容：

```kotlin
import org.junit.Test
import org.junit.Assert.*
import org.junit.Rule
import org.junit.rules.*

class Tests {
    // apply @Rule annotation to property getter
    @get:Rule val tempFolder = TemporaryFolder()

    @Test fun simple() {
        val f = tempFolder.newFile()
        assertEquals(42, getTheAnswer())
    }
}
```

因为 Java 没有定义注解参数的顺序，所以不能使用常规的函数调用语法来传递参数。不过可以利用命名参数的语法：

```java
// Java
public @interface Ann {
    int intValue();
    String stringValue();
}
```

```kotlin
// Kotlin
@Ann(intValue = 1, stringValue = "abc") class C
```

跟 Java 一样，有一个例外，`value` 参数的值无需指定参数名：

```java
// Java
public @interface AnnWithValue {
    String value()
}
```

```kotlin
// Kotlin
@AnnWithValue("abc") class C
```

### 数组作为注解参数
在 Java 中，如果 `value` 的实参是一个数组类型，Kotlin 会将其作为 `vararg` 来处理：

```java
// Java
public @interface AnnWithArrayValue {
    String[] value();
}
```

```kotlin
// Kotlin
@AnnWithArrayValue("abc", "foo", "bar") class C
```

对于其他数组类型的参数，需要使用数组字面量预发（Kotlin 1.2 之后开始支持）或者使用 `arrayOf(...)`：

```java
// Java
public @interface AnnWithArrayMethod {
    String[] names();
}
```

```kotlin
@AnnWithArrayMethod(names = ["abc", "foo", "bar"])
class C

// Older Kotlin versions:
@AnnWithArrayMethod(names = arrayOf("abc", "foo", "bar"))
class D
```

### 访问注解实例的属性
注解实例的值会暴露为 Kotlin 的属性：

```java
// Java
public @interface Ann {
    int value();
}
```

```kotlin
fun foo(ann: Ann) {
    val i = ann.value
}
```

