---

title: 关于nacos

date: 2020-11-24 14:33:17

---
##### 基础特点

1. 

##### 常见问题

1. 



##### Docker 启动

```sh
docker run -d \
-e MODE=standalone \
-e SPRING_DATASOURCE_PLATFORM=mysql \
-e MYSQL_SERVICE_HOST=10.32.7.86 \
-e MYSQL_SERVICE_PORT=3306 \
-e MYSQL_SERVICE_USER=root \
-e MYSQL_SERVICE_PASSWORD=1234 \
-e MYSQL_SERVICE_DB_NAME=nacos_config \
-p 8848:8848 \
--restart=always \
--name mynacos \
--net=bridge  \
nacos/nacos-server
```




##### 好玩的
```lua
-- 配合各种Escape Sequence
string.rep('hahaha\t', 10)
```
##### 相关介绍



##### 参考
