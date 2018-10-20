当前在做的项目分成两大块，一块是旧代码，基于Spring 3.x的传统Spring项目，另一块是新开发功能的模块，基于Spring Cloud，通信基于Eureka。


修改起来也不复杂，只需要调整普通Spring项目。

pom.xml添加以下内容：

```xml
<dependency>
			<groupId>com.netflix.eureka</groupId>
			<artifactId>eureka-client</artifactId>
			<version>1.4.12</version>
		</dependency>
		<dependency>
			<groupId>com.netflix.archaius</groupId>
			<artifactId>archaius-core</artifactId>
			<version>0.7.3</version>
		</dependency>
		<dependency>
			<groupId>com.fasterxml.jackson.core</groupId>
			<artifactId>jackson-annotations</artifactId>
			<version>2.8.8</version>
		</dependency>
		<dependency>
			<groupId>com.fasterxml.jackson.core</groupId>
			<artifactId>jackson-core</artifactId>
			<version>2.8.8</version>
		</dependency>
		<dependency>
			<groupId>com.fasterxml.jackson.core</groupId>
			<artifactId>jackson-databind</artifactId>
			<version>2.8.8</version>
		</dependency>
		<dependency>
			<groupId>com.netflix.servo</groupId>
			<artifactId>servo-core</artifactId>
			<version>0.10.0</version>
		</dependency>
```

添加配置文件eureka-client.properties，配置如下（hosts自行解决，我用docker+jenkins，可以直接在jenkins上配置）：

```
eureka.serviceUrl.default=http://ek1:8761/eureka/,http://ek2:8761/eureka/
eureka.preferSameZone=true
eureka.shouldUseDns=false
eureka.decoderName=JacksonJson
eureka.freshInterval=30000
```

关于是否注册到Eureka，不需要注册的话，可以继承DefaultEurekaClientConfig重写了shouldRegisterWithEureka方法。这里应该也可以通过配置properties来操作，因为我这边还涉及到其他统一配置的调整，就没采用。代码如下：
```java
public class XxxEurekaClientConfig extends DefaultEurekaClientConfig{

    /**
     * 不注册到eureka
     * @return false
     */
    @Override
    public boolean shouldRegisterWithEureka() {
        return false;
    }
    
    //其他调整
}
```

调用代码封装如下：

```java

public class EurekaConnectUtil {

    private static Logger logger = LoggerFactory.getLogger(EurekaConnectUtil.class);

    /**这里会提示已过时，忽略即可*/
    private static DiscoveryClient discoveryClient;

    static {
        DiscoveryManager discoveryManager = DiscoveryManager.getInstance();
        discoveryManager.initComponent(new MyDataCenterInstanceConfig(), new XxxEurekaClientConfig());
        discoveryClient = (DiscoveryClient) discoveryManager.getEurekaClient();
    }

    /**
     * 获取服务地址
     * @param servName 服务名
     * @return 服务的ip:port，如未发现，则返回null
     */
    public static String getServAddress(String servName){

        //最多尝试3次
        for (int i = 0; i < 3; i++) {
            //发现服务
            try {
                //这里是轮询的，无特殊要求的话，不需要太过考虑负载
                InstanceInfo nextServerInfo = discoveryClient.getNextServerFromEureka(servName, false);

                if (nextServerInfo == null) {
                    logger.error("Cannot get an instance of service with servName {},put serv address empty!",servName);
                } else {
                    logger.info("Found an instance of {} to talk to from eureka: {}:{}" ,
                            nextServerInfo.getVIPAddress(),nextServerInfo.getIPAddr(),nextServerInfo.getPort());

                    //服务健康检查
                    if (InstanceInfo.InstanceStatus.UP.equals(nextServerInfo.getStatus())) {
                        return "http://" + nextServerInfo.getIPAddr() + ":" + nextServerInfo.getPort();
                    } else {
                        logger.error("Service {} at ip {} and port {} is now not available!status is {}",
                                servName, nextServerInfo.getIPAddr(),
                                nextServerInfo.getPort(), nextServerInfo.getStatus());
                    }
                }
            } catch (Exception e) {
                logger.error("Cannot get an instance of service to talk to from eureka with servName {}",servName,e);
            }
        }
        return null;
    }
}


```

需要调用时，可以这样直接调用：
```
public void test(){
    RestTemplate template = new RestTemplate();
    logger.info("test discovery greeting: {}",template.getForObject(EurekaConnectUtil.getServAddress("service-A")+"/greeting", String.class)));
}
```


如果需要配置为注册，不用做配置重写，配一下注册名即可，具体可以去搜相关代码，这里不做赘述。