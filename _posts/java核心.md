---

title: Java核心知识理解记录

date: 2020-09-27 15:47:17

---
##### 关于编译期修改语法树

> 要是对于编译原理有不错的基础的话看这个应该就很简单了
> 编译期处理
> java使用AbstractProcessor、编译时注解和JCTree实现自动修改class文件
>
> 1. 提供了待处理的抽象语法树
> 2. TreeMaker 封装了创建AST节点的一些方法
> 3. Names 提供了创建标识符的方法

- 利用JDK的注解处理器，可在编译期间对注解处理，可以读取、修改、添加抽象语法树中的任意元素。利用注解处理器可以干涉编译器的行为，只要有足够的创意，可以利用注解处理器实现许多原本只能在编码中完成的事情。
- AST(抽象语法树)是一种用来描述程序代码语法结构的树形表示方式，语法树的每一个节点都代表着程序代码中的一个语法结构，例如包、类型、修饰符、运算符、接口、返回值，甚至代码注释等都可以是一个语法结构。



##### 好玩的
```lua
-- 配合各种Escape Sequence
string.rep('hahaha\t', 10)
```
##### 相关介绍



##### 参考

- [java注解处理器——在编译期修改语法树](https://blog.csdn.net/A_zhenzhen/article/details/86065063)

- [JCTree 分析](https://blog.csdn.net/u013998373/article/details/90050810)

- [HelloProcessor](https://gist.github.com/pietrocaselani/8624554)

- [修改语法树](https://segmentfault.com/a/1190000022157161)