# 类和继承

## 类

使用关键字 `class` 来定义类：

```kotlin
class Invoice {
}
```

类的声明包含：类名、类头（包括类型参数、首要构造函数等）、类体（由圆括号包裹）。类头和类体都是可选的；如果没有类体，圆括号也可以省略。

```kotlin
class Empty
```

### 构造器

类可以有一个**首要构造函数**和多个**次要构造函数**。首要构造函数是类头的一部分：位于类名（以及可选类型参数）之后。

```kotlin
class Person constructor(firstName: String) {
}
```

如果首要构造函数没有 annotation 或者可视化修饰符，`constructor` 关键字可以省略。

```kotlin
class Person(firstName: String) {
}
```

首要构造函数不能包含任何代码。初始化代码可位于**初始化区块（initializer block）**（`init` 关键字是前缀）。
