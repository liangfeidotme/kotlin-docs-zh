Kotlin 标准库
===

标准库提供了 Kotlin 持续更新精华。它们包含：

- 高阶函数，实现了符合语言设计的模式（let / apply / use / synchronized 等）
- 扩展函数，提供集合（eager）和序列（lazy）的查询操作
- 各钟工具，方便操作字符串和字符序列
- JDK 扩展，方便操作文件、IO 和线程

包
---
kotlin | 函数函数和类型，可用于所有支持的平台
kotlin.annotation | 标记功能的支持库
kotlin.browser *js* | 在浏览器环境中可访问最顶层属性（`document`、`window` 等）
kotlin.collections | 集合类型，例如 iterable、collection、list、set、map 以及相关的最顶层以及扩展函数
kotlin.comparisons | 创建 Comparator 实例的帮助函数
kotlin.concurrent *JVM* | 并发编程的工具函数
kotlin.coroutines.experimental *1.1* | 协程的支持库，包含 lazy sequences
kotlin.coroutines.experimental.intrinsics *1.1* | 为基于协程的 API 库提供底层的构建模块
kotlin.dom *js* | 操作浏览器 DOM 的工具函数
kotlin.experimental *1.1* | 实验性 API，可随着 Kotlin 版本发生改变
kotlin.io | 操作文件和流的 IO API
kotlin.js *js* | JavaScript 平台特定的函数和其他 API
kotlin.jvm *JVM* | Java 平台特定的函数和标记
kotlin.math *1.2* | 数学函数和常量
kotlin.properties | 代理和代理属性的标准实现以及实现自定义代理的帮助函数
kotlin.ranges | 范围、进度以及相关的最顶层和扩展函数
kotlin.reflect | Kotlin 反射的运行时 API
kotlin.reflect.full *JVM*/*1.1| 反射扩展，由 `kotlin-reflect` 库提供
kotlin.reflect.jvm *JVM* | Kotlin 和 Java 反射互操作的运行时 API，由 `kotlin-reflect` 提供
kotlin.sequences | `Sequence` 类型，用于表示懒计算的 collections。用于初始化序列的最顶层函数以及扩展函数。
kotlin.streams *JVM*/*1.2*/*JRE8* | Java8 streams 相关的工具函数
kotlin.systems *JVM* | 系统相关的工具函数
kotlin.text | 处理文本和正则表达式的函数
org.khronos.webgl *js*  | JS WebGL API 的 kotlin 封装
org.w3c.dom *js*  | js DOM API 的 kotlin 封装
org.w3c.dom.css *js*  | js DOM CSS API 的 kotlin 封装
org.w3c.dom.events *js*  | js DOM events API 的 kotlin 封装
org.w3c.dom.parsing *js*  | js DOM parsing API 的 kotlin 封装
org.w3c.dom.svg *js*  | js DOM SVG API 的 kotlin 封装
org.w3c.dom.url *js*  | js DOM URL API 的 kotlin 封装
org.w3c.fetch *js*  | js W3C fetch API 的 kotlin 封装
org.w3c.files *js*  | js W3C file API 的 kotlin 封装
org.w3c.notifications *js*  | js Web Notifications API 的 kotlin 封装
org.w3c.performance *js*  | js Navigation Timing API 的 kotlin 封装
org.w3c.workers *js* | js Web Workers API 的 kotlin 封装
org.w3c.xhr *js* | js XMLHttpRequest API 的 kotlin 封装
