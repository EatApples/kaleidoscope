### ping 与 telnet
```java
public class NetHelper {

    private NetHelper() {
    }

    private final static int TIMEOUT = 5000;

    /**
     * ping 远程IP
     *
     * @param ipStr
     * @return
     */
    public static boolean ping(String ipStr) {

        try {
            InetAddress inetAddress = InetAddress.getByName(ipStr);
            if (inetAddress.isReachable(TIMEOUT)) {
                return true;
            }
        } catch (Exception e) {
            // ignore
        }
        return false;
    }

    /**
     * telnet 远程IP和port
     *
     * @param hostname
     * @param port
     * @return
     */
    public static boolean telnet(String hostname, int port) {

        Socket server = null;
        try {
            server = new Socket();
            InetSocketAddress address = new InetSocketAddress(hostname, port);
            server.connect(address, TIMEOUT);
            return true;
        } catch (Exception e) {
            // ignore
        } finally {
            if (server != null)
                try {
                    server.close();
                } catch (Exception e) {
                    // ignore
                }
        }
        return false;
    }

}
```
