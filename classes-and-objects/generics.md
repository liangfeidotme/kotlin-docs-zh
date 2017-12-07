泛型
===

与 Java 一样，Kotlin 的类可以有类型参数（type parameter）：

```kotlin
class Box<T>(t: T) {
    var value = t
}
```

通常，为了初始化这个类，我们需要提供类型参数（type argument）：

```kotlin
val box: Box<Int> = Box<Int>(1)
```

如果参数可以被推导出（例如，通过构造器的入参或者其他手段），那么类型就可以省略掉。

```kotlin
val box = Box(1) // 1 has type Int, so the compiler figures out that we are talking about Box<Int>
```


