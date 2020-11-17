---

title: 微服务下的网关

date: 2020-09-15 10:02:13

---

> 以Spring Cloud Gateway为主熟悉一个高性能网关的组成，功能。
>
> 以及云原生网关，springCloudGateway的性能虽然比zull提高了不少，但是好像还是不行。

#####  Api网关介绍

> api网关的本质

**流量总入口，得以集中控制！**

api网关协议上最基本要支持HTTP 和 WebSocket，功能强大点的更会支持tcp/udp的负载均衡接入
正因为支持的是http协议，所以api网关不仅仅可以作为 RESTful API 接入，接入带页面的web都可以的，完全可以当成一个web负载均衡器使用

> api网关的作用

解决：认证、鉴权、安全、流量管控、缓存、服务路由，协议转换、服务编排、熔断、灰度发布、监控报警等问题
本质上，流量从我过，我就可以做想做的控制，上面列的就是我需要的控制。

##### 好玩的
```lua
-- 配合各种Escape Sequence
string.rep('hahaha\t', 10)
```
##### 相关介绍

##### 参考
- [开源API网关，你选对了么？](https://www.debugger.wiki/article/html/1578582035559256)
- https://www.lua.org/manual/5.3/manual.html

