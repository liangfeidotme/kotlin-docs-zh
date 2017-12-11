属性和字段
===

声明属性
---

Kotlin 可以给类定义定义属性。`var` 可声明可变类型，`val` 可声明只读类型。

```kotlin
class Address {
    var name: String = ...
    var street: String = ...
    var city: String = ...
    var state: String = ...
    var zip: String = ...
```

可以用名字指向一个属性，就好像 java 的字段：

```kotlin
fun copyAddress(address: Address): Address {
    val result = Address()  // there's no 'new' keyword in Kotlin
    result.name = address.name // accessors are called
    result.street = address.street
    // ...
    return result
}
```

getters 和 setters
---

声明一个属性的完整语法如下：


```kotlin
var <propertyName>[: <PropertyType>] [= <property_initializer>]
    [<getter>]
    [<setter>]
```

initializer、getter 和 setter 是可选的。
