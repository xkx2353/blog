---

title: Nginx

date: 2020-07-31 11:39:45

tags: [Nginx Http协议 Web服务器]

categories: [服务端的门口哨兵]

---
![nginx-logo](https://azou.tech/blog/static/image/nginx-logo.png)

#### Nginx出现在哪里

> 执行流程，module的功能在特定的流程处理阶段得到执行，和conf里面的配置顺序没有关系

- <span style="color: #00802b;"><strong>POST_READ</strong></span> The `ngx_http_realip_module` registers its handler at this phase to enable substitution of client addresses before any other module is invoked
-  <span style="color: #00802b;"><strong>SERVER_REWRITE</strong></span> Rewrite directives defined in a server block (but outside a location block) are processed
-  <span style="color: #00802b;"><strong>FIND_CONFIG</strong></span> Special phase where a location is chosen based on the request URI
- <span style="color: #00802b;"><strong>REWRITE</strong></span> For rewrite rules defined in the location, chosen in the `FIND_CONFIG` phase
- <span style="color: #00802b;"><strong>POST_REWRITE</strong></span> Special phase where the request is redirected to a new location if its URI changed during a rewrite
- <span style="color: #00802b;"><strong>PREACCESS</strong></span> Common phase for different types of handlers, not associated with access control. The standard nginx modules [ngx_http_limit_conn_module ](http://nginx.org/en/docs/http/ngx_http_limit_conn_module.html)and [ngx_http_limit_req_module](http://nginx.org/en/docs/http/ngx_http_limit_req_module.html) register their handlers at this phase
- <span style="color: #00802b;"><strong>ACCESS</strong></span> Phase where it is verified that the client is authorized to make the request. Standard nginx modules such as [ngx_http_access_module](http://nginx.org/en/docs/http/ngx_http_access_module.html) and [ngx_http_auth_basic_module ](http://nginx.org/en/docs/http/ngx_http_auth_basic_module.html)register their handlers at this phase
- <span style="color: #00802b;"><strong>POST_ACCESS</strong></span> Special phase where the [satisfy any](http://nginx.org/en/docs/http/ngx_http_core_module.html#satisfy) directive is processed. If some access phase handlers denied access and none explicitly allowed it, the request is finalized. No additional handlers can be registered at this phase
- <span style="color: #00802b;"><strong>PRECONTENT</strong></span> Handlers to be called prior to generating content. Standard modules such as [ngx_http_try_files_module](http://nginx.org/en/docs/http/ngx_http_core_module.html#try_files) and [ngx_http_mirror_module](http://nginx.org/en/docs/http/ngx_http_mirror_module.html) register their handlers at this phase
- <span style="color: #00802b;"><strong>CONTENT</strong></span> Response is normally generated. Multiple nginx standard modules register their handlers at this phase, including [ngx_http_index_module](http://nginx.org/en/docs/http/ngx_http_index_module.html) or `ngx_http_static_module`
- <span style="color: #00802b;"><strong>LOG</strong></span> Request logging is performed. Currently, only the [ngx_http_log_module](http://nginx.org/en/docs/http/ngx_http_log_module.html) registers its handler at this stage for access logging. Log phase handlers are called at the very end of request processing, right before freeing the request


> Modules

- realip[POST_READ PHASE]
  - break 停止执行rewrite模块中的指令集，影响其他模块的指令执行。

- rewrite
  - break 停止执行rewrite模块中的指令集，影响其他模块的指令执行
  - if  变量，正则表达式，文件，路径，是否可执行，和shell中有点类似
  - return code [text]; return code URL; return URL
  	- 301 permanent  may incorrectly sometimes be changed to a GET method
  	- 302 temporary some old clients were incorrectly changing the method to `GET`: the behavior with non-GET methods and 302 is then unpredictable on the Web, whereas the behavior with 307 is predictable. For GET requests, their behavior is identical
		- 303 temporary This response code is usually sent back as a result of `PUT`or `POST`. The method used to display this redirected page is always `GET`
		- 307 temporary guarantees that the method and the body will not be changed
		- 308 permanent  request method and the body will not be altered
  - rewrite 使用特定的正则表达式来匹配请求的uri，如果匹配成功，则可以更改uri。last,break
  - rewrite_log
  - set
- location
  - match rules
    - nginx first checks locations defined using the prefix strings (prefix locations). Among them, the location with the **longest matching prefix is selected and remembered**. 
    - Then regular expressions are checked, in **the order of their appearance in the configuration file**. The search of regular expressions terminates on the first match, and the corresponding configuration is used
    - If no match with a regular expression is found then **the configuration of the prefix location remembered earlier is used**

#### Nginx负载均衡策略

- 随机 random
- 轮询（默认）每个请求按时间顺序逐一分配到不同的后端服务器，如果后端服务器down掉，能自动剔除。
- weight 指定轮询几率，weight和访问比率成正比，用于后端服务器性能不均的情况。权重越高，在被访问的概率越大。
- ip_hash 可以采用ip_hash指令解决这个问题，如果客户已经访问了某个服务器，当用户再次访问时，会将该请求通过哈希算法，自动定位到该服务器。每个请求按访问ip的hash结果分配，这样每个访客固定访问一个后端服务器。
- fair（第三方）按后端服务器的响应时间来分配请求，响应时间短的优先分配。
- url_hash（第三方）使每个url定向到同一个（对应的）后端服务器，后端服务器为缓存时比较有效。
- least_conn（最少连接数）按照保持的连接数。一般来说保持的连接数越多说明处理的任务越多，也是最繁忙的，可以将请求分配给其他机器处理。

##### 故障节点摘除与恢复
- max_fails=number
这个参数决定了多少次请求后端失败后会暂停这个业务节点，不再给它发新的请求，默认值是1。此参数需要配合fail_timeout一起用。

题外话：如何定义失败，有很多种类型，这里因为主要处理HTTP代理，所以更关注proxy_next_upstream。
proxy_next_upstream：主要定义了当服务节点出现状况时，会将请求发给其他节点，也就是定义了怎么算作业务节点失败。

- fail_timeout=time

决定了当Nginx认定这个节点不可用时，暂停多久。不配置默认就是10s。

把上面两个参数联合起来考虑就是：当Nginx发现发送到这个节点上的请求失败了3次的时候，就会把这个节点摘除，摘除时间是30s，30s后才会再次发送请求到这个节点上。

- backup

类似于switch语句中的default，当主要节点都挂了的时候，会把请求打到这个backup节点。这是最后一个救兵了。


#### Nginx优秀的第三方扩展
1. Openresty

#### 熟悉HTTP
1. web服务器

---


> 参考

- [nginx-org](http://nginx.org/en/docs/)
- https://juejin.im/post/6844904019043811342

