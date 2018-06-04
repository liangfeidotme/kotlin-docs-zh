# 泛型
与 Java 一样，Kotlin 的类也可以拥有类型参数（parameter）：

```kotlin
class Box<T>(t: T) {
    var value = t
}
```

通常，为了创建这个类的实例，我们需要提供类型实参（argument）：

```kotlin
val box: Box<Int> = Box<Int>(1)
```

如果参数可以被推导出，例如，通过构造器的实参或者利用其他手段，那么类型实参就可以省略掉。

```kotlin
val box = Box(1) // 1 has type Int, so the compiler figures out that we are talking about Box<Int>
```

## 变型
在 Java 的类型系统中，最难搞的部分之一是通配符类型（wildcard type）。但是 Kotlin 没有任何通配符类型。相反，它有另外两个东西：声明点变型（declaration-site variance）和类型投影（type projection）。

我们先来思考一下为什么 Java 需要那些蜜汁通配符。Effective Java （第三版）解释了这个问题，第 28 条: *利用有限制通配符来提升API的灵活性*。首先，Java 的泛型是不可变的（invariant），这就意味着 `List<String>` 并**不是** `List<Object>` 的子类型。为什么会这样呢？如果 List 不是不可变的，那相比于数组就不会有任何优势，而且下面代码也会编译通过，但是运行时会引起异常：

```Java
// Java
List<String> strs = new ArrayList<String>();
List<Object> objs = strs; // !!! The cause of the upcoming problem sits here. Java prohibits this!
objs.add(1);    // Here we put an Integer into a list of Strings
String s= strs.get(0);  // !!! ClassCastException: Cannot cast Integer to String
```

因此，为了保证运行时安全，Java 直接禁止这么做。但是这么做也会带来其它影响。例如，想一下 `Collection` 接口的 `addAll()` 方法，它的签名是什么？符合直觉的写法如下：

```Java
// Java
interface Collection<E> ... {
    void addAll(Collection<E> items);
}
```

如果这么写，我们就无法做到下面这种简单的操作（它是绝对安全的）：

```java
// Java
void copyAll(Collection<Object> to, Collection<String> from) {
    to.addAll(from);    // !!! Would not compile with the naive declaration of addAll:
                        // Collection<String> is not a subtype of Collection<Object>
}
```

（在 Java 中，我们的教训比较深刻，参考 Effective Java，第三版, 第 25 条: *列表优先于数组*）

这也是为什么下面才是 `addAll()` 的正确签名：

```Java
interface Collection<E> ... {
    void addAll(Collection<? extends E> items);
}
```

**通配符类型实参**（wildcard type argument）`? extends E` 表示这个方法接收 `E` *或者它的某个子类型*的对象集合，并非只是 `E` 自身。这就意味着我们可以从 items 中读取 `E` 类型变量（这个集合的元素都是 E 的子类的实例），但是**并不能写入**，因为我们不知道 `E` 的子类的具体类型，写入的对象有可能跟类型并不匹配。虽然有这个限制，但是我们可以通过其他途径实现我们想要的行为：`Collection<String>` *是* `Collection<? extends Object>` 的一个子类型。用“聪明的话（clever words）”来说，带有 **extends**-限制（**上部**限制）的通配符使类型变为**协变的（covariant）**。

搞懂这个技巧是如何工作的其实很简单：如果你只能从一个集合中**取出**元素，那么使用 `String` 的集合，读取 `Object` 变量。相反，如果你只能往集合中**存放**元素，使用 `Object` 集合，放入 `String` 变量：在 Java 中，我们可以给 `List<? super String>` 找到一个 **超类型**（supertype） —— `List<Object>`。

后者称为**逆变性（contravariance）**，只能调用 `List<? super String>` 中参数为 `String` 的方法（例如，`add(String)` 或者 `set(int, String)`），如果 `List<T>` 的调用返回 `T` 值，那么不会得到 `String`， 而是 `Object`。

[Joshua Bloch](https://en.wikipedia.org/wiki/Joshua_Bloch) 把那些只读的对象称为**生产者**（Producer），只写的称为**消费者**（Consumer）。他建议：“*为了达到最大的灵活性，在表示生产者和消费者的输入参数上要使用通配符类型*”，同时提出了如下便于记忆的方法：

> PECS stands for Producer-Extends, Consumer-Super

***注意***：如果使用了生产者对象，例如，`List<? extends Foo>`，`add()` 或者 `set()` 方法不允许调用，但是这样并不意味着这个对象是**不可变的（immutable）**：例如，依然可以调用 `clear()` 来清空 list 中的所有元素，因为 `clear()` 没有任何参数。通配符（或者其他类型的变型）唯一能保证的是**类型安全**。不可变性（immutability）是一个完全不同的领域。

### 声明点变型
假设我们有一个泛型接口 `Source<T>`，它没有任何参数为 `T` 的方法，只有返回值为 `T` 的方法：

```java
// Java
interface Source<T> {
    T nextT();
}
```

然后，用类型为 `Source<Object>` 的变量来保存 `Source<String>` 实例的引用是非常安全的，因为没有消费者方法可调。但是 Java 并不知道这一点，依然会禁止这么做：

```java
// Java
void demo(Source<String> strs) {
    Source<Object> objects = strs;  // !!! Not allowed in Java
    // ...
}
```

为了解决这个问题，我们必须声明类型为 `Source<? extends Object>` 的对象，这么做其实没什么意义，因为我们依然可以调用这个变量上所有跟之前一样的方法，所以这个更复杂的类型并没有带来什么价值。但是编译器并不知道这些。

在 Kotlin 中有一个方式可以向编译器解释这种事情。叫做**声明点变型**（declaration-site variance）：我们可以通过标注 `Source` 的类型参数 `T` 来确保它只会被 `Source<T>` 的成员**返回**（生产），绝不会被消费。为了做到这一点，Kotlin 提供了 `out` 修饰符：

```kotlin
interface Source<out T> {
    fun nextT(): T
}

fun demo(strs: Source<String>) {
    val objects: Source<Any> = strs // This is OK, since T is an out-parameter
    // ...
}
```

通用规则是：当一个类 `C` 的类型参数 `T` 被声明为 **out** 时，它在 `C` 的成员中只能出现于 **out** 的位置，但是作为交换，`C<Base>` 可以安全地成为 `C<Deribed>` 的超类型。

用“聪明的话”来说，类 `C` 在参数 `T` 上是**协变的**（covariant），或者说，`T` 是一个**协变的**类型参数。可以认为 `C` 是一个 `T` 类型值的**生产者**，而**不是** `T` 类型值的**消费者**。

这个 **out** 修饰符被称作**变型注解**，因为它出现在类型参数的声明处，我们所讨论的是**声明点变型**。这一点和 Java 的**使用点变型**恰恰相反，因为只有在使用类型时所用的通配符才使得类型成为协变的。

除了**out**，Kotlin 还提供了一个互补的变型注解：`in`。它可以使类型参数变为**逆变的**：只能被消费，不能被生产。关于逆变类型的一个好例子是 `Comparable`：

```kotlin
interface Comparable<in T> {
    operator fun compareTo(other: T): Int
}

fun demo(x: Comparable<Number>) {
    x.compareTo(1.0) // 1.0 has type Double, which is a subtype of Number
                     // Thus, we can assign x to a variable of type Comparable<Double>
    val y: Comparable<Double> = x // OK!
}
```

我们相信单词 `in` 和 `out` 是自解释的（它们已经在 C# 中成功的使用了相当一段时间），因此上面提到的那种便于记忆的方式并不需要了，我们可以为了更高的目标把它改成：

**[存在主义](https://zh.wikipedia.org/wiki/%E5%AD%98%E5%9C%A8%E4%B8%BB%E4%B9%89)变形：消费者进来，生产者出去！**:-)

## 类型投影
### 使用点变型：类型投影

把类型参数 `T` 声明为 *out* 能够避免在使用点利用子类型时所带来的麻烦，这么做会比较方便，但是某些类**不能够**被限制成只返回 `T` 类型的值。Array 是一个很好的例子：

```kotlin
class Array<T>(val size: Int) {
    fun get(index: Int): T { /* ... */ }
    fun set(index: Int, value: T) { /* ... */ }
}
```

这个类与 `T` 之间既不是协变也不是逆变。如此一来就强加了某些不灵活性。看一下如下函数：

```kotlin
fun copy(from: Array<Any>, to: Array<Any>) {
    assert(from.size == to.size)
    for (i in from.indices)
        to[i] = from[i]
}
```

这个函数的期望功能是把一个数组的元素拷贝到另一个数组。我们尝试用一下：

```kotlin
val ints: Array<Int> = arrayOf(1, 2, 3)
val any = Array<Any>(3) { "" }
copy(ints, any) // Error: expects (Array<Any>, Array<Any>)
```

于是我们就遇到了同一个熟悉的问题：`Array<T>` 在 `T` 上是不可变的（invariant），因此 `Array<Int>` 和 `Array<Any>` 都不是彼此的子类型。为什么呢？再说一次，因为 `copy` **可能** 会干坏事，例如，它会尝试**写入** `from`，假设是一个字符串。如果我们实际上传入了一个 `Int` 数组，稍后就会抛出一个 `ClassCastException`。

然后，我们唯一想确保的就是 `copy` 不要做任何坏事。我们可以禁止它往 `from` 里写入，那么就可以这么做：

```kotlin
fun copy(from: Array<out Any>, to: Array<Any>) {
  // ...
}
```

这里所发生的事被称作**类型投影**（type projection）：我们说过 `from` 不是简单的一个数组，而是一个受限的（**投影的**）的数组：我们只能调用那些返回值类型为 `T` 的方法，这个 case 意味着我们只能调用 `get()`。这是**使用点变型**的实现方法，对应于 Java 的 `Array<? extends Object>`，但是方式更简单。

同样的，也可以用 `in` 来投影一个类型：

```kotlin
fun fill(dest: Array<in String>, value: String) {
    // ...
}
```

`Array<in String>` 对应 Java 的 `Array<? super String>`，也就是说，你可以传入一个 `CharSequence` 或 `Object` 的数组给 `fill()` 函数。

### 星投影（Star-projection）
有时候我们可能对类型实参一无所知，但是仍然想以一种安全的方式使用它。这里的安全方式就是给泛型类型定义这么一个投影，然后泛型类型的每一个具体实例都是投影的子类型。

Kotlin 为此提供了一个所谓的**星投影**语法：

* 对于 `Foo<out T : TUpper>`，`T` 是一个协变的类型参数，上限是 `TUpper`，`Foo<*>` 就等价于 `Foo<out TUpper>`。它意味着，当 `T` 不可知时，你可以安全地从 `Foo<*>` 中*读取* `TUpper` 类型的值。
* 对于 `Foo<in T>`，`T` 是一个逆变的类型参数，`Foo<*>` 等价于 `Foo<in Nothing>`。它意味着，当 `T` 不可知时，没有任何东西可以*写入* `Foo<*>`。
* 对于 `Foo<T : TUpper>`，`T` 是一个可变类型参数，上限是 `TUpper`，`Foo<*>` 如果用于读值，其等价于 `Foo<out TUpper>`，如果写值，其等价于 `Foo<in Nothing>`。

如果一个泛型类型有多个类型参数，它们中的每一个都能够被独立投影。例如，如果类型声明为 `interface Function<in T, out U>`，我们可以想象出如下的星投影：

* `Function<*, String>` 意思是 `Function<in Nothing, String>`；
* `Function<Int, *>` 意思是 `Function<Int, out Any?>`;
* `Function<*, *>` 意思是 `Function<in Nothing, out Any?>`。

> 注意：星投影很类似于 Java 的原始类型，但是更安全。
> 例如，List 是一个**原始类型**（raw type），List<String> 则是一个**参数化的类型**（parameterized type）

## 泛型函数
不只是类可以拥有类型参数。函数也可以。类型参数位于函数名之前：

```kotlin
fun <T> singletonList(item: T): List<T> {
    // ...
}

fun <T> T.basicToString(): String { // extension function
    // ...
}
```

如果要调用泛型函数，需要在调用处指明类型实参，位置在函数名**之后**。

```kotlin
val l = singletonList<Int>(1)
```

如果能从上下文推导出的话，类型实参也可以省略，下例可以正常运行：

```kotlin
val l = singletonList(1)
```

## 泛型约束
所有可能被指定类型参数替换掉的类型集合可以通过**泛型约束**加以限制。

### 上限
最普通的约束类型是**上限**，对应 Java 的 `extends` 关键字：

```kotlin
fun <T: Comparable<T>> sort(list: List<T>) {
    // ...
}
```

冒号之后指定的类型就是**上限**：只能是 `Comparable<T>` 的子类，可以取代 `T`。例如：

```kotlin
sort(listOf(1, 2, 3)) // OK. Int is subtype of Comparable<Int>
sort(listOf(HashMap<Int, String>())) // Error: HashMap<Int, String> is not a subtype of Comparable<HashMap<Int, String>>
```

默认的上限（如果没有指定）是 `Any?`。尖括号内只能指定一个上限。如果同一个类型需要多个上限，那么还需要一个额外的 **where** 语句：

```kotlin
fun <T> copyWhenGreater(list: List<T>, threshold: T): 
List<String 
    where T: CharSequence,
          T: Comparable<T> {
    return list.filter( it > threshold }.map { it.toString() }
}
```

## 类型擦除
Kotlin 为泛型声明的用法所执行的类型安全检查只在编译期时完成。在运行时，泛型类型的实例不会携带有关实际类型实参的任何信息。类型信息可以说被*擦除*了。例如，`Foo<Bar>` 和 `Foo<Baz?>` 被擦除之后的结果只是 `Foo<*>`。

因此，没有一种通用的方式能够在运行时去检测一个泛型实例的创建是否使用了某个类型实参，并且编译器也会禁止这种 is-check。

类型转换时，如果转换成携带具体类型实参的泛型类型，例如 `foo as List<String>`，不能在运行时被检查。
当类型安全隐含于高级的程序逻辑时，可以使用这些未检查的转换，但是编译器并不能直接推导出，而且编译器会给未检查的转换报一个警告。运行时只会检查非泛型的部分（等价于 `foo as List<*>`）。

泛型函数的类型实参也是只在编译时检查。在函数体内，类型形参不能用于类型检查，并且转换成类型实参（`foo as T`）的操作也不会被检查。但是，在调用处，内联函数的具体化类型参数会替换成函数体内的真实类型实参，因此可用于类型检查和转换，跟上述泛型类型的实例有同样的限制。

---

**术语约定**

* variance：变型
* declaration-site variance：声明点变型
* use-site variance：使用点变型

* invariant：不可变的
* covariant：协变的
* contravariant：逆变的

* type parameter：类型参数（形参）
* type argument：类型实参

---

关于变形修饰符（variance modifider）的解释可参考 [Kotlin generic variance modifiers](https://blog.kotlin-academy.com/kotlin-generics-variance-modifiers-36b82c7caa39)：

> When a generic type is invariant, like class `Box<T>`, there is no relation between any `Box<SomeType>` and `Box<AnotherType>`. So there is no relation between `Box<Number>` and `Box<Int>`.

> When a generic type is covariant, like class `Box<out T>`, when A is a subtype of B then `Box<A>` is a subtype of `Box<B>`. So `Box<Int>` is a subtype of `Box<Number>`.

> When a generic type is contravariant, like class `Box<in T>`, when A is a subtype of B then `Box<B>` is a subtype of `Box<A>`. So `Box<Number>` is a subtype of `Box<Int>`.
