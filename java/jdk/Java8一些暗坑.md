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