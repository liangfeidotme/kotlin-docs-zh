# 与 Java 的比较

## Kotlin 中已解决的一些 Java 问题

Kotlin 修复了一些列 Java 饱受其苦的问题：

- 空引用由类型系统来控制
- 无原始类型
- 数组在 Kotlin 中是协变的
- Kotlin 有特有的（proper）函数类型，与 Java 的 [SAM 转换][SAM-conversion] 截然相反
- 无通配符的使用处变形（use-site variance）
- Kotlin 没有检查型异常（checked exception）

## Java 有 Kotlin 没有的

- 检查型异常
- 不是类的基础类型
- 静态成员
- 非私有字段
- 通配符类型
- 三元操作符 a ? b : c

## Kotlin 有 Java 没有的
- lambda 表达式 + 内联函数 = 高效的自定义控制结构
- 扩展函数
- Null 安全
- 智能转换
- 字符串模板
- 属性
- 首要构造器
- 头等代理？
- 变量和属性的类型推到
- 单例
- 声明处变形 & 类型映射
- 范围表达式
- 操作符重载
- 伴生对象
- 数据类
- 只读和可变集合的独立接口
- 协程

[SAM-conversion]:https://stackoverflow.com/questions/17913409/what-is-a-sam-type-in-java?utm_medium=organic&utm_source=google_rich_qa&utm_campaign=google_rich_qa
