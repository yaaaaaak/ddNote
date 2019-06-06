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
//可能发生cpu切换导致执行了多次初始化，不建议。
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
public enum Singleton{
    INSTANCE;
    private String name1;
    public String getName1(){
        return name1;
    }
    public void setName1(String name1){
        this.name1 = name1;
    }
}
```

