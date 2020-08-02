---
title: Nginx
date: 2020-07-31 11:39:45
top: 9999
---
![nginx-logo](https://azou.tech/blog/static/image/nginx-logo.png)

#### Nginx出现在哪里

> 执行流程，module的功能在特定的流程处理阶段得到执行，和conf里面的配置顺序没有关系
- <span style="color: #00802b;"><strong>POST_READ</strong></span> The `ngx_http_realip_module` registers its handler at this phase to enable substitution of client addresses before any other module is invoked
-  <span style="color: #00802b;"><strong>SERVER_REWRITE</strong></span>  Rewrite directives defined in a server block (but outside a location block) are processed
-  <span style="color: #00802b;"><strong>FIND_CONFIG</strong></span> Special phase where a location is chosen based on the request URI
- <span style="color: #00802b;"><strong>REWRITE</strong></span> For rewrite rules defined in the location, chosen in the `FIND_CONFIG` phase
-  <span style="color: #00802b;"><strong>POST_REWRITE</strong></span> Special phase where the request is redirected to a new location if its URI changed during a rewrite
-  <span style="color: #00802b;"><strong>PREACCESS</strong></span>  Common phase for different types of handlers, not associated with access control. The standard nginx modules [ngx_http_limit_conn_module ](http://nginx.org/en/docs/http/ngx_http_limit_conn_module.html)and [ngx_http_limit_req_module](http://nginx.org/en/docs/http/ngx_http_limit_req_module.html) register their handlers at this phase.
-  <span style="color: #00802b;"><strong>ACCESS</strong></span> Phase where it is verified that the client is authorized to make the request. Standard nginx modules such as [ngx_http_access_module](http://nginx.org/en/docs/http/ngx_http_access_module.html) and [ngx_http_auth_basic_module ](http://nginx.org/en/docs/http/ngx_http_auth_basic_module.html)register their handlers at this phase.
-  <span style="color: #00802b;"><strong>POST_ACCESS</strong></span> Special phase where the [satisfy any](http://nginx.org/en/docs/http/ngx_http_core_module.html#satisfy) directive is processed. If some access phase handlers denied access and none explicitly allowed it, the request is finalized. No additional handlers can be registered at this phase
-  <span style="color: #00802b;"><strong>PRECONTENT</strong></span> Handlers to be called prior to generating content. Standard modules such as [ngx_http_try_files_module](http://nginx.org/en/docs/http/ngx_http_core_module.html#try_files) and [ngx_http_mirror_module](http://nginx.org/en/docs/http/ngx_http_mirror_module.html) register their handlers at this phase.
- <span style="color: #00802b;"><strong>CONTENT</strong></span> Response is normally generated. Multiple nginx standard modules register their handlers at this phase, including [ngx_http_index_module](http://nginx.org/en/docs/http/ngx_http_index_module.html) or `ngx_http_static_module`
-  <span style="color: #00802b;"><strong>LOG</strong></span> Request logging is performed. Currently, only the [ngx_http_log_module](http://nginx.org/en/docs/http/ngx_http_log_module.html) registers its handler at this stage for access logging. Log phase handlers are called at the very end of request processing, right before freeing the request


> Modules

- realip[POST_READ PHASE]
  - break 停止执行rewrite模块中的指令集，影响其他模块的指令执行。

- rewrite
  - break 停止执行rewrite模块中的指令集，影响其他模块的指令执行
  - if  变量，正则表达式，文件，路径，是否可执行，和shell中有点类似
  - return code [text]; return code URL; return URL
  	- 301 permanent  may incorrectly sometimes be changed to a GET method.
  	- 302 temporary some old clients were incorrectly changing the method to `GET`: the behavior with non-GET methods and 302 is then unpredictable on the Web, whereas the behavior with 307 is predictable. For GET requests, their behavior is identical.
		- 303 temporary This response code is usually sent back as a result of `PUT`or `POST`. The method used to display this redirected page is always `GET`
		- 307 temporary guarantees that the method and the body will not be changed
		- 308 permanent  request method and the body will not be altered
  - rewrite 使用特定的正则表达式来匹配请求的uri，如果匹配成功，则可以更改uri。last,break
  - rewrite_log
  - set
- location
  - match rules
    - nginx first checks locations defined using the prefix strings (prefix locations). Among them, the location with the **longest matching prefix is selected and remembered**. 
    - Then regular expressions are checked, in **the order of their appearance in the configuration file**. The search of regular expressions terminates on the first match, and the corresponding configuration is used. 
    - If no match with a regular expression is found then **the configuration of the prefix location remembered earlier is used.**


#### Nginx优秀的第三方扩展
1. Openresty

#### 熟悉HTTP
1. web服务器

---


> 参考
- [nginx-org](http://nginx.org/en/docs/)

