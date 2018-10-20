1、注册到eureka，可能会出现同实例id覆盖的问题。比如你明明同一个服务注册了多个实例，但是去eureka控制台只能看到一个：localhost:service-xxx:8000，基本可以确认是实例重名导致的。只需要配置一下即可：

```properties
eureka.instance.prefer-ip-address=true
eureka.instance.instance-id=${spring.cloud.client.ipAddress}:${spring.application.name}:${server.port}

```

2、如果为了安全，服务只允许内部访问，可以屏蔽掉外部ip对应的网卡
```properties
spring.cloud.inetutils.ignored-interfaces[0]=eth0
#docker
spring.cloud.inetutils.ignored-interfaces[1]=docker0
```

