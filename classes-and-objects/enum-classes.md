# 枚举类

枚举类最基本的用法是实现类型安全的枚举：

```kotlin
enum class Direction {
    NORTH, SOUTH, WEST, EAST
}
```
每一个枚举常量都是一个对象。枚举常量之间通过逗号分隔。

## 初始化

因为没一个枚举是一个枚举类的实例，他们可以按照如下方式初始化：

```kotlin
enum class Color(val rgb: Int) {
    RED(0xFF0000),
    GREEN(0x00FF00),
    BLUE(0x0000FF)
}
```

## 匿名类
枚举常量也可以声明他们自己的匿名类：

```kotlin
enum class ProtocolState {
    WAITING {
        override fun signal() = TALKING
    },

    TALKING {
        override fun signal() = WAITING
    };

    abstract fun signal(): ProtocolState
}
```


