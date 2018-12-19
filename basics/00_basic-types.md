# 基本类型
Kotlin 中，我们可以调用**任何变量**的成员函数和属性，从这个角度来说，一切皆对象。某些类型可以有特殊的内部表现 - 例如，数字、字符和布尔型在运行时可以表现为基础类型（primitive types），但是对用户来说，他们看上去就是是普通的类。这一章节主要描述 Kotlin 的基本类型：数字、字符、布尔、数组和字符串。

## 数值
Kotlin 处理数字的方式与 Java 类似，但不是完全一致。例如，数值没有隐式的拓宽转换（implicit widening conversions），某些情况下，字面意思也会稍有不同。

Kotlin 提供了如下内置类型来表示数值（接近 Java）：

类型 | 位宽
--- | ---
Double | 64
Float | 32
Long | 64
Int | 32
Short | 16
Byte | 8

> 注意：字符不是一种数值。

### 字面常量
整形值的字面常量有如下形式：

- 十进制：`123`
    - 长整型用 `L` 做标记：`123L`
- 十六进制：`0x0F`
- 二进制：`0b00001011`

> 注意：不支持八进制。

浮点数也支持约定的标记：

- double 类型：`123.5`，`123.5e10`
- float 用 `f` 或者 `F` 标记：`123.5f`

### 数值字面值中的下划线（1.1开始）

下划线可以使数值常量更具可读性：

```kotlin
val oneMillion = 1_000_000
val creditCardNumber = 1234_5678_9012_3456L
val socialSecurityNumber = 999_99_9999L
val hexBytes = 0xFF_EC_D5_5E
val bytes = 0b11010010_01101001_10010100_10010010
```

### 表现形式
Java 平台会把数值作为 JVM 基础类型来物理存储。除非是一个可为空的数值引用（例如 `Int?`）或者有泛型引入。如果是后者，数值会装箱。

注意：装箱后的数值不会保持 **identity**：

```kotlin
val a: Int = 10000
print(a === a) // Prints 'true'
val boxedA: Int? = a
val anotherBoxedA: Int? = a
print(boxedA === anotherBoxedA) // !!!Prints 'false`!!!
```

但是仍然会有相等性：

```kotlin
val a: Int = 10000
print(a == a) // Prints 'true'
val boxedA: Int? = a
val anotherBoxedA: Int? = a
print(boxedA == anotherBoxedA) // Prints 'true'
```

### 显示转换
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

缺少隐式转换并不会引起注意，因为通过上下文可以推导出类型，并且算术操作符也有支持类型转换的重载，例如：

```kotlin
val l = 1L + 3 // Long + Int => Long
```

### 运算
Kotlin 支持数值的标准算术运算，这些运算被声明为相应类的成员（但是编译器会把函数调用优化成相应的指令）。参考[操作符重载](../other/operator-overloading.md)。

位运算操作符也没有特殊之处，他们也只是支持中缀调用的命名函数，例如：

```kotlin
val x = (1 shl 2) and 0x000FF000
```

如下是位运算操作符的完整列表（只用于 `Int` 和 `Long`）：

* `shl(bits)` - 有符号左移（Java 的 `<<`）
* `shr(bits)` - 有符号右移（Java 的 `>>`）
* `ushr(bits)` - 无符号右移（Java 的 `>>>`）
* `and(bits)` - 位的与运算
* `or(bits)` - 位的或运算
* `xor(bits)` - 位的异或运算
* `inv(bits)` - 位的非运算

### 浮点数比较
本节所要讨论的浮点数运算符有：

* 相等检查：`a == b` 和 `a != b`
* 比较操作符：`a < b`，`a > b`，`a <= b`, `a >=b`
* 范围初始化和范围检查：`a..b`，`x in a..b`，`x !in a..b`

当操作数 `a` 和 `b` 静态已知为类型 `Float` 或 `Double`，以及它们对应的可空类型（得出方式包括：声明、推断或者[智能转换]()），数值的运算以及它们形成的范围（range）遵守 IEEE 754 制定的浮动点数运算规范。

但是为了支持通用的使用场景以及提供完整的排序，当操作数不是浮点数的静态类型（如 `Any`、`Comparable<...>`，类型参数）时，运算操作会使用 `Float` 和 `Double` 的 `equals` 和 `compareTo` 实现，这会导致异与标准，因此：

* `NaN` 等于它自己
* `NaN` 大与所有其他元素，包括 `POSITIVE_INFINITY`
* `-0.0` 小于 `0.0`


## 字符
`Char` 表示字符，不能直接用作数值：

```kotlin
fun check(c: Char) {
    if (c == 1) { // ERROR: incompatible types
        // ...
    }
}
```

字符用单引号来表示：`'1'`。特殊字符可以使用反斜杠来转义。

特殊字符可以用反斜杠转义。支持的转义序列有：`\t`、`\b`、`\n`、`\r`、`\'`、`\"`、`\\`、`\$`。如果要编译其他字符，可以使用 Unicode 转义序列语法：`\uFF00`。

我们可以显示地把一个字符转换成一个 `Int` 数值：

```kotlin
fun decimalDigitValue(c: Char): Int {
    if (c !in '0'..'9')
        throw IllegalArgumentException("Out of range")
    return c.toInt() - '0'.toInt() // Explicit conversions to numbers
}
```

就像数值那样，字符的空引用也会自动装箱。装箱操作不会保留字符的身份（identity）。

## 布尔型
`Boolean` 表示布尔型，有两个值：`true` 和 `false`。

布尔的可空引用会自动装箱。

内置操作符包括：

* `||` - lazy disjunction
* `&&` - lazy conjunction
* `!` - negation

## 数组
Kotlin 用类 `Array` 来表示数组，有 `get` 和 `set` 函数（利用操作符重载的约定可转换成 `[]` 操作），还有 `size` 属性，除此之外还有其他有用的成员函数：

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

注意：与 Java 不同，Kotlin 的数组是不可变的（invariant）。这就意味着 Kotlin 不允许我们把 `Array<String>` 赋给 `Array<Any>`，这样能避免运行时的失败（但是能用 `Array<out Any>`，可参考[类型映射](../classes-and-objects/generics.md#类型映射)）。

Kotlin 也有特定的类用于表示基础类型数组（没有装箱的开销）：`ByteArray`、`ShortArray`、`IntArray` 等。这几个类和 `Array` 没有直接的继承关系，但是他们有同样的方法和属性。每个类型都有相应的工厂函数：

```kotlin
val x: IntArray = intArrayOf(1, 2, 3)
x[0] = x[1] + x[2]
```

## 字符串
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

转义遵守约定俗成的方式（利用 \）。上面的字符那一章节已经列出了所有支持的转义序列。

纯字符串通过三个引号（`"""`）来界定，它不会包含转义而且能够包含换行和任意字符：

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

默认情况下，`|` 用作 margin 前缀，但是也可以使用其他字符作为参数传给 `trimMargin`，例如 `trimMargin(">")`。

### 字符串模板
字符串可以包含模板表达式，例如，可被求值的代码片段，求值结果可以连接到字符串中。模板表达式以美元符号（$）开始，由一个简单的名称组成：

```kotlin
val i = 10
val s = "i = $i" // evaluates to "i = 10"
```

或者是大括号内的任意表达式：

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
