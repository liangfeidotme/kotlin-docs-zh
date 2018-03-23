# 数据类

我们经常会新建一些仅仅用来携带数据的类。这种类的一些标准功能和工具函数都是机械式由数据衍生而来。Kotlin 称之为*数据类（data class）*，用 `data` 来标记。

```kotlin
data class User(val name: String, val age: Int)
```

编译器会自动根据首要构造函数中声明的属性生成如下代码：

- `equals()` / `hashCode()` 对
- `toString()`（格式：`"User(name=John, age=42)"`）
- `componentN()`函数，对应于属性的声明顺序
- `copy()` 函数（下有说明）

为了保证生成代码的一致性以及其行为的有意义性，数据类需要满足如下要求：

- 首要构造函数至少有一个参数
- 首要构造函数的所有参数都必须用 `val` 或者 `var` 标记
- 数据类不能是 abstract、open、sealed 以及 inner
- （1.1之前）数据类可以仅仅实现接口

另外，根据类成员的继承方式，类成员的生成要符合如下规则：

- 如果数据类本身显示地定义了 `equals()`、`hashCode()` 或者 `toString()` 函数，或者父类有 `final` 的实现，那么这些函数不会自动生成，直接使用现成的。
- 如果父类有 `componentN()` 函数（`open` 类型并且返回值类型兼容），数据类会生成并覆写父类函数。如果因为父类函数的签名不兼容或者是有 `final` 修饰导致无法覆写的话，会报错。
- 数据类可以继承一个带有 `copy()` 函数（函数签名一致）的类型的行为在 1.2 中被废弃，1.3 会禁用。
- 显示提供 `componentN()` 和 `copy()` 的实现是不被允许的。

从 1.1 开始，数据类可以扩展其他类（可参考[密封类](classes-and-objects/sealed-classes.md)）。

在 JVM 之上，如果类需要无参构造器，那么需要给属性指定默认值。

```kotlin
data class User(val name: String = "", val age: Int = 0)
```

## 类体中声明的属性
编译器只会使用首要构造函数中定义的属性来自动生成函数。如果要从生成的实现中排除一个属性，可以把它声明在类体中：

```kotlin
data class Person(val name: String) {
    var age: Int = 0
}
```

只有属性 `name` 会用于 `toString()`、`equals()`、`hashCode()` 和 `copy()` 的实现中，并且只会有一个组件函数 `component1()`。虽然两个 `Person` 会有不同的年龄，但是他们会被识为相等。

```kotlin
val person1 = Person("John")
val person2 = Person("John")

person1.age = 10
person2.age = 20
```

## 拷贝
很多时候，我们需要拷贝一个对象，修改某些属性，但是其他保持不变。这就是 `copy()` 函数的作用。例如，上例的 `User` 类，`copy()` 实现如下：

```kotlin
fun copy(name: String = this.name, age: Int = this.age) = User(name, age)
```

可以这么用：

```kotlin
val jack = User(name = "Jack", age = 1)
val olderJack = jack.copy(age = 2)
```

## 数据类和解构声明
数据类生成的组件函数在解构声明中会发挥作用：

```kotlin
val jane = User("Jane", 35)
val (name, age) = jane
println("$name, $age years of age") // prints "Jane, 35 years of age"
```

## 标准数据类
标准库提供的是 `Pair` 和 `Triple`。大部分情况下，自定义有具体名字的数据类是一个比较好的设计方式，因为提供了有意义的属性名字，所以代码可读性更高。
