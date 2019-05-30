1. Spring4中的@Profile有经过重构，实际上是通过@Conditional实现的。

2. @Scope有4种而不是2种，除了Singleton，Prototype，还有Session和Request（仅对web应用）。

   - 后两者在声明时需要注意设置代理模式，以Session域为例，bean的声明如下（注入不变）：

   ```java
   @Bean
   @Scope(value="Session",ProxyMode="interface or targetClass")
   
   ```

   - 配置了代理后，实际注入的是一个代理，详见p87的图。

3. Spring AOP只支持方法级别的切面，其余如构造器切面需要通过其他手段去实现。

   - 本质还是代理，而原生AspectJ是一门独立的语言编写的，Spring AOP编写时也只是借鉴其部分语法。
   - 要用到aop，不能调用自身的方法，public也不行，非得用的话做一下自我注入，再通过注入对象调用。

4. Spring AOP同一个方法被切多次，可以通过以下三种形式定义先后顺序：

   ```java
   //其一
   @AspectJ
   @Order(1)
   public class AspectDemo1{}
   
   //其二
   public class AspectDemo2 implements Ordered{
       
       @Override
       public int getOrder(){
           return 2;
       }
   }
   
   //其三 xml在aop命名空间配置 order="3"，不多赘述
   
   ```

5. Spring AOP还可以给代理的类通过增加接口实现扩展方法，达到无侵入扩展的目的，特别适合对他人写的底层或者无源码的外部包的切入。

   - 不过调用新的方法需要强转成扩展接口的类型，某种程度上让表层代码变得结构混乱了，个人用着不是太爽。
   - 具体搜关键字@DeclareParents，这里不细讲。

6. 上传文件可以用MultipartFile接收，也可以直接用javax.servlet.http.Part接收，后者还不用配专用的resolver，不过要怎么限制最大需要自行去查阅。

7. 自定义运行时异常可以直接@ResponseStatus配置http返回码以及对应的文案。

8. redirect将属性带去下一个页面，除了放session外，还可以使用RedirectAttributes，使用方法跟model一样，调用addFlashAttribute即可。

9. ContextLoaderListener负责加载Spring容器相关的bean，DispatcherServlet负责加载MVC相关的bean（废话，不过想想还是补上这一条）。

10. 书看完了，其他的都快速阅读过去了。以下为其他材料获取的补充。

11. spring容器的单例缓存在DefaultSingletonBeanRegistry里，其内部实现是一个HashMap

12. HierarchicalBeanFactory父子级联容器接口实现。子容器能读取到父容器的bean，反之不行。

    - Spring MVC是一个经典的实现，展示层能调用业务层持久层bean，而业务层持久层无需关注展示层。

13. BeanDefinitionRegistry提供手工注册BeanDefinition对象的方法。

14. 初始化顺序

    1. new
    2. PostConstruct
    3. afterPropertiesSet
    4. init-method
    5. ……其余不列举了，关键是要理解，如果bean实现了接口，会在初始化不同环节一层层执行各个初始化方法就行。

15. 调用自身需要用到aop的方法，可以通过AopContext.currentProxy()强转(推荐)，或者自我注入。

16. @Transactional默认隔离级别是DEFAULT，即使用数据库的配置，这个可以根据需要调整。

17. 

