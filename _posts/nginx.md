---
title: Nginx
date: 2020-07-31 11:39:45
top: 9999
---
![nginx-logo](https://azou.tech/blog/static/image/nginx-logo.png)

#### Nginx出现在哪里

> 执行流程，module的功能在特定的流程处理阶段得到执行，和conf里面的配置顺序没有关系
- POST_READ The `ngx_http_realip_module` registers its handler at this phase to enable substitution of client addresses before any other module is invoked
- SERVER_REWRITE
- FIND_CONFIG
- REWRITE
- POST_REWRITE
- PREACCESS
- ACCESS
- POST_ACCESS
- PRECONTENT
- CONTENT
- LOG


> core module 

- realip[POST_READ PHASE]
  - break 停止执行rewrite模块中的指令集，影响其他模块的指令执行。

- rewrite
  - break 停止执行rewrite模块中的指令集，影响其他模块的指令执行
  - if  变量，正则表达式，文件，路径，是否可执行，和shell中有点类似
  - return code [text]; return code URL; return URL
  - rewrite 使用特定的正则表达式来匹配请求的uri，如果匹配成功，则可以更改uri。last,break
  - rewrite_log
  - set



#### Nginx优秀的第三方扩展
1. Openresty

#### 熟悉HTTP
1. web服务器

---


> 参考
- [nginx-org](http://nginx.org/en/docs/)

