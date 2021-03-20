---

title: 参数校验

date: 2021-02-23 10:52:49

---
### 规范与实现

> JSR对于Bean Validation有一套规范
>
> Hibernate Validator是对规范的实现
>
> Spring中的Validation是对于Hibernate Validator的封装，一个适配转换「或者说提供一些别的功能」的过程
> 

### 使用

> @Valid 是Bean Validation的注解
>
> @Validated 是Spring的注解

### 自定义拓展

> 自定义注解
>
> 自定义校验逻辑
>
> 两种方式，一种就是上面的那种（自己写注解，字节写校验逻辑）
>
> 还有一种就是在Spring的支持下写自己的校验接口，参考：https://github.com/LyashenkoGS/spring-mvc-and-jms-validation-POC



### 参考

- https://developer.ibm.com/zh/articles/j-lo-beanvalid/
- https://blog.csdn.net/shenchaohao12321/article/details/100163991
- https://segmentfault.com/a/1190000023471742

