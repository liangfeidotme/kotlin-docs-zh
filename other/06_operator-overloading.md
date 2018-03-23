# 操作符重载
Kotlin 允许我们为类型上预先定义好的操作符提供自定义实现。这些操作符都有固定的符号（像 `+` 或者 `*`）和固定的优先级。如果要实现一个操作符，我们需要通过一个固定的名称为对应的类型提供成员函数或者扩展函数，也就是二元操作符左侧的类型以及一元操作符的参数类型。重载操作符的函数需要用 `operator` 修饰符标记。

下面的内容是约束不同操作符重载方式的规范。

## 一元操作符
### 一元前缀操作符
表达式 | 转化为
--- | ---
+a | a.unaryPlus()
-a | a.unaryMinus()
!a | a.not()

这个表格的意思是，编译器在处理像 `+a` 这样的表达式时，会执行如下操作步骤：

* 确定 `a` 的类型，假设是 `T`；
* 寻找 `inc()` 函数（带 `operator` 修饰并且无参，接收者类型是 `T`），即成员函数或者扩展函数；
* 如果函数不存在或者有歧义，那么就引发编译时错误；
* 如果函数存在并且它的返回值类型是 `R`，那么 `+a` 表达式的类型就是 `R`。

注意，这些以及剩下的其他操作符，都会针对基本类型做优化，不会引入函数调用开销。

下面的例子说明了如何来重载一个一元减号操作：

```kotlin
data class Point(val x: Int, val y: Int)

operator fun Point.unaryMinus() = Point(-x, -y)

val point = Point(10, 20)
println(-point) // prints "(-10, -20)"
```

### 增量和减量
表达式 | 转化为
--- | ---
a++ | a.inc() + 参考下面
a-- | a.dec() + 参考下面

`inc()` 和 `dec()` 函数必须有返回值，而且返回值要赋值给 `++` 或者 `--` 操作符所作用的变量。但是这些操作不能改动调用 `inc` 或 `dec` 的对象。

编译器会执行如下步骤来解析一个*后缀**形式的操作符，例如 `a++`：

* 确定 `a` 的类型，假设是 `T`；
* 寻找 `inc()` 函数（带 `operator` 修饰并且无参，接收者类型是 `T`）。
* 检查函数返回值类型是否是 `T` 的子类型。
* 计算表达式的结果如下：

表达式的计算过程如下：

* 把 `a` 的初始值临时存储给 `a0`；
* 把 `a.inc()` 的结果赋值给 `a`；
* 把 `a0` 作为表达式结果返回。

`a--` 的步骤完全类似。

前缀形式的 `++a` 和 `--a` 的解析原理一样，过程如下：

* 把 `a.inc()` 的结果赋值给 `a`；
* 把 `a` 的新值作为表达式结果返回。

## 二元操作符
### 算术运算符
表达式 | 转化为
--- | ---
a + b | a.plus(b)
a - b | a.minus(b)
a * b | a.times(b)
a / b | a.div(b)
a % b | a.rem(b), a.mod(b)（已废弃）
a..b | a.rangeTo(b)

对于表格中的操作，编译器会把它们解析成*转化为*那一栏的内容。

注意，`rem` 操作符从 Kotlin 1.1 开始支持。Kotlin 1.0 使用 `mod` 操作符，Kotlin 1.1 中被废弃。

**例子**

下面是一个 Counter 类的例子，用一个给定值初始化之后，利用重载的 `+` 操作符进行递增：

```kotlin
data class Counter(val dayIndex: Int) {
    operator fun plus(increment: Int): Counter {
        return Counter(dayIndex + increment)
    }
}
```

### 'in' 操作符
表达式 | 转化为
--- | ---
a in b | b.contains(a)
a !in b | !b.contains(a)

`in` 和 `!in` 的流程是一样的，但是参数顺序是反的。

### 索引访问操作符
表达式 | 转化为
--- | ---
a[i] | a.get(i)
a[i, j] | a.get(i, j)
a[i_1, ..., i_n] | a.get(i_1, ..., i_n)
a[i] = b | a.set(i, b)
a[i, j] = b | a.set(i, j, b)
a[i_1, ..., i_n] = b | a.set(i_1, ..., i_n, b)

方括号会转换成对 `get` 和 `set` 的调用（使用对应数量的参数）。

### 调用操作符
表达式 | 转化为
--- | ---
a() | a.invoke()
a(i) | a.invoke(i)
a(i, j) | a.invoke(i, j)
a(i_1, ..., i_n) | a.invoke(i_1, ..., i_n)


### 增加操作符
表达式 | 转化为
--- | ---
a += b | a.plusAssign(b)
a -= b | a.minusAssign(b)
a *= b | a.timesAssign(b)
a /= b | a.divAssign(b)
a %= b | a.remAssign(b), a.modAssign(b)（已废弃）

对于赋值操作，例如，`a += b`，编译器会执行如下步骤：

* 如果表格右侧的方法可用
    * 如果对应的二元函数（即 `plusAssign` 对应的 `plus`）也可用，报错（有歧义）
    * 确保返回值是 `Unit`，否则报错
    * 为 `e.plusAssign(b)` 生成代码
* 否则，尝试生成 `a = a + b` 的代码（包含类型检查：`a + b` 的类型必须是 `a` 的子类型）

注意：赋值在 Kotlin 中**不是**表达式。

### 相等和不等操作符
表达式 | 转化为
--- | ---
a == b | a?.equals(b) ?: (b === null)
a != b | !(a?.equals(b)) ?: (b === null)

*注意：`===` 和 `!==`（身份检查）不可重载，所以没有针对它们的约定。*

`==` 操作符是特殊的：它会为 `null` 转换成一个复杂一些的表达式。`null == null` 永远为 true，并且 `x == null` 对于一个非空 `x` 来说，永远为 false，而且不会调用 `x.equals()`。

### 比较操作符
表达式 | 转化为
--- | ---
a > b | a.compareTo(b) > 0
a < b | a.compareTo(b) < 0
a >= b | a.compareTo(b) >= 0
a <= b | a.compareTo(b) <= 0

所有比较都会转化为 `compareTo` 的调用，并且需要返回 `Int`。

### 属性代理操作符
`provideDelegate`，`getValue` 和 `setValue` 操作符函数的具体内容可参考代理属性一节。

## 命名函数的中缀调用
我们可以使用中缀函数调用来模拟自定义中缀操作。
