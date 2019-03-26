1. 序列化相关

   - 如果没有指定serialVersionUID，会随机生成一个，反序列化会用上，到时候判断对象是否equal就可能出问题
   - transient可以阻止字段被序列化和反序列化
   - 静态变量不会序列化
   - 父类要被序列化也必须要实现Serializable接口

2. java复制

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

3. 