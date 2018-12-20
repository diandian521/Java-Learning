---
title: SpringBoot核心知识清单
tags: 自动配置,起步依赖,Actuator,命令行界面(CLI)
grammar_cjkRuby: true
---


欢迎使用 **{小书匠}(xiaoshujiang)编辑器**，您可以通过 `小书匠主按钮>模板` 里的模板管理来改变新建文章的内容。

### 简说
		Spring Boot的魅力主要集中在其四大核心特性：==自动配置==、==起步依赖==、==Actuator==、==命令行界面(CLI)== 。
### Spring IoC容器
		基于Spring Boot框架的程序，其入口是SpringApplication.run()方法，二其本质上则是通过Spring容器来启动的。IoC 容器想要管理各个业务对象以及它们之间的依赖关系，需要通过某种途径来记录和管理这些信息，而BeanDefinition对象就承担了这个责任，其工作原理及关系如图1所示：
![图1 SpringIoc容器](spring-IoC.JPG)

		Spring IoC 容器的整个工作流程大致可以分为两个阶段：容器启动阶段和Bean的实例化阶段
		
##### 容器启动阶段：
		容器通过某种途径加载配置元数据(Configuration MetaData).除了书写代码外，大多数的时候是通过其依赖的工具类，这些工具类加载 Configuration MetaData，并进行解析与分析，然后组装相应的 BeanDefinition，最后会将这些保存了 bean 定义的 BeanDefinition，注册到相应的 BeanDefinitionRegistry 中，以此完成容器的启动工作。
		此阶段主要完成准备性工作，侧重于bean对象管理信息的收集及一些验证或辅助性工作。
		
##### Bean的实例化阶段：
		第一阶段所有bean定义都通过 BeanDefinition 的方式注册到 BeanDefinitionRegistry 中，当某个请求通过容器的 getBean 方法请求某个对象，或者因为依赖关系容器需要隐式的调用 getBean 时，就会触发第二阶段的活动：容器会首先检查所请求的对象之前是否已经实例化完成。如果没有，则会根据注册的 BeanDefinition 所提供的信息实例化被请求对象，并为其注入依赖。当该对象装配完毕后，容器会立即将其返回给请求方法使用。
		而在实际场景下，我们更多的使用另外一种类型的容器： ApplicationContext，它构建在 BeanFactory 之上，属于更高级的容器，除了具有 BeanFactory 的所有能力之外，还提供对事件监听机制以及国际化的支持等。它管理的 bean，在容器启动时全部完成初始化和依赖注入操作。
		
##### Spring容器扩展机制：
		IoC 容器负责管理容器中所有bean的生命周期，而在 bean 生命周期的不同阶段，Spring 提供了不同的扩展点来改变 bean 的命运。
		在容器的启动阶段， BeanFactoryPostProcessor允许我们在容器实例化相应对象之前，对注册到容器的 BeanDefinition 所保存的信息做一些额外的操作，比如修改 bean 定义的某些属性或者增加其他信息等。如果要自定义扩展类，通常需要实现 org.springframework.beans.factory.config.BeanFactoryPostProcessor接口，与此同时，因为容器中可能有多个BeanFactoryPostProcessor，可能还需要实现 org.springframework.core.Ordered接口，以保证BeanFactoryPostProcessor按照顺序执行。
		与之相似的，还有 BeanPostProcessor，其存在于对象实例化阶段。跟BeanFactoryPostProcessor类似，它会处理容器内所有符合条件并且已经实例化后的对象。简单的对比，BeanFactoryPostProcessor处理bean的定义，而BeanPostProcessor则处理bean完成实例化后的对象。
	
##### 注释：
 
###### 1）IoC（Inversion of Control-控制反转）:
		1）定义：
		它是是面向对象编程中的一种设计原则，可以用来减低计算机代码之间的耦合度。其中最常见的方式叫做依赖注入（Dependency Injection，简称DI），还有一种方式叫“依赖查找”（Dependency Lookup）。通过控制反转，对象在被创建的时候，由一个调控系统内所有对象的外界实体，将其所依赖的对象的引用传递给它。也可以说，依赖被注入到对象中。
		
		2）描述：
		Class A中用到了Class B的对象b，一般情况下，需要在A的代码中显式的new一个B的对象。
		采用依赖注入技术之后，A的代码只需要定义一个私有的B对象，不需要直接new来获得这个对象，而是通过相关的容器控制程序来将B对象在外部new出来并注入到A类里的引用中。而具体获取的方法、对象被获取时的状态由配置文件（如XML）来指定。
		
		3）理解：
		可以认为是一种全新的设计模式，但是理论和时间成熟相对较晚，并没有包含在GoF（设计模式）中。
		实现策略

		IoC是一个很大的概念,可以用不同的方式实现。其主要形式有两种：

    	依赖查找：容器提供回调接口和上下文条件给组件。EJB和Apache Avalon 都使用这种方式。这样一来，组件就必须使用容器提供的API来查找资源和协作对象，仅有的控制反转只体现在那些回调方法上，容器将调用这些回调方法，从而让应用代码获得相关资源。
    	依赖注入：组件不做定位查询，只提供普通的Java方法让容器去决定依赖关系。容器全权负责的组件的装配，它会把符合依赖关系的对象通过JavaBean属性或者构造函数传递给需要的对象。通过JavaBean属性注射依赖关系的做法称为设值方法注入(Setter Injection)；将依赖关系作为构造函数参数传入的做法称为构造器注入（Constructor Injection） 

### JavaConfig与常见Annotation
##### JavaConfig（java配置）
		1）定义：JavaConfig就是使用注释来描述Bean配置的组件。JavaConfig 是Spring的一个子项目, 比起Spring，它还是一个非常年青的项目。它基于Java代码和Annotation注解来描述bean之间的依赖绑定关系，目前的版本是1.0 M2。
		
		2）从Spring3开始，JavaConfig功能已经包含在Spring核心模块中，它允许开发者将bean定义和Spring配置放到Java类中来实现。同时，仍允许使用经典的XML方式来定义bean和配置，所以在Spring3以后的版本中，支持xml方式和javaConfig两种Spring配置方式。
		
		3）示例：
``` java
/*<------------------------------------一个简短的组件实例------------------------------------>*/
/*（Bean）*/
package com.spring;

public interface Helloeorld {
    void printHelloWorld(String msg);
}


/**/
package com.spring;

public class HelloworldImpl implements Helloeorld{

    @Override
    public void printHelloWorld(String msg) {
        System.out.println("Hello : " + msg);
    }
}
```