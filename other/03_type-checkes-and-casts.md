类型检查和转换：`is` 和 `as`
===

`is` 和 `!is` 操作符
---

在运行时可以通过 `is` 或者它的否定形 `!is` 来判断一个对象是否是某个类型：

```kotlin
if (obj is String) {
    print(obj.length)
}

if (obj !is String) { // same as !(obj is String)
    print("Not a String")
} else {
    print(obj.length)
}
```

只能转换
---
大多数情况下，我们不会用到显示转换操作符，因为编译器会追踪 `is` 检查以及不可变值的显示转换，然后在必要时自动插入（安全）转换代码：

```kotlin
fun demo(x: Any) {
    if (x is String) {
        print(x.length) // x is automatically cast to String
    }
}
```

编译器很聪明，如果否定检查导致了 return 
