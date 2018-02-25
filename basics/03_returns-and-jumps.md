# 返回和跳转

Kotlin 支持三个结构化的跳转表达式：

- `return`      默认返回就近的封闭（enclosing）函数或者匿名函数
- `break`       中断就近的封闭循环
- `continue`    继续执行就近封闭循环的下一步操作

以上表达式可作为更大表达式的一部分：

```kotlin
val s = person.name ?: return
```

以上**表达式的类型**是 `Nothing`。

## break & continue 标签
Kotlin 的每个表达式都可以用一个 `label` 来标记。标签的形式是标识符后面带一个 `@` 符号，例如 `abc@`、`fooBar@` 都是合法的标签。如果要**标记一个表达式**，只需要在前面放一个 label：

```kotlin
loop@ for (i in 1..100) {
    // ...
}
```

现在，我们可以用一个 label 来限定 `break` 和 `continue` 的作用目标：

```kotlin
loop@ for (i in 1..100) {
    for (j in 1..100) {
        if (...) break@loop
    }
}
```

这里用标签限定过的 `break` 会跳转到标签所标记的循环语句执行处。`continue` 会继续执行循环的下一次迭代。

## 标签返回
借助 function literals、local functions 和 object expression，Kotlin 还支持函数嵌套。限定过的 `return` 允许我们直接从外围函数中返回。最重要的使用场景是从 lambda 表达式中返回。当我们写下下面的代码时：

```kotlin
fun foo() {
    ints.forEach {
        if (it == 0) return // non-local return directly to the caller of foo()
        print(it)
    }
}
```

`return` 表达式从就近的外围函数中返回 - `foo`。（注意：只有传入到内联函数的 lambda 表达式才支持非局部（non-local）返回。）。如果要从 lambda 表达式中返回，我们需要加标签并且限定 `return`。

```kotlin
fun foo() {
    ints.forEach lit@ {
        if (it == 0) return@lit // local return to the caller of the lambda, i.e. the forEach loop
        print(it)
    }
}
```

现在，它只会从 lambda 表达式中返回。多数情况下，更便利的方式是使用隐式标签：标签与接收 lambda 表达式作为参数的函数名同名。

```kotlin
fun foo() {
    ints.forEach {
        if (it == 0) return@forEach
        print(it)
    }
}
```

除此之外，我们可以用一个匿名函数来代替 lambda。匿名函数中的 `return` 声明会从它自己当中返回。

```kotlin
fun foo() {
    ints.forEach(fun(value: Int) {
        if (value == 0) return  // local return to the caller of the annoymous fun, ie.e. the forEach loop
        print(value)
    }
}
```

注意，上面三个例子当中 local return 的用法和常规循环中 `continue` 的作用类似。没有与 `break` 直接等价的用法，但是我们可以采用增加一个嵌套 lambda 并从中局部返回的做法来模拟实现：

```kotlin
fun foo() {
    run loop@{
        listOf(1, 2, 3, 4, 5).forEach {
            if (it == 3) return@loop // non-local return from the lambda passed to run
            print(it)
        }
    }

    print("done with nested loop")
}
```

当返回一个值时，parser 给予限定 return 更高的优先级。

```kotlin
return@a 1
```

上面的代码意味着：“在标签`@a`处返回`1`”，而不是“返回一个标记过的表达式 `(@a 1)`”。

---

术语约定：

0. enclosing function -> 封闭函数
0. outer function -> 外围函数
0. qualified -> 限定的