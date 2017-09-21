# AOP实例

### 对整个类进行拦截

1. 拦截操作类继承通知接口
   * 通知(Advice)之前 - 该方法执行前运行 
   * 通知(Advice)返回之后 – 运行后，该方法返回一个结果 
   * 通知(Advice)抛出之后 – 运行方法抛出异常后
   * 环绕通知 – 环绕方法执行运行，结合以上这三个通知
2. 配置被拦截类spring bean配置
3. 配置拦截操作类bean配置
4. 配置代理工程类 org.springframework.aop.framework.ProxyFactoryBean 
   * 指定其属性target为被拦截类
   * 拦截操作类interceptorNames类list 添加value为拦截操作器类

> 以上方法仅能对整个被拦截类进行拦截操作
>
> 该处理有一个基本认知，即在方法的声明周期内都会产生通知Advice



## 对类的方法进行拦截

> 属于介绍
>
> * Advice – 指示之前或方法执行后采取的行动。 
> * Yiibaicut – 指明哪些方法应该拦截，通过方法的名称或正则表达式模式。
> * Advisor – 分组"通知"和”切入点“成为一个单元，并把它传递到代理工厂对象。

1. 声明切入点
   - 声明切入点class   org.springframework.aop.support.NameMatchMethodYiibaicut
   - 声明属性mappedName 映射名称的 value为被拦截方法
2. 声明advisor
   * 声明advisor关联类 class org.springframework.aop.support.DefaultYiibaicutAdviso
     * 声明切入点属性 pointcut 为已声明的切入点
     * 声明操作属性advice为已实现对应接口的属性
   * 声明advisor关联类 class org.springframework.aop.support.RegexpMethodYiibaicutAdvisor
     * 声明切入点正则规则 pattern 属性列表 添加正则匹配值
     * 声明操作属性advice为以实现对应接口的属性
3. 声明代理器类和代理器类的target
   * 声明target属性为被拦截类的bean id
   * 声明interceptorNames拦截器名称属性列表 添加advisor bean

