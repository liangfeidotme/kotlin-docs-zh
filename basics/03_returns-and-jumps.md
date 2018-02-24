返回和跳转
===

Kotlin 有三个结构化的跳转表达式：

- `return`      默认返回最近的外围函数或者匿名函数
- `break`       中断最近的外围循环
- `continue`    继续执行外围循环的下一步操作

以上表达式可作为更大表达式的一部分：

```kotlin
val s = person.name ?: return
```

以上表达式的类型是 `Nothing`。

break & continue 标签
---

Kotlin 的每个表达式都可以用一个 `label` 来标记。标签的形式是标识符后面带一个 `@` 符号，例如 `abc@`、`fooBar@` 都是合法的标签。如果要标记一个表达式，只需要在前面放一个 label：

```kotlin
loop@ for (i in 1..100) {
    // ...
}
```

现在，我们可以用一个 label 来修饰 `break` 和 `continue`：

```kotlin
loop@ for (i in 1..100) {
    for (j in 1..100) {
        if (...) break@loop
    }
}
```

这里用标签修饰的 `break` 会跳转到标签所标记的循环语句执行处。`continue` 会继续执行循环的下一次迭代。

标签返回
---

利用 `function literals`、`local functions` 和 `object expression`，Kotlin 支持函数嵌套。修饰过的 `return` 允许我们直接返回外部函数。最重要的使用 case 是从 lambda 表达式中返回。例如：

```kotlin
fun foo() {
    ints.forEach {
        if (it == 0) return // nonlocal return from inside lambda directly to the caller of foo()
        print(it)
    }
}
```

`return` 表达式返回最内层的外围函数 - `foo`。（注意：只有传入到内联函数的 lambda 表达式才支持非局部（non-local）返回。）。如果要从 lambda 表达式中返回，我们需要加标签并且修饰 `return`。

```kotlin
fun foo() {
    ints.forEach lit@ {
        if (it == 0) return@lit
        print(it)
    }
}
```

这样一来只会从 lambda 表达式中返回。多数情况下，使用隐式标签会更方便：标签与传入 lambda 表达式的函数名同名：

```kotlin
fun foo() {
    ints.forEach {
        if (it == 0) return@forEach
        print(it)
    }
}
```

还有另一种办法，我们可以用一个匿名函数来代替 lambda。匿名函数中的 `return` 语句只返回它自己。

```kotlin
fun foo() {
    ints.forEach(fun(value: Int) {
        if (value == 0) return  // local return to the caller of the annoymous fun, ie.e. the forEach loop
        print(value)
    }
}
```

如果 `return` 有返回值，那么标签的优先级高于返回值。

```kotlin
return@a 1
```

上例代码的含义是：“在标签`@a`处返回`1`”，而不是“返回一个用于比较的表达式 `(@a 1)`”。

