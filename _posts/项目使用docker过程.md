---
title: 项目使用docker历程
date:  2020-08-14 09:47:50
---

1. 什么是镜像分层

2. 镜像分类
   ```
   1.一种是类似centos这样的基础镜像，称为基础或根镜像。这些镜像是由Docker公司创建、验证、支持、提供。这样的镜像往往使用单个单词作为名字
   2.还有一种类型，比如ansible/centos7-ansible镜像，它是由Docker用户ansible创建并维护的，带有用户名称为前缀，表明是某用户下的某仓库
   
   ```

3. 数据管理
	
	```
	1.数据卷（Data Volumes）：容器内数据直接映射到本地主机环境
	2.数据卷容器（Data Volume Containers）：使用特定容器维护数据卷
	```
	
	
	
4. 端口映射以及容器和容器之间互相访问

5. Dockerfile

   ```
   A Dockerfile is a text document that contains all the commands a user could call on the command line to assemble an image.
   
   Dockerfile分为四部分：基础镜像信息、维护者信息、镜像操作指令和容器启动时执行指令
   
   
   ```

   - The instruction is **not case-sensitive**. However, convention is for them to be UPPERCASE to distinguish them from arguments more easily.

   - Docker runs instructions in a Dockerfile in order. **A Dockerfile must begin with a FROM instruction**

   - Docker treats lines that *begin* with `#` as a comment. A `#` marker anywhere else in a line is treated as an argument. 

   - There can **only be one `CMD` instruction in a `Dockerfile`**. If you list more than one `CMD` then only the last `CMD` will take effect.

6. Docker Compose. Compose is **a tool for defining and running multi-container Docker applications**. With Compose, you use a YAML file to configure your application’s services. Then, with a single command, you create and start all the services from your configuration.

   Using Compose is basically a three-step process:

   1. Define your app’s environment with a **`Dockerfile`** so it can be reproduced anywhere.
   2. Define the services that make up your app in **`docker-compose.yml`** so they can be run together in an isolated environment.
   3. Run **`docker-compose up`** and Compose starts and runs your entire app

7.
8.ewrwer
9.dfsdfsd
10.dfdsf
11.dsfsd
12.sdfsdf
13.sdfsd
14.

#### 项目引入
> 个人觉着本质就是对于生命周期的理解
1. 项目是使用maven构建，所以找maven插件，找到了这个[Dockerfile Maven](https://github.com/spotify/dockerfile-maven),也发现了一个中文翻译的[VERSION](http://blog.geekidentity.com/java/maven/dockerfile-maven-cn/)；配置本身很简单，但是要明白maven本身的构建生命周期呀这些东西，如何将docker嵌入进去。
2. 




#### 参考

- Docker技术入门与实战（第二版）
- https://docs.docker.com/