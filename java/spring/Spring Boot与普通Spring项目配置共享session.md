当前在做的项目分成两大块，一块是旧代码，基于Spring 3.x的传统Spring项目，另一块是新开发功能的模块，基于Spring Boot。

操作台的登录等基础功能还是基于旧代码，也就是在原有基础上做这块，于是做了简单的基于redis的Spring session共享。

### 对普通Spring项目的调整

pom.xml里添加依赖

```xml
		<!-- spring-session -->
		<dependency>
			<groupId>org.springframework.session</groupId>
			<artifactId>spring-session</artifactId>
			<version>1.2.0.RELEASE</version>
		</dependency>

		<dependency>
			<groupId>org.springframework.session</groupId>
			<artifactId>spring-session-data-redis</artifactId>
			<version>1.2.0.RELEASE</version>
		</dependency>
```

web.xml加过滤器

```xml
	<filter>
        <filter-name>springSessionRepositoryFilter</filter-name>
        <filter-class>org.springframework.web.filter.DelegatingFilterProxy</filter-class>
    </filter>
    <filter-mapping>
        <filter-name>springSessionRepositoryFilter</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>
```

bean配置

```xml
	<bean id="jedisPoolConfig" class="redis.clients.jedis.JedisPoolConfig">
		<property name="maxTotal" value="100" />
		<property name="maxIdle" value="10" />
		<property name="testWhileIdle" value="true" />
		<property name="maxWaitMillis" value="2000" />
	</bean>

	<bean id="redisHttpSessionConfiguration"  class="org.springframework.session.data.redis.config.annotation.web.http.RedisHttpSessionConfiguration" >
       <property name="maxInactiveIntervalInSeconds" value="10800" />
    </bean>
    
    <bean class="org.springframework.data.redis.connection.jedis.JedisConnectionFactory">
        <property name="hostName" value="${your_redis_ip}" />
        <property name="port" value="${your_redis_port}" />
        <property name="password" value="${your_redis_password}" />
        <property name="poolConfig" ref="jedisPoolConfig" />
    </bean>
```

properties配置

```
your_redis_ip=127.0.0.1
your_redis_port=6379
your_redis_password=123456
```

暂时没考虑redis的高可用性，这里可以做一层优化

### 对Spring Boot项目的调整

pom.xml添加依赖

```xml
		<dependency>
			<groupId>org.springframework.session</groupId>
			<artifactId>spring-session</artifactId>
		</dependency>
```

代码新增：

```java
@Configuration
public class RedisConfig {

    @Bean
    @ConfigurationProperties(prefix="spring.redis.pool")
    public JedisPoolConfig jedisPoolConfig(){
        return new JedisPoolConfig();
    }

}
```

```java
@Configuration
@EnableRedisHttpSession(maxInactiveIntervalInSeconds = 10800)
public class SpringSessionConfig {


	@Bean
	@ConfigurationProperties(prefix = "spring.session.redis")
    public RedisConnectionFactory springSessionRedisConnectionFactory(@Qualifier("jedisPoolConfig") JedisPoolConfig jedisPoolConfig) {
        JedisConnectionFactory connectionFactory = new JedisConnectionFactory();
        connectionFactory.setPoolConfig(jedisPoolConfig);
        return connectionFactory;  
    }  
	
	@Bean
	public HttpSessionStrategy httpSessionStrategy() {
	    //基于Cookie交互
		return new CookieHttpSessionStrategy();
	}
	
	@Bean
	public org.springframework.session.data.redis.config.ConfigureRedisAction configureRedisAction() {

		return org.springframework.session.data.redis.config.ConfigureRedisAction.NO_OP;
	}

	
	
}
```

```java
@Configuration
public class WebBootConfig extends WebMvcConfigurerAdapter {

    private Logger logger = LoggerFactory.getLogger(this.getClass());


    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        //登录拦截，排除error
        LoginInterceptor loginInterceptor = new LoginInterceptor();
        registry.addInterceptor(loginInterceptor).excludePathPatterns("/error");
    }
    
    /**
     * 登录校验
     */
    class LoginInterceptor implements HandlerInterceptor{

        private Logger logger = LoggerFactory.getLogger(this.getClass());

        @Override
        public boolean preHandle(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Object o) throws Exception {
            //OperInfo是登录时存储的一个object，需要支持序列化。根据自己实际项目调整
            OperInfo info = (OperInfo)httpServletRequest.getSession().getAttribute(Constants.SESSION_OPER_NAME);
            if(info == null || info.getOperId() == null ){
                logger.error("User's login status invalid. Remote address:{}, requestURI:{}",httpServletRequest.getRemoteAddr(),httpServletRequest.getRequestURI());
                /*这里可以根据需要调整成需要的方式，我的项目有统一的异常捕获，
                所以从这里可以直接抛出，如不需要这种形式，也可以改成filter*/
                throw new BizException(ErrorCode.LOGIN_STATUS_INVALID);
            }
            return true;
        }

        @Override
        public void postHandle(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Object o, ModelAndView modelAndView) throws Exception {

        }

        @Override
        public void afterCompletion(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Object o, Exception e) throws Exception {

        }
    }
}

```

为了方便，也可以自己封装一个类似SessionHelper，获取当前操作用户之类的，这里不赘述。

配置文件如下(yml)

```yml
spring:
  session:
    redis:
      #这里不是host，需要跟属性一一对应
      hostName: 127.0.0.1
      port: 6379
      password: 123456
  #这里是跟其他池共用的配置
  redis:
    pool:
      max-idle: 10
      max-wait: 2000
      min-idle: 5
```

### 暗坑

SpringSession实现里，RedisHttpSessionConfiguration自动注入的配置会获取默认的RedisConnectionFactory。因此如果代码层面上只配置了一个RedisConnectionFactory（如上springSessionRedisConnectionFactory），就会默认使用这个显式声明的factory，并且其他redisTemplate（如果有）也会也会用此配置。而如果配了多个，会报类似but found 2的错误。

要区分开的话，需要再显式配factory，并在引用处指定factory的名字。

```java
    @Bean
    @ConfigurationProperties(prefix = "spring.redis")
    public RedisConnectionFactory redisConnectionFactory1(@Qualifier("jedisPoolConfig") JedisPoolConfig jedisPoolConfig) {
        JedisConnectionFactory connectionFactory = new JedisConnectionFactory();
        connectionFactory.setPoolConfig(jedisPoolConfig);
        return connectionFactory;
    }

    @Bean
    public RedisTemplate<Object, Object> redisTemplate(@Qualifier("redisConnectionFactory1") RedisConnectionFactory redisConnectionFactory) {
        RedisTemplate<Object, Object> redisTemplate = new RedisTemplate<>();
        redisTemplate.setConnectionFactory(redisConnectionFactory);

        // 使用Jackson2JsonRedisSerialize 替换默认序列化
        Jackson2JsonRedisSerializer jackson2JsonRedisSerializer = new Jackson2JsonRedisSerializer(Object.class);

        ObjectMapper objectMapper = new ObjectMapper();
        objectMapper.setVisibility(PropertyAccessor.ALL, JsonAutoDetect.Visibility.ANY);
        objectMapper.enableDefaultTyping(ObjectMapper.DefaultTyping.NON_FINAL);

        jackson2JsonRedisSerializer.setObjectMapper(objectMapper);

        // 设置value的序列化规则和 key的序列化规则
        redisTemplate.setValueSerializer(jackson2JsonRedisSerializer);
        redisTemplate.setKeySerializer(new StringRedisSerializer());
        redisTemplate.setHashKeySerializer(new StringRedisSerializer());
        redisTemplate.setHashValueSerializer(jackson2JsonRedisSerializer);
        redisTemplate.afterPropertiesSet();
        return redisTemplate;
    }
```

因为指定了ConfigurationProperties的prefix，配置也有微调(yml)

```yaml
spring:
  redis:
    #微调部分。这里不是host，需要跟属性一一对应
    hostName: 127.0.0.1
    port: 6379
    password: 123456
```

**并且**需要给SpringSession默认的RedisConnectionFactory加上@Primary注解，标注为默认factory

```java
    /**
     * session基础配置，{@link RedisHttpSessionConfiguration}使用的默认factory
     * @param jedisPoolConfig 池配置
     * @return factory
     */
	@Bean
	@ConfigurationProperties(prefix = "spring.session.redis")
	@Primary
    public RedisConnectionFactory springSessionRedisConnectionFactory(@Qualifier("jedisPoolConfig") JedisPoolConfig jedisPoolConfig) {
        JedisConnectionFactory connectionFactory = new JedisConnectionFactory();
        connectionFactory.setPoolConfig(jedisPoolConfig);
        return connectionFactory;  
    }  
```

### 其他

需要提及的是，Spring Session只适合中小规模的项目，大规模的应该考虑其他解决方案。有兴趣的可以去看下实现原理或者源码，其实内部的实现效率并不高。