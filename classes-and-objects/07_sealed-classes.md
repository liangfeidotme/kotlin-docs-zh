密封类
===

密封类是为了限制类的结构体系，这样可以确保一个值的类型范围可穷尽，不能是有限范围之外的其他类型。某种程度上是枚举类的扩展，因为枚举类型限制了值的范围，但是枚举常量只存在一个实例，然而密封类的子类可以有多个实例，并拥有自己的状态。

在类名签名加一个 `sealed` 修饰符来定义密封类。可以定义密封类的子类，但是子类必须跟这个密封类位于同一个文件（1.1 之前规则更严格：子类必须嵌套在密封类内部）。

```kotlin
sealed class Expr
data class Const(val number: Double) : Expr()
data class Sum(val e1: Expr, val e2: Expr) : Expr()
object NotANumber : Expr()
```

> 上例中用到了一个 1.1 中新增的特性：数据类可以继承自其他类（包括密封类）

密封类本身是一个抽象类，所以不能被实例化，但是可以有抽象成员。

密封类不能有非私有的构造函数（构造函数默认为私有）。

密封类子类的子类可以在其他文件中定义，不一定在同一个文件中。

when 表达式可以充分利用密封类的优势。如果可以验证 when 中的声明覆盖了所有情况，那么就可以省略 `else` 语句。但是这个特性只在 `when` 是表达式（expression）的情况下生效，不能用于声明（statement）的情况。

```kotlin
fun eval(expr: Expr): Double = when(expr) {
    is Const -> expr.number
    is Sum -> eval(expr.e1) + eval(expr.e2)
    NotANumber -> Double.NaN
    // the `else` clause is not required because we've covered all the cases
}
```
