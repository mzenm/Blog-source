# 注解

## 什么是注解

​	注解（Annotation），也叫（`metadata`）元数据。一种代码级别的说明。它是JDK1.5及以后版本引入的一个特性，与类、接口、枚举是在同一个层次。它可以声明在包、类、字段、方法、局部变量、方法参数等的前面，用来对这些元素进行说明，注释。这些信息被存储在Annotation的“name=value”结构对中。

那么你可能又有疑问什么是（`metadata`）元数据呢？

这个解释起来比较抽象，我的理解是：“数据的数据”。

那它有什么作用呢，如果要对于元数据的作用进行分类，还没有明确的定义，不过我们可以根据它所起的作用，大致可分为三类：

- 编写文档：通过代码里标识的元数据生成文档。
- 代码分析：通过代码里标识的元数据对代码进行分析。
- 编译检查：通过代码里标识的元数据让编译器能实现基本的编译检查



<!-- more -->

## 注解的分类

### 注解的分类

根据注解的参数个数分类：

- 标记注解。一个没有成员的 Annotation，这种类型仅仅使用自身的存在与否来为我们提供信息。标记注解非常常见，比如上面所说的 @Override
- 单值注解。成员的参数为单个参数
- 完整注解。成员的参数为多个参数

根据注解使用的方法和用途分类：

- JDK内置系统注解
- 元注解
- 自定义注解

​	它们都不会直接影响到程序的语义，只是作为注解（标识）存在，我们可以通过反射机制编程实现对这些元数据（用来描述数据的数据）的访问。另外，你可以在编译时选择代码里的注解是否只存在于源代码级，或者它也能在class文件、或者运行时中出现（SOURCE/CLASS/RUNTIME）。下面我具体根据注解使用的方法和用途分类来讲解。



### 内置注解

- @Override  
- @Deprecated  
- @SuppressWarnings  

----

##### **@Override**

```java
@Override
protected void onStart() {
  super.onStart();
}
```

```java
package java.lang;

import java.lang.annotation.*;

/**
 * 指定一个方法准备重写一个超类的方法体。如果一个方法带有这样的注释，
 * 在至少满足以下状态中至少一个的情况下，编译器才不会生成一个错误消息
 * 
 * 方法的确重写或者声明了一个声明在超类中的方法。
 * 该方法包含一个签名，此签名等同于重写了声明在其他链接对象中的任何一个
 * 公有方法
 */
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.SOURCE)
public @interface Override {
}
```



**@Deprecated** 

```java
public class User {

    @Deprecated
    public static String getName(){
        return "user";
    }

}
```

```java
package java.lang;

import java.lang.annotation.*;
import static java.lang.annotation.ElementType.*;

/**
 * 如果一个程序元素被标示为@Deprecated意味着这个不建立程序员使用这个元素。
 * 通常因为这个元素的使用存在危险或者有更好的替代品存在。当一个被弃用的程序
 * 元素被在一个未被弃用的代码中使用或者复写时，编译器发出一个警告
 */
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target(value={CONSTRUCTOR, FIELD, LOCAL_VARIABLE, METHOD, PACKAGE, PARAMETER, TYPE})
public @interface Deprecated {
}

```

**@SuppressWarnings **

```java
public class User {

    @SuppressWarnings("deprecation")
    public static int getAge() {
        return 18;
    }

}
```

```java
package java.lang;

import java.lang.annotation.*;
import static java.lang.annotation.ElementType.*;

/**
 * 指示在带有该注释的元素(以及包含在该元素中的所有程序元素)中的(指定的)编译器警告应该被抑制。
 * 注意在给定元素中 抑制警告 的集合是该元素 包含 的所有程序元素的 抑制警告 的 超集。比方说如果
 * 你在一个类上抑制了某个警告，在其内部的方法上抑制了另一个警告，那么对于这个方法，两个警告
 * 都会被抑制。
 * 为了获得良好的编程风格，程序员应该总在最深层嵌套的元素中使用该注释，因为这样更有效率。如果
 * 你想要为抑制在一个特定方法中的警告，就应该注释这个方法而不是他的类。
 */
@Target({TYPE, FIELD, METHOD, PARAMETER, CONSTRUCTOR, LOCAL_VARIABLE})
@Retention(RetentionPolicy.SOURCE)
public @interface SuppressWarnings {
    /**
     * 在被标注元素中被编译器抑制的警告集合。该集合允许重复的警告名称。第二个和后续
     * 的同名警告将被忽略。不能被识别的警告并不会引起错误：编译器会忽视它们无法识别的
     * 任何错误名称。然而如果一个注释包含一种不能识别的警告名，编译器将自由的发出警告
     * 
     * 可以使用{@code "unchecked"}来抑制未检查的错误。编译器提供商应该在文档中指明
     * 该注释支持的附加警告种类。我们鼓励供应商合作来确保在多种编译器下同样的命名正常
     * 生效
     */
    String[] value();
}

```



### 元注解

元注解就是`定义注解`的注解，由Java API提供，

- @Target
- @Retention
- @Documented 
- @Inherited  



**@Target**

​	用于描述注解的使用范围。修饰的对象范围：packages、types（类、接口、枚举、Annotation类型）、类型成员（方法、构造方法、成员变量、枚举值）、方法参数和本地变量（如 catch 参数）。

```java
package java.lang.annotation;

/**
 * 指定当前上下文中可以使用那种注释类型。关于在声明的上下文和内容上下文中哪种注释类型
 * 可以使用 在  JLS 9.6.4.1 中说明，并且在 ElementType 中作为枚举常量储量。
 *
 * 如果一个注释类型{@code T}上不存在{@code @Target}元注解，那么该声明类型可以作为修
 * 饰符写在除了类型元素声明中的所有声明上。
 *
 * 如果存在{ @code }元注释,则编译器将根据jls 9.7.4 ,强制执行{ @ code elementtype}枚
 * 举常量指示的使用限制。
 * 
 * 比如说{@code @Target}元注释指示被声明的类型本事就是一个元注释类型。这种情况只能用在
 * 注释类型的声明上
 * @Target(ElementType.ANNOTATION_TYPE)
 * public @interface MetaAnnotationType{}
 * 
 * 下面的{@code @Target}元注释指示被声明类型仅用来复杂注释类型的声明中，它本身不能直接
 * 注释任何东西。
 * @Target({})
 * public @interface MetaAnnotationType{}
 *
 * 在一个{@code @Target} 注释中,单一的{@code ElementType}长岭不能出现1次以上，否则
 * 将产生一个编译错误。
 * @Target({ElementType.FIELD, ElementType.METHOD, ElementType.FIELD})
 * public @interface MetaAnnotationType{}
 */
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.ANNOTATION_TYPE)
public @interface Target {
    /**
     * 返回可以使用 当前注释类型 的 元素种类 的数组上。
     */
    ElementType[] value();
}
```

```java
package java.lang.annotation;

/**
 * 这个枚举类型的常量提供了注释可能出现在java程序中 语法位置 的简单分类。这些常量
 * 可以用在{@link Target java.lang.annotation.Target}元注释中，用来标示在给定类型
 * 上添加注释是合法的。
 *
 * 这些注释可能出现的语法定位被分为 声明上下文 和 类型上下文。声明上下文中注释用来声明
 * ，而类型上下文中，注释应用于声明和表达式中使用的类型。
 *
 * 常量{#ANNOTATION_TYPE} , {#CONSTRUCTOR} , {#FIELD} , {#LOCAL_VARIABLE} ，
 * , {#METHOD} , {#PACKAGE} ,{#PARAMETER} , {#TYPE} 和 { #TYPE_PARAMETER} 
 * 在JLS 9.6.4.1中都是声明上下文。
 *
 * 比方说， 带有@Target(ElementType.FIELD)元注释的注释类型只可以作为一个域声明的
 * 修饰符。
 * 
 * 常量 {#TYPE_USE} 对应于jls中的15个类型上下文,以及两个声明上下文:
 * 类型声明(包括注释类型声明)和类型参数声明。
 * 
 * 比方说一个带有元数据类型{ @Target(ElementType.TYPE_USE)} 的注释可能可以卸载一个域的
 * 类型上面或者在域的类型中，如果他是嵌入的，参数化的或者数据类型，或者同样可以作为一个
 * 作为类声明的修饰符。
 * 
 * {@code TYPE_USE} 常量包含类型声明和类型参数声明，这为类型检查器设计提供语义到注释类
 * 型的方便。比方说某个注释类型NonNull是带有元注释 {@Target(ElementType.TYPE_USE)}
 * 的注释，那么{@NonNull} {class C {...}} 
 *
 * @author  Joshua Bloch
 * @since 1.5
 * @jls 9.6.4.1 @Target
 * @jls 4.1 The Kinds of Types and Values
 */
public enum ElementType {
    /** Class, interface (including annotation type), or enum declaration */
    TYPE,

    /** Field declaration (includes enum constants) */
    FIELD,

    /** Method declaration */
    METHOD,

    /** Formal parameter declaration */
    PARAMETER,

    /** Constructor declaration */
    CONSTRUCTOR,

    /** Local variable declaration */
    LOCAL_VARIABLE,

    /** Annotation type declaration */
    ANNOTATION_TYPE,

    /** Package declaration */
    PACKAGE,

    /**
     * Type parameter declaration
     *
     * @since 1.8
     */
    TYPE_PARAMETER,

    /**
     * Use of a type
     *
     * @since 1.8
     */
    TYPE_USE
}
```

**@Retention**

​	表示需要在什么级别保存该注释信息，用于描述注解的生命周期（即：被描述的注解在什么范围内有效），定义了该Annotation被保留的时间长短：某些Annotation仅出现在源代码中，而被编译器丢弃；而另一些却被编译在class文件中；编译在class文件中的Annotation可能会被虚拟机忽略，而另一些在class被装载时将被读取。

```java
package java.lang.annotation;

/**
 * Indicates how long annotations with the annotated type are to
 * be retained.  If no Retention annotation is present on
 * an annotation type declaration, the retention policy defaults to
 * {@code RetentionPolicy.CLASS}.
 *
 * <p>A Retention meta-annotation has effect only if the
 * meta-annotated type is used directly for annotation.  It has no
 * effect if the meta-annotated type is used as a member type in
 * another annotation type.
 *
 * @author  Joshua Bloch
 * @since 1.5
 * @jls 9.6.3.2 @Retention
 */
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.ANNOTATION_TYPE)
public @interface Retention {
    /**
     * Returns the retention policy.
     * @return the retention policy
     */
    RetentionPolicy value();
}

```

```java
package java.lang.annotation;

/**
 * Annotation retention policy.  The constants of this enumerated type
 * describe the various policies for retaining annotations.  They are used
 * in conjunction with the {@link Retention} meta-annotation type to specify
 * how long annotations are to be retained.
 *
 * @author  Joshua Bloch
 * @since 1.5
 */
public enum RetentionPolicy {
    /**
     * Annotations are to be discarded by the compiler.
     */
    SOURCE,

    /**
     * Annotations are to be recorded in the class file by the compiler
     * but need not be retained by the VM at run time.  This is the default
     * behavior.
     */
    CLASS,

    /**
     * Annotations are to be recorded in the class file by the compiler and
     * retained by the VM at run time, so they may be read reflectively.
     *
     * @see java.lang.reflect.AnnotatedElement
     */
    RUNTIME
}

```

**@Documented**

​	用于描述其它类型的annotation应该被作为被标注的程序成员的公共API，Documented 是一个标记注解，没有成员。将此注解包含在 
javadoc 中 ，它代表着此注解会被javadoc工具提取成文档。在doc文档中的内容会因为此注解的信息内容不同而不同。

```java
package java.lang.annotation;

/**
 * Indicates that annotations with a type are to be documented by javadoc
 * and similar tools by default.  This type should be used to annotate the
 * declarations of types whose annotations affect the use of annotated
 * elements by their clients.  If a type declaration is annotated with
 * Documented, its annotations become part of the public API
 * of the annotated elements.
 *
 * @author  Joshua Bloch
 * @since 1.5
 */
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.ANNOTATION_TYPE)
public @interface Documented {
}

```



##### **@Inherited**

允许子类继承父类中的注解，是一个标记注解，@Inherited阐述了某个被标注的类型是被继承的。如果一个使用了@Inherited修饰的annotation类型被用于一个class，则这个annotation将被用于该class的子类。

```java
package java.lang.annotation;

/**
 * Indicates that an annotation type is automatically inherited.  If
 * an Inherited meta-annotation is present on an annotation type
 * declaration, and the user queries the annotation type on a class
 * declaration, and the class declaration has no annotation for this type,
 * then the class's superclass will automatically be queried for the
 * annotation type.  This process will be repeated until an annotation for this
 * type is found, or the top of the class hierarchy (Object)
 * is reached.  If no superclass has an annotation for this type, then
 * the query will indicate that the class in question has no such annotation.
 *
 * <p>Note that this meta-annotation type has no effect if the annotated
 * type is used to annotate anything other than a class.  Note also
 * that this meta-annotation only causes annotations to be inherited
 * from superclasses; annotations on implemented interfaces have no
 * effect.
 *
 * @author  Joshua Bloch
 * @since 1.5
 * @jls 9.6.3.3 @Inherited
 */
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.ANNOTATION_TYPE)
public @interface Inherited {
}
```



### 自定义注解

​	使用@interface自定义注解时，自动继承了java.lang.annotation.Annotation接口，由编译程序自动完成其他细节。在定义注解时，不能继承其他的注解或接口。@interface用来声明一个注解，其中的每一个方法实际上是声明了一个配置参数。方法的名称就是参数的名称，返回值类型就是参数的类型（返回值类型只能是基本类型、Class、String、enum）。可以通过default来声明参数的默认值。以上所有例子都属于自定义注解。自定义注解具有以下固定格式：

```java
 public @interface 注解名{注解体}
```

- 所有基本数据类型（int,float,boolean,byte,double,char,long,short)
- String类型
- Class类型
- enum类型
- Annotation类型
- 以上所有类型的数组

注意：只能有public或默认(default)这两个访问权修饰，参数成员只能用以上6种类型，如果只有一个参数成员，最好把参数名称设为”value”。

```java
@Target(ElementType.FIELD)
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface Shade {

    public enum ShadeType {
        Triangle("三角"),
        Four("四边"),
        Five("五角");

        private String type;

        ShadeType(String type) {
            this.type = type;
        }

        @Override
        public String toString() {
            return type;
        }
    }

    public ShadeType shader() default ShadeType.Triangle;

}
```



## 读取注解

​	这里我们将使用反射去读取注解。Java在java.lang.reflect 包下新增了AnnotatedElement接口，该接口代表程序中可以接受注解的程序元素。该接口主要有如下几个方法：

1. 判断该程序元素上是否包含指定类型的注解，存在则返回true，否则返回false。

   ```java
   default boolean isAnnotationPresent(Class<? extends Annotation> annotationClass)
   ```

2. 返回该程序元素上存在的、指定类型的注解，如果该类型注解不存在，则返回null。

   ```java
   <T extends Annotation> T getAnnotation(Class<T> var1);
   ```

3. 返回该程序元素上存在的所有注解。

   ```java
   Annotation[] getAnnotations();
   ```

4. 返回直接存在于此元素上的所有注释。与此接口中的其他方法不同，该方法将忽略继承的注释。（如果没有注释直接存在于此元素上，则返回长度为零的一个数组。）该方法的调用者可以随意修改返回的数组；这不会对其他调用者返回的数组产生任何影响。

   ```java
    Annotation[] getDeclaredAnnotations();
   ```

注意：使用反射去读取注解，必须将Retention的值选为Runtime。

```java
/***********注解声明***************/

/**
 * 水果名称注解
 * @author peida
 *
 */
@Target(ElementType.FIELD)
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface FruitName {
    String value() default "";
}

/**
 * 水果颜色注解
 * @author peida
 *
 */
@Target(ElementType.FIELD)
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface FruitColor {
    /**
     * 颜色枚举
     * @author peida
     *
     */
    public enum Color{ BULE,RED,GREEN};
    
    /**
     * 颜色属性
     * @return
     */
    Color fruitColor() default Color.GREEN;

}

/**
 * 水果供应者注解
 * @author peida
 *
 */
@Target(ElementType.FIELD)
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface FruitProvider {
    /**
     * 供应商编号
     * @return
     */
    public int id() default -1;
    
    /**
     * 供应商名称
     * @return
     */
    public String name() default "";
    
    /**
     * 供应商地址
     * @return
     */
    public String address() default "";
}

/***********注解使用***************/

public class Apple {
    
    @FruitName("Apple")
    private String appleName;
    
    @FruitColor(fruitColor=Color.RED)
    private String appleColor;
    
    @FruitProvider(id=1,name="陕西红富士集团",address="陕西省西安市延安路89号红富士大厦")
    private String appleProvider;
    
    public void setAppleColor(String appleColor) {
        this.appleColor = appleColor;
    }
    public String getAppleColor() {
        return appleColor;
    }
    
    public void setAppleName(String appleName) {
        this.appleName = appleName;
    }
    public String getAppleName() {
        return appleName;
    }
    
    public void setAppleProvider(String appleProvider) {
        this.appleProvider = appleProvider;
    }
    public String getAppleProvider() {
        return appleProvider;
    }
    
    public void displayName(){
        System.out.println("水果的名字是：苹果");
    }
}

/***********注解处理器***************/

public class FruitInfoUtil {
    public static void getFruitInfo(Class<?> clazz){
        
        String strFruitName=" 水果名称：";
        String strFruitColor=" 水果颜色：";
        String strFruitProvicer="供应商信息：";
        
        Field[] fields = clazz.getDeclaredFields();
        
        for(Field field :fields){
            if(field.isAnnotationPresent(FruitName.class)){
                FruitName fruitName = (FruitName) field.getAnnotation(FruitName.class);
                strFruitName=strFruitName+fruitName.value();
                System.out.println(strFruitName);
            }
            else if(field.isAnnotationPresent(FruitColor.class)){
                FruitColor fruitColor= (FruitColor) field.getAnnotation(FruitColor.class);
                strFruitColor=strFruitColor+fruitColor.fruitColor().toString();
                System.out.println(strFruitColor);
            }
            else if(field.isAnnotationPresent(FruitProvider.class)){
                FruitProvider fruitProvider= (FruitProvider) field.getAnnotation(FruitProvider.class);
                strFruitProvicer=" 供应商编号："+fruitProvider.id()+" 供应商名称："+fruitProvider.name()+" 供应商地址："+fruitProvider.address();
                System.out.println(strFruitProvicer);
            }
        }
    }
}

/***********输出结果***************/
public class FruitRun {

    /**
     * @param args
     */
    public static void main(String[] args) {
        
        FruitInfoUtil.getFruitInfo(Apple.class);
        
    }

}

====================================
 水果名称：Apple
 水果颜色：RED
 供应商编号：1 供应商名称：陕西红富士集团 供应商地址：陕西省西安市延安路89号红富士大厦
```

![java注释导图](http://oq3ecl7n4.bkt.clouddn.com/java%E6%B3%A8%E9%87%8A.jpg)