##什么是spring?

Spring是**一个轻量级Java开发框架** 。

两个核心特性，也就是**依赖注入（dependency  injection，DI）和面向切面编程（aspect-oriented programming， AOP）**。

 为了降低Java开发的复杂性，Spring采取了以下4种关键策略 

- 基于POJO的轻量级和最小侵入性编程； 
- 通过依赖注入和面向接口实现松耦合； 
- 基于切面和惯例进行声明式编程； 
- 通过切面和模板减少样板式代码。 



##Spring由哪些模块组成？

*  核心容器（Core Container）：**Beans  + core + context +SpEl**

* **AOP +Aspects + 设备支持（Instrmentation）+消息（Messaging）**

* **数据访问与集成（Data Access/Integeration)  + web + Test **



<img src="https://ljjblog.oss-cn-beijing.aliyuncs.com/img/image-20210906150712057.png" alt="image-20210906150712057" style="zoom:80%;" />

- spring **core**：提供了框架的基本组成部分，包括控**制反转（Inversion of  Control，IOC）和依赖注入（Dependency Injection，DI）**功能。 
- spring **beans**：提供了**BeanFactory**，是工厂模式的一个经典实现，Spring将管 理对象称为Bean。 
- spring **context**：构建于 core 封装包基础上的 context 封装包，提供了一种框 架式的对象访问方法。 
- spring jdbc：提供了一个JDBC的抽象层，消除了烦琐的JDBC编码和数据库厂 商特有的错误代码解析， 用于简化JDBC。 
- spring aop：提供了面向切面的编程实现，让你可以自定义拦截器、切点等。 
- spring Web：提供了针对 Web 开发的集成特性，例如文件上传，利用 servlet  listeners 进行 ioc 容器初始化和针对 Web 的 ApplicationContext。 
- spring test：主要为测试提供支持的，支持使用JUnit或TestNG对Spring组件进 行单元测试和集成测试。 

## 6. Spring 框架中都用到了哪些设计模式？ 

1. **工厂模式**：BeanFactory就是简单工厂模式的体现，用来创建对象的实 例； 
2. **单例模式**：Bean默认为单例模式。
3. **代理模式**：Spring的AOP功能用到了**JDK的动态代理和CGLIB字节码生 成技术**； 
4. 模板方法：用来解决代码重复的问题。比如. RestTemplate,  JmsTemplate, JpaTemplate。 
5. 观察者模式：定义对象键一种一对多的依赖关系，当一个对象的状态发 生改变时，所有依赖于它的对象都会得到通知被制动更新，如Spring中 listener的实现–ApplicationListener。 

## JDK的动态代理和CGLIB字节码生成技术区别？

* 基于接口 -- JDK动态代理
* 基于类 --   cglib： 基于[ASM](http://www.baeldung.com/java-asm)的字节码生成库，它允许我们在**运行时对字节码进行修改和动态生成**。

## spring中使用代理

 代理对象调用（jdk代理 ，cglib代理）**@proxyTragetClass为true或者false  来指定** 

######@EnableAspectJAutoProxy(exposeProxy = true)

* proxyTargetClass = true 强制使用cglib代理
* exposeProxy = true      与此线程关联的 AOP 代理的 ThreadLocal 持有者。 除非控制代理配置上的“exposeProxy”属性已设置为“true”，否则将包含null 。

##Spring FactoryBean和BeanFactory 区别

* **BeanFactory** **是ioc容器的底层实现接口**，是**ApplicationContext 顶级接口**
* FactoryBean 是spirng提供的工厂bean的一个接口

```java
public interface FactoryBean<T> {

//创建的具体bean对象的类型
	@Nullable
	T getObject() throws Exception;

 //工厂bean 具体创建具体对象是由此getObject()方法来返回的
	@Nullable
	Class<?> getObjectType();
	
  //是否单例
	default boolean isSingleton() {
		return true;
	}

}
```

## 什么是IOC？

我们设计软件尽量要满足**高内聚，低耦合**。ioc是控制反转，**资源不由使用者管理，而有spring控制资源**，

好处

第一，**资源集中管理**，由IoC容器实现资源的可配置和易管理。

第二，**降低了使用资源双方的依赖程度**，也就是我们说的耦合度

## Spring指定作用域

`@Scope(value = "prototype")`

@Scope的取值

* singleton 单实例的(默认)
  * :**单实例的bean **,**饿汉加载(容器启动实例就创建好了)**
* prototype 多实例的
  * :**懒汉模式加载（IOC容器启动的时候，并不会创建对象，而是在第一次使用的时候才会创建）**
* request 同一次请求
* session 同一个会话级别

## 往IOC容器中添加组件的方式

* ①:通过**@CompentScan +@Controller @Service @Respository @compent**
  适用场景: 针对我们**自己写的组件可以通过该方式**来进行加载到容器中。
* ②:通过@**Bean**的方式来导入组件(适用于导入**第三方组件的类**)
* ③:通过**@Import**来导入组件 （导入三方组件的**id为全类名路径**）
* ④:通过**实现FacotryBean接口**来实现注册组件

@import

```java
@Configuration
@Import(value = {Person.class, Car.class})
@Import(value = {Person.class, Car.class, TulingImportSelector.class, TulingBeanDefinitionRegister.class})
public class MainConfig {
    
}
```

1.**ImportSeletor，springboot使用这个**

通过**@Import 的ImportSeletor类**实现组件的**导入** (导入组件的id为**全类名路径**)

```java
	public class TulingImportSelector implements ImportSelector {
	//可以获取导入类的注解信息
	@Override
	public String[] selectImports(AnnotationMetadata importingClassMetadata) {
		return new String[]{"com.tuling.testimport.compent.Dog"};
 }
}	
```

2.**ImportBeanDefinitionRegister** -------把**bean定义对象导入到容器中**

```java
public class TulingBeanDefinitionRegister implements ImportBeanDefinitionRegistrar {
	@Override
	public void registerBeanDefinitions(AnnotationMetadata importingClassMetadata, BeanDefinitionRegistry registry) {
		//创建一个bean定义对象
		RootBeanDefinition rootBeanDefinition = new RootBeanDefinition(Cat.class);
		//把bean定义对象导入到容器中
		registry.registerBeanDefinition("cat",rootBeanDefinition);
	}
}
```

**FactorBean**接口使用:

sqlSessionFactorBean使用了这个

```java
public class CarFactoryBean implements FactoryBean<Car> {

    @Override
    public Car getObject() throws Exception {
        return new Car();
    }

    @Override
    public Class<?> getObjectType() {
        return Car.class;
    }

    @Override
    public boolean isSingleton() {
        return false;
    }
}
```

两种获取:

```java
   Object bean = ctx.getBean("carFactoryBean");      //Car@8198b689       获取car
   Object bean2 = ctx.getBean("&carFactoryBean"); //CarFactoryBean@1797edb8  获取FactorBean本身
```

####

## SpringBoot自动装配的原理

* 

* @**EnableAutoConfiguration**注解，使用@Import加载**AutoConfigurationImportSelector**.class类。

* 而**AutoConfigurationImportSelector**实现 ImportSelector并重写了selectImports()方法。

* selectImports()方法扫描了Meata-Inf目录下 的spring.factories中的全限定类名
* 至此，加类添加到spring的bean定义中，等待被创建。

![image-20210913173018435](https://ljjblog.oss-cn-beijing.aliyuncs.com/img/image-20210913173018435.png)



![image-20210913173051166](https://ljjblog.oss-cn-beijing.aliyuncs.com/img/image-20210913173051166.png)

##Spring的生命周期



## Spring的前置后置方法



## spring中Bean的生命周期

* refresh()方法中**invoke**BeanFactoryPostProcessors(beanFactory)对注解进行扫描。

* refresh()使用**finishBeanFactoryInitialization（）**容器初始化时，创建bean。

* doGetBean()方法获取bean，
  * 第一步：从单例缓存池中获取对象
  * 没有，调用CreteBean方法
    * 1.创建早期象  ------**createBeanInstance**()：
    * 2.将早期对象放入缓存中，为用户解决循环依赖。**addSingletonFactory**()
    * 3.给属性赋值------**populateBean**()
    * 4.初始化对象------**initializeBean**()

* 容器关闭，销毁bean。

#####initializeBean()------调用到AOP的**BeanPostProcessor**方法

- postProcessBeforeInitialization：调用bean的初始化方法之前调用
- postProcessAfterInitialization：调用bean的初始化之后调用

## spring循环依赖如何解决？

两种循环依赖：

* constructor-arg：通过**构造函数注入。** 构造函数注入---不能解决
* property：通过**setter对应的方法注入**。**属性循环依赖-------能解决**

### 解决办法属性循环依赖----三级缓存：

一级缓存：**singletonObjects**           **完整的对象**

二级缓存：**earlySingletonObjects**   **早期对象**，**new 了但属性没有赋值**

三级缓存：**singletonFactories**       在createBeanInstance()中addSingletonFactory()方法中放入

三级缓存singletonFactory 是一个包裹对象，通过**getObject获取早期对象**。

## 为什么不能解决构造器循环依赖

createBeanInstance() 

* set方法注入，创建早期对象。
* 构造器注入。

构造器注入，**createBeanInstance()方法在放入早期缓存池(addSingletonFactory)之前，无法解决，循环依赖。**

## 什么是aop?

AOP(Aspect-Oriented Programming)，一般称为面向切面编程，作为面向对象的一种补充，用于将那些与业务无关，但却对多个对象产生影响的公共行为和逻辑，抽取并封装为一个可重用的模块，这个模块被命名为“切面”（Aspect），减少系统中的重复代码，降低了模块间的耦合度，同时提高了系统的可维护性。可用于权限认证、日志、事务处理等。

- Joinpoint(**连接点**)：就是方法

- Pointcut(**切入点**)：就是挖掉共性功能的方法

- Advice(**通知**)：就是**共性功能**，最终以一个方法的形式呈现

- Aspect(**切面**)：就是**通知与切入点对应关系**

- Target**(目标对象**)：就是挖掉功能的方法对应的类产生的对象，这种对象是无法直接完成最终工作的

- Weaving(**织入**)：就是将挖掉的功能回填的动态过程

- Proxy(**代理)**：目标对象无法直接完成工作，需要对其进行功能回填，通过创建原始对象的代理对象实现

- Introduction(引入/引介) ：就是对原始对象无中生有的添加成员变量或成员方法

​	

![](https://ljjblog.oss-cn-beijing.aliyuncs.com/img/image-20210906165226035.png)



![image-20210906165234025](https://ljjblog.oss-cn-beijing.aliyuncs.com/img/image-20210906165234025.png)

##如何用aop?



##aop的实现原理？

第三点: **@EnableAspectJAutoProxy**=====> 导入了**AnnotationAwareAspectJAutoProxyCreator**组件
我们分析出AnnotationAwareAspectJAutoProxyCreator 的继承关系图发现了他具有**BeanPostProcessor 接口特性**和InstantiationAwareBeanPostProcessor
的特性.我们发现InstantiationAwareBeanPostProcessor 在实例化之前(调用构造方法之前)执行的.

创建第一个bean   

根据Bean的生命周期中的**createBean的环节  resolveBeforeInstantiation连触发后置处理器的前置方法.**
 **切面信息(@Aspectj)的信息找出来做成增强器， 然后进行缓存.**

第二步：(创建代理对象)
**后置处理器的 .after方法中**
创建要切的对象的时候,根据方法进行**匹配去找自己的切面(增强器),**然后把增强器和被切的对象**创成一个代理对象.**

第三步:
通过**责任链模式+递归**的来进行调用
先执行我们的**异常通知**.......(catch里面**执行异常通知的方**法)
**返回通知**:(有异常就不会执行)
**后置通知:**正是因为在后置通知中,**代码在finally里中 所以他才是总是被执行的.**
**前置通知**:执行我们的前置通知.
递归终止条件满足:**执行我们的目标方法**:

##spring的事务

- 编程式
- 声明式（XML）



## 说一下Spring的事务传播行为

spring事务的传播行为说的是，当多个事务同时存在的时候，spring如何处理这些事务的行为。

① PROPAGATION_**REQUIRED**：如果当**前没有事务，就创建一个新事务**，如果当前存在事务，就加入该事务，该设置是最常用的设置。

② PROPAGATION_**SUPPORTS**：支持当前事务，**如果当前存在事务，就加入该事务**，如果当前不存在事务，就以非事务执行。

③ PROPAGATION_**MANDATORY**：支持当前事务，如果当前**存在事务，就加入该事务**，如果当前**不存在事务，就抛出异常**。

④ PROPAGATION_**REQUIRES_NEW**：**创建新事务，无论当前存不存在事务，都创建新事务**。

⑤ PROPAGATION_NOT_SUPPORTED：以非事务方式执行操作，如果当前存在事务，就把当前事务挂起。

⑥ PROPAGATION_NEVER：以非事务方式执行，如果当前存在事务，则抛出异常。

⑦ PROPAGATION_NESTED：如果当前存在事务，则在嵌套事务内执行。如果当前没有事务，则按REQUIRED属性执行。

## springboot自动装配

@EnableAutoConfiguration调用了**AutoConfigurationImportSelector**.class类。

这个类中filter（）会功能句MEATA-INF下的spring.fafctories文件中包含的类，

调用refresh方法中的**invokeBeanFactoryPostProcessor**方法，调用了**parse**解析调用了**AutoConfigurationImportSelector** 将类扫描进容器。进行创建。

## SpringMVC执行流程

1. DispatcherServlet 根据url 找到对应HandlerMapping
2. HandlerMappingg根据handler找到handlerAdapter
3. 调用handler执行业务方法
4. ViewResolve根据view名字找到对应view
5. 返回view



1. Spring由哪些模块组成
2. Spring是怎么解决循环依赖的？
3. Spring Boot手动装配有哪几种方式？
4. Spring Boot自动配置原理
5. 谈谈自己对于Spring IOC的理解
6. 谈谈自己对于Spring AOP的理解
7. Spring AOP和AspectJ AOP有什么区别？
8. Spring中的bean的作用域有哪些？
9. Spring中的单例bean的线程安全问题了解吗？
10. Spring中的bean生命周期了解过吗？
11. Spring MVC的工作原理了解嘛？
12. Spring框架中用到了哪些设计模式？
13. @Component和@Bean的区别是什么？
14. 将一个类声明为Spring的bean的注解有哪些？
15. Spring事务管理的方式有几种？
16. Spring事务中的隔离级别有哪几种？
17. Spring事务中有哪几种事务传播行为？
18. Spring 事务底层原理
19. BeanFactory和ApplicationContext有什么区别？
20. Resource 是如何被查找、加载的？
21. 解释自动装配的各种模式？
22. 有哪些不同类型的IOC(依赖注入)？
23. Spring AOP 实现原理
24. ApplicationContext通常的实现是什么?
25. Bean 工厂和 Application contexts 有什么区别？

