###### 1、Collectors.toMap的暗坑

List转Map一般会这样操作：

```java
Optional.ofNullable(staffInfoRepository.findAll())
        .orElse(new ArrayList<>())
        .stream()
        .collect(Collectors.toMap(StaffInfo::getUid, Function.identity()));
```

但是当有用户uid相同（一般都是不正常数据）时，会出现java.lang.IllegalStateException: Duplicate key com.xxx.StaffInfo的异常。要处理这种异常，要么不用，要么需要调整一下，改成后者覆盖前者（如下代码），或者不操作。

```java
Optional.ofNullable(staffInfoRepository.findAll())
        .orElse(new ArrayList<>())
        .stream()
        .collect(Collectors.toMap(StaffInfo::getUid, Function.identity(),(s1,s2)-> s2));
```



###### 2、File.lines，需要使用try-with-resources

否则可能会出现Too many open files。包括其他Stream的terminal操作估计也有类似的问题，需要研究一下。

使用如下：

```java
try(Stream<String> stream = Files.lines("xxx")){
    stream.foreach(s->{...});
}
```

参考：[stackoverflow](https://stackoverflow.com/questions/43067269/java-8-path-stream-and-filesystemexception-too-many-open-files)  [其他技术博客](https://www.kernelhcy.info/?p=124)



