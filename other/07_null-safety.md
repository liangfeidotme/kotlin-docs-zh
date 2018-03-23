# Null 安全
Kotlin 类型系统旨在从源码层面消除 null 引用的风险（十亿美金的错误）。

很多其他编程语言（包括 Java）最常见的一个缺陷是：访问 null 引用的成员时会导致 null 引用异常。Java 世界里的 `NullPointerException`，简称 NPE。

Kotlin 类型系统旨在从源码层面排除 `NullPointerException`。所有导致 `NPE` 的原因可能是：

* 显示调用 `throw NullPointerException()`；
* 使用 `!!` 操作符（下文有介绍）；
* 初始化相关的数据不一致性，例如：
    * 在构造器中可用的 `this` 变量没有初始化，却在其他地方被使用；
    * 超类构造器调用了开放成员，但是它在子类中的实现使用了未初始化的状态；
* 与 Java 的互操作
    * 尝试访问 `null` 引用（平台类型）的成员；
    * 使用泛型做互操作性时，可空性处理不正确，例如，Java 代码可能给 Kotlin 的 `MutableList<String>` 新增一个 `null` 变量，正确的做法应该使用 `MutableList<String?>`。
    * 其他由外部 Java 代码引起的问题；

Kotlin 的类型系统会明确区分可空和不可空。例如，常规的 `String` 类型变量不能够为空：

```kotlin
var a: String = "abc"
a = null // compilation error
```

为了允许可空，我们把变量声明为可空字符串，用 `String?`：

```kotlin
var b: String? = "abc"
b = null // ok
```

这样一来，访问 `a` 的方法和属性时，肯定不会导致 NPE，所以下面的写法是安全的：

```kotlin
val l = a.length
```

但是，如果要访问 `b` 的同样的属性，就会变成不安全的，编译器会报错：

```kotlin
val l = b.length // error: variable 'b' can be null
```

但是，我们依然要访问那个属性，下面介绍了几种访问方式。

## 在条件表达式中检查 `null`
首先，可以显示地去检查 `b` 是否为空，然后分别处理两种选项：

```kotlin
val l = if (b != null) b.length else -1
```

编译器会追踪已经执行过的检查的相关信息，然后允许在 `if` 内调用 `length`。也会支持更加复杂的条件：

```kotlin
if (b != null && b.length > 0) {
    print("String of length ${b.length}")
} else {
    print("Empty string")
}
```

需要注意的是：只有当 `b` 不可变时以上才会工作，即：

* 检查和使用之间不能被修改的局部变量
* 一个有幕后字段的 `val` 成员，并且不能被覆写

如果不是这样的话，`b` 在检查之后可能会变为 `null`。

## 安全调用
第二个选项是使用安全调用操作符，写成 `?.`：

```kotlin
b?.length
```

如果 `b` 不为空，返回 `b.length`，否则是 `null`。这个表达式的类型是 `Int?`。

安全调用很适合链式调用。例如，如果雇员 Bob 归属于某个部门（也可能不是），然后另一个雇员是这个部门的老大，如果要获取 Bob 部门老大的名字：

```kotlin
bob?.department?.head?.name
```

只要任何一个属性的值为空，这样一个调用链的值就是 `null`。

如果只有在变量不为空时才执行某个操作，那么可以用安全调用操作符配合 `let` 来使用：

```kotlin
val listWithNulls: List<String?> = listOf("A", null)
for (item in listWithNulls) {
    item?.let { println(it) } // prints A and ignores null
}
```

安全调用可位于赋值操作的左侧。然后，只要是安全调用链上任何一个接收者为空，赋值就会被跳过，右侧的表达式也就不会被计算。

```kotlin
// If either `person` or `person.department` is null, the function is not called:
person?.department?.head = managerPool.getManager();
```

## 猫王（Elvis）操作符
如果 `r` 是一个可空引用，我们可以说：如果 `r` 非空，使用它，否则使用一个非空的值 `x`：

```kotlin
val l: Int = if (b != null) b.length else -1
```

这个完整的 `if` 表达式可以简写，使用猫王操作符：`?:`:

```kotlin
val l = b?.length ?: -1
```

如果 `?:` 左侧的表达式不为空，猫王操作符会直接返回它，如果会返回右侧的表达式。需要注意，只有当右侧的表达式不为空时，左侧才会被计算。

注意，因为 `throw` 和 `return` 都是表达式，所以可以位于猫王操作符的右侧。这个用法非常方便，例如，检查函数参数。

```kotlin
fun foo(node: Node): String? {
    val parent = node.getParent() ?: return null
    val name = node.getName() ?: throw IllegalArgumentException("name expected")
    // ...
}
```

> SpEL ( Spring Expression Language ) 中 ?: 操作符的名称。因为这个操作符顺时针90度看很像30年代美国摇滚歌手猫王 Elvis Presley的发型，所以取名 Elvis operator。

## !! 操作符
第三种选项是给 NPE 爱好者准备的：这个非空断言操作符（`!!`）会把任何值转变成一个非空类型，如果值为空，则会抛出异常。我们可以写成 `b!!`，这样就会返回一个值不为空的 `b`（例如，例子中的 `String`），或者如果 `b` 是 null 的话抛出一个 NPE。

```kotlin
val l = b!!.length
```

因此，如果需要 NPE，可以使用 `!!`，但是需要明确地指定，否则他不会“出乎意思”地出现。

## 安全类型转换
在常规的类型转换中，如果对象不属于目标类型，会导致 `ClassCastException`。另一个选项是使用安全类型转化：如果转换不成功，返回 `null`。

```kotlin
val aInt: Int? = a as? Int
```

## 可空类型的集合
如果一个集合含有可空类型的元素，使用 `filterNotNull` 可以过滤出不可空类型的元素：

```kotlin
val nullableList: List<Int?> = listOf(1, 2, null, 4)
val initList: List<Int> = nullabelList.filterNotNull()
```
