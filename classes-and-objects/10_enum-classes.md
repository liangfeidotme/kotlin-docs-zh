# 枚举类
枚举类最基本的用法是实现类型安全的枚举：

```kotlin
enum class Direction {
    NORTH, SOUTH, WEST, EAST
}
```
每一个枚举常量都是一个对象。枚举常量之间通过逗号分隔。

## 初始化
因为每个枚举都是一个枚举类的实例，他们可以按照如下方式初始化：

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
可以定义它们对应的方法，也可以覆写基类方法。注意，如果枚举类定义了任何成员，你需要用分号把枚举常量的定义从成员的定义中隔离开，跟 Java 一样。

枚举条目不能包含除了内部类之外的嵌套类型（在 Kotlin 1.2 版本中废弃了）。

## 使用枚举常量
与 Java 一样，Kotlin 的枚举类提供了合成的（synthetic）方法，可以列出所有定义的枚举常量以及根据名字获取枚举常量。这些方法的签名如下（假设枚举类的名字是 `EnumClass`）：

```kotlin
EnumClass.valueOf(value: String): EnumClass
EnumClass.values(): Array<EnumClass>
```

如果指定的名字没有匹配到任何一个类中所定义的枚举常量，那么 `valueOf()` 方法会抛出 `IllegalArgumentException` 的异常。

从 Kotlin 1.1 开始，可以以泛型方式来访问枚举类中的常量，使用 `enumValues<T>` 和 `enumValueOf<T>()` 函数：

```kotlin
enum class RGB { RED, GREEN, BLUE }

inline fun <reified T : Enum<T>> printAllValues() {
    print(enumValues<T>().joinToString { it.name })
}

printAllValues<RGB>()   // prints RED, GREEN, BLUE
```

每一个枚举常量都有属性来获取它的名字以及在枚举类声明中的位置：

```kotlin
val name: String
val ordinal: Int
```

枚举类常量也实现了 `Comparable` 接口，排序的顺序是它们在枚举类中定义的顺序。
