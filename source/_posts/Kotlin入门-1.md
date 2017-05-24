title: Kotlin入门
author: 卫振家
tags: []
categories:
  - Kotlin
date: 2017-05-24 20:18:00
---
Kotlin 基本语法介绍

基本数据类型

在Kotlin中，所有的东西都是一个对象，所以我们可以在任何变量上调用成员函数和属性。有一些类型是内置的，因为他们的实现经过了优化。但对于用户来说，他们看起来就像普通的类。在本节主要描述这些内置类型:numbers、booleans、arrays;

Numbers

Kotlin适用于Java相似的方式来操作数字，但是并不完全一样。比如说，不存在针对数字的隐式扩展转换，有些字面上 也不太一样。Kotlin提供以下内置类型表示数字

  Type  	Bit width
  Double	64       
  Float 	32       
  Long  	64       
  Int   	32       
  Short 	16       
  Byte  	8        

注意在Kotlin中characters 类型并不属于数字类型。

字符常量

对于整形有以下类型的字符常量

- 十进制：123
  - Long型使用大写L标记：123L
- 16进制：0x0F
- 二进制：0b00001011

Note:不支持8进制

Kotlin对浮点数也有方便的定义方式

- 默认的Double: 123.5、123.5e10
- Float型使用f或F标记：123.5f

在数字表示中使用下划线(从1.1开始)

你可以使用下划线以确保数字变量具有更好的可读性

    val oneMillion = 1_000_000
    val creditCardNumber = 1234_5678_9012_3456L
    val socialSecurityNumber = 999_99_9999L
    val hexBytes = 0xFF_EC_DE_5E
    val bytes = 0b11010010_01101001_10010100_10010010

表示-包装

在Java平台上，数字类型实际上被存储为JVM的基本数据类型，除非我们需要一个可以为null的数字引用或者作为泛型引用。在后面这种情况下，数字就会被包装。

注意，数字类型的包装不必保护一致性。

    val a: Int = 10000
    print(a === a) // Prints 'true'
    val boxedA: Int? = a
    val anotherBoxedA: Int? = a
    print(boxedA === anotherBoxedA) // !!!Prints 'false'!!!

但是，另一方面包装保护对等性。

    val a: Int = 10000
    print(a == a) // Prints 'true'
    val boxedA: Int? = a
    val anotherBoxedA: Int? = a
    print(boxedA == anotherBoxedA) // Prints 'true'

显式转换

由于不同的表现形式，小类型不是大类型的子类型。如果如，我们会在下列适用中遇到问题。

    //虚构代码，不要实际运行:
    val a: Int? = 1 // 一个包装int型 (Java.lang.Integer)
    val b: Long? = a // 隐式的转化为Long项 (Java.lang.Long)
    print(a == b) // 打印"false"因为Long.equals()的方法也会检测long数据的其他部分

所以，不只是一致性，甚至对等性也静静的消失掉了。

	结论是，小类型不能隐式的转换为大类型。这就意味着，我们只有在使用显式转换的情况在，可以把一个Byte型的值赋值给Int型的变量:

    val b: Byte = 1 // OK, literals are checked statically
    val i: Int = b // ERROR

我们可以使用显式转换为更宽类型的数字

    val i: Int = b.toInt() // OK: explicitly widened

每种数字类型都支持以下方式的转换操作

- toByte(): Byte
- toShort(): Short
- toInt(): Int
- toLong(): Long
- toFloat(): Float
- toDouble: Double
- toChar(): Char

隐式的类型转换都很少被注意到，因为这种类型都是通过上下文推断得出，而在算数运算中类型也不自动被恰当的重载。

    val l = 1L + 3 // Long + Int => Long

运算

Kotlin支持数字中标准的算数运算集合。这些运算集合被定义为对应类的成员发膜护发。但是编译器优化这些调用为对应的指定。

对于按位运算，并没有提供特殊的运算符，但是提供了内嵌格式的函数来达到相似的调用效果，比如说：

    val x = (1 shl 2) and 0x000FF000

一下是位运算的完整类型（仅对Int和Long型可用）

- shl(bits) – signed shift left (Java's <<)
- shl(bits) – signed shift left (Java's <<)
- ushr(bits) – unsigned shift right (Java's >>>)
- and(bits) – bitwise and
- or(bits) – bitwise or
- xor(bits) – bitwise xor
- inv() – bitwise inversion

Characters

字符使用类型Char表示。字符型不能直接当做数字类型对待

    fun check(c: Char) {
        if (c == 1) { // ERROR: incompatible types
            // ...
        }
    }

字符类型通过单引号'进行字面表示：'1'。特殊的字符可以使用反斜杠进行转义。支持下列转义顺序:\t, \b, \n, \r, \', \", \\和 \$。为了转码其他字符，可以使用Unicode转义序列语法：'\uFF00'。

我们可以显式把一个字符转化为Int型数字。

    fun decimalDigitValue(c: Char): Int {
        if (c !in '0'..'9')
            throw IllegalArgumentException("Out of range")
        return c.toInt() - '0'.toInt() // Explicit conversions to numbers
    }

跟数字 一样，需要时字符型也可以包装为一个可null的引用。在包装运算中不保证一致性。

Boolean

Boolean类型表示布尔值，布尔值只有两个:true和false;

需要时布尔值也会被包装为一个可null得引用

内置的布尔运算符包括：

- || – lazy disjunction
- && – lazy conjunction
- ! - negation

Array

Kotlin中的数组用类Array来标示。Array类包含get和set函数（通过运算符重载协议转化为[]）和size属性。除此之外还有其他一些有用的成员函数：

    class Array<T> private constructor() {
        val size: Int
        operator fun get(index: Int): T
        operator fun set(index: Int, value: T): Unit
    
        operator fun iterator(): Iterator<T>
        // ...
    }

为了创建一个数组，我们可以使用库函数arrayOf()，并向该函数传递一列值，这样arrayOf(1, 2, 3)就创建一个一个数组[1, 2, 3]。另外也可以使用arrayOfNulls()库函数创建填充了指定数量Null元素的数组。

另一个选择是使用工厂方法，接受一个数组大小参数和一个可以根据指针返回对应数组元素初始化值得函数。

    // Creates an Array<String> with values ["0", "1", "4", "9", "16"]
    val asc = Array(5, { i -> (i * i).toString() })

就如我上面所说，[]操作符表示调用成员函数get()和set();

注意：与Java不同，Kotlin的数组是不可变的。这意味着在Kotlin中我们不能将Array<String>赋值给Array<Any>,这可能会导致一个运行时错误。但是我们可以使用Array<out Any>。细节可以查看类型投影。

Kotlin也提供了特殊的类以在不需要额外包装开销的情况下标示基础类型数组：ByteArray,ShortArray, IntArray 等。这些类与Array类不存在继承关系，但是具有与Array类相同的方法和属性集合。每个基础类数组也都有对应的工厂函数:

    val x: IntArray = intArrayOf(1, 2, 3)
    x[0] = x[1] + x[2]

Strings

字符串使用类型String表示。字符串是不可变的。字符串的每一个元素都是一个字符，我们可以通过索引运算符$[i]来获取这些字符。一个字符串可以用for-loop结构来遍历字符。

    for (c in str) {
        println(c)
    }

字符串的语言标示

Kotlin有两种字符串的字面值：可能包含转义字符的转义字符串和包含换行和任意文本的原生字符串。转义字符串与Java字符串非常相似

    val s = "Hello, world!\n"

通过简单的\就能完成转义。可以查看上面关于字节的描述查看支持的转义序列。

原生字符串使用三个双引号"""来限定，包含不进行转义的字符，且可以包含换行符和其他任何字符。

    val text = """
        for (c in "foo")
            print(c)
    """

你可以使用 trimMargin() 函数来移除主要的空白符。

    val text = """
        |Tell me and I forget.
        |Teach me and I remember.
        |Involve me and I learn.
        |(Benjamin Franklin)
        """.trimMargin()

默认上我们用|作为边界前缀，但是你可选择其他字符作为前缀，并作为参数传递，比如trimMargin(">").

字符串模板

字符串可以包含模板表达式。比如一些可以求值的程序片段，他们的结果被嵌入到目标字符串中。一个模板表达式以一个$符开始，后面要么跟随一个简单变量名

    val i = 10
    val s = "i = $i" // evaluates to "i = 10"

要么跟随一个{}包裹的任意表达式

    val s = "abc"
    val str = "$s.length is ${s.length}" // evaluates to "abc.length is 3"

字符串末班在原声字符串和转移字符串中一样支持。如果你需要在一个原生字符产表示字符$(因为原生字符串不支持转义)。你可以使用以下的语法结构。

    val price = """
    ${'$'}9.99
    """



Package

一个原文源文件可以以包声明开始。

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

这样，这个源文件中的所有内容都被包含在哪个被生命的包中。这样，在上面的例子中，函数baz()的全名应该是basic.kotlin.baz，类Goo的全名是basic.kotlin.Goo;

如果未指定包名，这样一个文件的上下文都归属于一个没有名字的默认包。

默认注入

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

注入

除了默认注入外，每一个文件还可以包含自己的注入指令。主语的语法规则被定义在语法部分.

我们可以注入单个命名，比如：

    import foo.Bar // Bar is now accessible without qualification

或者注入一个范围内的所有可接入内容(包、类、对象等);

    import foo.* // everything in 'foo' becomes accessible

如果存在命名冲突，我们可以使用as关键字在当前环境下重命名冲突实体来清除歧义。

    import foo.Bar // Bar is accessible
    import bar.Bar as bBar // bBar stands for 'bar.Bar'

import关键字并不仅限于引入类，你也可以用它来引入其他声明：

- 顶级函数和属性
- 对象声明(单例声明)中的函数和属性
- 枚举常量

与Java不同，Kotlin并没有一个单独的import static语法；所有生命的引入都要使用常规的import 关键字。

顶级对象的可见性声明

如果一个顶级声明被标记为private,那么他对于声明他的文件而言是私有的(请看可视性修饰符)。

	Kotlin可以像绝大多数面向对象语言一样，我们可以定义一个包，用来划分模块功能和防止重复命名。同样与绝大多数面向对象语言相同，Kotlin支持使用Import关键字引用类，用as关键字设置类在当前源文件中的名称。但是相比与Java相比，kotlin在引入以及文件系统上有非常大的不同。

1. kotlin允许在源文件中，不必须建立与源文件名相同的类。实际上，这本是是一种良好的编程风格，大家依旧可以保持。
2. kotlin允许用import方法包含对应包的类、顶层方法、顶层属性、声明在对象声明(一种单例声明方式)的属性和方法以及枚举常量。
3. kotlin的import关键字可以使用as 关键词配合来定义引入的别名。

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



Function

函数声明

	kotlin使用fun关键字声明函数。

1. 标准函数定义
       fun sum(a: Int, b: Int): Int {
           return a + b
       }
2. 单行函数
       fun sum(a: Int, b: Int) = a + b
3. 无返回值函数
       fun printSum(a: Int, b: Int): Unit {
           println("sum of $a and $b is ${a + b}")
       }
   无返回值函数的Unit关键字可以忽略
       fun printSum(a: Int, b: Int) {
           println("sum of $a and $b is ${a + b}")
       }

函数调用

经典方式调用函数

    val result = double(2)

使用.dot调用成员方法

    Sample().foo() //创建类Sample的实例并调用foo

使用Infix 标识调用

Infix 示意为 To fix in the mind; instill灌输注入

中缀表达式（Infix notations ）就是用运算符连接起来有运算意义的式子

最常见的中缀运算符包括+ - * / > < == !等

函数在满足以下条件是，也可以使用Infix notations方式条用

- 这些函数是成员函数或者扩展函数
- 仅有一个参数
- 被infix关键字标记

    // 为Int定义扩展函数
    infix fun Int.shl(x: Int): Int {
    ...
    }
    
    //使用Infix方式调用函数
    
    1 shl 2
    
    //等同于使用
    
    1.shl(2)

函数的形参

	kotlin使用Pascal 标志定义形参，例如name: type，形参使用:分割形参名称和类型。每一个形参必须有明确的类型标示

    fun powerOf(number: Int, exponent: Int) {
    ...
    }

函数的默认实参

	函数的形参可以有默认值。默认值在对应的形参被忽略时使用。这与其他语言相比，减少了override的次数。

    fun read(b: Array<Byte>, off: Int = 0, len: Int = b.size()) {
    ...
    }

默认值可以在目标值的类型后用=定义。

	override方法总是使用与base方法（根方法，基础方法）相同的默认值形参值。当我们override一个方法时，默认的参数值必须从签名中忽略:

    open class A {
        open fun foo(i: Int = 10) { ... }
    }
    
    class B : A() {
        override fun foo(i: Int) { ... }  // no default value allowed
    }	

具名实参

当调用函数时，函数的形参可以被重命名。当一个函数有多个形参或者默认形参时，将会变得更加方便:

    fun reformat(str: String,
                 normalizeCase: Boolean = true,
                 upperCaseFirstLetter: Boolean = true,
                 divideByCamelHumps: Boolean = false,
                 wordSeparator: Char = ' ') {
    ...
    }

我们可以用默认实参来调用函数

    reformat(str)

然而，当我们不是用默认值来调用参数时，调用方式应该如下

    reformat(str, true, true, false, '_')

使用被命名的实参可以让我们的代码更有可读性

    reformat(str,
        normalizeCase = true,
        upperCaseFirstLetter = true,
        divideByCamelHumps = false,
        wordSeparator = '_'
    )

而且如果我们不需要所有参数，我们还可以这样

    reformat(str, wordSeparator = '_')

注意，在调用Java函数（方法时）,命名实参方式是不可用的，因为Java字节码文件并不总是保留函数参数的名称。

单元返回参数

如果一个函数不返回任何有用值，那么他的返回类型就是Unit;Unit是一种只能关联一个值Unit的类型。这个值不强制显式返回的。

    fun printHello(name: String?): Unit {
        if (name != null)
            println("Hello ${name}")
        else
            println("Hi there!")
        // `return Unit` or `return` is optional
    }

函数签名中的Unit返回类型也是可选的，上面的代码也可以写成

    fun printHello(name: String?) {
        ...
    }

行函数

当一个函数只返回单一表达式时，花括号可以被省略，函数体可用用后置的a = symbol来表达

    fun double(x: Int): Int = x * 2

如果编译器可以推断返回值类型，那么返回值的显式声明也是可选的；

    fun double(x: Int) = x * 2

显式返回值类型

	块函数必须显式的指定返回值得类型，除非函数计划返回Unit。这种情况下，是否显式指定是可选的。

	Kotlin并不会对块函数进行返回类型推断，因为这种函数通常在函数体内具有复杂的控制流，并且返回值的类型对于reader而言是不明显的，甚至有时对于编译器而言都是如此。

可变数量的实参-Varargs

一个函数的形参（通常是最后一个）可能会被vararg修饰符标记

    fun <T> asList(vararg ts: T): List<T> {
        val result = ArrayList<T>()
        for (t in ts) // ts is an Array
            result.add(t)
        return result
    }

这种情况下，允许响函数传递可变数量的实参。

    val list = asList(1, 2, 3);

在一个函数中，T类型的vararg参数可以视为T的数组，比如说，按上述案例中的ts变量具有Array<out T>类型。

	函数中，只有一个形参可以被标记为vararg。如果vararg形参不是形参列表的最后一个，那么后续形参的值，可以使用具名实参方式传递，或者，如果是函数类型的形参，可以用圆括号外的lambda来传递参数。

	当我们调用一个vararg函数式，我们可以逐一的传递参数，比如说asList(1,2,3) 。或者，如果我们已经有一个数组了，想要将数组的内容传递给这个函数，我们使用扩散运算符（使用*号作为数组的前缀）。

    val a = arrayOf(1, 2, 3)
    val list = asList(-1, 0, *a, 4)

函数作用域

在kotlin文件中，函数可以被声明为顶层项。这意味着你不需要向Java、C#或者Scala那样，创建一个类来持有一个函数。除了全局函数外，Kotlin函数也可以被声明为局部的，比如说成员函数和扩展函数。

局部函数

Kolin支持局部函数，比方说，在一个函数中创建另一个函数：

    fun dfs(graph: Graph) {
        fun dfs(current: Vertex, visited: Set<Vertex>) {
            if (!visited.add(current)) return
            for (v in current.neighbors)
                dfs(v, visited)
        }
    
        dfs(graph.vertices[0], HashSet())
    }

局部函数可以处理外层函数的局部变量，比如说闭包。所以在上面的案例中，visited可以使一个局部变量。

    fun dfs(graph: Graph) {
        val visited = HashSet<Vertex>()
        fun dfs(current: Vertex) {
            if (!visited.add(current)) return
            for (v in current.neighbors)
                dfs(v)
        }
    
        dfs(graph.vertices[0])
    }

成员函数

在一个类或者对象中定义的函数是一个成员函数。

    class Sample() {
        fun foo() { print("Foo") }
    }

成员函数可以通过.来调用

    Sample().foo() // creates instance of class Sample and calls foo

泛型函数

函数可以拥有泛型形参。这些形参必须在函数名前使用<>指明。

单行函数

扩展函数

尾递归函数

Kotlin支持一种称之为尾递归的函数式编程风格。他允许将通常写成循环的算法使用函数递归来替代，但是这种风格不会造成栈溢出风险。当一个函数被tailrec修饰符标记，并满足所需形式时，编译器就会丢弃快速高效的基于循环的版本，二区优化递归。

    tailrec fun findFixPoint(x: Double = 1.0): Double
            = if (x == Math.cos(x)) x else findFixPoint(Math.cos(x))

上述代码用来计算余弦的不动点，这是一个数学常量。它仅仅从1.0开始重复的调用Math.cos函数知道结果不在发生改变位置，获得一个值为0.7390851332151607的结果。

以上求解的代码完全等价于下面更为传统的风格：

    private fun findFixPoint(): Double {
        var x = 1.0
        while (true) {
            val y = Math.cos(x)
            if (x == y) return y
            x = y
        }
    }

为了适用talrec修饰符，一个函数必须在它执行的最后调用自身。如果在递归调用之后还有其他代码，你就不能使用尾递归，并且你不能在try/catch/finally块中使用尾递归。当前版本，尾递归仅在JVM后端中支持使用。


