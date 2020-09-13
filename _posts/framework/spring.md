---
title: Spring中的一些细节
date: 2020-09-13 19:10:13
---
##### 关于BeanDefinition
> A BeanDefinition describes a bean instance, which has property values,
 constructor argument values
 This is just a minimal interface: The main intention is to allow a
 {BeanFactoryPostProcessor} to introspect and modify property values
 and other bean metadata.

1. BeanDefinition及抽象基类属性介绍

   ```
   scope
   role
   parentName
   beanClassName
   lazyInit
   String... dependsOn,the names of the beans that this bean depends on being initialized
   	autowireCandidate,whether this bean is a candidate for getting autowired into some     other bean
   primary,whether this bean is a primary autowire candidate
   factoryBeanName
   factoryMethodName
   constructorArgumentValues
   propertyValues
   initMethodName
   destroyMethodName
   description
   resolvableType
   abstractFlag,whether this bean is "abstract", that is, not meant to be instantiated
   resourceDescription
   qualifiers,AutowireCandidateQualifier
   methodOverrides
   synthetic,Set whether this bean definition is 'synthetic', that is, not defined by the     application itself (for example, an infrastructure bean such as a helper for auto-     proxying, created through {<aop:config>})
   
   ```

2. Full-fledged classes

   ```
   RootBeanDefinition,
   ChildBeanDefinition,
   GenericBeanDefinition,
   
   ```

3. 注解相关AnnotatedBeanDefinition

   ```
   AnnotatedGenericBeanDefinition,
   ScannedGenericBeanDefinition,
   ConfigurationClassBeanDefinition
   ```

   





##### BeanDefinition中的autowireMode和@Autowire，@Resource注入的时候的byType和 byName区别







##### 好玩的

```lua
-- 配合各种Escape Sequence
string.rep('hahaha\t', 10)
```




##### 参考
- https://juejin.im/post/6844903959136567310

