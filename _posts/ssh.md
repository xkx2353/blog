---

title: 关于SSH

date: 2020-10-28 18:11:19

---
##### ssh知识相关
- SSH只是一种协议，存在多种实现，既有商业实现，也有开源实现；OpenSSH，PuTTY「Windows」
-  的

##### ssh服务端配置

##### ssh端口转发：ssh隧道

> 1、保护tcp会话，保护会话中明文传输的内容。
>
> 2、绕过防火墙或者穿透到内网，访问对应的服务。

- 本地端口转发（Local Port Forwarding）

	ssh -N -f -g  -L localhost:20000:localhost:49160 azou@azou.tech
	
	**访问本地的20000端口，通过ssh隧道，访问的是azou.tech的49160端口**

- 远程端口转发（Remote Port Forwarding）

	ssh -N -g -f -R 18098:localhost:8098 azou.tech
	
	**访问的是azou.tech的18098端口，通过ssh隧道，访问本地的8098端口**
	
- 动态端口转发（Dynamic Port Forwarding）

  ssh -g -f -N -D localhost:20002 azou.tech

  这条命令创建了一个SOCKS代理，所以通过该SOCKS代理发出的数据包将经过azou.tech转发出去。

  测试验证的时候我在浏览器配置好代理就是不生效，也不知道为什么，但是在终端进行了验证：

  curl -x socks5h://localhost:20002 http://localhost:49160



##### HTTP代理，SOCKS代理，VPN的区别

HTTP代理：能够代理客户机的HTTP访问，主要是代理浏览器访问网页。

SOCKS代理与其他类型的代理不同，它只是简单地传递数据包，而并不关心是何种应用协议，既可以是HTTP请求，所以SOCKS代理服务器比其他类型的代理服务器速度要快得多。SOCKS代理又分为SOCKS4和SOCKS5，二者不同的是SOCKS4代理只支持TCP协议（即传输控制协议），而SOCKS5代理则既支持TCP协议又支持UDP协议（即用户数据包协议），还支持各种身份验证机制、服务器端域名解析等。**Socks代理通常只是代理电脑中的某一个具体的程序**

VPN（虚拟专网），你接入VPN就是接入了一个专有网络，那么你访问网络都是从这个专有网络的出口出去，好比你在家，你家路由器后面的网络设备是在同一个网络，而VPN则是让你的设备进入了另一个网络。同时你的IP地址也变成了由VPN分配的一个IP地址。通常是一个私网地址。你和VPN服务器之间的通信是否加密取决于连接VPN的具体方式/协议。



##### HTTP 隧道代理

为什么需要HTTP隧道？

HTTP tunnel是HTTP/1.1中引入的一个功能，主要为了解决明文的HTTP proxy无法代理跑在TLS中的流量（也就是https）的问题，同时**提供了作为任意流量的TCP通道的能力。**

在HTTP tunnel出来之前，HTTP proxy工作在中间人模式。
也就是说在一次请求中，客户端（浏览器）明文的请求代理服务器。代理服务器明文去请求远端服务器（网站），拿到返回结果，再将返回结果返回给客户端。整个过程对代理服务器来说都是可见的，代理能看到你要请求的path，你请求中的header（包括例如auth头里面的用户名密码），代理也能看到网站返回给你的cookie。和中间人攻击的中间人是一种情况

HTTP tunnel以及CONNECT报文解决了这个问题，代理服务器不再作为中间人，不再改写浏览器的请求，而是把浏览器和远端服务器之间通信的数据原样透传，这样浏览器就可以直接和远端服务器进行TLS握手并传输加密的数据。



连接过程：

（a）客户端先发送 CONNECT 请求到隧道代理服务器，告诉它建立和服务器的 TCP 连接（因为是 TCP 连接，只需要 ip 和端口就行，不需要关注上层的协议类型）
（b，c）代理服务器成功和后端服务器建立 TCP 连接
（d）代理服务器返回 HTTP 200 Connection Established 报文，告诉客户端连接已经成功建立
（e）这个时候就建立起了连接，所有发给代理的 TCP 报文都会直接转发，从而实现服务器和客户端的通信

`CONNECT` 请求的内容和其他 HTTP 方法的语法一样，只不过它在状态栏（status line）指定了真正服务器的地址。请求 URI 替换成了 hostname 和 port 的字符串

如：CONNECT realserver.com:443 HTTP/1.0，其他 HTTP 请求的状态栏对应位置是路径地址：GET /about HTTP/1.0

需要注意的是，客户端应该尽量少地暴露其他信息，最好只有状态栏一行的内容，因为 `CONNECT` 请求是没有经过加密的



**可以将代 理的认证支持与隧道配合使用，对客户端使用隧道的权利进行认证**

**隧道网关无法验证目前使用的协议是否就是它原本打算经过隧道传输的协议，为了降低对隧道的滥用，网关应该只为特定的知名端口，比如HTTPS的端口443， 打开隧道**

##### 好玩的
```lua	
-- 什么是灿烂
```



##### 参考
- http://www.ruanyifeng.com/blog/2011/12/ssh_remote_login.html
- https://www.gudaos.com/981.html
- https://cizixs.com/2017/03/22/http-tunnel-proxy-and-golang-implementation/
- https://blog.csdn.net/qq_41453285/article/details/98587279

