---
title: Spring中的一些细节
date: 2020-09-13 19:10:13
---

##### spring容器启动的一个过程（refresh过程）
1. 准备资源以及校验必要参数，property source；
2. 创建基本ioc容器，BeanDefinition加载入IOC容器中；
3. 设置beanFactory的基本属性（类加载器，表达式解析器，添加beanPostProcessor,添加忽略自动装配的接口，添加特定bean对应的依赖注册environment，systemProperties，systemEnvironment）
4. 添加一些beanPostProcessor；
5. 就是先查找BeanDefinitionRegistryPostProcessor接口实现并执行postProcessBeanDefinitionRegistry方法，然后再查找执行BeanFactoryPostProcessor接口的实现，并执行postProcessBeanFactory方法；
6. 注册BeanPostProcessor实现类；
7. 初始化消息资源的默认空处理DelegatingMessageSource；
8. 初始化事件广播器SimpleApplicationEventMulticaster；
9. 实例化web服务，比如ServletContext上下文配置，tomcat初始化配置等（ServletWebServerApplicationContext的实现）；
10. 查找注册ApplicationListener实现的监听器；
11. 遍历BeanFacotry当中的BeanDefinition数据，然后调用getBean方法来实例化及依赖注入等，完成容器所有bean（non-lazy-init）的实例化工作；
12. 完成刷新工作，清除缓存；注册LifecycleProcessor的实现DefaultLifecycleProcessor并调用onRefresh方法；发布刷新事件；启动web服务等。在整个refresh方法的最后，就是调用各种清除缓存的方法了。

##### spring中Bean的一个生命周期
> 实例化和初始化的区别
> 实例化：是对象创建的过程。比如使用构造方法new对象，为对象在内存中分配空间；
> 初始化：是为对象中的属性赋值的过程；
> 这个初始化和bean中的init-method初始化方法是两个概念
> 这里仅仅考虑单例bean
1. 实例化前置处理 InstantiationAwareBeanPostProcessor#postProcessBeforeInstantiation
2. 实例化
3. 实例化后置处理 InstantiationAwareBeanPostProcessor#postProcessAfterInstantiation
4. 实例化后，属性设置前调用 InstantiationAwareBeanPostProcessor#postProcessProperties
5. 初始化（属性注入）
6. setBeanName（BeanNameAware）
7. setBeanClassLoader（BeanClassLoaderAware）
8. setBeanFactory（BeanFactoryAware）
9. 调用初始化方法前置处理BeanPostProcessor#applyBeanPostProcessorsBeforeInitialization
10. 属性注入完成后调用InitializingBean#afterPropertiesSet
10. 调用初始化方法invokeInitMethods
11. 调用初始化方法后置处理applyBeanPostProcessorsAfterInitialization
12. 放入缓存池
13. 容器销毁DisposableBean#destory
14. 指定的destroy-method

##### spring中单例是如何实现的


##### spring中是如何解决循环依赖的
> spring处理循环依赖的三种情况：
> 构造器的循环依赖：这种依赖spring是处理不了的，直接抛BeanCurrentlylnCreationException异常； 
> 单例模式下的setter循环依赖：通过“三级缓存”处理循环依赖；
> 非单例循环依赖：无法处理；

单例模式下setter处理
```java

/** 
* Cache of singleton objects: bean name –> bean instance
* 完成初始化的单例对象的cache（一级缓存）
*/
private final Map singletonObjects = new ConcurrentHashMap(256);

/** 
* Cache of singleton factories: bean name –> ObjectFactory
* 进入实例化阶段的单例对象工厂的cache
* allowEarlyReference=true
* 加入singletonFactories三级缓存的前提是执行了构造器
*/
private final Map> singletonFactories = new HashMap>(16);

/** 
* Cache of early singleton objects: bean name –> bean instance
* 完成实例化但是尚未初始化（属性填充）的，提前暴光的单例对象的Cache 
*/
private final Map earlySingletonObjects = new HashMap(16);

/** Names of beans that are currently in creation. */
// 这个缓存也十分重要，它表示bean创建过程中都会在里面呆着
// 它在Bean开始创建时放值，创建完成时会将其移出
private final Set<String> singletonsCurrentlyInCreation = Collections.newSetFromMap(new ConcurrentHashMap<>(16));

```

##### ignoreDependencyInterface和ignoreDependencyType
> ignoreDependencyInterface 自动装配时忽略该接口实现类中和setter方法入参相同的类型，也就是忽略该接口实现类中存在依赖外部的bean属性注入。\
> ignoreDependencyType 自动装配时忽略某个类或者接口的实现

##### 关于BeanDefinition
>  A BeanDefinition describes a bean instance, which has property values,
 constructor argument values
 This is just a minimal interface: The main intention is to allow a
 {BeanFactoryPostProcessor} to introspect and modify property values
 and other bean metadata.

1. BeanDefinition及抽象基类属性介绍
	```latex
	 scope
   role
   parentName
   beanClassName
   autowireMode
   lazyInit
   String... dependsOn,the names of the beans that this bean depends on being initialized
   	autowireCandidate,whether this bean is a candidate for getting autowired into some     other bean
   primary,whether this bean is a primary autowire candidate
   instanceSupplier:
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
  ```

  ```

2. Full-fledged classes

   ```latex
   RootBeanDefinition,
   ChildBeanDefinition,
   GenericBeanDefinition,
    
   ```
```

3. 注解相关AnnotatedBeanDefinition

   ```latex
   AnnotatedGenericBeanDefinition,
   ScannedGenericBeanDefinition,
   ConfigurationClassBeanDefinition
```


##### id和name属性都可以在XML元数据中唯一标识某个Bean实例
1. 通常情况下，id用于定义Bean实例的名称(beanName)，而name用于定义Bean实例的别名(aliases)。一个Bean实例可以有多个别名，在name属性中用半角逗号或分号分隔各个别名即可；
2. bean元素的id和name属性都不是必填项，但如果配置了id或name，其属性值不能与另一个bean元素的id或name属性值重复，必须保证当前Bean在整个容器环境中的唯一性；
3. 通常bean元素指定id属性值是最常用的一种配置方式。对于这种方式，容器会将id值当作beanName来完成后续的解析和编组工作。如果不指定id，也可通过name属性来配置bean；
4. 如果只通过name属性标识bean元素，则容器将name属性解析到aliases数组后又会取出第一个别名值用作beanName（将这个属性从别名中移除，然后设置成beanName）；
5. id和name属性可以联合使用，并且同一个bean元素中的id和name属性值可以重复（但不能与其它bean元素的id或name属性值重复）；
6. 唯一的必填项是class属性；
7. 既没有配置id，也没有配置name，只有配置class，则生成根据class全限定名称和不同的策略进行生成，内部bean使用{className}#{BeanDefinition实例的16进制hashCode码}，普通bean使用{className}#{BeanDefinition实例的16进制hashCode码}，使用class的全限定名称来作为别名；

##### BeanDefinition中的autowireMode和@Autowire，@Resource注入的时候的byType和 byName区别

> 本质上都是自动装配，就是在使用XML配置和使用注解来自动装配Bean的属性
>

``` java
// autowireMode
public interface AutowireCapableBeanFactory extends BeanFactory {
	/**
	 * Constant that indicates no externally defined autowiring. Note that
	 * BeanFactoryAware etc and annotation-driven injection will still be applied.
	 * 通过显式设置ref 属性来进行装配(xml)
	 */
	int AUTOWIRE_NO = 0;
	/**
	 * indicates autowiring bean properties by name
	 * (applying to all bean property setters).
	 */
	int AUTOWIRE_BY_NAME = 1;
	/**
	 * indicates autowiring bean properties by type
	 * (applying to all bean property setters).
	 */
	int AUTOWIRE_BY_TYPE = 2;
	/**
	 * indicates autowiring the greediest constructor that
	 * can be satisfied (involves resolving the appropriate constructor).
	 * 也是要根据constructor中的参数type
	 */
	int AUTOWIRE_CONSTRUCTOR = 3;
	/**
	 * indicates determining an appropriate autowire strategy
	 * through introspection of the bean class.
	 * @deprecated as of Spring 3.0: If you are using mixed autowiring strategies,
	 * prefer annotation-based autowiring for clearer demarcation of autowiring needs.
	 */
	@Deprecated
	int AUTOWIRE_AUTODETECT = 4;
```

##### 不同自动注入的区别
> @Autowired根据类型来自动注入，如果有多个可以配合@Qualifier，或者可以使用@Primary，或者可以使用@Priority注解进行优先级排序（有个int类型的属性value，可以配置优先级大小。数字越小的，就被优先匹配）
>
> 在Spring3.0之后，有效的自动装配策略分为`byType、byName、constructor`三种方式。注解Autowired默认使用byType来自动装配，如果存在类型的多个实例，先通过Primary和Priority注解来确定，如果也确定不了，最后通过byName。

>
>@Resource如果指定了name属性, 那么就按name属性的名称装配;如果没有指定name属性, 那就按照要注入对象的字段名查找依赖对象;如果按默认名称查找不到依赖对象, 那么就按照类型查找。

##### Bean实例化策略(createBeanInstance)

Supplier回调方式
工厂方法初始化
构造函数自动注入初始化
默认构造函数初始化:
	如果该bean没有配置lookup-method、replace-method或者@LookUp注解，则直接通过反射的方式实例化bean对象即可；
	如果存在覆盖，则需要使用CGLIB进行动态代理，因为在创建代理的同时可将动态方法织入类中；

##### DependencyDescriptor 源码
##### 好玩的

``` lua
-- 配合各种Escape Sequence
string.rep('hahaha\t', 10)

```




##### 参考
- https://juejin.im/post/6844903959136567310
- https://blog.csdn.net/weixin_41927463/article/details/106089245
- https://juejin.im/post/6844904142750613511
- https://cloud.tencent.com/developer/article/1497692


