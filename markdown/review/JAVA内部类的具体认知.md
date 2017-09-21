---
title: JAVA内部类的具体认知

author: 卫振家

tags:

- 内部类
- 静态方法
  categories:
- JAVA
  date: 2017-05-22 10:46:00
---



## JAVA内部类处理

今天遇到一个比较奇怪的代码题：

```java
package cn.hash.hashcode;

public class Outter {
	public void someOuterMethod(){
		//Inner a = new Inner();
	}
	
	public static void main(String[] args){
      
	}
	
	public class Inner{
		public int b = 100;
	}
}
```

就是关于内部类的引用的，关于在一个类的成员方法中实例化内部类时，是否需要首先实例化外部类。那么静态方法是否需要吶。

经过测试

```java
package cn.hash.hashcode;

public class Outter {
	public void someOuterMethod(){
		Inner a = new Inner(); //$1合法
	}
	
	public static void main(String[] args){
      	Inner a = new Inner(); //$2不合法
      
		Outter o = new Outter();
        new o.Inner();//$3不合法
	}
	
	public class Inner{
		public int b = 100;
	}
}
```

$1合法是因为，在类的内部类可以认为是该类的一个成员变量或者成员方法，当在一个类的内部使用类的成员变量或者成员方法是完全可合理的。

$2不合法的原因也很简单，因为类的静态方法只能处理类的静态成员。假设类的静态方法可以处理类的成员变量，因为成员变量是跟随对象一起存在内存对中，静态方法却存储在静态存储区域，当静态方法调用成员变量时是无法判断其纯在的形式的。

$3不合法，这完全是一个语法错误，学习的时候没有好好看，正确的写法如下

```java
o.new Inner();
```

这才是最终的正确写法，如果Inner类是静态类，则可以使用

```java
package cn.hash.hashcode;

public class Outter {
	public void someOuterMethod(){
		Inner a = new Inner();
	}
	
	public static void main(String[] args){
		new Inner();
      	new Outter.Inner();
        new Outter.new Inner();//不合法
	}
	
	public static class Inner{
		public int b = 100;
	}	
}

```

是完全合法的处理，具体原因也是一样的，大家可以从普通的静态成员和非静态成员关系也可以得出以上的结论。

​	类的静态成员被存储在RAM的静态存储区域，句柄（变量）被存储在RAM堆栈中(还有基础类型)，对象被存储在RAM内存堆中。这本身就能说明一些问题。静态成员在最初就被初始化完成，所以static代码块会在类被存储时就运行完毕，这也是可以理解的。

​	句柄(handler)和对象的分栈存储也可以说明为什么，变量类型可变，而实际引用的类却类型不可以变。还是那句话，把变量当做遥控器，对象当做电视机，赋值就是将两者关联。

​	当一个变量超出其生命周期后（主要因为离开了该变量的作用范围{}），对象依旧存在(电视机，通常一个对象可以跟多个句柄相连)，但是已经没有遥控器与之相关联，这是垃圾收集器就会调用其根类型Object的销毁方法，销毁这个对象。