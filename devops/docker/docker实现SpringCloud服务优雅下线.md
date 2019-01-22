

项目使用的是spring cloud，包含多个服务。刚开始制作镜像的时候会在Dockerfile里这样写：

```
ENTRYPOINT [ "sh", "-c", "java $JAVA_OPTS -jar /app.jar" ]
```

后来发现这样子服务下线后并没有及时通知，经常会访问到下线的服务，等于高可用比较差。

与一个朋友聊到这里，搜了下，其实只需要将sh改成bash就行。Alpine Linux下sh不会传递信号量给子进程而bash可以。

至于客户端的缓存，可以通过其他手段去兼容，这里暂时就不做拓展了。

[参考资料](https://www.do1618.com/archives/1180/graceful-shutdown-of-pods-developed-by-go-with-kubernetes/)

