# Zookeeper与Java

## 一、导包

```java
<dependency>
	<groupId>org.apache.zookeeper</groupId>
	<artifactId>zookeeper</artifactId>
	<version>3.4.6</version>
</dependency>
```

## 二、连接ZK客户端

```java
/**
 * ZK连接帮助类——countdownlatch用于阻塞主线程，等待ZK连接上后再结束阻塞
 * ZK通过监视器回调process回复连接状态，一旦客户端与ZK连接，监视器回调起作用
 * 回调起作用标志ZK连接完毕，结束主线程阻塞
 * 阻塞解决主程序太快，zk还没来得及连接上就结束
 */
public class ZkHelper {
    private static String connectString = "localhost:2181";
    private static int sessionTimeout = 9000;
    private static ZooKeeper zooKeeper = null;
    final CountDownLatch countDownLatch = new CountDownLatch(1);//阻塞主线程

    public ZooKeeper connect() throws InterruptedException, IOException {
        System.out.println("ZK客户端初始化");
        zooKeeper = new ZooKeeper(connectString, sessionTimeout, new Watcher() {
            @Override
            public void process(WatchedEvent event) {
                if (event.getState() == Event.KeeperState.SyncConnected) {
                    System.out.println("ZK客户端建立连接");
                    countDownLatch.countDown();
                }
            }
        });
        countDownLatch.await();
        return zooKeeper;
    }

    public void close() throws InterruptedException {
        zooKeeper.close();
    }

    public String getConnectString() {
        return connectString;
    }
}
-----------------------------------------
//客户端
main(){
    zkHelper = new ZkHelper();
    zooKeeper = zkHelper.connect();
    //业务代码
    zkHelper.close();
    System.out.println("ZK客户端运行完毕，关闭连接");
}
```

## 三、创建节点

```java
private static ZkHelper zkHelper;
private static ZooKeeper zooKeeper;

public static void create(String path, byte[] data) throws KeeperException, InterruptedException {
    //方案一：写列表  create -s -e /node[path] 'hello'[data] ACL[List<Acl> = 权限+schema+id]
    //ACL参数   1.perms:二进制表示权限cdrwa 有1则有权限  全有则为11111=31
    //         2.ACL：schema+id
    /*List<ACL> list = new ArrayList<>();
    ACL acl1 = new ACL(31, new Id("world", "anyone"));
    list.add(acl1);
    ACL acl2 = new ACL(31, new Id("ip", "127.0.0.1"));
    list.add(acl2);
    String node = zooKeeper.create(path, data, list, CreateMode.PERSISTENT);*/

    //方案二
    String node = zooKeeper.create(path, data, ZooDefs.Ids.OPEN_ACL_UNSAFE, CreateMode.PERSISTENT);
    System.out.println("创建节点：" + node);
    /**
     * 看底层  public String create(final String path, byte data[], List<ACL> acl,CreateMode createMode)
     * 1.String path：节点名
     * 2.byte data[]：节点数据
     * 3.--> List<ACL>
     *     ACL 构造方法初始化两个参数，最后将ACL包装成List
     *          1> perms：二进制形式表示权限cdrwa，有1则有权限 ，全有则为11111=31
     *          2> Id id：构造函数初始化两个参数 scheme+id
     *  -->ZooDefs.Ids.OPEN_ACL_UNSAFE
     *      ArrayList<ACL> OPEN_ACL_UNSAFE = new ArrayList<ACL>(Collections.singletonList(new ACL(Perms.ALL, ANYONE_ID_UNSAFE)));
     *          1.singletonList；将ACL转为List格式-->适配器模式
     *          2.new ACL(Perms.ALL, ANYONE_ID_UNSAFE)
     *              1>.Perms：位运算传入权限
     *                  public interface Perms {
     *                      int READ = 1 << 0;      //1
     *                      int WRITE = 1 << 1;     //10
     *                      int CREATE = 1 << 2;    //100
     *                      int DELETE = 1 << 3;    //1000
     *                      int ADMIN = 1 << 4;     //10000
     *                      int ALL = READ | WRITE | CREATE | DELETE | ADMIN;   //11111
     *                  }
     *              2>.ANYONE_ID_UNSAFE
     *                   Id ANYONE_ID_UNSAFE = new Id("world", "anyone");
     *        实际就是不用手动写权限+schema+id信息，使用工具
     * 4.CreateMode.PERSISTENT ：节点类型 -e -s -->枚举实现(大量用到枚举)
     */
}
------------------------------------------
main(){
	String path = "/mynode1";
    byte[] data = "hello mynode1".getBytes();
    zkHelper = new ZkHelper();
    zooKeeper = zkHelper.connect();
    create(path, data);
    zkHelper.close();
    System.out.println("关闭ZK");
}
```

## 四、判断节点是否存在

```java
//检查znode存在，并获取此节点信息  用到countdownlatch
public class Test03_nodeExist_stat_update {
    private static ZkHelper zkHelper;
    private static ZooKeeper zooKeeper;

    //此监听器最多起到三次作用
    private static CountDownLatch countDownLatch = new CountDownLatch(3);

    public static Stat znodeExits(String path) throws KeeperException, InterruptedException {
        return zooKeeper.exists(path, true);
    }

    public static void main(String[] args) throws IOException, InterruptedException, KeeperException {
        String path = "/mynode1";

        zkHelper = new ZkHelper();
        zooKeeper = zkHelper.connect();

        Stat stat = znodeExits(path);

        if (stat == null) {
            System.out.println("节点不存在！");
            return;
        }

        System.out.println("**********************");
        System.out.println("初始状态：" + ZnodeUtil.printZnodeInfo(stat));
        System.out.println("**********************");

        byte[] bytes = zooKeeper.getData(path, new MyWatcher(path, countDownLatch, zooKeeper), stat);
        //打印节点信息
        String da = new String(bytes, "UTF-8");
        System.out.println("主线程获取节点" + path + " 原始数据 " + da);
        System.out.println(path + " 新信息");
        System.out.println(ZnodeUtil.printZnodeInfo(stat));

        /**
         * 程序执行过程讲解：
         * 1.运行主程序，在getData时绑定了自定义实现的监听器MyWatcher，直到await主程序阻塞，若主程序不阻塞则主程序会直接结束，绑定的监听不起作用
         * 2.当当前节点内容改变，回调绑定的自定义监听器中的process方法，打印节点信息并且再次绑定监听，同时countDown
         * 3.每次修改操作会countDown一次，同时继续绑定监听，直到countDown减为0，主线程解除阻塞，程序运行结束
         */

        countDownLatch.await();

        zkHelper.close();
        System.out.println("关闭ZK");
    }
}

class MyWatcher implements Watcher {
    private String path;
    private CountDownLatch countDownLatch;
    private ZooKeeper zooKeeper;

    public MyWatcher(String path, CountDownLatch countDownLatch, ZooKeeper zooKeeper) {
        this.path = path;
        this.countDownLatch = countDownLatch;
        this.zooKeeper = zooKeeper;
    }

    @SneakyThrows
    @Override
    public void process(WatchedEvent event) {
        if (event.getType() == Event.EventType.NodeDataChanged) {
            Stat stat = new Stat();
            //获取子节点更新后的数据
            //1.只绑定一次监听，监听器用完会被删除
            //byte[] data = zooKeeper.getData(path,false,stat);
            //2.继续绑定监听器
            byte[] data = zooKeeper.getData(path, MyWatcher.this, stat);

            //打印节点信息
            String da = new String(data, "UTF-8");
            System.out.println(path + " 新数据 " + da);
            System.out.println(path + " 新信息");
            System.out.println(ZnodeUtil.printZnodeInfo(stat));

            //监听次数-1
            countDownLatch.countDown();
        } else {
            //若需要输出其他监控信息，继续添加else-if
            System.out.println("事件类型：" + event.getType());
        }
    }
}
```

## 五、更新节点

```java
public static void update(String path, byte[] data) throws KeeperException, InterruptedException {
	//更新节点最小版本，exists返回Stat获取版本号
	Stat stat = zooKeeper.exists(path, true);
	//获取version
	zooKeeper.setData(path, data, stat.getVersion());
}
```

## 六、获取子节点

### 1.一级子节点

```java
public static Stat znodeExits(String path) throws KeeperException, InterruptedException {
    return zooKeeper.exists(path, true);
}

public static void main(String[] args) throws IOException, InterruptedException, KeeperException {
    String path = "/";

    zkHelper = new ZkHelper();
    zooKeeper = zkHelper.connect();

    Stat stat = znodeExits(path);
    if (stat == null) {
        System.out.println("子节点不存在");
        return;
    }
    List<String> list = zooKeeper.getChildren(path, true);
    list.forEach(System.out::println);

    zkHelper.close();
    System.out.println("关闭ZK");
}
```

### 2.本身与所有子节点

```java
public static Stat znodeExits(String path) throws KeeperException, InterruptedException {
    return zooKeeper.exists(path, true);
}

public static void showChildNode(String path, int level) throws Exception {
    Stat stat = znodeExits(path);
    if (stat == null) {
        System.out.println("子节点不存在");
        return;
    }

    //拼接不同目录级的缩进
    StringBuffer sb = new StringBuffer();
    for (int i = 0; i < level; i++) {
        sb.append("    ");
    }
    System.out.println(sb.toString() + path);

    List<String> list;
    try {
        list = zooKeeper.getChildren(path, true);
    } catch (Exception e) {
        //有异常结束递归
        return;
    }
    
    list.forEach(childNode -> {
        if (level == 0) {
            try {
                showChildNode(path + childNode, level + 1);
            } catch (Exception e) {
                e.printStackTrace();
            }
        } else {
            try {
                showChildNode(path + "/" + childNode, level + 1);
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
    });
}

public static void main(String[] args) throws Exception {
    String path = "/";

    zkHelper = new ZkHelper();
    zooKeeper = zkHelper.connect();

    showChildNode(path, 0);

    zkHelper.close();
    System.out.println("关闭ZK");
}
```

## 七、删除节点

```java
public static void delete(String path) throws KeeperException, InterruptedException {
    zooKeeper.delete(path, zooKeeper.exists(path, true).getVersion());
}
```

## 八、格式化节点信息

```java
import org.apache.zookeeper.data.Stat;
import java.text.SimpleDateFormat;

public class ZnodeUtil {
    private static SimpleDateFormat df = new SimpleDateFormat("yyyy年MM月dd日 HH:mm:ss");

    public static String printZnodeInfo(Stat stat) {
        StringBuffer sb = new StringBuffer();
        sb.append("\n*******************************\n");
        sb.append("创建znode的事务id czxid:" + stat.getCzxid() + "\n");
        //格式化时间
        sb.append("创建znode的时间 ctime:" + df.format(stat.getCtime()) + "\n");
        sb.append("更新znode的事务id mzxid:" + stat.getMzxid() + "\n");
        sb.append("更新znode的时间 mtime:" + df.format(stat.getMtime()) + "\n");
        sb.append("更新或删除本节点或子节点的事务id pzxid:" + stat.getPzxid() + "\n");
        sb.append("子节点数据更新次数 cversion:" + stat.getCversion() + "\n");
        sb.append("本节点数据更新次数 dataVersion:" + stat.getVersion() + "\n");
        sb.append("节点ACL(授权信息)的更新次数 aclVersion:" + stat.getAversion() + "\n");
        if (stat.getEphemeralOwner() == 0) {
            sb.append("本节点为持久节点\n");
        } else {
            sb.append("本节点为临时节点,创建客户端id为:" + stat.getEphemeralOwner() + "\n");
        }
        sb.append("数据长度为:" + stat.getDataLength() + "字节\n");
        sb.append("子节点个数:" + stat.getNumChildren() + "\n");
        sb.append("\n*******************************\n");
        return sb.toString();
    }
}
```









