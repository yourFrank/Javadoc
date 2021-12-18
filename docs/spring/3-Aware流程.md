---
title: 3、Aware流程
tags:
  - spring源码
categories:
  - spring
  - spring源码
cover: 'https://image.imxyu.cn/file/image-20211207193924210.png'
description: Aware、@Autowired流程
abbrlink: a0b8b788
date: 2021-12-7 21:28:43
---





## 核心接口ApplicaitonContext

 ![image-20211205153759992](https://image.imxyu.cn/file/image-20211205153759992.png)

ApplicationContext拥有如下功能

* ioc事件派发器
* 国际化解析
* bean工厂功能-- 自动装配是被组合进来的
* 资源解析功能

## Aware接口

如果我们想在自己的类中使用ApplicationContext 有下面两种方式：

1、通过自动注入

 ```java
  @Component
   public class Person  {
   
      @Autowired
      ApplicationContext context;  //可以要到ioc容器
      MessageSource messageSource;
       ...
   }
 ```

2、实现Aware接口

```java
/**
 * Aware接口；帮我们装配Spring底层的一些组件
 * 1、Bean的功能增强全部都是有 BeanPostProcessor+InitializingBean  （合起来完成的）
 * 2、骚操作就是 BeanPostProcessor+InitializingBean
 *
 * 你猜Autowired是怎么完成的
 *
 */
public class Person implements ApplicationContextAware,MessageSourceAware{
    
	@Override
	public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
		//利用回调机制，把ioc容器传入
		this.context = applicationContext;

	}
    
    @Override
	public void setMessageSource(MessageSource messageSource) {
		this.messageSource = messageSource;
	}

}
```

这几个接口，实现后 在ioc 创建完Bean后都会将ioc容器（ApplicationContext）通过我们写的实现函数（回调）传入

![image-20211205154343395](https://image.imxyu.cn/file/image-20211205154343395.png)

## 断点1-aware赋值和对象创建

我们打两个断点看看何时创建Bean对象和 XXX Aware是什么时候把applicationContext赋值进来的

![image-20211205155259145](https://image.imxyu.cn/file/image-20211205155259145.png)



```java
/**
 * 因为使用的是注解，我们使用注解测试类
 */
public class AnnotationMainTest {

   public static void main(String[] args) {
      ApplicationContext applicationContext = new AnnotationConfigApplicationContext(MainConfig.class);
      Person bean = applicationContext.getBean(Person.class);
      ApplicationContext context = bean.getContext();

      System.out.println(context == applicationContext);
   }
```



**具体步骤可以自己debug，已经在源码上写好了注释**

这里主要截几个关键步骤的图

![image-20211207202316726](https://image.imxyu.cn/file/image-20211207202316726.png)

![image-20211207202639242](https://image.imxyu.cn/file/image-20211207202639242.png)



![image-20211207203024083](https://image.imxyu.cn/file/image-20211207203024083.png)



执行后置处理器：

![image-20211207203127952](https://image.imxyu.cn/file/image-20211207203127952.png)

![image-20211207203215828](https://image.imxyu.cn/file/image-20211207203215828.png)

## 断点2-属性赋值

在Person的setName处打断点，看何时属性赋值

![image-20211207203959724](https://image.imxyu.cn/file/image-20211207203959724.png)

```java
public class MainTest {

	public static void main(String[] args) {
		ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("beans.xml");
		Person bean = context.getBean(Person.class);
		System.out.println(bean);
	}

}
```

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	   xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	   xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context https://www.springframework.org/schema/context/spring-context.xsd">

	<bean class="cn.imlql.spring.bean.Person" id="person">
		<property name="name" value="张三"/>
 	</bean>

</beans>
```



同样贴几个关键位置

![image-20211207203024083](https://image.imxyu.cn/file/image-20211207203024083.png)



```java
protected void populateBean(String beanName, RootBeanDefinition mbd, @Nullable BeanWrapper bw) {
    // 这一步就是拿到属性值
    PropertyValues pvs = (mbd.hasPropertyValues() ? mbd.getPropertyValues() : null);
    
    // ......      

    if (pvs != null) {
        applyPropertyValues(beanName, mbd, bw, pvs); //xml版的所有配置会来到这里给属性赋值
    }
}
```

![image-20211207204519614](https://image.imxyu.cn/file/image-20211207204519614.png)



![image-20211207212600461](https://image.imxyu.cn/file/image-20211207212600461.png)

![image-20211207212707788](https://image.imxyu.cn/file/image-20211207212707788.png)



## 断点3-@Autowired

![image-20211207213027784](https://image.imxyu.cn/file/image-20211207213027784.png)

```java
//使用注解类debug @Autowired
public class AnnotationMainTest {
   public static void main(String[] args) {

      ApplicationContext applicationContext =
            new AnnotationConfigApplicationContext(MainConfig.class);

      Person bean = applicationContext.getBean(Person.class);
      ApplicationContext context = bean.getContext();
      System.out.println(context == applicationContext);
   }
}
```

列举关键方法：

![image-20211207213421689](https://image.imxyu.cn/file/image-20211207213421689.png)



![image-20211207213537810](https://image.imxyu.cn/file/image-20211207213537810.png)

将标注@Autowired注解的属性和方法放到element集合中

![image-20211207213612147](https://image.imxyu.cn/file/image-20211207213639729.png)

![image-20211207213807930](https://image.imxyu.cn/file/image-20211207213807930.png)

![image-20211207213915037](https://image.imxyu.cn/file/image-20211207213915037.png)



## 流程图

详细步骤可按照相应debug的断点，对源码进行debug，https://gitee.com/youFrank/spring-framework 已经添加源码注释

![image-20211207193924210](https://image.imxyu.cn/file/image-20211207193924210.png)

