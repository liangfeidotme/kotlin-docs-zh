# 密封类
密封类用于表示受限制的类结构体系，这样可以确保一个值的类型范围可穷尽，不能是有限范围之外的其他类型。从某种程度上来说，密封类是枚举类的扩展，因为枚举类型限制了值的范围，但是枚举常量只存在一个实例，然而密封类的子类可以有多个实例，并拥有自己的状态。

定义密封类需要在类名之前加一个 `sealed` 修饰符。密封类可以拥有子类，但是子类必须跟这个密封类位于同一个文件（1.1 之前规则更严格：子类必须嵌套在密封类内部）。

```kotlin
sealed class Expr
data class Const(val number: Double) : Expr()
data class Sum(val e1: Expr, val e2: Expr) : Expr()
object NotANumber : Expr()
```

> 上例中用到了一个 1.1 中新增的特性：数据类可以继承自其他类（包括密封类）

密封类本身是一个抽象类，所以无法直接实例化，但是它可以有抽象成员。

密封类不能拥有非私有的构造函数（构造函数默认为私有）。

继承了密封类子类（直接继承者）的类可以在其他文件中定义，不一定在同一个文件中。

使用密封类的巨大优势在遇到 when 表达式时就可以体现出来。如果可以验证 when 中的声明覆盖了所有情况，那么就可以省略 `else` 语句。但是这个特性只在 `when` 是表达式而不是声明的的情况下生效。

```kotlin
fun eval(expr: Expr): Double = when(expr) {
    is Const -> expr.number
    is Sum -> eval(expr.e1) + eval(expr.e2)
    NotANumber -> Double.NaN
    // the `else` clause is not required because we've covered all the cases
}
```
