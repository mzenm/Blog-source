# Kotlin函数

## Function

### 函数声明

​	kotlin使用`fun`关键字`声明`函数。

1. 标准函数定义

   ```kotlin
   fun sum(a: Int, b: Int): Int {
       return a + b
   }
   ```

2. 单行函数

   ```kotlin
   fun sum(a: Int, b: Int) = a + b
   ```

3. 无返回值函数

   ```kotlin
   fun printSum(a: Int, b: Int): Unit {
       println("sum of $a and $b is ${a + b}")
   }
   ```

   无返回值函数的`Unit`关键字可以忽略

   ```kotlin
   fun printSum(a: Int, b: Int) {
       println("sum of $a and $b is ${a + b}")
   }
   ```


----

### 函数调用

#### 经典方式调用函数

```kotlin
val result = double(2)
```

#### 使用`.`dot调用成员方法

```kotlin
Sample().foo() //创建类Sample的实例并调用foo
```

#### 使用`Infix` 标识调用

> Infix 示意为 `To fix in the mind; instill`灌输注入
>
> 中缀表达式（Infix notations ）就是用运算符连接起来有运算意义的式子
>
> 最常见的中缀运算符包括`+ - * / > < == ! `等

函数在满足以下条件是，也可以使用`Infix notations`方式条用

- 这些函数是成员函数或者扩展函数
- 仅有一个参数
- 被infix关键字标记

```kotlin
// 为Int定义扩展函数
infix fun Int.shl(x: Int): Int {
...
}

//使用Infix方式调用函数

1 shl 2

//等同于使用

1.shl(2)
```

#### 函数的形参

​	kotlin使用Pascal 标志定义形参，例如` name: type`，形参使用`:`分割形参名称和类型。每一个形参必须有明确的类型标示

```kotlin
fun powerOf(number: Int, exponent: Int) {
...
}
```

#### 函数的默认实参

​	函数的形参可以有默认值。默认值在对应的形参被忽略时使用。这与其他语言相比，减少了`override`的次数。

```kotlin
fun read(b: Array<Byte>, off: Int = 0, len: Int = b.size()) {
...
}
```

默认值可以在目标值的类型后用`=`定义。

​	`override`方法总是使用与`base`方法（根方法，基础方法）相同的默认值形参值。当我们`override`一个方法时，默认的参数值必须从签名中忽略:

```kotlin
open class A {
    open fun foo(i: Int = 10) { ... }
}

class B : A() {
    override fun foo(i: Int) { ... }  // no default value allowed
}	
```

#### 具名实参

当调用函数时，函数的形参可以被重命名。当一个函数有多个形参或者默认形参时，将会变得更加方便:

```kotlin
fun reformat(str: String,
             normalizeCase: Boolean = true,
             upperCaseFirstLetter: Boolean = true,
             divideByCamelHumps: Boolean = false,
             wordSeparator: Char = ' ') {
...
}
```

我们可以用默认实参来调用函数

```kotlin
reformat(str)
```

然而，当我们不是用默认值来调用参数时，调用方式应该如下

```kotlin
reformat(str, true, true, false, '_')
```

使用被命名的实参可以让我们的代码更有可读性

```kotlin
reformat(str,
    normalizeCase = true,
    upperCaseFirstLetter = true,
    divideByCamelHumps = false,
    wordSeparator = '_'
)
```

而且如果我们不需要所有参数，我们还可以这样

```kotlin
reformat(str, wordSeparator = '_')
```

> 注意，在调用Java函数（方法时）,命名实参方式是不可用的，因为Java字节码文件并不总是保留函数参数的名称。

#### 单元返回参数

如果一个函数不返回任何有用值，那么他的返回类型就是`Unit`;`Unit`是一种只能关联一个值`Unit`的类型。这个值不强制显式返回的。

```java
fun printHello(name: String?): Unit {
    if (name != null)
        println("Hello ${name}")
    else
        println("Hi there!")
    // `return Unit` or `return` is optional
}
```

函数签名中的`Unit`返回类型也是可选的，上面的代码也可以写成

```java
fun printHello(name: String?) {
    ...
}
```

#### 行函数

当一个函数只返回单一表达式时，花括号可以被省略，函数体可用用后置的`a = symbol`来表达

```kotlin
fun double(x: Int): Int = x * 2
```

如果编译器可以`推断`返回值类型，那么返回值的显式声明也是`可选`的；

```kotlin
fun double(x: Int) = x * 2
```

#### 显式返回值类型

​	块函数必须显式的指定返回值得类型，除非函数计划返回`Unit`。这种情况下，是否显式指定是可选的。

​	Kotlin并不会对块函数进行返回类型推断，因为这种函数通常在函数体内具有复杂的控制流，并且返回值的类型对于`reader`而言是不明显的，甚至有时对于编译器而言都是如此。

#### 可变数量的实参-Varargs

一个函数的形参（通常是最后一个）可能会被`vararg`修饰符标记

```kotlin
fun <T> asList(vararg ts: T): List<T> {
    val result = ArrayList<T>()
    for (t in ts) // ts is an Array
        result.add(t)
    return result
}
```

这种情况下，允许响函数传递可变数量的实参。

```kotlin
val list = asList(1, 2, 3);
```

在一个函数中，`T`类型的`vararg`参数可以视为`T`的数组，比如说，按上述案例中的`ts`变量具有`Array<out T>`类型。

​	函数中，只有一个形参可以被标记为`vararg`。如果`vararg`形参不是形参列表的最后一个，那么后续形参的值，可以使用具名实参方式传递，或者，如果是函数类型的形参，可以用圆括号外的lambda来传递参数。

​	当我们调用一个`vararg`函数式，我们可以逐一的传递参数，比如说`asList(1,2,3)` 。或者，如果我们已经有一个数组了，想要将数组的内容传递给这个函数，我们使用扩散运算符（使用*号作为数组的前缀）。

```kotlin
val a = arrayOf(1, 2, 3)
val list = asList(-1, 0, *a, 4)
```

----

### 函数作用域

在kotlin文件中，函数可以被声明为顶层项。这意味着你不需要向Java、C#或者Scala那样，创建一个类来持有一个函数。除了全局函数外，Kotlin函数也可以被声明为局部的，比如说成员函数和扩展函数。

#### 局部函数

Kotlin支持局部函数，比方说，在一个函数中创建另一个函数：

```kotlin
fun dfs(graph: Graph) {
    fun dfs(current: Vertex, visited: Set<Vertex>) {
        if (!visited.add(current)) return
        for (v in current.neighbors)
            dfs(v, visited)
    }

    dfs(graph.vertices[0], HashSet())
}
```

局部函数可以处理外层函数的局部变量，比如说闭包。所以在上面的案例中，`visited`可以使一个局部变量。

```kotlin
fun dfs(graph: Graph) {
    val visited = HashSet<Vertex>()
    fun dfs(current: Vertex) {
        if (!visited.add(current)) return
        for (v in current.neighbors)
            dfs(v)
    }

    dfs(graph.vertices[0])
}
```

#### 成员函数

在一个类或者对象中定义的函数是一个成员函数。

```kotlin
class Sample() {
    fun foo() { print("Foo") }
}
```

成员函数可以通过`.`来调用

```kotlin
Sample().foo() // creates instance of class Sample and calls foo
```

----

### 泛型函数

函数可以拥有泛型形参。这些形参必须在函数名前使用`<>`指明。

----

### 单行函数

----

### 扩展函数

----

### 尾递归函数

Kotlin支持一种称之为尾递归的函数式编程风格。他允许将通常写成循环的算法使用函数递归来替代，但是这种风格不会造成栈溢出风险。当一个函数被`tailrec`修饰符标记，并满足所需形式时，编译器就会丢弃快速高效的基于循环的版本，二区优化递归。

```kotlin
tailrec fun findFixPoint(x: Double = 1.0): Double
        = if (x == Math.cos(x)) x else findFixPoint(Math.cos(x))
```

上述代码用来计算余弦的不动点，这是一个数学常量。它仅仅从1.0开始重复的调用Math.cos函数知道结果不在发生改变位置，获得一个值为0.7390851332151607的结果。

以上求解的代码完全等价于下面更为传统的风格：

```kotlin
private fun findFixPoint(): Double {
    var x = 1.0
    while (true) {
        val y = Math.cos(x)
        if (x == y) return y
        x = y
    }
}
```

为了适用`talrec`修饰符，一个函数必须在它执行的最后调用自身。如果在递归调用之后还有其他代码，你就不能使用尾递归，并且你不能在try/catch/finally块中使用尾递归。当前版本，尾递归仅在JVM后端中支持使用。

----

## 高阶函数和Lambdas表达式

### 高阶函数

高阶函数一个可以接受函数做为形参，或者可以返回一个函数的函数。`lock()`就是这样一个很好的例子，他接受一个锁对象和一个函数，获取到锁，运行函数，释放锁。

```kotlin
fun <T> lock(lock: Lock, body: () -> T): T {
    lock.lock()
    try {
        return body()
    }
    finally {
        lock.unlock()
    }
}
```

让我们检查以上上面的函数，body变量具有一个函数类型`() -> T`,所以body变量假定为一个不接受参数返回`T`类型值得函数。该函数在`try`块中被调用

，同时被指定`lock`保护，他的运算结果被`local()`函数返回。

如果我们想要调用`lock()`函数，我们可以将另一个函数作为实参传递给它。

```kotlin
fun toBeSynchronized() = sharedResource.operation()

val result = lock(lock, ::toBeSynchronized)
```

另外，通常更方便的方式是传一个lambda表达式到`lock()`函数。

```kotlin
val result = lock(lock, { sharedResource.operation() })
```

lambda表达式更多的描述将放在下面，但为了继续本节，我们可以简要的了解一下。

- 一个Lambda表达式总是被花括号包裹。
- Lambda表达式如果有参数，那么应该声明在`->`之前（参数`类型`通常会被忽略）。
- Lambda表达式得表达体如果存在，那么放在`->`后面

在Kotlin中，如果一个函数的最后一个参数也是一个函数的，还有种更便捷的写法。你可以讲一个Lambda表达式作为对应的参数传递给高阶函数，并将其放在括号外面。

```kotlin
lock (lock) {
    sharedResource.operation()
}
```



下面是一个`map()`的高阶函数示例：

```kotlin
fun <T, R> List<T>.map(transform: (T) -> R): List<R> {
    val result = arrayListOf<R>()
    for (item in this)
        result.add(transform(item))
    return result
}
```

这个函数可以用以下方式调用，记住，对于Lambda表达式：参数`->`运算题

```kotlin
val doubled = ints.map { value -> value * 2 }
```

Note：如果在某次调用中，仅有Lambda一个参数的话，那么调用的()可以忽略;

#### it: 单个参数的 隐式

另一个比较有用的简化是，如果一个函数字面只有一个参数，那么这个参数的声明可以忽略（得带上`->`），Kotlin为这个参数提供了默认声明`it`（）

```kotlin
ints.map { it * 2 }
```

这种简化还允许写入[LINQ](https://docs.microsoft.com/zh-cn/dotnet/articles/csharp/linq/index)风格的代码:

```kotlin
strings.filter { it.length == 5 }.sortBy { it }.map { it.toUpperCase() }
```

> 语言集成查询 (LINQ) 是一系列直接将查询功能集成到 C# 语言的技术统称。 数据查询历来都表示为简单的字符串，没有编译时类型检查或 
> IntelliSense 支持。 此外，对于每种数据源，还需要学习不同的查询语言：SQL 数据库、XML 文档、各种 Web 服务等。 借助 
> LINQ，查询成为了最高级的语言构造，就像类、方法和事件一样。
>
> 对于编写查询的开发者来说，LINQ 最明显的“语言集成”部分就是查询表达式。 查询表达式采用声明性*查询语法*编写而成。 使用查询语法，可以用最少的代码对数据源执行筛选、排序和分组操作。 可使用相同的基本查询表达式模式来查询和转换 SQL 数据库、ADO .NET 数据集、XML 文档和流以及 .NET 集合中的数据。

#### 下划线`_`替换未使用变量(since 1.1)

如果一个Lambda参数未使用，你可以用一个下划线来替代作为他的名字 替代。

```kotlin
map.forEach { _, value -> println("$value!") }
```

----

### 内联函数

----

### Lambda表达式和匿名函数

一个lambda表达式或者一个匿名函数是一种字面函数，比如一个函数没有进行声明，但可以作为一个表达式立即传递（参考主类型的字面表达如`123`,`"String"`）。思考下面的案例：

```kotlin
max(strings, { a, b -> a.length < b.length })
```

比如，函数`max`是一个高阶函数，接受 一个函数值作为第二个参数。这个第二参数是一个表达式，但其自身也是一个函数，比如一种字面函数。这种写法等价于

```kotlin
fun compare(a:String,b:String):Boolean = a.length < b.length
```

#### 函数类型

为了让一个函数可以作为另一个函数的参数，我们不得不为那个参数指定具体的参数类型。比方说，上面提到的函数`max`是如下定义的:

```kotlin
fun <T> max(collection: Collection<T>, less: (T, T) -> Boolean): T? {
	var max: T? =null;
	for(it in collection)
  		if (max == null || less(max, it))
  			max = it
  	return max
}
```

形参`less`的类型是`(T, T) -> Boolean`，标示一个函数接受两个`T`类型参数，返回Boolean型：如果一个比第二小，那么返回true。

在函数体的第四行，less被作为一个函数调用：通过传递两个`T`类型的实参调用。

上面写的函数类型，如果你想要存档每一个参数的含义，也可以使用具名参数，即去声明函数变量

```kotlin
val compare: (x: T, y: T) -> Int = ... //compare 变量名 ： (x: T, y: T) -> Int 变量类型
```

#### Lambda表达式语法

Lambda表达式得完整语法如下，即函数类型的字面表达，是一下格式

```kotlin
val sum = { x:Int, y:Int -> x + y }
```

一个Lambda表达式总是被一堆花括号`{}`包裹。在完整语法格式下，参数应该在小括号`()`中声明，且具有可选的类型注解。方法体通常都跟在`->`标识后。如果相关的Lambda表达式返回类型不为Unit，那么Lambda表达式的最后一句（当然可能只有一句）被当做返回值对待。

如果我们去除所有可选的注解，剩下的部分大致是这样:

```kotlin
var sum: (Int, Int) -> Int = { x, y -> x + y }
```

只有一个参数的Lambda表达式非常普遍。如果Kotlin可以自行判断出这个参数的签名，他允许我们不去声明这个唯一参数，而在程序中使用`it`隐式为我们声明它。

```kotlin
ints.filter { it>0 } //这个字面表达的类型是(it: Int)->Boolean
```

我们可以通过使用[qualified return](http://kotlinlang.org/docs/reference/returns.html#return-at-labels)语法，显式的从一个Lambda中返回一个值。此外，这个最后一个表达式的值是隐式返回的。因此，下面两个代码片段式等价的。

```kotlin
ints.filter {
  var shouldFilter = it > 0
  shouldFilter
}

ints.filter {
  var shouldFilter = it > 0
  return@filter shouldFilter
}
```

> qualified return 合格返回。默认return仅返回到最近的闭合函数。但是我们 qualified return 可以帮我们从一个更外层的函数返回。当我们需要从lambda表达式中返回时，我们可以标记返回并授予返回资格。
>
> ```kolin
> fun foo() {
>   ints.forEach lit@ {
>     if (it == 0 ) return@lit
>     print(it)
>   }
> }
> ```
>
> 现在，这个函数只允许从lambda表达式中返回了。当然很多时候，我们都是用隐式标签。这个隐式标签与`被传递到lambda表达式`的函数`同名`。
>
> 另外，我们可以使用匿名函数替换lambda表达式。一个匿名函数的返回值将从他本身返回。
>
> 当返回一个值时，解析器倾向于将
>
> ```kotlin
> return@a 1
> ```
>
> 认为“在标签`@a`处返回1”，而不是返回一个标签化得表达式`@a 1`;
>
> 注意：这种非本地返回方式，仅支持传递 给内联函数的lambda表达式。

注意，如果一个函数接受另一个函数作为最后一个参数，那么这个lambda表达式参数可以放在防止参数的括号外面。

#### 匿名函数

从上面所说的`lambda表达式语法`中丢失掉的东西就是没有明确说明函数的返回值类型。在大多数情况下，因为编译器可以自动推断返回值类型，所以我们并不要明确指出。然而，如果你需要显式的指定返回值类型，你可以选择另一种语法：`一个匿名函数`:

```kotlin
fun(x: Int, y: Int): int = x + y;
```

匿名函数看起来跟一个常规函数声明非常相似，除了函数名被忽略了。他的函数体可以是一个表达式，也可以是一个代码块。

```kotlin
fun(x: Int, y: Int): Int {
    return x + y
}
```

参数和返回类型的指定方式与常规函数相同，但不同之处在于如果编译器可以从上下文推到出来，参数类型可以省略。

```kotlin
ints.filter(fun(item) = item > 0)
```

匿名函数相关的返回值类型与普通函数相同：对于表达式型函数，返回值类型可以自行推断，而对于代码块型函数，除非返回值类型是`Unit`，否则必须显式的指定返回值类型。

请注意,匿名函数参数总是在括号中传递。允许在括号外传递参数的函数简写语法只适用于lambda表达式。

lambda表达式和匿名函数的另一个区别是非本地返回。一个不带标签的返回标示总是从使用fun关键字的函数那返回。这句话的意思是在一个lambda表达式中的返回将最终将从包含函数返回，而匿名函数的返回将会从匿名函数自身返回。

#### 闭包

一个lambda表达式或者匿名函数（还有局部函数和对象表达式）可以获取他的闭包。比如说，外层范围内声明的那个变量。与java不同，闭包占据的变量可以更改：

```kotlin
var sum =0;
ints.filter( it > 0 ).forEach {
  sum += it;
}
print(sum);
```

#### 带有接受者的字面函数

Kotlin提供了`带有接收者对象状态下`调用`字面函数`的能力。在一个`字面函数`的函数体中，你不使用任何附加修饰符（可以对比在成员变量）的调用接受者对象上的方法。这个与扩展函数有些相似，扩展函数允许你函数的函数体内调用接受者对象的成员。最重要的应用实例是[Type-safe Groovy-style builders](http://kotlinlang.org/docs/reference/type-safe-builders.html).

> receiver 接受者或者接收人，大意就是 就是将 当前声明的函数，方法发送给另一个对象，这样，这个方法就变成了 另一个对象的方法，类似于成员方法。这个对象就是接受者。因此在声明带有接受者的函数，如带有接受者的匿名函数，带有接受者的Lambda表达式时，就可以像定义成员函数一样直接使用this关键字和接受者对象的其他方法，而不去要携带任何标志。

下面就是一个带有接收者的字面函数类型:

```kotlin
sum: Int.(other: Int) ->Int; //定义一个函数sum,这个函数在Int对象上调用，接受另一个Int对象作为参数，返回值类型也是一个Int
```

如果字面函数是带有接受者的，那么就该这个函数就可以当做成员函数被调用.

```kotlin
1.sum(2)
```

匿名函数语法允许你直接指定字面函数的`接受者类型`。当你想要声明一个`类型是`带有接受者的函数的`变量`时这会非常有用，我们可以稍后会使用它。

```kotlin
val sum = fun Int.(other : Int): Int = this + other 
```

当接受者类型可以从上下文推断是，Lambda表达式也可以作为一个带有接受者的字面函数使用。

```kotlin
class HTML {
  fun body() {}
}

fun html(init: HTML.() -> Unit): HTML {
  var html = HTML(); //创建一个接受者对象。
  html.init()	//将接受者对象粗旱地给Lambda
  return html;
}

html {  //带有接收器的lambda从这里开始
  body()  //在接收器对象上调用一个方法
}
```

----

### 内联函数

> 内联函数的基本思想在于将每个函数调用以它的代码体来替换，很可能会增加整个目标代码的体积过分地使用内联所产生的程序会因为有太大的体积而导致可用空间不够。
>
> 根据这个基本概念，这是一种特殊的函数调用规则，必须需要编译器的支持，调用函数也必须声明为内联。可以假定调用函数时，直接替换为函数体的内容。

使用高阶函数将造成一定的运行时惩罚：每个函数都是一个对象，所以每一个函数都占有一个闭包，比如说在该函数的函数体内引用的所有变量。内存分配（无论函数对象还是函数类）和虚拟调用产生额外运行时开销。

> 闭包 closure 在计算机科学中，**闭包**（*Closure*）是**词法闭包**（*Lexical Closure*）的简称，是引用了自由变量的函数。这个被引用的自由变量将和这个函数一同存在，即使已经离开了创造它的环境也不例外。所以，有另一种说法认为闭包是由函数和与其相关的引用环境组合而成的实体。
>
> 所以，一个闭包就是一个“捕获”或“携带”了其被`生成的环境中、所属的变量范围内所引用的所有变量`的函数。的确，很难描述，但当你看完了这些代码后，你就很容易理解了。
>
> ```c#
> var x = 1;
>
> Action action = () =>
> {
>     var y = 2;
>     var result = x + y;
>     Console.Out.WriteLine("result = {0}", result);
> };
>
> action();
> ```
>
> 这里我们首先定义了一个变量“x”，值为1。然后我们定义了一个匿名函数(一个lambda表达式)赋给类型Action。Action没有参数，没有返回值，但如果你观察“action”里的定义，你会发现它使用了“x”变量。这是变量是被action“捕获”或“携带”的，自动被添加到了action的运行环境中了。
>
> 当我们执行action时，它输出了我们预期的结果。请注意，当我们执行时，原始的“x”此时已经脱离了它当初的变量环境，但它仍然能用。
>
> 按照我的理解，闭包相当于封装了一个对象，对象中使用其成员变量和成员函数保存了函数定义时的各种条件和环境。为什么称为闭包，因为这个函数本身包含了一个块状代码。

但是，我们发现，在很多案例中，这种开销都可以通过内联的lambda表达式来消除。下面展示的函数就是这一情况的很好案例。考虑一下案例，`lock()`函数可以使用简单地内联方式调用函数

```kotlin
lock(1){ foo() }
```

而不是使用 为参数创建函数对象并生成一次调用的方式，这样编译器可以省略一下复杂的代码。

```kotlin
1.lock()
try{
  foo()
}
finally{
  1.unlock();
}
```

这不就我们最初的时候想要的东西。

想要编译器实现内联，我们需要就两`local`使用`inline`修饰符标记

```kotlin
inline fun lock<T>(lock: Lock, body:()->T): T {
  // ...
} 
```

这个`inline`修饰符同时影响函数自身和传递给函数的Lambdas表达式：内联传入调用点的所有东西。

> 调用点 class site
>
> In programming, a **call site** of a function or subroutine is the location (line of code) where the function is called (or may be called, through dynamic dispatch. A call site is where zero or more arguments are passed to the function, and zero or more return values are received.
>
> 在编程中，一个函数或者子例程的调用点是指函数被调用的位置(代码的行)（或者通过动态分发可能被调用的位置）。一个调用点是一个或多个实参被传递给函数，一个或多个返回值被接受的位置。

内联方法可能隐去生成代码增长，但是如我们能够用合理的方式使用它，他将会给与我们更高的性能，特别是对于循环内的可变调用点。

#### 不内联标识

在一种情况下，如果你只想传递给一个内联函数的lambda表达式部分内联，你可以使用`noline`修饰符标记你的部分函数参数。

```kotlin
inline fun foo(inlined: () -> Unit, noinline noteInlined: ()-> Unit){
  // ...
}
```

内联的lambda表达式仅允许在内联函数中调用或者将其传递给一个可以内联的实参。但是`noinline`的lambda表达式可以用你喜欢的任何方式操作：存入成员变量，循环调用等。

注意，如果一个内联函数没有可以内联的函数参数，且没有引用类型参数，编译器将生成一个警告，因为这样的内联函数通常不会产生任何收益（如果你确定需要使用内联，你可以抑制这个警告）。

#### 非局部返回

在Kotlin中，我们可以只用一个普通的，无条件的`return`来退出一个具名函数或者一个匿名函数。这意味着，当我们想从一个Lambda表达式退出时，我们必须使用标签`label`来返回，只有`return`在lambda是禁止的，因为一个lambda表达式不能产生封闭函数的返回。

```kotlin
fun foo() {
  oridinaryFunction {
    return // ERROR: 不能再这里返回。
  }
}
```

但如果lambda作为参数传递的函数是内联的，因为返回值可以内联，所以这么写是允许的

```kotlin
fun foo() {
  inlineFunction {
    return // OK: 这个lambda被内联到foo中。
  }
}
```

这样位于lambda中，但是退出到包函数的返回被称为非本地返回。我们可以在循环（循环结构中经常包裹了内联）中使用这种结构：

```kotlin
fun hasZeros(ints: List<Int>): Boolean {
  ints.forEach {
    if (it==0) return true
  }
  return false
}
```

注意，有一些内联函数可以`不`直接从`该函数体中调用`作为参数传递给他的`lambda表达式`，而是从`其他执行结构中调用`，比如局部对象或者嵌入函数。在这种情况下，lambda的非局部控制结构一样不被允许。为了指明这一现象，对应的lambda把参数需要使用`crossinline`修饰符标记。

```kotlin
inline fun f(crossinline body: () -> Unit) {
  val f = object: Runnable {
    override fun run() = body();
  }
}
```

,break和continue在行内lambda中还不能用，但我们稍后应该会加入它。

#### 实例化类型参数

有时，我们需要讲一个类型做为参数传给给我们

```kotlin
fun <T> TreeNode.findParentOfType(clazz: Class<T>): T? {
	var p = parent
	while(p != null && !clazz.isInstance(p)){
    	p = p.parent
	}
    @Suppress("UNCHECKED_CAST")
    return p as T?
}
```

在这里，我们编译 一个tree，并使用反射来检查是否每一个节点都有一个特定类型，但是调用不是而别的优雅

```kotlin
treeNode.findParentOfType(MyTreeNode::class.java)
```

我们实际上想要的是简单地讲一个类型传递给这个函数，所以，我们可以这样调用

```kotlin
treeNode.findParentOfType<MyTreeNode>()
```

为了让内联函数支持实例化类型参数，我们可以这样写：

```kotlin
inline fun <reified T> TreeNode.findParentOfType(): T? {
    var p = parent
    while (p != null && p !is T) {
        p = p.parent
    }
    return p as T?
}
```

我们使用`reified`修饰符来获取类型参数的试用资格，现在可以在函数内使用类型参数了，几乎就像在普通的类中一样。因为函数是内联的，所以不需要反射，正常的操作比如`!is`和`as`工作的很好。通常我们也可以用上面提到的方式调用它：

```kotlin
myTree.findParentOfType<MyTreeNodeType>().
```

虽然在很多案例中，我们并不需要反射，但是我们依然可以在使用类型参数时使用它

```kotlin
inline fun <reified T> membersOf() = T::class.members

fun main(s: Array<String>) {
    println(membersOf<StringBuilder>().joinToString("\n"))
}
```

普通函数（未标记为内联）不能拥有具象化参数。一个没有运行时表示的类型（比如一个非具体类型参数，或者函数类型比如`Nothing`）不能作为具体类型参数的实参使用。

更底层的描述，可以查看[spec document](https://github.com/JetBrains/kotlin/blob/master/spec-docs/reified-type-parameters.md).

#### 内联属性

内联修饰符可以在没有后台字段的属性访问器上使用。可以注释单个属性访问器:

```kotlin
val foo: Foo
    inline get() = Foo()

var bar: Bar
    get() = ...
    inline set(v) { ... }
```

您还可以注释整个属性,它将其两个访问器标记为内联:

```kotlin
inline var bar: Bar
    get() = ...
    set(v) { ... }
```

在调用点,内联访问者会想常规内联函数一样被内联。

