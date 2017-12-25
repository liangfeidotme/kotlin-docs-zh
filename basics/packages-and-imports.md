包名
===

源文件的开头可以是 package 的声明：

```kotlin
package foo.bar

fun baz() {}

class Goo {}

// ...
```

源文件的所有的内容（例如类和函数）都会包含在所声明的 package 中。所以，上例中 `baz()` 的完整名称是 `foo.bar.baz`，`Goo` 是 `foo.bar.Goo`。

package 没有指明的情况下，文件的内容归属于一个无名的“默认” package。


默认导入
---

每一个 kotlin 文件都会默认导入一些 package：

- `kotlin.*`
- `kotlin.annotation.*`
- `kotlin.collections.*`
- `kotlin.comparisons.*`（1.1 开始支持）
- `kotlin.io.*`
- `kotlin.ranges.*`
- `kotlin.sequences.*`
- `kotlin.text.*`

不同的目标平台还会导入额外的 package：

- JVM:
    - `java.lang.*`
    - `kotlin.jvm.*`
- JS:
    - `kotlin.js.*`

导入
---
默认的导入之外，每个文件都有自己的导入指令。

既可以只导入一个名称：

```kotlin
import foo.Bar // Bar is now accessible without qulification
```

或者导入一个 scope（package，class, object etc）内所有可被访问的内容：

```kotlin
import foo.* // everything in 'foo' becomes accessible
```

如果有名称冲突，可以使用 `as` 来消除歧义：

```kotlin
import foo.Bar // Bar is accessible
import bar.Bar as bBar // bBar stands for `bar.Bar`
```

`import` 关键字并不仅限于导入类；还可以导入下面的声明：

- 顶层函数和属性；
- 对象声明中定义的函数和属性
- 枚举常量

与 Java 不同，Kotlin 没有一个额外的 `import static` 预发；所有的声明都是由常规的 `import` 来导入。

顶层声明的可见性
---
标记为 `private` 的顶层声明只属于它所在的文件。

