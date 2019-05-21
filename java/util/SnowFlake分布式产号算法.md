一个简单的分布式产号算法，源于twitter，实现相对比较简单，实用性也比较高。详见代码。

如果系统时钟回拨，还是有可能会出现重复，此处实现并没有去做这类的异常处理，正常来说应该抛异常会好一些。有几种做法可以优化（完全避免需要付出极大代价，不值得）：

- 减少dataCenterId或者machineId的位数，取出来做回拨兼容
- 引入第三方组件，设计好数据结构做回拨兼容（影响响应，得不偿失）
- 逆序获取，如果发生回拨，往当前毫秒戳最大的取，数据量不是极大的情况下，也能大概率避免这种事。
  - 可以设置一个全局变量记录毫秒戳逆拨id，回拨时设置一下这个id并-1，小于一定数（看并发度来定）时重新重置到毫秒戳最大值。

```java

import org.apache.commons.lang3.ArrayUtils;
import org.apache.commons.lang3.StringUtils;
import org.apache.commons.lang3.math.NumberUtils;

import java.util.Date;

/**
 * twitter的SnowFlake产号算法，在启动处调用SnowFlake.init()初始化后，在代码处调用SnowFlake.getNextId()即可。
 *
 * @author dd
 * @date 2018/2/9 17:31
 */
public class SnowFlake {

    /**
     * 起始的时间戳 2018-01-01
     */
    private final static long START_STMP = 1514736000000L;

    /**
     * 序列号占用的位数
     */
    private final static long SEQUENCE_BIT = 12;
    /**
     * 机器标识占用的位数
     */
    private final static long MACHINE_BIT = 5;
    /**
     * 数据中心占用的位数
     */
    private final static long DATACENTER_BIT = 5;
    /**
     * 数据中心最大值
     */
    private final static long MAX_DATACENTER_NUM = ~(-1L << DATACENTER_BIT);
    /**
     * 机器标识最大值
     */
    private final static long MAX_MACHINE_NUM = ~(-1L << MACHINE_BIT);
    /**
     * 序列最大值
     */
    private final static long MAX_SEQUENCE = ~(-1L << SEQUENCE_BIT);

    /**
     * 机器标识向左的位移
     */
    private final static long MACHINE_LEFT = SEQUENCE_BIT;
    /**
     * 数据中心向左的位移
     */
    private final static long DATACENTER_LEFT = SEQUENCE_BIT + MACHINE_BIT;
    /**
     * 时间戳向左的位移
     */
    private final static long TIMESTMP_LEFT = DATACENTER_LEFT + DATACENTER_BIT;

    /**
     * 数据中心
     */
    private long datacenterId;
    /**
     * 机器标识
     */
    private long machineId;
    /**
     * 序列号
     */
    private long sequence = 0L;
    /**
     * 上一次时间戳
     */
    private long lastStmp = -1L;

    private static SnowFlake instance;

    private SnowFlake(long datacenterId, long machineId) {
        this.datacenterId = datacenterId;
        this.machineId = machineId;
    }

    /**
     * 初始化方法，需要使用时在应用初始化调用
     */
    public static void init() {
        init(null);
    }

    /**
     * 初始化方法，需要使用时在应用初始化调用。这里需要指定一个ip
     */
    public static void init(String ip) {
        if(ip == null) {
            ip = IpUtil.getDefaultIp();
        }

        long dataCenterId = 1;
        long machineId = 1;
        if (StringUtils.isNotBlank(ip)) {
            //当前使用比较简单的策略，获取第一张网卡的ip，ip第三段做dataCenterId，第四段做machineId，后续可根据需要调整
            if(StringUtils.isNotBlank(ip)){
                String[] ipParts = ip.split("\\.");
                int correctPart = 4;
                if (ArrayUtils.getLength(ipParts) == correctPart) {
                    dataCenterId = NumberUtils.toLong(ipParts[2]) % (1 << DATACENTER_BIT);
                    machineId = NumberUtils.toLong(ipParts[3]) % (1 << MACHINE_BIT);
                }
            }
        }
        instance = new SnowFlake(dataCenterId, machineId);
    }



    /**
     * 产生下一个ID
     *
     * @return 产生的唯一id
     */
    private synchronized long nextId() {
        long currStmp = getNewstmp();
        if (currStmp < lastStmp) {
            throw new RuntimeException("Clock moved backwards.  Refusing to generate id");
        }

        if (currStmp == lastStmp) {
            //相同毫秒内，序列号自增
            sequence = (sequence + 1) & MAX_SEQUENCE;
            //同一毫秒的序列数已经达到最大
            if (sequence == 0L) {
                currStmp = getNextMill();
            }
        } else {
            //不同毫秒内，序列号置为0
            sequence = 0L;
        }

        lastStmp = currStmp;

        //第一行时间戳部分
        //第二行数据中心部分
        //第三行机器标识部分
        //第四行序列号部分
        return (currStmp - START_STMP) << TIMESTMP_LEFT
                | datacenterId << DATACENTER_LEFT
                | machineId << MACHINE_LEFT
                | sequence;
    }

    /**
     * 静态形式获取下一个唯一id
     *
     * @return 唯一id
     */
    public static long getNextId() {
        if (instance == null) {
            throw new RuntimeException("SnowFlake hasn't been initialized!");
        }
        return instance.nextId();
    }

    /**
     * 获取唯一id(字符串类型)
     * @return 唯一id
     */
    public static String getNextIdStr(){
        return getNextId() + "";
    }

    /**
     * 获取下一个毫秒戳
     *
     * @return 毫秒戳
     */
    private long getNextMill() {
        long mill = getNewstmp();
        while (mill <= lastStmp) {
            mill = getNewstmp();
        }
        return mill;
    }

    /**
     * 获取毫秒戳
     *
     * @return 毫秒戳
     */
    private long getNewstmp() {
        return System.currentTimeMillis();
    }

    /**
     * 获取id创建毫秒戳
     *
     * @param id id
     * @return 毫秒戳
     */
    public static long getCreateMillis(long id) {
        return (id >> TIMESTMP_LEFT) + START_STMP;
    }

    /**
     * 获取创建的机器id
     *
     * @param id id
     * @return 机器id
     */
    public static long getCreateMachineId(long id) {
        return (id & (~(Long.MAX_VALUE >> DATACENTER_LEFT << DATACENTER_LEFT))) >> MACHINE_LEFT;
    }

    /**
     * 获取创建的数据中心id
     *
     * @param id id
     * @return 数据中心id
     */
    public static long getCreateDataCenterId(long id) {
        return (id & (~(Long.MAX_VALUE >> TIMESTMP_LEFT << TIMESTMP_LEFT))) >> DATACENTER_LEFT;
    }


    public static void main(String[] args) throws Exception {
        SnowFlake.init();
        for (int i = 0; i < MAX_MACHINE_NUM; i++) {
            long x = SnowFlake.getNextId();
            System.out.println(x);
            System.out.println("create time :" + SnowFlake.getCreateMillis(x));
            System.out.println("create time :" + new Date(SnowFlake.getCreateMillis(x)));
            System.out.println("create dataCenterId :" + SnowFlake.getCreateDataCenterId(x));
            System.out.println("create machineId :" + SnowFlake.getCreateMachineId(x));
            System.out.println("================");
        }
    }


}
```



ip获取简单实现如下。

```java

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.net.InetAddress;
import java.net.NetworkInterface;
import java.net.SocketException;
import java.util.*;

/**
 * ip工具
 *
 * @author :dd
 * @date :2018/2/27 10:49
 */
public class IpUtil {

    private static Logger logger = LoggerFactory.getLogger(IpUtil.class);

    /**
     * 默认获取eth0网卡的ip
     */
    private final static String ETH0 = "eth0";
    /**
     * 默认ip，找不到时返回
     */
    private final static String DEFAULT_IP = "127.0.0.1";

    public static void main(String[] args) {
        System.out.println("default :"+getDefaultIp());
        System.out.println("eth3 :"+getExactIp("eth3"));
    }

    /**
     * 获取本机所有可用ip
     *
     * @return ip地址集合 key-网卡,value-ip集
     */
    public static Map<String, List<String>> getIpMap() {
        Map<String, List<String>> ipMap = new HashMap<>();

        try {
            for (Enumeration<NetworkInterface> en = NetworkInterface.getNetworkInterfaces(); en.hasMoreElements(); ) {
                NetworkInterface intf = en.nextElement();
                String ethName = intf.getName();
                ipMap.put(ethName, new ArrayList<>());
                for (Enumeration<InetAddress> enumIpAddr = intf.getInetAddresses(); enumIpAddr.hasMoreElements(); ) {
                    InetAddress inetAddress = enumIpAddr.nextElement();
                    if (!inetAddress.isLoopbackAddress() && !inetAddress.isLinkLocalAddress()) {
                        ipMap.get(ethName).add(inetAddress.getHostAddress());
                    }
                }
            }
        } catch (SocketException e) {
            logger.error("getIp error!", e);
        }

        logger.info("Current ips are as follows:{}", ipMap);

        return ipMap;
    }

    /**
     * 获取默认网卡的本机ip
     *
     * @return ip地址
     */
    public static String getDefaultIp() {
        return getExactIp(ETH0);
    }

    /**
     * 获取指定网卡的本机ip
     * @param ethName 网卡名
     * @return ip地址
     */
    public static String getExactIp(String ethName) {
        if(ethName == null){
            throw new RuntimeException("ethName cannot be null!");
        }
        Map<String, List<String>> ipMap = getIpMap();
        List<String> ipList = ipMap.get(ethName);
        if (ipList != null && ipList.size() > 0 ) {
            return ipList.get(0);
        }
        logger.warn("Cannot get exact ip with ethName {}, return default ip.",ethName);
        return DEFAULT_IP;
    }
}

```