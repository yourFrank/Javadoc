上一节我们看了根据我们的请求拿到了处理器链（封装了目标方法和所有的拦截器）

而HandlerAdapter 就是一个**超级反射工具**，通过反射直接我们的目标方法的

这个适配器没有我们想的那么简单直接一个反射调用就完事了，它要处理方法中的各种参数赋值等，得到处理器进行方法执行后返回到相应的页面

下面我们来看一下

当拿到这个合适的handler之后，会去找相应的适配器，看哪个适配器能处理，我们点进去看一下

![image-20220221211220759](https://image.imxyu.cn/file/image-20220221211220759.png)

会拿到所有的适配器，这个适配器也是在八大组件初始化的时候加入的。同时我们也可以自己定义适配器去处理

![image-20220221211411868](https://image.imxyu.cn/file/image-20220221211411868.png)

我们来看一下这几个适配器

每个适配器中都有两个重要的方法

1、第一个是supports 方法，会判断当前的适配器能否处理这个handler

2、第二个是handle（），处理逻辑

**adapter适配器是策略模式+适配器模式的体现**

**在各个adapter策略中寻找适合的策略，然后调用handle方法，将handle请求转化成view 视图返回**

## HandlerMapping和HandlerAdapter交互

当这个handle处理器满足Adapter的support方法时，就会使用这个adapter适配器的handle（）方法处理这个请求

### HttpRequestHandlerAdapter

第一个适配器是HttpRequestHandlerAdapter，这个适配器会判断

![image-20220221211650959](https://image.imxyu.cn/file/image-20220221211650959.png)

它的工作supports方法就是判断这个handler看是否实现了HttpRequestHandler接口。

我们如果想让它工作可以写一个组件@Controller，名字以/开始，并且 实现HttpRequestHandler接口即可

### SimpleControllerHandlerAdapter

第二个适配器是SimpleControllerHandlerAdapter

它的support逻辑是实现了controller接口的handler

![image-20220221212047777](https://image.imxyu.cn/file/image-20220221212047777.png)

### HandlerFunctionAdapter

HandlerFunctionAdapter的条件是实现了HandlerFunction接口的handler

![image-20220221212256003](https://image.imxyu.cn/file/image-20220221212256003.png)

### RequestMappingHandlerAdapter

这个adapter的要求是HandlerMethod类型的

因此我们的之前的hello请求对应的就是方法，因此会走这个RequestMappingHandlerAdapter进行处理

![image-20220221212438117](https://image.imxyu.cn/file/image-20220221212438117.png)

### 满足HttpRequestHandlerAdapter的handler请求举例

下面我们来构造两个能满足HttpRequestHandlerAdapter的和SimpleControllerHandlerAdapter的请求

使用我们之前介绍的BeanNameUrlHandlerMapping来注册handler

```java
//BeanNameUrlHandlerMapping 创建好对象以后也要初始化，启动拿到容器中所有组件，看谁的名字是以/开始的，就把这个组件注册为处理器
@Controller("/helloReq") //BeanNameUrlHandlerMapping 就会把他注册进去
public class HelloHttpRequestHandler implements HttpRequestHandler {
   //启用 HttpRequestHandlerAdapter

   //处理请求
   @Override
   public void handleRequest(HttpServletRequest request,
                       HttpServletResponse response) throws ServletException, IOException {
      response.getWriter().write("HelloHttpRequestHandler....");
   }
}
```



```java
@org.springframework.stereotype.Controller("/helloSimple")
public class HelloSimpleController implements Controller {
	@Override
	public ModelAndView handleRequest(HttpServletRequest request, HttpServletResponse response) throws Exception {
		return null;
	}
}

```

我们之前说过三大HandlerMapping 中BeanNameUrlHandlerMapping这个是根据beanName放入路径的

具体的逻辑和RequestHandlerMapping之前一样，创建完对象后调用BeanNameUrlHandlerMapping初始化方法来加载

这个会加载所所有的是以"/" 斜杠开头的beanName。把这个beanName作为路径，保存到BeanNameUrlHandlerMapping的registry中

![image-20220221212813990](https://image.imxyu.cn/file/image-20220221212813990.png)

> 因此我们上面创建的两个类都会保存到BeanNameUrlHandlerMapping 中
>
> 当我们访问/helloReq和/helloSimple都会去BeanNameUrlHandlerMapping 这个中找到

比如说现在我们访问/helloReq 请求，就会在BeanNameUrlHandlerMapping中找到相应的处理器HelloHttpRequestHandler，然后寻找合适的adapter

![image-20220221213510622](https://image.imxyu.cn/file/image-20220221213510622.png)

然后寻找能处理HelloHttpRequestHandler的 处理器适配器，然后发现这个HttpRequestHandlerAdapter支持处理这个，因为HelloHttpRequestHandler实现了HttpRequestHandler接口，所以返回HttpRequestHandlerAdapter

![image-20220221213558840](https://image.imxyu.cn/file/image-20220221213558840.png)

然后就直接调用adapter的handler（）方法

![image-20220221213841352](https://image.imxyu.cn/file/image-20220221213841352.png)

这个handle方法会直接将我们的handler转换成HttpRequestHandler接口，然后调用handleRequest方法

![image-20220221213925695](https://image.imxyu.cn/file/image-20220221213925695.png)

这就是我们实现的handleRequest方法进行处理

![image-20220221214009225](https://image.imxyu.cn/file/image-20220221214009225.png)

>  同样的对于SimpleControllerHandlerAdapter这个适配器
>
> 也是判断满足controller 这个接口的可以处理，然后转换成controller 接口调用我们实现的handleRequest方法



当然我们也可以创建自己的adpter，当满足我们定义的adpater的support要求时，走我们的adapter的handle方法

对于这两种HttpRequestHandlerAdapter和SimpleControllerHandlerAdapter，直接转换成接口调用实现的handleRequest方法就行了

重点是RequestMappingHandlerAdapter，这个是处理**方法**的handler



我们来看一下这个接口

![image-20220222202814572](https://image.imxyu.cn/file/image-20220222202814572.png)

```java
@Nullable //适配的过程，把request、response和handler连接起来进行处理，处理完成得到 ModelAndView（页面解析过程就靠它）
ModelAndView handle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception;
```

## 交互图

![image-20220221215251507](https://image.imxyu.cn/file/image-20220221215251507.png)

## 处理RequestMappingHandlerAdapter的请求

### HandlerAdapter中参数解析和返回值处理器

我们继续看如果是要执行一个方法，handlerAdapter是如何处理的？我们看之前的/hello请求

首先找到能处理的handler,返回handler链，然后开始寻找能处理这个handler的adapter



![image-20220222204955917](https://image.imxyu.cn/file/image-20220222204955917.png)

因为handler是一个方法，可以看到这里找到了RequestMappingHandlerAdapter来处理

![image-20220222205055907](https://image.imxyu.cn/file/image-20220222205055907.png)

接下来先执行拦截器的preHandle方法，这里我们先跳过，后面再说

我们重点来看一下真正执行目标方法的

```java
mv = ha.handle(processedRequest, response, mappedHandler.getHandler());
```

这个就是adapter适配器的作用，把request, response, 和handler转换成modelAndview返回的过程

![image-20220222205141687](https://image.imxyu.cn/file/image-20220222205141687.png)



![image-20220222205754171](https://image.imxyu.cn/file/image-20220222205754171.png)

这里上来会有一个锁

```
这是会话锁，每一个用户和服务器交互无论发了多少请求都只有一个会话，限制用户的线程
如果想控制用户的请求次数一个一个进入，防止用户做一些其他的操作扰乱程序。就可以用这个，一般用在高并发事务中
```

我们主要看mav = invokeHandlerMethod(request, response, handlerMethod); 目标方法的执行

![image-20220222205843935](https://image.imxyu.cn/file/image-20220222205843935.png)

这个方法比较长，我们一点一点来看

![image-20220222210100174](https://image.imxyu.cn/file/image-20220222210100174.png)

```java
//首先这句是将我们的request 和response包装到一个对象中，因为以后要经常用到这两个参数，同时对他们的方法进行一些增强（因此是装饰器模式）
ServletWebRequest webRequest = new ServletWebRequest(request, response);
```

```java
//这是一个数据绑定器，我们之前知道在方法的参数上可以直接写一个Person这样的对象，它可以将我们前台传入的所有有关Person的属性都包装放入到这个对象当中
WebDataBinderFactory binderFactory = getDataBinderFactory(handlerMethod);
```

```java
//这个是准备了一个工厂，因为我们最后要返回ModelAndView Model（要交给页面的数据） View（我们要去的 视图）。model对象就是通过这个工厂产生的
ModelFactory modelFactory = getModelFactory(handlerMethod, binderFactory);
```

```java
//对handlerMethod进行增强，可以快速得到handlerMethod中的一些属性比如参数和返回值什么的，这也是一个装饰器模式
ServletInvocableHandlerMethod invocableMethod = createInvocableHandlerMethod(handlerMethod); 
```

```java
//下面这两个是参数解析器和返回值解析器，专门用来处理请求的参数和返回值
if (this.argumentResolvers != null) { //参数解析器
   invocableMethod.setHandlerMethodArgumentResolvers(this.argumentResolvers);
}
if (this.returnValueHandlers != null) { //返回值解析器（无论原生的返回值是什么最后都要返回modelAndView）
   invocableMethod.setHandlerMethodReturnValueHandlers(this.returnValueHandlers);
}
```

![image-20220222211128996](https://image.imxyu.cn/file/image-20220222211128996.png)

![image-20220222211143843](https://image.imxyu.cn/file/image-20220222211143843.png)

可以看到这个参数解析器有27个，对于不同类型的参数会用不同的解析器进行处理

返回值解析器有15个

> 那么我们就有疑惑了，这个两个组件是什么时候放入进来的呢？

答案和我们之前的handlerMapping一样，都是在默认的组件创建的时候利用initializingBean的afterPropertiesSet()方法，设置的这些

![image-20220222211417702](https://image.imxyu.cn/file/image-20220222211417702.png)

可以看到这个方法中的逻辑如下

![image-20220222211841363](https://image.imxyu.cn/file/image-20220222211841363.png)