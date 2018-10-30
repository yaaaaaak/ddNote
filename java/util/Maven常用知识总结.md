clean install整个parent的话，如果子模块很多，要打很久，而且大多数对于你的开发并没有什么用处。因此可以考虑只打基础依赖。具体含义可以查阅相关api，这里不做赘述。

```shell
mvn clean install -pl base -am -DskipTests -f pom.xml
```

