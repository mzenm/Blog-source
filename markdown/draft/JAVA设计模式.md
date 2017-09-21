# JAVA设计模式

### 工厂模式 Factory

> 工厂模式即生产者模式，调用类函数生成某接口类型的对象

​	![工厂模式](E:/markdown/resource/Java/factory.jpg)

### 抽象工厂模式

> 抽象工厂是生产工厂的工厂

![抽象工厂模式](E:/markdown/resource/Java/abstractFactory.jpg)

### 单例模式

> 单例模式将实例赋值给类变量，在类声明完成即在类中完成了实例化。另一方面将构造器设置为禁止外部访问，最终只能透过类方法获取类变量，所以所有该类的引用都指向了同一个对象，称之为单例模式。

![单例模式](E:/markdown/resource/Java/singleObject.jpg)

Builder模式

> 使用简单对象并使用逐步方法构建复杂对象。即多种零件类型，逐个拼接，从小组件大组件，大组件拼接成完整机械。即Builder建筑工模式

​	![单例模式](E:/markdown/resource/Java/builder.jpg)

### 原型模式

> 原型模式是为了降低在昂贵的操作之后对对象进行缓存，来减少对数据库的调用。专门提供一个Cache类，当完成昂贵的操作之后将结果保存在cache类的静态变量中map 方便下次调用，而不用再去调用昂贵操作。

![原型](E:/markdown/resource/Java/prototype.jpg)



### 适配器模式

> 适配器模式，相当于一个外置的设备。好比你的电脑无法播放正常处理某些格式的图像文件，但是使用其他设备就可以。这种情况下，就建立一个实现当前设备接口的适配器类，然后在这个类中操纵其他可用设备。实现使用。

![适配器](E:/markdown/resource/Java/adapter.jpg)