1、Spring4中的@Profile有经过重构，实际上是通过@Conditional实现的。



2、@Scope有4种而不是2种，除了Singleton，Prototype，还有Session和Request（仅对web应用）。后两者在声明时需要注意设置代理模式，以Session域为例，bean的声明如下（注入不变）：

```java
@Bean
@Scope(value="Session",ProxyMode="自行查api，interface或者targetClass")

```

配置了代理后，实际注入的是一个代理，详见p87的图。



