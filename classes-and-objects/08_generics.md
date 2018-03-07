# 泛型

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

## 变异

Java 的类型系统当中，最难搞部分之一是通配符类型。然而 Kotlin 一点都没有。相反，它有另外两个东西：声明点变异（declaration-site variance）和类型映射（type projection）。

我们先来思考一下为什么 Java 需要那些神奇的通配符。Effective Java 对此作出了解释，Item 28: *Use bounded wildcards to increate API flexibility*。首先，Java 的泛型是不可变的，这就意味着 `List<String>` 并**不是** `List<Object>` 的子类型。为什么会这样呢？如果 List 不是不可变，那么它不会比数据更好用，如下代码会编译通过并且在运行时引起异常：

```Java
// Java
List<String> strs = new ArrayList<String>();
List<Object> objs = strs; // !!! The cause of the upcoming problem sits here. Java prohibits this!
objs.add(1);    // Here we put an Integer into a list of Strings
String s= strs.get(0);  // !!! ClassCastException: Cannot cast Integer to String
```

因此，为了保证运行时安全，Java 禁止这么做。但是这么做会带来其他影响。例如，`Collection` 接口的 `addAll()` 方法，它的签名是什么？符合直觉的话，我们会写成如下形式：

```Java
// Java
interface Collection<E> ... {
    void addAll(Collection<E> items);
}
```

如果这么写，我们就无法完成下面这种简单操作（它是绝对安全的）：

```java
// Java
void copyAll(Collection<Object> to, Collection<String> from) {
    to.addAll(from);    // !!! Would not compile with the naive declaration of addAll:
                        // Collection<String> is not a subtype of Collection<Object>
}
```

（在 Java 中，我们的教训比较深刻，参考 Effective Java, Item 25: *Prefer lists to arrays）

这也是为什么下面才是 `addAll()` 的正确签名：

```Java
interface Collection<E> ... {
    void addAll(Collection<? extends E> items);
}
```

**通配符类型参数（wildcard type argument）**`? extends E` 表示这个方法接收 `E` 或者它的子类型的对象集合，并非只是 `E` 自身。这就意味着我们可以从 items 中读取 `E` 类型变量（这个集合的元素都是 E 的子类的实例），但是**并不能写入**，因为我们不知道 `E` 的子类的具体类型，写入的对象有可能跟类型并不匹配。虽然有这个限制，但是我们可以通过其他途径实现我们想要的行为：`Collection<String>` 是 `Collection<? extends Object>` 的子类。用“聪明的话”来说，带有 **extends**-bound（**upper** bound）把类型变为**协变的（covariant）**。

搞懂这个 trick 的关键点很简单：如果你只能从一个集合中**取出**元素，那么使用 `String` 的集合，读取 `Object` 变量。想法，如果你只能往集合中**存放**元素，使用 `Object` 集合，放入 `String` 变量：在 Java 中，我们可以给 `List<? super String>` 找到一个 **supertype** —— `List<Object>`。

后者称为**逆变性（contravariance）**，只能调用 `List<? super String>` 中参数为 `String` 的方法（例如，`add(String)` 或者 `set(int, String)`），如果 `List<T>` 的调用返回 `T`，不会得到 `String`， 而是一个 `Object`。

[Joshua Bloch](https://en.wikipedia.org/wiki/Joshua_Bloch) 把那些只读的对象称为 **Producer**，只写的称为 **Consumer**。他建议：“为了达到最大的灵活性，在表示生产者和消费者的输入参数上使用通配符类型”，同时提出了如下便于记忆的方法：

> PECS stands for Producer-Extends, Consumer-Super

注意：如果使用了生产生产者对象，例如，`List<? extends Foo>`，`add()` 或者 `set()` 方法不允许调用，但是这样并不意味着这个对象是**不可变的（immutable）**：例如，依然可以调用 `clear()` 来清空 list 中的所有 item，因为 `clear()` 没有任何参数。通配符（或者其他异变类型）唯一能保证的是**类型安全（type safety）**。不可变性（immutability）是一个完全不同的领域。

### 声明点异变

假设我们有一个泛型接口 `Source<T>`，它没有任何参数类型为 `T` 的方法，只有返回值为 `T` 的方法：

```java
// Java
interface Source<T> {
    T nextT();
}
```

然后，用一个类型为 `Source<Object>` 的变量保存 `Source<String>` 实例的引用是非常安全的，没有消费者方法可调。但是 Java 并不知道这一点，依然会禁止这么做：

```java
// Java
void demo(Source<String> strs) {
    Source<Object> objects = strs;  // !!! Not allowed in Java
    // ...
}
```
