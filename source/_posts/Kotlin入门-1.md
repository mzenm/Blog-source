title: Kotlin基础
author: 卫振家
tags: []
categories:
  - Kotlin
date: 2017-05-24 20:18:00
---
# Kotlin 基本语法介绍

## 基本数据类型

在Kotlin中，所有的东西都是一个对象，所以我们可以在任何变量上调用成员函数和属性。有一些类型是内置的，因为他们的实现经过了优化。但对于用户来说，他们看起来就像普通的类。在本节主要描述这些内置类型:numbers、booleans、arrays;

### Numbers

Kotlin适用于Java相似的方式来操作数字，但是并不完全一样。比如说，不存在针对数字的隐式扩展转换，有些字面上 也不太一样。Kotlin提供以下内置类型表示数字

| Type   | Bit width |
| ------ | --------- |
| Double | 64        |
| Float  | 32        |
| Long   | 64        |
| Int    | 32        |
| Short  | 16        |
| Byte   | 8         |

注意在Kotlin中characters 类型并不属于数字类型。

#### 字符常量

对于整形有以下类型的字符常量

- 十进制：`123`
  - Long型使用大写`L`标记：`123L`
- 16进制：`0x0F`
- 二进制：`0b00001011`

Note:不支持8进制

Kotlin对浮点数也有方便的定义方式

- 默认的Double: `123.5`、`123.5e10`
- Float型使用`f`或`F`标记：`123.5f`

#### 在数字表示中使用下划线(从1.1开始)

你可以使用下划线以确保数字变量具有更好的可读性

```kotlin
val oneMillion = 1_000_000
val creditCardNumber = 1234_5678_9012_3456L
val socialSecurityNumber = 999_99_9999L
val hexBytes = 0xFF_EC_DE_5E
val bytes = 0b11010010_01101001_10010100_10010010
```

#### 表示-包装

在Java平台上，数字类型实际上被存储为JVM的基本数据类型，除非我们需要一个可以为null的数字引用或者作为泛型引用。在后面这种情况下，数字就会被包装。

注意，数字类型的包装不必保护一致性。

```kotlin
val a: Int = 10000
print(a === a) // Prints 'true'
val boxedA: Int? = a
val anotherBoxedA: Int? = a
print(boxedA === anotherBoxedA) // !!!Prints 'false'!!!
```

但是，另一方面包装保护对等性。

```kotlin
val a: Int = 10000
print(a == a) // Prints 'true'
val boxedA: Int? = a
val anotherBoxedA: Int? = a
print(boxedA == anotherBoxedA) // Prints 'true'
```

#### 显式转换

由于不同的表现形式，小类型不是大类型的子类型。如果如，我们会在下列适用中遇到问题。

```kotlin
//虚构代码，不要实际运行:
val a: Int? = 1 // 一个包装int型 (Java.lang.Integer)
val b: Long? = a // 隐式的转化为Long项 (Java.lang.Long)
print(a == b) // 打印"false"因为Long.equals()的方法也会检测long数据的其他部分
```

所以，不只是一致性，甚至对等性也静静的消失掉了。

​	结论是，小类型`不能`隐式的转换为大类型。这就意味着，我们`只有`在使用显式转换的情况在，可以把一个Byte型的值赋值给Int型的变量:

```kotlin
val b: Byte = 1 // OK, literals are checked statically
val i: Int = b // ERROR
```

我们可以使用`显式`转换为更宽类型的数字

```kotlin
val i: Int = b.toInt() // OK: explicitly widened
```

每种数字类型都支持以下方式的转换操作

- `toByte(): Byte`
- `toShort(): Short`
- `toInt(): Int`
- `toLong(): Long`
- `toFloat(): Float`
- `toDouble: Double`
- `toChar(): Char`

隐式的类型转换都很少被注意到，因为这种类型都是通过上下文推断得出，而在算数运算中类型也不自动被恰当的重载。

```kotlin
val l = 1L + 3 // Long + Int => Long
```

#### 运算

Kotlin支持数字中标准的算数运算集合。这些运算集合被定义为对应类的成员发膜护发。但是编译器优化这些调用为对应的指定。

对于按位运算，并没有提供特殊的运算符，但是提供了内嵌格式的函数来达到相似的调用效果，比如说：

```kotlin
val x = (1 shl 2) and 0x000FF000
```

一下是位运算的完整类型（仅对Int和Long型可用）

- `shl(bits)` – signed shift left (Java's `<<`)
- `shl(bits)` – signed shift left (Java's `<<`)
- `ushr(bits)` – unsigned shift right (Java's `>>>`)
- `and(bits)` – bitwise and
- `or(bits)` – bitwise or
- `xor(bits)` – bitwise xor
- `inv()` – bitwise inversion

### Characters

字符使用`类型Char`表示。字符型不能直接当做数字类型对待

```kotlin
fun check(c: Char) {
    if (c == 1) { // ERROR: incompatible types
        // ...
    }
}
```

字符类型通过单引号`'`进行字面表示：`'1'`。特殊的字符可以使用反斜杠进行转义。支持下列转义顺序:`\t`, `\b`, `\n`, `\r`, `\'`, `\"`, `\\`和 `\$`。为了转码其他字符，可以使用Unicode转义序列语法：`'\uFF00'`。

我们可以显式把一个字符转化为`Int`型数字。

```kotlin
fun decimalDigitValue(c: Char): Int {
    if (c !in '0'..'9')
        throw IllegalArgumentException("Out of range")
    return c.toInt() - '0'.toInt() // Explicit conversions to numbers
}
```

跟数字 一样，需要时字符型也可以包装为一个可null的引用。在包装运算中不保证一致性。

### Boolean

`Boolean`类型表示布尔值，布尔值只有两个:`true`和`false`;

需要时布尔值也会被包装为一个可null得引用

内置的布尔运算符包括：

- `||` – lazy disjunction
- `&&` – lazy conjunction
- `!` - negation

### Array

Kotlin中的数组用`类Array`来标示。`Array`类包含`get`和`set`函数（通过运算符重载协议转化为[]）和`size`属性。除此之外还有其他一些有用的成员函数：

```kotlin
class Array<T> private constructor() {
    val size: Int
    operator fun get(index: Int): T
    operator fun set(index: Int, value: T): Unit

    operator fun iterator(): Iterator<T>
    // ...
}
```

为了创建一个数组，我们可以使用库函数`arrayOf()`，并向该函数传递一列值，这样`arrayOf(1, 2, 3)`就创建一个一个数组`[1, 2, 3]`。另外也可以使用`arrayOfNulls()`库函数创建填充了指定数量Null元素的数组。

另一个选择是使用工厂方法，接受一个数组大小参数和一个可以根据指针返回对应数组元素初始化值得函数。

```kotlin
// Creates an Array<String> with values ["0", "1", "4", "9", "16"]
val asc = Array(5, { i -> (i * i).toString() })
```

就如我上面所说，`[]`操作符表示调用成员函数`get()`和`set()`;

注意：与Java不同，Kotlin的数组是不可变的。这意味着在Kotlin中我们不能将`Array<String>`赋值给`Array<Any>`,这可能会导致一个运行时错误。但是我们可以使用`Array<out Any>`。细节可以查看[类型投影](https://kotlinlang.org/docs/reference/generics.html#type-projections)。

Kotlin也提供了特殊的类以在不需要额外包装开销的情况下标示基础类型数组：`ByteArray`,`ShortArray`, `IntArray` 等。这些类与`Array`类不存在继承关系，但是具有与`Array`类相同的方法和属性集合。每个基础类数组也都有对应的工厂函数:

```kotlin
val x: IntArray = intArrayOf(1, 2, 3)
x[0] = x[1] + x[2]
```

### Strings

字符串使用`类型String`表示。字符串是不可变的。字符串的每一个元素都是一个字符，我们可以通过索引运算符`$[i]`来获取这些字符。一个字符串可以用for-loop结构来遍历字符。

```kotlin
for (c in str) {
    println(c)
}
```

#### 字符串的语言标示

Kotlin有两种字符串的字面值：可能包含转义字符的转义字符串和包含换行和任意文本的原生字符串。转义字符串与Java字符串非常相似

```kotlin
val s = "Hello, world!\n"
```

通过简单的`\`就能完成转义。可以查看上面关于字节的描述查看支持的转义序列。

原生字符串使用三个双引号`"""`来限定，包含不进行转义的字符，且可以包含换行符和其他任何字符。

```kotlin
val text = """
    for (c in "foo")
        print(c)
"""
```

你可以使用 [`trimMargin()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.text/trim-margin.html) 函数来移除主要的空白符。

```kotlin
val text = """
    |Tell me and I forget.
    |Teach me and I remember.
    |Involve me and I learn.
    |(Benjamin Franklin)
    """.trimMargin()
```

默认上我们用`|`作为边界前缀，但是你可选择其他字符作为前缀，并作为参数传递，比如`trimMargin(">")`.

#### 字符串模板

字符串可以包含模板表达式。比如一些可以求值的程序片段，他们的结果被嵌入到目标字符串中。一个模板表达式以一个`$`符开始，后面要么跟随一个简单变量名

```kotlin
val i = 10
val s = "i = $i" // evaluates to "i = 10"
```

要么跟随一个`{}`包裹的任意表达式

```kotlin
val s = "abc"
val str = "$s.length is ${s.length}" // evaluates to "abc.length is 3"
```

字符串末班在原声字符串和转移字符串中一样支持。如果你需要在一个原生字符产表示字符`$`(因为原生字符串不支持转义)。你可以使用以下的语法结构。

```kotlin
val price = """
${'$'}9.99
"""
```

----

## Package

一个原文源文件可以以包声明开始。

```kotlin
#创建一个Kt文件
package basic.kotlin

//变量声明
var a:Int = 100;

//函数声明
fun baz():Boolean {
	return true
}
//定义类
class Goo {
	var a:DataProviderManager = DataProviderManager
	var b:A = A.B;
	fun test():Int{
		return 1;
	}
	class Doo{
	}
}

//枚举类
enum class A {
    B,C,D,E;
}

//一种单例声明格式
object DataProviderManager {
    var a:String = "test";
    fun c(){}
}
```

这样，这个源文件中的所有内容都被包含在哪个被生命的包中。这样，在上面的例子中，函数`baz()`的全名应该是`basic.kotlin.baz`，类`Goo`的全名是`basic.kotlin.Goo`;

如果未指定包名，这样一个文件的上下文都归属于一个没有名字的默认包。

### 默认注入

在任意一个Kotlin文件中，有一些包都会被默认注入：

- kotlin.*
- kotlin.annotation.*
- kotlin.collections.*
- kotlin.comparisons.*  (since 1.1)
- kotlin.io.*
- kotlin.ranges.*
- kotlin.sequences.*
- kotlin.text.*

根据目标平台，还回引入其他额外的依赖包：

- JVM:    
  - java.lang.*
  - kotlin.jvm.*
- JS:    
  - kotlin.js.*

### 注入

除了默认注入外，每一个文件还可以包含自己的注入指令。主语的语法规则被定义在[语法部分](https://kotlinlang.org/docs/reference/grammar.html#import).

我们可以注入单个命名，比如：

```kotlin
import foo.Bar // Bar is now accessible without qualification
```

或者注入一个范围内的所有可接入内容(包、类、对象等);

```kotlin
import foo.* // everything in 'foo' becomes accessible
```

如果存在命名冲突，我们可以使用`as`关键字在当前环境下重命名冲突实体来清除歧义。

```kotlin
import foo.Bar // Bar is accessible
import bar.Bar as bBar // bBar stands for 'bar.Bar'
```

`import`关键字并不仅限于引入类，你也可以用它来引入其他声明：

- 顶级函数和属性
- [对象声明](https://kotlinlang.org/docs/reference/object-declarations.html#object-declarations)(单例声明)中的函数和属性
- [枚举常量](https://kotlinlang.org/docs/reference/enum-classes.html)

与Java不同，Kotlin并没有一个单独的`import static`语法；所有生命的引入都要使用常规的`import` 关键字。

### 顶级对象的可见性声明

如果一个顶级声明被标记为`private`,那么他对于声明他的文件而言是私有的(请看[可视性修饰符](https://kotlinlang.org/docs/reference/visibility-modifiers.html))。

​	Kotlin可以像绝大多数面向对象语言一样，我们可以定义一个包，用来划分模块功能和防止重复命名。同样与绝大多数面向对象语言相同，Kotlin支持使用Import关键字引用类，用as关键字设置类在当前源文件中的名称。但是相比与Java相比，kotlin在引入以及文件系统上有非常大的不同。

1. kotlin允许在源文件中，不必须建立与源文件名相同的类。实际上，这本是是一种良好的编程风格，大家依旧可以保持。
2. kotlin允许用import方法包含对应包的类、顶层方法、顶层属性、声明在对象声明(一种单例声明方式)的属性和方法以及枚举常量。
3. kotlin的import关键字可以使用as 关键词配合来定义引入的别名。


```kotlin
//kotlin的引用方式
package basic.kotlin

import basic.kotlin.*;
import basic.kotlin.DataProviderManager as DataMg
import basic.kotlin.DataProviderManager.a;
import basic.kotlin.DataProviderManager.c;
import basic.kotlin.a;
import basic.kotlin.A;
import basic.kotlin.Goo;
import basic.kotlin.Magic

class Magic {
}
```

----

## 流程控制

### if 表达式

在 Kotlin 中，if 是表达式，比如它可以返回一个值。因此Kotlin没有三元运算符(condition ? then : else),因为if语句已经满足了效果。

```kotlin
// 传统用法
var max = a
if (a < b)
    max = b

// 带 else
var max: Int
if (a > b)
    max = a
else
    max = b

// 作为表达式
val max = if (a > b) a else b
```

if分支可以作为块，最后一个表达是是该块的值：

```kotlin
val max = if (a > b){
    print("Choose a")
    a
}
else{
    print("Choose b")
    b
}
```

如果使用If作为表达式而不是语句(例如，返回其值或将其赋值给一个变量)，则`必须`需要有一个`else分支`。

参看[if语法](http://kotlinlang.org/docs/reference/grammar.html#if)

> 表达式和语句最大的不同时，表达式可以有返回值，语句没有返回值。表达式表示是什么，语句表示做什么。表达式是一个计算过程，完成一些事情，最终目的是返回一个结果；语句则是一个独立的东西，不在于依赖于其结果的另一个东西。

### When 表达式

when 取代了 C 风格语言的 switch 。最简单的用法像下面这样

```kotlin
when (x) {
    1 -> print("x == 1")
    2 -> print("x == 2")
    else -> { // 注意这个语句块
        print("x is neither 1 nor 2")
    }
}
```

when会对所有的分支进行检查直到有一个条件满足。when 可以用做`表达式`或声明。如果用作表达式的话，那么满足条件的分支就是总表达式。如果用做声明，那么分支的的的值会被忽略。(像 if 表达式一样，每个分支是一个语句块，而且它的值就是最后一个表达式的值)

在其它分支都不匹配的时候默认匹配 else 分支。如果把 when 做为表达式的话 else 分支是强制的，除非编译器可以证明分支条件已经覆盖所有可能性。

如果有分支可以用同样的方式处理的话，分支条件可以连在一起：

```kotlin
when (x) {
    0,1 -> print("x == 0 or x == 1")
    else -> print("otherwise")
}
```

可以用任意表达式作为分支的条件

```
when (x) {
    parseInt(s) -> print("s encode x")
    else -> print("s does not encode x")
}
```

甚至可以用 in 或者 !in 检查值是否值在一个[范围](http://kotlinlang.org/docs/reference/ranges.html)或一个集合中：

```kotlin
when (x) {
    in 1..10 -> print("x is in the range")
    in validNumbers -> print("x is valid")
    !in 10..20 -> print("x is outside the range")
    else -> print("none of the above")
}
```

也可以用 is 或者 !is 来判断值是否是某个类型。注意，由于 [smart casts](http://kotlinlang.org/docs/reference/typecasts.html#smart-casts) ，你可以不用另外的检查就可以使用相应的属性或方法。

```kotlin
val hasPrefix = when (x) {
    is String -> x.startsWith("prefix")
    else -> false
}
```

when 也可以用来代替 if-else if 。如果没有任何参数提供，那么分支的条件就是简单的布尔表达式，当条件为真时执行相应的分支：

```kotlin
when {
    x.isOdd() -> print("x is odd")
    x.isEven() -> print("x is even")
    else -> print("x is funny")
}
```

参看[when语法](http://kotlinlang.org/docs/reference/grammar.html#when)



### for 循环

for 循环通过任何提供的迭代器进行迭代。语法是下面这样的：

```kotlin
for (item in collection)
    print(item)
```

内容可以是一个语句块

```kotlin
for (item: Int in ints){
    // ...
}
```

像之前提到的， for 可以对任何提供的迭代器进行迭代，比如：

- 包含一个`成员或扩展`函数`iterator()`，且该函数返回值类型
  - 包含一个`成员或扩展`函数`next()`，且
  - 包含一个返回值类型为`Boolean`的`成员或扩展`函数`hasNext()

这三个函数都需要标记为运算符`operator`.

----

对数组的for循环不会创建迭代器对象，而是被编译成一个基于索引的循环.

如果你想通过 list 或者 array 的索引进行迭代，你可以这样做：

```kotlin
for (i in array.indices)
    print(array[i])
```

在没有其它对象创建的时候 "iteration through a range " 会被自动编译成最优的实现。

或者，您可以使用`withIndex`库函数

```kotlin
for ((index, value) in array.withIndex()) {
    println("the element at $index is $value")
}
```

参看[for语法](http://kotlinlang.org/docs/reference/grammar.html#for)



### while 循环

while 和 do...while 像往常那样

```kotlin
while (x > 0) {
    x--
}

do {
    val y = retrieveData()
} while (y != null) // y 在这是可见的
```

参看[while 语法](http://kotlinlang.org/docs/reference/grammar.html#while)

### 在循环中使用 break 和 continue

kotlin 支持传统的 break 和 continue 操作符。参看[返回和跳转](http://kotlinlang.org/docs/reference/returns.html)

----

## 返回与跳转

Kotlin 有三种结构跳转表达式：

> -- return. 默认情况下，返回到最近的闭合函数或匿名函数（`fun`关键字），那么lambda会怎么样呢？
> -- break 结束`最近的闭合`循环
> -- continue 执行`最近的闭合`循环的`下一次循环`  

上述表达式都可以作为更大的表达式的一部分来使用：

```kotlin
val s = person.name ?: return
```

这些表达式的类型是 [Nothing type](http://kotlinlang.org/docs/reference/exceptions.html#the-nothing-type)

> Nothing type; `Nothing` 该类型没有值,用于标记无法达到的代码位置。在您自己的代码中,您可以使用`Nothing` 来标记一个永远不会返回的函数:

### break 和 continue 标签

在 Kotlin 中表达式可以添加标签。标签通过 @ 结尾来表示，比如：`abc@`，`fooBar@` 都是有效的(参看[语法](http://kotlinlang.org/docs/reference/grammar.html#label))。使用标签语法只需像这样：

```kotlin
loop@ for (i in 1..100){
    // ...
}
```

现在我们可以用标签实现 break 或者 continue 的快速跳转：

```kotlin
loop@ for (i in 1..100) {
    for (j in i..100) {
        if (...)
            break@loop
    }
}
```

break 是跳转标签后面的表达式，continue 是执行该循环的下一次迭代。



### 返回到标签

对于字面函数，局部函数，以及对象表达式，在Kotlin中其他函数函数可以嵌套这些函数。有资格的`return` 允许我们返回到外层函数。最重要的例子就是从如何从字面函数中返回，还记得我们之前的写法吗：

```kotlin
fun foo() {
    ints.forEach {
        if (it  == 0) return
        print(it)
    }
}
```

return 表达式返回到最近的闭合函数，比如 `foo` (注意这样非局部返回仅仅可以在[内联函数](http://kotlinlang.org/docs/reference/inline-functions.html)中使用)。如果我们需要从一个字面函数返回可以使用标签修饰 `return` :

```kotlin
fun foo() {
    ints.forEach lit@ {
        if (it ==0) return＠lit
        print(it)
    }
}
```

现在它仅仅从字面函数中返回。经常用一种更方便的含蓄的标签：比如用和传入的 lambda 表达式名字相同的标签。

```kotlin
fun foo() {
    ints.forEach {
        if (it == 0) return@forEach
        print(it)
    }
}
```

另外，我们可以用函数表达式替代匿名函数。在函数表达式中使用 return 语句可以从函数表达式中返回。

```kotlin
fun foo() {
    ints.forEach(fun(value:  Int){
        if (value == 0) return
        print(value)
    })
}
```

当返回一个值时，解析器给了一个参考，比如(原文When returning a value, the parser gives preference to the qualified return, i.e.)：

```kotlin
return@a 1
```

表示 “在标签 `@a` 返回 `1` ” 而不是返回一个标签表达式 `(@a 1)`

命名函数自动定义标签：

```kotlin
foo outer() {
    foo inner() {
        return@outer
    }
}
```