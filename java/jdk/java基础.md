###### 序列化相关

- 如果没有指定serialVersionUID，会随机生成一个，反序列化会用上，到时候判断对象是否equal就可能出问题
- transient可以阻止字段被序列化和反序列化
- 静态变量不会序列化
- 父类要被序列化也必须要实现Serializable接口

###### java复制

- 直接赋值复制

  - 实际指向同一个对象，a1字段变化时a2也会变

  ```java
  Test a2 = a1;
  ```

- 浅复制

  - 实现Cloneable接口强转，字段为值类型时复制，为引用类型时也只是复制引用。

  ```java
  
      public class Test implements Cloneable {
          private String str;
          
          @Override
          public Ba clone() throws CloneNotSupportedException {
              return (Ba) super.clone();
          }
  ```

- 深复制

  - 只是在clone方法内将所有引用类型字段执行一次clone操作

- 反序列化

  - 实际上只是从流里读出数据反序列化再生成原来的对象，能彻底解决浅复制不完善和深复制麻烦的问题，但是需要考虑性能问题



###### 单例代码

```java


//双重检查所，提高并发度，但是在多线程情况下，某线程执行判断完第一个if(instance == null)之后，
//可能发生指令重排导致分配内存先于初始化，此时其他线程误以为初始化完成了直接拿去用，然后出错了，不建议。
//给instance加上volatile可以解决这个问题
public class Singleton{
    private static Singleton instance = null;
    private Singleton(){}
    public static Singleton getInstance(){
        if(instance == null){
            synchronized(Singleton.class){
                if(instance == null){
                    instance = new Sinleton();
                }
            }
        }
        return instance;
    }
}

//饿汉法，线程安全，但是不能实现延迟加载
public class Singleton{
    private static Singleton instance = new Singleton();
    private Singleton(){}
    public static Singleton getInstance(){
        return instance;
    }
}

//较优雅的实现，能实现延迟加载。java并发编程实战推荐
public class Singleton{
    private static class Holder{
        private static Singleton instance = new Singleton();
    }
    private Singleton(){}
    public static Singleton getInstance(){
        return Holder.instance;
    }
}

//枚举，自身就是单例，线程安全，支持序列化，effective java推荐
public class Singleton{
    
    private Singleton(){}
    
    public static Singleton getInstance(){
        return SingletonEnum.E.getInstance();
    }
    
    private enum SingletonEnum{
        E;
        private Singleton singleton;
        
        SingletonEnum(){
            singleton = new Singleton();
        }
        
        public Singleton getInstance(){
            return singleton;
        }
        
    }
}
```



###### atomic包的一些比较少用的类

- LongAdder

  - 当有cas不成功时或者判断到已经有Cell数组时，主动拆分成Cell数组（已有则忽略）分散执行cas从而减少死循环概率，计算时再拿出各Cell的数据汇总，比较巧妙用上了分布式设计，具体可以看源码。
  - 可能会出现统计上的小误差，如果需要很精准的计数，不要使用。

- AtomicReference

  - 对指定类型（通过泛型指定）做atomic操作

- AtomicStampedReference

  - 用于解决ABA问题

- AtomicIntegerFieldUpdater

  - cas更新某个实例的Integer域
  - 该域必须为非static，且有volatile修饰

  

###### 禁止集合的修改

可以使用Collections.unmodifiableXxx一系列方法，传入指定的引用之后，不允许再修改内部数值。

看源码实现，其实就是简单将修改操作改成抛出异常。

guava包里的ImmutableXxx一系列类也是类似的功能。

