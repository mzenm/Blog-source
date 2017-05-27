title: Kotlin类与对象
author: 卫振家
date: 2017-05-27 18:10:36
tags:
---
# 类和对象                                             

## 类和继承

### 类

在 Kotlin 中类用 class 声明：

```Kotlin
class Invoice{
}
```

类的声明包含`类名`，`类头`(指定类型参数，主构造器等等)，以及`类主体`，用大括号包裹。类头和类体是`可选`的；如果没有类体可以省略大括号。

```Kotlin
class Empty
```

### 构造器

在 Kotlin 中类可以有一个主构造器以及多个二级构造器。主构造器是`类头`的一部分：`跟在类名后面`(可以有`可选`的类型参数)。

```Kotlin
class Person constructor(firstName: String){
}
```

如果主构造器没有注解或可见性说明，则 constructor 关键字是可以省略：

```kotlin
//具有可见性说明的主构造器
class Person private constructor(firstName: String){
}

//具有注释的主构造器
 
//无注解、无可见性说明的则主构造器可以省略
class Person(firstName: String){
}
```

主构造器`不能`包含`任意`代码。初始化代码可以放在以 init 做前缀的`初始化块`内

```kotlin
class Customer(name: String){
    init {
        logger,info("Customer initialized with value ${name}")
    }
}
```

注意主构造器的参数可以在初始化块内使用，也可以用在类主体中`属性`（成员变量）的初始化声明：

```kotlin
class Customer(name: String) {
    val customerKry = name.toUpperCase()
}
```

事实上，在 Kotlin 中有更简单的语法来使用主构造器声明并初始化类`属性`：

```kotlin
class Person(val firstName: String, val lastName: String, var age: Int){
}
```

跟普通的类属性一样，在主构造器中的声明的属性也可以是可变或只读。

----

如果构造器有注解或可见性声明，则 constructor 关键字是不可少的，并且可见性应该在前：

```kotlin
class Customer public @inject constructor (name: String) {...}
```

参看[可见性](http://kotlinlang.org/docs/reference/visibility-modifiers.html#constructors)

### 二级构造器

类也可以声明前缀为`constructor`的二级构造器:

```kotlin
class Person { 
    constructor(parent: Person) {
        parent.children.add(this) 
    }
}
```

如果类有主构造器，每个二级构造器都应该委托给那个主构造器，不管直接委托给构造器，还是间接通过另一个二级构造器委托给主构造器。我们使用`this`关键词将一个二级构造器委托给同一个类的其他构造器（主构造器也行哦）：

```kotlin
class Person(val name: String) {
    constructor (name: String, paret: Person) : this(name) {
        parent.children.add(this)
    }
}
```

如果一个`非抽象类`没有声明构造器(主构造器或二级构造器)，它会产生一个`没有参数`的构造器。构造器是 public 。如果你不想你的类有公共的构造器，你就得声明一个空的主构造器：

```kotlin
class DontCreateMe private constructor () {
}
```

> 注意：在 JVM 虚拟机中，如果主构造器的所有参数都有默认值，编译器会生成一个附加的无参的构造器，这个构造器会直接使用这些默认值。这使得 Kotlin 可以更简单的使用像 Jackson 或者 JPA 这样使用无参构造器来创建类实例的库。

```kotlin
class Customer(val customerName: String = "")
```

### 创建类的实例

我们可以像使用普通函数那样使用构造器创建类实例：

```kotlin
val invoice = Invoice()
val customer = Customer("Joe Smith")
```

`注意 Kotlin 没有 new 关键字。`

关于如何创建一个嵌入类，内部类，匿名内部类的实例，被描述在 [Nested classes](https://kotlinlang.org/docs/reference/nested-classes.html).中

### 类成员

类可以包含：

> 构造器和初始化代码块
>
> [函数](https://huanglizhuo.gitbooks.io/kotlin-in-chinese/content/ClassesAndObjects/FunctionsAndLambdas/Functions.md) 
>
> [属性](https://huanglizhuo.gitbooks.io/kotlin-in-chinese/content/ClassesAndObjects/ClassesAndObjects/Properties-and-Filds.md)　
>
> [内部类](https://huanglizhuo.gitbooks.io/kotlin-in-chinese/content/ClassesAndObjects/ClassesAndObjects/NestedClasses.md) 
>
> [对象声明](https://huanglizhuo.gitbooks.io/kotlin-in-chinese/content/ClassesAndObjects/ClassesAndObjects/ObjectExpressicAndDeclarations.md) 

### 继承

Kotlin 中所有的类都有共同的父类 Any ，下面是一个没有父类声明的类：

```kotlin
class Example //　隐式继承于 Any
```

`Any` 不是 `java.lang.Object`；事实上它除了 `equals()`,`hashCode()`以及`toString()`外没有任何成员了。参看[ Java interoperability](https://huanglizhuo.gitbooks.io/kotlin-in-chinese/content/ClassesAndObjects/Java%20interoperability)了解更多详情。

为了声明一个显式的父类，我们需要在类头中，将父类类型放在`:`后面：

```kotlin
open class Base(p: Int)

class Derived(p: Int) : Base(p)
```

如果类有主构造器，则基类可以且必须使用该主构造器的参数，在该主构造器中初始化。

如果类没有主构造器，则必须在每一个构造器中用` super `关键字初始化基类，或者在委托另一个构造器做这件事。注意在这种情形中不同的二级构造器可以调用基类不同的构造方法：

```kotlin
class MyView : View {
    constructor(ctx: Context) : super(ctx) {
    }
    constructor(ctx: Context, attrs: AttributeSet) : super(ctx,attrs) {
    }
}
```

open 注解与java 中的 final相反:它允许别的类继承这个类。默认情形下，kotlin 中所有的类都是 final ,对应 [Effective Java](http://www.oracle.com/technetwork/java/effectivejava-136174.html) ：Design and document for inheritance or else prohibit it.

### 复写方法

像之前提到的，我们在 kotlin 中坚持做明确的事。不像 Java ，kotlin 需要把可以复写的成员都`明确注解`出来，并且`重写`它们：

```kotlin
open class Base {
    open fun v() {}
    fun nv() {}
}

class Derived() : Base() {
    override fun v() {}
}
```

对于 `Derived.v()` 来说 override 注解是必须的。如果没有加的话，编译器会提示。如果`没有` open 注解，像 `Base.nv()` ,在子类中声明一个同样的函数是`不合法`的，要么加 override 要么不要复写。在 final 类(就是没有open注解的类)中，open 类型的成员是不允许的。

标记为 override 的成员是 open的，它可以在子类中被复写。如果你不想被重写就要加 final:

```kotlin
open class AnotherDerived() : Base() {
    final override fun v() {}
}
```

### 复写属性

复写属性的工作方式跟复写方法很像；声明在父类中的属性，如果想要在子类中复写就必须以`override`开头，并且这两个属性必须有可以兼容的类型。但是每个被声明的属性都可以通过初始化函数或者某个属性的getter方法。

```kotlin
open class Foo {
    open val x: Int get { ... }
}

class Bar1 : Foo() {
    override val x: Int = ...
}
```

你也可以用一个`var`属性复写一个`val`属性，反之不然。允许这样是因为，一个`val`属性必须声明一个getter方法，将其复写为一个var属性则另外需要在分支类的声明一个setter方法。

注意：你可以将`override`关键字作为主构造器中声明语法的一部分。

```kotlin
interface Foo {
    val count: Int
}

class Bar1(override val count: Int) : Foo

class Bar2 : Foo {
    override var count: Int = 0
}
```



### 复写规则

在 kotlin 中，实现继承通常遵循如下规则：如果一个类从它的直接父类继承了同一个成员的多个实现，那么它必须复写这个成员并且提供自己的实现(或许只是直接用了继承来的实现)。为表示使用父类中提供的方法我们用 `super<Base>`表示:

```
open class A {
    open fun f () { print("A") }
    fun a() { print("a") }
}

interface B {
    fun f() { print("B") } //接口的成员变量默认是 open 的
    fun b() { print("b") }
}

class C() : A() , B{
    override fun f() {
        super<A>.f()//调用 A.f()
        super<B>.f()//调用 B.f()
    }
}
```

可以同时从 A B 中继承方法，而且 C 继承 a() 或 b() 的实现没有任何问题，因为它们都只有一个实现。但是 f() 有俩个实现，因此我们在 C 中必须复写 f() 并且提供自己的实现来消除歧义。

### 抽象类

一个类或一些成员可能被声明成 abstract 。一个抽象方法在它的类中没有实现方法。记住我们不用给一个抽象类或函数添加 open 注解，它默认是带着的。

我们可以用一个抽象成员去复写一个带 open 注解的非抽象方法。

```
open class Base {
    open fun f() {}
}

abstract class Derived : Base() {
    override abstract fun f()
}
```

### 伴随对象

在 kotlin 中不像 java 或者 C# 它没有静态方法。在大多数情形下，我们建议只用包级别的函数。

如果你要写一个没有实例类就可以调用，且需要访问到类的内部信息的方法，但需要访问到类内部(比如说一个工厂方法)，你可以将其书写成这个类中的一个对象声明的成员。

更高效的方法是，你可以在你的类中声明一个[伴随对象](http://kotlinlang.org/docs/reference/object-declarations.html#companion-objects)，这样你就可以像 java/c# 那样把它当做静态方法调用，只需要它的类名做一个识别就好了



## 属性和字段

### 属性声明

在 Kotlin 中类可以有属性，我们可以使用 var 关键字声明可变属性，或者用 val 关键字声明只读属性。

```
public class Address {     
    public var name: String = ...
      public var street: String = ...
    public var city: String = ...
      public var state: String? = ...
    public var zip: String = ...
}
```

我们可以像使用 java 中的字段那样,通过名字直接使用一个属性：

```
fun copyAddress(address: Address) : Address {
    val result = Address() //在 kotlin 中没有 new 关键字
    result.name = address.name //accessors are called
    result.street = address.street
}
```

### Getter 和 Setter

声明一个属性的完整语法如下：

```
var <propertyName>: <PropertyType> [ = <property_initializer> ]
    <getter>
    <setter>

```

语法中的初始化语句，getter 和 setter 都是可选的。如果属性类型可以从初始化语句或者类的成员函数中推断出来,那么他的类型也是忽略的。

例子：

```
var allByDefault: Int? // 错误: 需要一个初始化语句, 默认实现了 getter 和 setter 方法
var initialized = 1 // 类型为 Int, 默认实现了 getter 和 setter

```

只读属性的声明语法和可变属性的声明语法相比有两点不同: 它以 val 而不是 var 开头，不允许 setter 函数：

```
val simple: Int? // 类型为 Int ，默认实现 getter ，但必须在构造函数中初始化

val inferredType = 1 // 类型为 Int 类型,默认实现 getter

```

我们可以像写普通函数那样在属性声明中自定义的访问器，下面是一个自定义的 getter 的例子:

```
var isEmpty: Boolean
    get() = this.size == 0

```

下面是一个自定义的setter:

```
var stringRepresentation: String
    get() = this.toString()
    set (value) {
        setDataFormString(value) // 格式化字符串,并且将值重新赋值给其他元素
}

```

为了方便起见,setter 方法的参数名是value,你也可以自己任选一个自己喜欢的名称.

如果你需要改变一个访问器的可见性或者给它添加注解，但又不想改变默认的实现，那么你可以定义一个不带函数体的访问器:

```
var settVisibilite: String = "abc"//非空类型必须初始化
    private set // setter 是私有的并且有默认的实现
var setterVithAnnotation: Any?
    @Inject set // 用 Inject 注解 setter

```

### 备用字段

在 kotlin 中类不可以有字段。然而当使用自定义的访问器时有时候需要备用字段。出于这些原因 kotlin 使用 `field` 关键词提供了自动备用字段，

```
var counter = 0 //初始化值会直接写入备用字段
    set(value) {
        if (value >= 0)
            field  = value
    }

```

`field` 关键词只能用于属性的访问器.

编译器会检查访问器的代码,如果使用了备用字段(或者访问器是默认的实现逻辑)，就会自动生成备用字段,否则就不会.

比如下面的例子中就不会有备用字段：

```
val isEmpty: Boolean
    get() = this.size == 0

```

### 备用属性

如果你想要做一些事情但不适合这种 "隐含备用字段" 方案，你可以试着用备用属性的方式：

```
private var _table: Map<String, Int>? = null
public val table: Map<String, Int>
    get() {
    if (_table == null)
        _table = HashMap() //参数类型是推导出来的
        return _table ?: throw AssertionError("Set to null by another thread")
    }

```

综合来讲，这些和 java 很相似，可以避免函数访问私有属性而破坏它的结构

### 编译时常量

那些在编译时就能知道具体值的属性可以使用 `const` 修饰符标记为 *编译时常量*. 这种属性需要同时满足以下条件:

- 在"top-level"声明的 或者 是一个object的成员(Top-level or member of an object)
- 以`String`或基本类型进行初始化
- 没有自定义getter

这种属性可以被当做注解使用:

```
const val SUBSYSTEM_DEPRECATED: String = "This subsystem is deprecated"
@Deprected(SUBSYSTEM_DEPRECATED) fun foo() { ... }

```

### 延迟初始化属性

通常,那些被定义为拥有非空类型的属性,都需要在构造器中初始化.但有时候这并没有那么方便.例如在单元测试中,属性应该通过依赖注入进行初始化,或者通过一个 setup 方法进行初始化.在这种条件下,你不能在构造器中提供一个非空的初始化语句,但是你仍然希望在访问这个属性的时候,避免非空检查.

为了处理这种情况,你可以为这个属性加上 `lateinit` 修饰符

```
public class MyTest {
    lateinit var subject: TestSubject

    @SetUp fun setup() {
        subject = TestSubject()
    }

    @Test fun test() {
        subject.method() 
    }
}
```

这个修饰符只能够被用在类的 var 类型的可变属性定义中,不能用在构造方法中.并且属性不能有自定义的 getter 和 setter访问器.这个属性的类型必须是非空的,同样也不能为一个基本类型.

在一个延迟初始化的属性初始化前访问他,会导致一个特定异常,告诉你访问的时候值还没有初始化.

### 复写属性

参看[复写成员](http://kotlinlang.org/docs/reference/classes.html#overriding-members)

### 代理属性

最常见的属性就是从备用属性中读（或者写）。另一方面，自定义的 getter 和 setter 可以实现属性的任何操作。有些像懒值( lazy values )，根据给定的关键字从 map 中读出，读取数据库，通知一个监听者等等，像这些操作介于 getter setter 模式之间。

像这样常用操作可以通过代理属性作为库来实现。更多请参看[这里](http://kotlinlang.org/docs/reference/delegated-properties.html)。