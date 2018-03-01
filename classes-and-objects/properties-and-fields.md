# 属性和字段

## 声明属性
Kotlin 中的类可以拥有属性。`var` 表示可变，`val` 表示只读。

```kotlin
class Address {
    var name: String = ...
    var street: String = ...
    var city: String = ...
    var state: String = ...
    var zip: String = ...
```

可以用名字指代一个属性，就好像 java 的字段：

```kotlin
fun copyAddress(address: Address): Address {
    val result = Address()  // there's no 'new' keyword in Kotlin
    result.name = address.name // accessors are called
    result.street = address.street
    // ...
    return result
}
```

## Getter 和 Setter
声明一个属性的完整语法如下：

```kotlin
var <propertyName>[: <PropertyType>] [= <property_initializer>]
    [<getter>]
    [<setter>]
```

initializer、getter 和 setter 是可选的。类型如果能从 initializer 或者返回值中推导得出，也可以省略。

例如：

```kotlin
var allByDefault: Int? // error: explicit initializer required, default getter and setter implied
var initialized = 1 // has type Int, default getter and setter
```

只读属性和可变属性的完整语法有两点不同：

* 只读属性用 `val` 而不是 `var`
* 只读属性不允许有 setter

```kotlin
val simple: Int? // has type Int, default getter, must be initialized in constructor
val inferredType = 1 // has type Int and a default getter
```

我们可以在属性的声明中自定义访问器，跟普通函数非常类似。自定义 getter 的方式如下：

```kotlin
val isEmpty: Boolean
    get() = this.size == 0
```

自定义 setter 的方式如下：

```kotlin
var stringRepresentation: String
    get() = this.toString()
    set(value) {
        setDataFromString(value) // parses the string and assigns values to other properties
    }
}
```

按照约定，setter 的参数名是 `value`，当然也可以使用其他名字。

从 Kotlin 1.1 开始，如果能从 getter 中推断出类型，可以省略属性类型：

```kotlin
val isEmpty get() = this.size == 0 // has type Boolean
```

如果需要改变访问器的可见性或者添加注解，但是并不需要改变它的实现，可以直接定义访问器，并不需要 body。

```kotlin
var setterVisiblity: String = "abc"
    private set // the setter is private and has the default implementation

var setterWithAnnotation: Any? = null
    @Inject set // annotate the setter with Inject
```

## 备份字段
字段可以直接声明在 Kotlin 类中。但是，当属性需要备份字段时，Koltin 会自动提供。在访问器中可以通过 `field` 标识符来引用备份字段：

```kotlin
var counter = 0 // the initializer value is written directly to the backing field
    set(value) {
        if (value >= 0) field = value
    }
```

`field` 标识符只能用于属性的访问器中。

备份字段的生成需要属性满足一定的条件：备份字段使用了至少一个访问器的默认实现，或者，自定义访问器通过 `field` 标识符来引用它。

例如，下面这个 case 就不会生成备份字段：

```kotlin
val isEmpty: Boolean
    get() = this.size == 0
```

## 备份属性

如果你不想落入“隐式备份字段”的窠臼，可以退而求其次，使用备份属性：

```kotlin
private var _table: Map<String, Int>? = null
public val table: Map<String, Int>
    get() {
        if (_table == null) {
            _table = HashMap() // Type parameters are inferred
        }
        return _table ?: throw AssertionError("Set to null by another thread")
    }
```
从各个方面来看，这点跟 Java 非常类似，因为通过默认 getter 和 setter 来访问私有属性都可以被优化掉，以避免函数调用带来的开销。

## 编译时常量
如果一个属性在编译时可以确定它的取值，那么可以使用 `const` 修饰符标记为 *编译时常量*。这种属性需要满足如下条件：

* 顶层或者一个 `object` 的成员
* 初始化为一个 `String` 或者基本类型的值
* 没有自定义 getter

这些属性可用于注解：
```kotlin
const val SUBSYSTEM_DEPRECATED: String = "This subsystem is deprecated"

@Deprecated(SUBSYSTEM_DEPRECATED) fun foo() { ... }
```

## 延迟初始化属性和变量
通常，声明为非空类型的属性必须在构造器中初始化。但是，这种做法经常会不方便。例如，属性可以通过依赖注入进行初始化，或者在 ut 的 setup 方法中初始化。这种情况下就无法在构造器中初始化为非空，但是我们仍然希望在类内部使用这个属性时可以避免做空检查。

这种 case 的处理方式是：用 `lateinit` 修饰符来标记这个属性：

```kotlin
public class MyTest {
    lateinit var subject: TestSubject

    @SetUp fun setup() {
        subject = TestSubject()
    }

    @Test fun test() {
        subject.method()    // dereference directly
    }
}
```

这个修饰符可作用于 `var` 定义的属性，声明位于类内部，但不是首要构造器，而且属性不能有自定义的 getter 或者 setter，并且，从 Kotlin 1.2 开始，也可以用于顶层属性和局部变量。属性或者变量的类型必须为非空，且不能是基本类型。

在初始化之前访问一个 `lateinit` 属性会抛出一个特定的异常，这个异常会指明被访问的属性并且会说明它还没有被初始化。

## 覆写属性
可见[属性覆写](classes-and-inheritance.md#属性覆写)章节。

## 代理属性
最常见的属性只是简单读取（或者写入）一个备份字段。从另一方面来说，利用自定义 getter 和 setter，我们可以实现一个属性的任意行为。介乎两者之间存在着一些属性如何工作的通过用模式。可以举几个例子：lazy value，通过给定的 key 读取 map 值，访问数据库，通知访问的监听器，等等。

这些共同的行为可以基于[代理属性](delegated-properties.md)封装成库。
