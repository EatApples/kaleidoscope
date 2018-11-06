### 获取本机IP地址

问题：SpringBoot 中 `spring.cloud.client.ipAddress` 中是怎么搞的？

回答：类 HostInfoEnvironmentPostProcessor 配置了 spring.cloud.client.ipAddress 属性，而 ip 又是通过工具类 `org.springframework.cloud.commons.util.InetUtils` 抓取

```java
public class NetworkHelper {

    private NetworkHelper() {

    }

    /**
     * 通过方法
     *
     * <pre>
     * InetAddress.getLocalHost().getHostAddress()
     * </pre>
     *
     * 来获取IP。可能返回回环地址。
     *
     * @return 如果 UnknownHostException 则返回 null
     */
    public static String getIpByHostAddress() {

        try {
            return InetAddress.getLocalHost().getHostAddress();
        }
        catch (UnknownHostException e) {
            // ignore
        }
        return null;

    }

    /**
     * 根据网络接口获取IP地址。
     *
     * @param ethNum
     *            网络接口名，Linux下一般是eth0
     * @return 可能返回 null
     */
    public static String getIpByEthNum(String ethNum) {

        Map<String, String> IPs = getAllIpsByInetAddress();
        return IPs.get(ethNum);

    }

    /**
     * 返回所有有效的IP地址，格式为
     *
     * <pre>
     * <网卡号，有效的IP地址>
     * </pre>
     *
     * @return 可能返回空 Map
     */
    public static Map<String, String> getAllIpsByInetAddress() {

        Map<String, String> IPs = new HashMap<String, String>();

        InetAddress address;
        Enumeration<NetworkInterface> allNetInterfaces;
        try {
            allNetInterfaces = NetworkInterface.getNetworkInterfaces();

            while (allNetInterfaces.hasMoreElements()) {
                NetworkInterface netInterface = allNetInterfaces.nextElement();
                // network interface is up and running
                if (netInterface.isUp()) {
                    Enumeration<InetAddress> ips = netInterface.getInetAddresses();
                    while (ips.hasMoreElements()) {
                        address = ips.nextElement();
                        // IPV4 and non-loopback address
                        if (address != null && address instanceof Inet4Address && !address.isLoopbackAddress()
                                && address.getHostAddress().indexOf(":") == -1) {
                            String ethNum = netInterface.getName();
                            String ip = address.getHostAddress();
                            IPs.put(ethNum, ip);
                        }
                    }
                }
            }
        }
        catch (SocketException e) {
            // ignore
        }

        return IPs;
    }

    /**
     * 获取服务器地址，返回一个可用的IP地址
     *
     * @return 若无有效的IP地址，则返回 127.0.0.1
     */
    public static String getServerIp() {

        // return first valid address
        Map<String, String> IPs = getAllIpsByInetAddress();
        for (Map.Entry<String, String> item : IPs.entrySet()) {
            return item.getValue();
        }
        // backup
        String resp = getIpByHostAddress();
        if (resp == null) {
            // default
            resp = "127.0.0.1";
        }

        return resp;
    }

    /**
     * COPY FROM {@code org.springframework.cloud.commons.util.InetUtils}
     *
     * @return may return null
     */
    public static InetAddress findFirstNonLoopbackAddress() {

        InetAddress result = null;
        try {
            int lowest = Integer.MAX_VALUE;
            for (Enumeration<NetworkInterface> nics = NetworkInterface.getNetworkInterfaces(); nics
                    .hasMoreElements();) {
                NetworkInterface ifc = nics.nextElement();
                if (ifc.isUp()) {

                    if (ifc.getIndex() < lowest || result == null) {
                        lowest = ifc.getIndex();
                    }
                    else if (result != null) {
                        continue;
                    }

                    for (Enumeration<InetAddress> addrs = ifc.getInetAddresses(); addrs.hasMoreElements();) {
                        InetAddress address = addrs.nextElement();
                        if (address instanceof Inet4Address && !address.isLoopbackAddress()) {

                            result = address;
                        }
                    }

                }
            }
        }
        catch (IOException ex) {
            // ignore
        }

        if (result != null) {
            return result;
        }

        try {
            return InetAddress.getLocalHost();
        }
        catch (UnknownHostException e) {
            // ignore
        }

        return null;
    }
}
```
### 扩展阅读
#### 1. 利用InetAddress类确定特殊IP地址
http://rainbow702.iteye.com/blog/2066431

#### 2. Spring Cloud Netflix Eureka: 多网卡环境下Eureka服务注册IP选择问题
https://blog.csdn.net/neosmith/article/details/53126924

#### 3. Spring Cloud Eureka 多网卡配置最终版
https://blog.csdn.net/xichenguan/article/details/76632033
