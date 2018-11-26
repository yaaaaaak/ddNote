###### 1、注册到eureka，可能会出现同实例id覆盖的问题。

比如你明明同一个服务注册了多个实例，但是去eureka控制台只能看到一个：localhost:service-xxx:8000，基本可以确认是实例重名导致的。只需要配置一下即可：

```properties
eureka.instance.prefer-ip-address=true
eureka.instance.instance-id=${spring.cloud.client.ipAddress}:${spring.application.name}:${server.port}

```



###### 2、如果为了安全，服务只允许内部访问，可以屏蔽掉外部ip对应的网卡

```properties
spring.cloud.inetutils.ignored-interfaces[0]=eth0
#docker
spring.cloud.inetutils.ignored-interfaces[1]=docker0
```



###### 3、eureka注册相关

eureka注册中心基于心跳检测和上下线主动通知修改注册数据，客户端也是定时从服务器（eureka）获取新服务列表的，所以服务发现的变更肯定会延时。由于客户端（client）和注册中心都有缓存，默认时间还特别长(30/90s)，所以最好根据需要配置一下缓存和主动刷新时间，目前小规模集群1-3秒还算合适，当然具体要看业务场景分析。

另外最好不要暴力干掉注册到eureka的服务实例，否则eureka认为是服务故障，将处于长期自我保护状态。譬如用到docker时，最好不要使用docker stop来停止服务。

简单点的话可以通过配置actuator的shutdown这个endpoint，通过http post访问http://ip:port/shutdown 优雅下线，这样会通知到注册中心去下掉当前实例。注意权限控制，就算是内网，如果兄弟部门的同事好奇执行了下这个，你的服务实例就down了。追查起来即使你不负主要责任，领导也要批评你安全意识不行，也够糗的。

具体配置后续再补充。



###### 4、FeignClient使用RequestMapping的一个坑

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



​	