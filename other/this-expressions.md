this 表达式
===

[*原文链接*](http://kotlinlang.org/docs/reference/this-expressions.html)

我们用 `this` 表示当前的接收器（receiver）：

- 在类的成员中，`this` 指代类对象
- 在扩展函数中和普通的函数接收器中，`this` 指代点号左侧的接收器参数

如果 `this` 没有修饰符，它指代_最内部的封闭区域（innermost enclosing scope）_。其他区域的 `this` 要通过标签修饰符（label qualifier）来引用。

限定 this
---
如果要访问外部区域（类、扩展函数、带标签的函数接收器）的 `this`，要写成 `this@label`，其中 `@label` 表示 `this` 的来源。

```kotlin
class A { // implicit label @A
    inner class B { // implicit label @B
        fun Int.foo() { // implicit label @foo
            val a = this@A // A's this
            val b = this@B // B's this

            val c = this // foo()'s receiver, an Int
            val c1 = this@foo // foo()'s receiver, an Int

            val funLit = lambda@ fun String.() {
                val d = this // funLit's receiver
            }


            val funLit2 = { s: string -> 
                // foo()'s receiver, since enclosing lambda expression
                // doesn't have any receiver
                val d1 = this
            }
        }
    }
}
```
