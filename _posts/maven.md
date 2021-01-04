---

title: MAVEN

date: 2020-12-22 20:40:17

---
##### 关键词

- 生命周期分为三部分
  - clean
  - default
  - site
- 阶段 每一部分有一些阶段组成，比如clean部分包含三个阶段：pre-clean，clean，post-clean
  - 调用maven的阶段时，会顺序执行**本周期**当前阶段之前的所有阶段，直至当前阶段
  - 那么各个阶段到底是如何执行的呢？Maven内部是通过**插件**中的**目标**来实现。
- 目标 
  - 一个插件中包括若干目标，而每个目标会执行一个特定的任务task。例如：mvn  clean  使用的参数是**clean**阶段，而实际执行的是maven-clean-plugin插件的clean目标

##### 使用

可使用`插件名:目标名`的参数形式直接运行某插件的某目标。

例如：mvn mybatis-generator:generate

执行了 mybatis-generator 插件的 generate目标



阶段(phase)和插件目标(goal)可以同时使用

mvn clean dependency:copy-dependencies package

以上命令执行了clean周期的pre-clean和clean阶段，dependency插件的copy-dependencies目标，default周期package阶段及package之前的所有阶段



maven插件中包含goal。这些**goal可以被绑定到不同的maven的构建阶段**上。goal表示一个特定的任务，是一个比构建阶段更细粒度的构建步骤，用于构建和管理当前项目。

要使用maven插件提供的goal，首先得知道这些插件提供了哪些goal以及他们的用法。

maven的核心插件之一 --- help插件（Maven Help Plugin）可以用于查看插件提供了哪些goal

mvn help:describe -Dplugin=org.mybatis.generator:mybatis-generator-maven-plugin:1.3.2 -Ddetail



help插件的goal： `mvn help:describe -Dplugin=help` 

有8个goal，可以自行查看

help:active-profiles
列出当前构建中活动的Profile（项目的，用户的，全局的）。
help:effective-pom
显示当前构建的实际POM，包含活动的Profile。
help:effective-settings
打印出项目的实际settings, 包括从全局的settings和用户级别settings继承的
配置。
help:describe
描述插件的属性。它不需要在项目目录下运行。但是你必须提供你想要描述插件
的 groupId 和 artifactId。

maven的设计思想



##### 相关介绍



##### 参考

- https://blog.csdn.net/just4you/article/details/74225386
- https://www.cnblogs.com/yitouniu/p/7573885.html