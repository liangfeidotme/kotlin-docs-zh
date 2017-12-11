基本类型
===

Kotlin 中的所有变量都是对象，所以我们可以调用任何变量的成员函数和属性。某些类型可以有特殊的内部表现 - 例如，数字、字符和布尔型在运行时可以表现为基础类型（primitive types），但是对用户来说，他们看上去就是是普通的类。这一章节主要描述 Kotlin  基本类型：数字、字符、布尔、数组和字符串。

数值
---
Kotlin 处理数字的方式与 Java 类似，但不是完全一致。例如，数值没有隐士的类型升级（implicit widening conversions），某些情况下，字面意思也有所不同。

Kotlin 提供了如下内置类型来表示数值（Java 接近）：

类型 | 位宽
--- | ---
Double | 64
Float | 32
Long | 64
Int | 32
Short | 16
Byte | 8

注意：字符不是数值。

### 字面常量
整形的字面常量有如下形式：

- 十进制：`123`
    - 长整型用 `L` 表示：`123L`
- 十六进制：`0x0F`
- 二进制：`0b00001011`

注意：八进制不支持

浮点数也支持约定的标记：

- double 类型：`123.5`，`123.5e10`
- float 用 `f` 或者 `F` 表示：`123.5f`

### 数值字面值中的下划线（1.1开始）

下划线可以使数值常量更具有可读性：

```kotlin
val oneMillion = 1_000_000
val creditCardNumber = 1234_5678_9012_3456L
val socialSecurityNumber = 999_99_9999L
val hexBytes = 0xFF_EC_D5_5E
val bytes = 0b11010010_01101001_10010100_10010010
```

### 表现形式
Java 平台会把数值作为 JVM 基础类型来物理存储。除非是一个可为空的数值引用（例如 `Int?`）或者有泛型引入。如果是后者，数值会装箱。

注意：装箱后的数值不会保持身份（preserve identity）：

```kotlin
val a: Int = 10000
print(a === a) // Prints 'true'
val boxedA: Int? = a
val anotherBoxedA: Int? = a
print(boxedA === anotherBoxedA) // !!!Prints 'false`!!!
```

在另一个领域，它会保持身份：

```kotlin
val a: Int = 10000
print(a == a) // Prints 'true'
val boxedA: Int? = a
val anotherBoxedA: Int? = a
print(boxedA == anotherBoxedA) // Prints 'true'
```

隐式转换
---
由于不同的表现形式，小类型并非大类型的子类型。如果是的话，可能会带来如下麻烦：

```kotlin
// Hypothetical code, does not actually compile:
val a: Int? = 1 // A boxed Int (java.lang.Integer)
val b: Long? = a // implicit conversion yields a boxed Long (java.long.Long)
print(a == b) // Surprise! This prints "false" as Long's equals() check for other part to be Long as well
```

所以，不只是身份（identity），连相等性（equality）也会静默丢失。

因此，小类型**不会**隐式转换成大类型。这就意味着：不通过显示转换，我们无法把一个 `Byte` 赋值给 `Int`。

```kotlin
val b: Byte = 1 // OK, literals are checked statically
val i: Int = b // ERROR
```

通过显示转换可以“拓宽（widen）”数值。

```kotlin
val i: Int = b.toInt()  // OK: explicitly widened
```

每个数值类型都支持如下转换：

* `toByte(): Byte`
* `toShort(): Short`
* `toInt(): Int`
* `toLong(): Long`
* `toFloat(): Float`
* `toDouble(): Double`
* `toChar(): Char`

隐式转换的缺席很少会被注意到，因为类型可以从上下文推导出，并且算术操作符也有重载：

```kotlin
val l = 1L + 3 // Long + Int => Long
```

### 操作

数组
---
Kotlin 用 `Array` 类来表示数组，有 `get` 和 `set` 函数（利用操作符重载的约定可转换成 `[]` 操作），还有 `size` 属性，初次之外还有其他有用的成员函数：

```kotlin
class Array<T> private constructor() {
    val size: Int
    operator fun get(index: Int): T
    operator fun set(index: Int, value: T): Unit
    operator fun iterator(): Iterator<T>
    // ...
}
```

使用库函数 `arrayOf()` 并传入元素值可以创建一个数组：`arrayOf(1, 2, 3)` 创建了 [1, 2, 3]。另外，`arrayOfNulls()` 可以创建一个所有元素都是 null 的数组。

另一种创建方式是调用 `Array` 的构造函数，传入数组大小和一个根据下标返回初始值的函数：

```kotlin
// Creates an Array<String> with values ["0", "1", "4", "9", "16"]
val asc = Array(5, { i -> (i * i).toString() })
```

上面已经说过，`[]` 操作等价于调用成员函数 `get()` 和 `set()`。

注意：与 Java 不同，Kotlin 的数组是不可变的（invariant）。这就意味着 



字符串
---

字符串由 `String` 表示。字符串是不可变的。字符串的元素可通过下标访问：`s[i]`。字符串可通过 `for` 循环遍历：

```kotlin
for (c in str) {
    println(c)
}
```

### 字符串字面值

Kotlin 支持两种类型的字符串字面值：包含转义字符的转义字符串和包含换行和任意文本的纯字符串。转义字符串跟 Java 类似：

```kotlin
val s = "Hello, world\n"
```

转义遵守约定俗称的方式（利用 \）。字符章节列出了所有支持的转义序列：

```kotlin
val text = """
    for (c in "foo")
        print(c)
"""
```

可以通过 `trimMargin()` 去除开头的空字符：

```kotlin
val text = """
    |Tell me and I forget
    |Teach me and I remember.
    |Involve me and I learn.
    |(Benjamin Franklin)
    """.trimMargin()
```

默认情况下，`|` 用作 margin 前缀，但是也可以使用其他字符，例如 `trimMargin(">")`。

### 字符串模板

字符串可以包含模板表达式，例如，代码可以被执行，执行结果可以被嵌入到字符串中。模板表达式以美元符号（$）开始，可以由一个简单的名称组成：

```kotlin
val i = 10
val s = "i = $i" // evaluates to "i = 10"
```

或者是大括号包裹的任意表达式：

```kotlin
val s = "abc"
val str = "$s.length is ${s.length}" // evaluates to "abc.length is 3"
```

模板可用于纯字符串和转义后的字符串内。如果要在纯字符串（不支持转义）中展示 `$` 符号，可以使用如下语法：

```kotlin
val price = """
${'$'}9.99
"""
```
