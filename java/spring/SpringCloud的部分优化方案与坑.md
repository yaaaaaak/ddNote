1. ###### 注册到eureka，可能会出现同实例id覆盖的问题。


比如你明明同一个服务注册了多个实例，但是去eureka控制台只能看到一个：localhost:service-xxx:8000，基本可以确认是实例重名导致的。只需要配置一下即可：

```properties
eureka.instance.prefer-ip-address=true
eureka.instance.instance-id=${spring.cloud.client.ipAddress}:${spring.application.name}:${server.port}

```



2. ###### 如果为了安全，服务只允许内部访问，可以屏蔽掉外部ip对应的网卡


```properties
spring.cloud.inetutils.ignored-interfaces[0]=eth0
#docker
spring.cloud.inetutils.ignored-interfaces[1]=docker0
```



3. ###### eureka注册相关


eureka注册中心基于心跳检测和上下线主动通知修改注册数据，客户端也是定时从服务器（eureka）获取新服务列表的，所以服务发现的变更肯定会延时。由于客户端（client）和注册中心都有缓存，默认时间还特别长(30/90s)，所以最好根据需要配置一下缓存和主动刷新时间，目前小规模集群1-3秒还算合适，当然具体要看业务场景分析。

另外最好不要暴力干掉注册到eureka的服务实例，否则eureka认为是服务故障，将处于长期自我保护状态。譬如用到docker时，最好不要使用docker stop来停止服务。

简单点的话可以通过配置actuator的shutdown这个endpoint，通过http post访问http://ip:port/shutdown 优雅下线，这样会通知到注册中心去下掉当前实例。注意权限控制，就算是内网，如果兄弟部门的同事好奇执行了下这个，你的服务实例就down了。追查起来即使你不负主要责任，领导也要批评你安全意识不行，也够糗的。

具体配置后续再补充。



4. ###### FeignClient使用RequestMapping的一个坑


如果你在@FeignClient类上使用@requestMapping，那么这个声明默认会被当做一个外部映射扫描到。所以某些时候可能会出现一些莫名其妙的问题，比如提示 request method 'xxx' not supported或者映射冲突。

项目开发中，一个服务没有配置根目录，而在一个Feign中为抽出共有的header配了，并指定post。本来访问根目录应该返回404的，结果返回request method 'get' not supported。

原因是RequestMappingHandlerMapping的isHandler判断有点问题：

``` java
@Override
protected boolean isHandler(Class<?> beanType) {
	return (AnnotatedElementUtils.hasAnnotation(beanType, Controller.class) ||
			AnnotatedElementUtils.hasAnnotation(beanType, RequestMapping.class));
}
```

意思是只要有Controller注解或者RequestMapping注解，都会认为是一个映射。

参考[网上文章](http://blog.didispace.com/spring-cloud-feignclient-problem/)，处理方法有两个：

- 去掉类上的RequestMapping。

- 重写RequestMappingHandlerMapping的isHandler，排除掉有FeignClient注解的类。

  ```java
  @Override
  protected boolean isHandler(Class<?> beanType) {
  	return super.isHandler(beanType) &&
  			!AnnotatedElementUtils.hasAnnotation(beanType, FeignClient.class));
  }
  ```



5. ###### Ribbon设置Rule的一个坑


在配置客户端负载均衡时，一般会通过@Configuration和@Bean指定一个IRule的实现。但是[官方文档](https://cloud.spring.io/spring-cloud-netflix/multi/multi_spring-cloud-ribbon.html)中有明确写明：

> The `CustomConfiguration` clas must be a `@Configuration` class, but take care that it is not in a `@ComponentScan` for the main application context. Otherwise, it is shared by all the `@RibbonClients`. If you use `@ComponentScan` (or `@SpringBootApplication`), you need to take steps to avoid it being included (for instance, you can put it in a separate, non-overlapping package or specify the packages to scan explicitly in the `@ComponentScan`).
>
>

大意是，要指定使用Ribbon的Rule，要有@Configuration但是又不能被Spring上下文扫描到，即不能被@ComponentScan或者@SpringBootApplication等注解扫到，否则全局共享，因此最好扫描的时候排除掉，非常奇葩。具体排除方法自行查阅相关文章，很简单。



6. ###### short-circuited and no fallback available


Hystrix自我保护的一种，书暂时还没看到这里，先记一下。当一段时间请求异常率超过一定数值（50%）时，客户端会自我保护一段时间，对该服务发起的请求都默认标记为失败。

参考：[同样的问题](http://mzeroo.github.io/2015/02/06/thread-hang.html) 和 [一些博主的解读](https://blog.csdn.net/github_38592071/article/details/78878716#) 



7. ###### spring cloud项目获取本机ip

   项目启动后就可以获取，不需要其他第三方工具或者手写IpUtils。

   ```java
           ConfigurableApplicationContext context = SpringApplication.run(Application.class,args);
           String ip = context.getEnvironment().getProperty("spring.cloud.client.ipAddress");
   ```

   