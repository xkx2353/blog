---
title: 关于Tomcat[一个web应用服务器相关的边边角角]

date: 2020-09-01 16:18:19
---
### 架构设计

![tomcat_arch](https://azou.tech/blog/static/image/tomcat_arch.jpg)

#### 组件设计 

> 从简单到复杂
>

- 服务抽象Server，代表整个servlet容器
- 一个Server包含多个Service（相互独立，共享一个JVM和系统类库），一个Service服务维护多个Connector和Container
- 网络协议处理Connector
- 请求处理Engine
- Host虚拟主机，可以包含多个Context
- Context表示一个Web应用
- 一个Web应用中可以包含多个Servlet实例处理不同链接的请求，Servlet定义为Wrapper，一个Servlet就是一个Wrapper

> 容器的概念，容器就是用来处理请求的，只是容器内部会有不同的组件。所以将Engine，Host，Context，Wrapper都可以继承自容器，通过Container来来维护组件之间的父子关系




#### 生命周期管理

### Catalina

> 2001年，Tomcat发布了重要的具有里程碑意义的版本4.0，该版本，Tomcat完全重新设计了气Servlet容器的架构。该部分工作由Craig McClanahan「Servlet及JSP规范专家组成员，Apache Struts作者，Apache Tomcat Servlet容器Catalina的架构师」完成，他将这个新版本的Servlet容器实现命名为Catalina。
> 

#### Catalina Web应用的加载过程

#### Catalina Web应用的请求过程

### Coyote

> Coyote是Tomcat链接器框架的名称，是Tomcat服务器提供的供客户端访问的外部接口。客户端通过Coyote与服务器建议连接、发送请求并接收响应。
>
> 封装了底层的网络通信，为Catalina容器提供了统一的接口，是Catalina容器与具体的请求协议及I/O方式解耦。
>
> 作为独立的模块，只负责具体协议和I/O的处理，与Servlet规范实现没有直接关系，因此即便是Request和Response对象也没有实现Servlet规范对应的接口，而是在Catalina中将它们进行进一步封装为ServletRequest和ServletResponse。

#### Connector组件

> 主要任务是负责接收浏览器的发过来的 tcp 连接请求，创建一个 Request 和 Response 对象分别用于和请求端交换数据，然后会产生一个线程来处理这个请求并把产生的 Request 和 Response 对象传给处理这个请求的线程，处理这个请求的线程就是 Container 组件要做的事了。
>
> 整个Connector组件包含三部分「拿Nio来举例」：Http11NioProtocol、Mapper、CoyoteAdapter。Http11NioProtocol包含NioEndpoint和Http11ConnectionHandler，NioEndpoint是Http11NioProtocol中负责接收处理socket的主要模块；Http11ConnectionHandler是连接处理器。NioEndpoint主要是实现了socket请求监听线程Acceptor、socket NIO poller线程、以及请求处理线程池。

### SpringBoot内置

> Spring容器和Servlet容器的关联
>
>  

- WebServer和Server的区别『Spring中和Tomcat源码中定义的区分』
  - WebServer，是Spring中的一个概念，代表一个完整的可配置的Web应用服务器，例如Tomcat，Jetty，Netty等
  - Server，是Tomcat中的一个概念，代表Catalina容器「来处理请求」

- 内置Tomcat启动过程
  - 创建WebServer「createWebServer」
    - 初始化一些在Spring环境中的配置，同时将Spring环境中对于WebServer的回调引入到Tomcat的生命周期中「比如回调配置DispatcherServlet」
    - Catalina相关：创建Server，Service，Engine，Host，Wrapper；
    - Coyote相关：创建Connector，EndPoint
  - 启动WebServer
    - 开始端口监听

### EndPoint 

#### NIO相关类
- ServerSocketChannel 开启  设置 ServerSocketChannel 为阻塞模式

- 设置 acceptor 和 poller 的数量

- NioSelectorPool

- Acceptor AcceptorState

  - 封装成 SocketChannel同时将它注册到 Poller 上。因为我们知道，acceptor 应该尽可能的简单，只做 accept 的工作，简单处理下就往后面扔。acceptor 还得回到之前的循环去 accept 新的连接呢。

    我们只需要明白，此时，往 poller 中注册了一个 NioChannel 实例，此实例包含客户端过来的 SocketChannel 和一个 SocketBufferHandler 实例。

- Poller

  - 每个 poller 关联了一个 Selector。
  - PollerEvent
  - Processor SocketProcessor 

#### 使用NIO

> Tomcat当启用NIO的时候，却配置serverSocketChannel.blocking=true

考虑我们为什么使用NIO,而放弃BIO.因为BIO的accept是个阻塞方法，而且write和read操作也是阻塞的。因此我们只能当新连接到来时去创建新线程出处理这个连接，**这样带来的最大问题是不能同时处理大量连接，因为大量连接带来的是创建很多线程，线程成本很高，大量线程很容易让操作系统崩溃，而且虽然并发度很高，但是很多线程都在空转，很多时间都浪费在线程空跑和线程切换上，效率很差。**

但事实上，处理连接的操作不必放在后台线程中，因为后台线程很可能会处理连接建立不及时，不如将其置于主线程中，增加并发度（虽然优势并不是十分明显）。我们需要重点关心的是连接建立后获得的与客户端交互的那个socket，他的操作必须是非阻塞的，这很显然。因为在处理长连接的时候，我们往往关心的是在本次连接之内数据的读写

因此tomcat的这种处理方式还是不错的。当然将server socket接收连接设置为非阻塞也是可以的，但是没有必要

[NIO案例](https://www.cnblogs.com/zhuawang/p/3843723.html)

#### NIO中的选择器Selector

> NIO中的选择器依赖操作系统内核的这些系统调用

选择器的宏观理解
“有这么一种检查员，她工作在养鸡场，每天的工作就是不停的查看特定的鸡舍，如果有鸡生蛋了，或者需要喂食，或者有鸡生病了，就把相应信息记录下来，这样一来，鸡舍负责人想知道鸡舍的情况，只需要到检查员那里查询即可，当然，鸡舍负责人得事先告知检查员去查询哪些鸡舍。“

实际上选择器为通道服务，通道事先告诉选择器：“我对某些事件感兴趣，如可读、可写等“，选择器在接受了一个或多个通道的委托后，开始选择工作，它的选择工作就完全交给操作系统，linux下即为poll或epoll

##### SelectionKey

> 每次通道注册到选择器上时都会创建一个SelectionKey储存在该选择器上，该SelectionKey保存了注册的通道、注册的选择器、通道事件类型操作符等信息。
>
> 源码：https://www.cnblogs.com/gaofei200/p/13974084.html


#### NIO提高效率的关键

![tomcat_connector](https://azou.tech/blog/static/image/Tomcat-NIO-Connector.jpg)

> BIO： Acceptor接收socket，然后从Worker线程池中找出空闲的线程处理socket，如果worker线程池没有空闲线程，则Acceptor将阻塞。其中Worker是Tomcat自带的线程池，如果通过\<Executor>配置了其他线程池，原理与Worker类似
>
> NIO：Acceptor接收socket后，不是直接使用Worker中的线程处理请求，而是先将请求发送给了Poller，而Poller是实现NIO的关键。Acceptor向Poller发送请求通过队列实现，使用了典型的生产者-消费者模式。在Poller中，维护了一个Selector对象；当Poller从队列中取出socket后，注册到该Selector中；然后通过遍历Selector，找出其中可读的socket，并使用Worker中的线程处理相应请求。与BIO类似，Worker也可以被自定义的线程池代替。
>
> 在NIOEndpoint处理请求的过程中，无论是Acceptor接收socket，还是线程处理请求，使用的仍然是阻塞方式；但在“读取socket并交给Worker中的线程”的这个过程中，使用非阻塞的NIO实现，这是NIO模式与BIO模式的最主要区别（其他区别对性能影响较小，暂时略去不提）。而这个区别，在并发量较大的情形下可以带来Tomcat效率的显著提升：
>
> 目前大多数HTTP请求使用的是长连接（HTTP/1.1默认keep-alive为true），而长连接意味着，一个TCP的socket在当前请求结束后，如果没有新的请求到来，socket不会立马释放，而是等timeout后再释放。如果使用BIO，“读取socket并交给Worker中的线程”这个过程是阻塞的，也就意味着在socket等待下一个请求或等待释放的过程中，处理这个socket的工作线程会一直被占用，无法释放；因此Tomcat可以同时处理的socket数目不能超过最大线程数，性能受到了极大限制。而使用NIO，“读取socket并交给Worker中的线程”这个过程是非阻塞的，当socket在等待下一个请求或等待释放时，并不会占用工作线程，因此Tomcat可以同时处理的socket数目远大于最大线程数，并发性能大大提高。

#####SelectorProvider

#### 协议处理相关类

- SocketWrapperBase
- SocketProcessorBase
- ConnectionHandler
- Processor
- Http11Processor

### 线程池

### 异步SERVLET



### 单独列出来

#### SynchronizedStack作用

#### LimitLatch

> Tomcat使用LimitLatch类实现了对请求连接数的控制
>
> https://blog.csdn.net/BryantLmm/article/details/85068066


##### 好玩的
```lua
厚道的人 运气都不会太差--[Xiaomi]
```



##### 参考
- https://programmer.help/blogs/tomcat-configuration-and-principle.html
- https://www.javadoop.com/post/tomcat-nio
- https://www.cnblogs.com/kismetv/p/7806063.html
- https://www.baeldung.com/spring-webflux-concurrency

