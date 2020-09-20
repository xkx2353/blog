---

title: Dubbo学习和对于RPC的理解

date: 2020-09-18 15:42:17

---

##### 什么是RPC

> Remote Procedure Call 远程过程调用，就是一种通过网络从远程计算机程序上请求服务，而不需要了解底层网络技术的协议。满足分布式系统架构中不同的系统之间的远程通信和相互调用

- 从通信协议来看可以分为：
基于HTTP协议：Rest（Json）、Hession（binary）；
基于TCP协议（以Netty高性能网络框架，然后使用自己实现的高效率应用协议）。
 
- 从语言和平台理解：
 仅支持特定语言： Java的RMI；
 可以跨平台：HTTP Rest、Thrift。
 
- 从调用过程来看
 同步调用;
 异步调用。
 
- 不同的序列化协议
 序列化：将对象转成二进制流;
 反序列化：将二进制流转换成对象。
 使用不同的序列化框架对RPC的性能有影响,所以要选择高效率的序列化框架，考虑速度，空间，能否跨语言
 
- 网络协议，网络模型，线程模型
 
- RPC安全服务接口的鉴权和访问控制
 
- 常见的RPC框架：
> 
> Dubbo alibaba Java
> 
> Motan weibo Java
> 
> Tars tencent C++
> 
> gRPC google multi language
> 
> Thrift facebook multi language
> 
> Spring Cloud 全套的微服务下的服务治理方案
> 

##### 架构

- Provider
- Consumer 从提供者地址列表中，基于软负载均衡算法，选一台提供者进行调用，如果调用失败，再选另一台调用。
- Registry 服务注册与发现的注册中心，注册中心返回服务提供者地址列表给消费者，如果有变更，注册中心将基于长连接推送变更数据给消费者；负责服务地址的注册与查找，相当于目录服务，服务提供者和消费者只在启动时与注册中心交互，注册中心不转发请求，压力较小。
- Monitor 统计服务的调用次数和调用时间的监控中心，异步
- Container

> 连通性 注册中心，服务提供者，服务消费者三者之间均为长连接，监控中心除外
> 
> 健壮性  注册中心和监控中心全部宕机，不影响已运行的提供者和消费者，消费者在本地缓存了提供者列表
> 注册中心和监控中心都是可选的，服务消费者可以直连服务提供者
> 
> 伸缩性
> 注册中心为对等集群，可动态增加机器部署实例，所有客户端将自动发现新的注册中心
服务提供者无状态，可动态增加机器部署实例，注册中心将推送新的服务提供者信息给消费者

##### 源码结构模块

> 写在前面
> 
> dubbo的基本设计原则是采用URL作为配置信息的统一格式，所有拓展点都通过传递URL携带配置信息。
> 以URL为总线，运行过程中所有的状态数据信息都可以通过URL来获取，比如当前系统采用什么序列化，采用什么通信，采用什么负载均衡等信息，都是通过URL的参数来呈现的，所以在框架运行过程中，运行到某个阶段需要相应的数据，都可以通过对应的Key从URL的参数列表中获取

###### 注册中心模块

> dubbo的注册中心实现有Multicast、Zookeeper、Redis、Nacos、Consul、Etcd、Eureka、Sofa、Simple，这个模块就是封装了dubbo所支持的注册中心的实现。官方推荐Zookeeper。

###### 集群模块
> 将多个服务提供方伪装为一个提供方，包括：负载均衡, 容错，路由等，集群的地址列表可以是静态配置的，也可以是由注册中心下发。

###### 公共逻辑模块
> 包括 Util 类和通用模型
> 

###### 配置模块
> Dubbo对外的 API，用户通过 Config 使用Dubbo，隐藏 Dubbo 所有细节。用户都是使用配置来使用dubbo，dubbo也提供了四种配置方式，包括XML配置、属性配置、API配置、注解配置，配置模块就是实现了这四种配置的功能。
> 

###### 远程调用模块
> 抽象各种协议，以及动态代理，只包含一对一的调用，不关心集群的管理。远程调用，最主要的肯定是协议，dubbo提供了许许多多的协议实现，不过官方推荐时使用dubbo自己的协议，还给出了一份性能测试报告。这个模块依赖于dubbo-remoting模块，抽象了各类的协议
> 

###### 远程通信模块
> 相当于 Dubbo 协议的实现，如果 RPC 用 RMI协议则不需要使用此包。提供了多种客户端和服务端通信功能，比如基于Grizzly、Netty、Etcd3、Http、Mina等等，RPC用除了RMI的协议都要用到此模块。
> 

###### 容器模块
>  因为后台服务不需要Tomcat/JBoss 等 Web 容器的功能，不需要用这些厚实的容器去加载服务提供方，既资源浪费，又增加复杂度。服务容器只是一个简单的Main方法，加载一些内置的容器，也支持扩展容器。
> 

###### 监控模块
>  统计服务调用次数，调用时间的，调用链跟踪的服务。这个模块很清楚，就是对服务的监控。
> 

###### 过滤器模块
> 内置的一些过滤器：提供缓存过滤器。提供参数验证过滤器。
> 

###### 插件模块
>  内置的插件：提供了在线运维的命令。
> 

###### 序列化模块
>  封装了各类序列化框架的支持实现。序列化也支持扩展。
> 

###### SPI机制
> Jdk的SPI，Dubbo的SPI，SpringBoot的SPI
>
>SPI的全名为Service Provider Interface，面向对象的设计里面，模块之间推荐基于接口编程，而不是对实现类进行硬编码，这样做也是为了模块设计的可拔插原则。
>为了在模块装配的时候不在程序里指明是哪个实现，就需要一种服务发现的机制。
>
>jdk的spi就是为某个接口寻找服务实现。jdk提供了服务实现查找的工具类：java.util.ServiceLoader，它会去加载META-INF/service/目录下的配置文件。
>JDK标准的SPI只能通过遍历来查找扩展点和实例化，有可能导致一次性加载所有的扩展点，如果不是所有的扩展点都被用到，就会导致资源的浪费。
>
>Dubbo把SPI配置文件中扩展实现的格式修改，例如META-INF/dubbo/com.xxx.Protocol里的com.foo.XxxProtocol格式改为了xxx = com.foo.XxxProtocol这种以键值对的形式，这样做的目的是为了让我们更容易的定位到问题，
>
>
>dubbo的SPI机制增加了对IOC、AOP的支持，一个扩展点可以直接通过setter注入到其他扩展点。
>
>
>
>
1. @SPI，在某个接口上加上@SPI注解后，表明该接口为可扩展接口。如下:

```java
//在Protocol上有@SPI("dubbo")注解。
// 而这个protocol属性值或者默认值会被当作该接口的实现类中的一个key，
// dubbo会去META-INF/dubbo/internal/com.alibaba.dubbo.rpc.Protocol文件中找该key对应的value
@SPI("dubbo")
public interface Protocol {

}
```

2. @Adaptive，该注解为了保证dubbo在内部调用具体实现的时候不是硬编码来指定引用哪个实现，也就是为了适配一个接口的多种实现。
这样做符合模块接口设计的可插拔原则，也增加了整个框架的灵活性，该注解也实现了扩展点自动装配的特性。
- 实现类上面加上@Adaptive注解，表明该实现类是该接口的适配器。
dubbo中的ExtensionFactory接口就有一个实现类AdaptiveExtensionFactory，加了@Adaptive注解，AdaptiveExtensionFactory就不提供具体业务支持，用来适配ExtensionFactory的SpiExtensionFactory和SpringExtensionFactory这两种实现。AdaptiveExtensionFactory会根据在运行时的一些状态来选择具体调用ExtensionFactory的哪个实现，
- 在接口方法上加@Adaptive注解，dubbo会动态生成适配器类。有该注解的方法的参数必须包含URL，ExtensionLoader会通过createAdaptiveExtensionClassCode方法动态生成一个Transporter$Adaptive类。
```java
//@Adaptive注解中有一些key值，比如connect方法的注解中有两个key，分别为“client”和“transporter”，
// URL会首先去取client对应的value来作为注解@SPI中写到的key值，
// 如果为空，则去取transporter对应的value，
// 如果还是为空，则会根据SPI默认的key，也就是netty去调用扩展的实现类，
// 如果@SPI没有设定默认值，则会抛出IllegalStateException异常。
@SPI("netty")
public interface Transporter {
    /**
     * Connect to a server.
     */
    @Adaptive({Constants.CLIENT_KEY, Constants.TRANSPORTER_KEY})
    Client connect(URL url, ChannelHandler handler) throws RemotingException;
```

3. @Activate，扩展点自动激活加载的注解，
就是用条件来控制该扩展点实现是否被自动激活加载，在扩展实现类上面使用，实现了扩展点自动激活的特性，它可以设置两个参数，分别是group和value。

```java
//META-INF/dubbo/com.alibaba.dubbo.rpc.Filter，内容为：
//clearXbootContext=com.xboot.core.config.dubbo.ClearXbootContextFilter
//当配置了xxx参数，并且参数为有效值时激活，比如配了cache="lru"，自动激活CacheFilter。
@Activate(group = "provider", value = "xxx", order = -10000) // 只对提供方激活，group可选"provider"或"consumer"
public class ClearXbootContextFilter implements Filter {
    // ...
}
```


##### 好玩的
```lua
-- 配合各种Escape Sequence
string.rep('hahaha\t', 10)
```
##### 相关介绍




##### 参考
- https://dubbo.apache.org/zh-cn/docs/user/quick-start.html
- https://segmentfault.com/a/1190000016741532

