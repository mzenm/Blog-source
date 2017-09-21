# Netty入门

### 概念

​	Channel代表一个用于**连接到实体**如硬件设备、文件、网络套件字或程序组件，能够**执行**一个或多个不同的**I/O操作**(例如读或写)的**开放连接**。Netty 内部使用回调处理事件时，一旦这样的回调被触发，事件可以由接口 ChannelHandler 的实现来处理。

​	Future代表一个**异步操作结果**的**占位符**,它将在**将来**的某个时候**完成并提供结果**。Future 提供了另外一种通知应用操作已经完成的*方式*。JDK自带的Future所提供的的实现只允许您手动该检查是否完成或者阻塞了。所以 Netty 提供自己了的实现ChannelFuture用于在执行异步操作时使用。

​	ChannelFuture 提供多个附件方法来允许一个或者多个 ChannelFutureListener 实例。这个回调方法 operationComplete() 会在操作完成时调用。每个 Netty 的 outbound I/O 操作都会返回一个 ChannelFuture;这样就不会阻塞。

​	

1. 当channel异步连接到远程对等节点时，调用立即返回并提供ChannelFuture;
2. 操作完成后通知注册一个ChannelFutureListener.
3. 当 operationComplete() 调用时检查操作的状态。
4. 如果成功就创建一个 ByteBuf 来保存数据。
5. 异步发送数据到远程。再次返回ChannelFuture。
6. 如果有一个错误则抛出 Throwable,描述错误原因。



图1.3显示了一个事件可以由一连串的事件处理器来处理

![事件流](https://waylau.gitbooks.io/essential-netty-in-action/images/Figure%201.3%20Event%20Flow.jpg)

- ChannelHandler 是给不同类型的事件调用
- 应用程序实现或扩展 ChannelHandler 挂接到事件生命周期和提供自定义应用逻辑。

​	当一个新的连接被接受，一个新的子 Channel 将被创建， ChannelInitializer 会添加EchoServerHandler
 的实例到 Channel 的 ChannelPipeline。正如我们如前所述，如果有入站信息，这个处理器将被通知。

​	*每个 Channel 都有一个关联的 ChannelPipeline，它代表了 ChannelHandler 实例的链。适配器处理的实现只是将一个处理方法调用转发到链中的下一个处理器。因此，如果一个 Netty 应用程序不覆盖exceptionCaught ，那么这些错误将最终到达 ChannelPipeline，并且结束警告将被记录。出于这个原因，你应该提供至少一个 实现 exceptionCaught 的 ChannelHandler。*



### 服务器主代码组件

- EchoServerHandler 实现了的业务逻辑
- 在 main() 方法，引导了服务器。

##### 执行后者需要的步骤是

- 创建 ServerBootstrap 实例来引导服务器并随后绑定
- 创建并分配一个 NioEventLoopGroup 实例来处理事件的处理，如接受新的连接和读/写数据。
- 指定本地 InetSocketAddress 给服务器绑定
- 通过 EchoServerHandler 实例给每一个新的 Channel 初始化
- 最后调用 ServerBootstrap.bind() 绑定服务器

​	