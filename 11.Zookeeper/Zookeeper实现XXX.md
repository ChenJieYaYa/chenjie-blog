# Zookeeper实现XXX

## 一、命名服务

在ZK中文件系统中创建**全局唯一Path**，该Path可作为名称

* 创建持久节点`/services/MyService`用于存储各种服务的名称

* 服务上线在/services/MyService下创建**临时**服务节点，节点内容为IP:Port
* 服务下线时临时节点被自动删除

## 二、配置管理(发布与订阅)

### 1.原理

程序有许多配置，当程序分布于多态机器，显然一台台该数据非常不显示，发布订阅即将数据发布到ZK节点上，供订阅者监听进而动态获取数据，实现配置信息的**集中管理与动态更新**

* 创建持久节点`/mysql`用于存储DB的各种配置信息
* 创建`/mysql`下的配置持久节点`/url`、`/port`、`/username`、`/password`用于存储DB的各种具体配置信息

### 2.代码实现要求

实现部分分为启动客户端与程序客户端两部分

* **启动客户端**：启动客户端启动则添加auth权限认证信息到ZK，保证获取DB连接的用户必须是认证用户，即`zookeeper.addAuthInfo(scheme,auth);`，启动后将DB的连接信息保存到ZK对应节点下
* **程序客户端**：程序客户端启动则创建与ZK的连接，读取ZK中DB配置信息，尝试连接数据库，同时添加监听事件，当ZK中DB信息改变时再次初始化DB连接，注意阻塞主线程，否则主线程一旦结束监听也将不再起作用，可使用`CountDownLatch`或死循环

### 3.代码实现

①**ZkHelper**帮助连接ZK

```java
import org.apache.zookeeper.WatchedEvent;
import org.apache.zookeeper.Watcher;
import org.apache.zookeeper.ZooKeeper;
import java.io.IOException;
import java.util.concurrent.CountDownLatch;

/**
 * ZK连接帮助类——countdownlatch用于阻塞主线程，等待ZK连接上后再接触阻塞
 * ZK通过监视器回调回复连接状态，一旦客户端与ZK连接，监视器回调起作用
 * 回调起作用标志ZK连接完毕，借助主线程阻塞
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
                if (event.getType() == Event.EventType.None) {
                    System.out.println("服务器联接成功");
                }
                if (event.getType() == Event.EventType.NodeCreated) {
                    System.out.println("节点创建成功");
                }
                if (event.getType() == Event.EventType.NodeDataChanged) {
                    System.out.println("节点数据更新成功");

                    //****************再次获取接节点数据
                    MyClient myClient = new MyClient();
                    myClient.initDataSource();
                }
                if (event.getType() == Event.EventType.NodeChildrenChanged) {
                    System.out.println("子节点节点修改成功");
                }
                if (event.getType() == Event.EventType.NodeDeleted) {
                    System.out.println("节点删除成功");
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
```

②**SetConfigClient**启动客户端，用于注册DB配置信息

```java
import org.apache.zookeeper.CreateMode;
import org.apache.zookeeper.KeeperException;
import org.apache.zookeeper.ZooDefs;
import org.apache.zookeeper.ZooKeeper;
import java.io.IOException;

/**
 * 向zk中添加mysql配置信息
 */
public class SetConfigClient {
    //配置信息
    private final static String dburl = "jdbc:mysql://localhost:3306/res?characterEncoding=utf-8";
    private final static String dbport = "3306";
    private final static String dbuname = "root";
    private final static String dbpwd = "xbzz7789";
    
    //节点
    private final static String root = "/mysql";
    private final static String urlNode = root + "/url";
    private final static String portNode = root + "/port";
    private final static String userNode = root + "/username";
    private final static String password = root + "/password";

    //权限
    private final static String auth_type = "digest";
    private final static String auth_password = "yc:a";

    public static void main(String[] args) throws IOException, InterruptedException, KeeperException {
        ZkHelper zkHelper = new ZkHelper();
        ZooKeeper zooKeeper = zkHelper.connect();

        //设置权限
        zooKeeper.addAuthInfo(auth_type, auth_password.getBytes());

        if (zooKeeper.exists(root, true) == null) {
            zooKeeper.create(root, "root".getBytes(), ZooDefs.Ids.CREATOR_ALL_ACL, CreateMode.PERSISTENT);
        }
        if (zooKeeper.exists(urlNode, true) == null) {
            zooKeeper.create(urlNode, dburl.getBytes(), ZooDefs.Ids.CREATOR_ALL_ACL, CreateMode.PERSISTENT);
        }
        if (zooKeeper.exists(portNode, true) == null) {
            zooKeeper.create(portNode, dbport.getBytes(), ZooDefs.Ids.CREATOR_ALL_ACL, CreateMode.PERSISTENT);
        }
        if (zooKeeper.exists(userNode, true) == null) {
            zooKeeper.create(userNode, dbuname.getBytes(), ZooDefs.Ids.CREATOR_ALL_ACL, CreateMode.PERSISTENT);
        }
        if (zooKeeper.exists(password, true) == null) {
            zooKeeper.create(password, dbpwd.getBytes(), ZooDefs.Ids.CREATOR_ALL_ACL, CreateMode.PERSISTENT);
        }
        zkHelper.close();
    }
}
```

③**MyClient**程序客户端，用于获取DB配置信息，并监听

```java
import lombok.Data;
import org.apache.zookeeper.ZooKeeper;
import java.io.IOException;
import java.util.concurrent.CountDownLatch;

/**
 * 获取节点信息，设置数据源
 */
@Data
public class MyClient {
    private static CountDownLatch countDownLatch = new CountDownLatch(10);
    //节点
    private final static String root = "/mysql";
    private final static String urlNode = root + "/url";
    private final static String portNode = root + "/port";
    private final static String userNode = root + "/username";
    private final static String password = root + "/password";

    //权限
    private final static String auth_type = "digest";
    private final static String auth_password = "yc:a";

    //配置信息
    private static String dburl;
    private static String dbport;
    private static String dbuname;
    private static String dbpwd;

    private static ZooKeeper zooKeeper;

    public void initDataSource() {
        zooKeeper.addAuthInfo(auth_type, auth_password.getBytes());
        try {
            dburl = new String(zooKeeper.getData(urlNode, true, null));
            dbport = new String(zooKeeper.getData(portNode, true, null));
            dbuname = new String(zooKeeper.getData(userNode, true, null));
            dbpwd = new String(zooKeeper.getData(password, true, null));
        } catch (Exception e) {
            e.printStackTrace();
        }

        System.out.println("获取到数据库连接池信息，将来可以连接DB");
        System.out.println("url--" + dburl);
        System.out.println("port--" + dbport);
        System.out.println("uname--" + dbuname);
        System.out.println("pwd--" + dbpwd);

        countDownLatch.countDown();
    }

    public static void main(String[] args) throws IOException, InterruptedException {
        ZkHelper zkHelper = new ZkHelper();
        zooKeeper = zkHelper.connect();

        MyClient myClient = new MyClient();
        myClient.initDataSource();

        countDownLatch.await();
    }
}
```

## 三、分步式通知协调

### 1.原理

很好的实现分布式环境下**不同系统之间的通知与协调**，实现对数据变更的**实时处理**，通常做法是不同系统都向ZK上同一个子节点注册，并监听该节点及子节点，当有变化其他系统很快接收到通知并处理

* **心跳检测机制**：检测与被检测系统之间没有直接关联，而是通过ZK某节点关联，大大减小耦合
* **系统调度模式**：某系统由控制台与推送系统两部分组成，控制台的职责是控制推送系统进行相应的推送工作，管理员对控制台操作实际是修改ZK内节点状态，而ZK将变化通知给推送系统，推送系统所出相应推送工作
* **工作汇报模式**：子任务启动则在ZK注册临时节点，并将任务进度写回该临时节点，使管理者实时直到子任务进度

### 2.代码实现要求

模拟Hadoop中NameNode与DataNode的Master-Slaves关系，并反应出心跳信息，DataNode中存放的是对象`Properties`经过序列化后的`byte[]`，利用`ByteArrayOutputStream`将数据以`byte[]`形式写到内存中，当DataNode上线时向ZK中`/hdfs`下注册临时顺序节点，使用`CountDownLatch`保持NameNode服务一直在线，NameNode监听DataNode节点变化，DataNode上线下线NameNode都可以感知到，**首先在ZK中创建`\hdfs`节点**，因为实现中没写这一步

### 3.代码实现

①**ZkHelper**帮助连接ZK

②**DataNode**向ZK注册自己

```java
import org.apache.zookeeper.CreateMode;
import org.apache.zookeeper.KeeperException;
import org.apache.zookeeper.ZooDefs;
import org.apache.zookeeper.ZooKeeper;
import java.io.ByteArrayOutputStream;
import java.io.IOException;
import java.io.ObjectOutputStream;
import java.net.InetAddress;
import java.util.Properties;
import java.util.Scanner;

/**
 * 每个datanode注册信息到zk
 */
public class DataNode {
    private static ZkHelper zkHelper;
    private static ZooKeeper zooKeeper;
    private static String curConnectString;
    
    private static final String hdfsnode = "/hdfs";
    
    private static Scanner scanner = new Scanner(System.in);

    //连接zk
    public void getConnect() throws IOException, InterruptedException {
        if (curConnectString != null) {
            ZkHelper.connectString = curConnectString;
        }
        zkHelper = new ZkHelper();
        zooKeeper = zkHelper.connect();
    }

    //节点保存的数据--随便保存一点数据到DataNode
    public static Properties getnodeinfo() {
        Runtime runtime = Runtime.getRuntime();//jvm-info
        Properties properties = System.getProperties();//os-info
        InetAddress address = null;
        try {
            address = InetAddress.getLocalHost();//ip
        } catch (Exception e) {
            e.printStackTrace();
        }
        String ip = address.getHostAddress();//地址

        properties.setProperty("ip", ip);
        properties.setProperty("hostname", address.getHostName());
        properties.setProperty("totalMemory", String.valueOf(runtime.totalMemory()));
        properties.setProperty("freeMemory", String.valueOf(runtime.freeMemory()));
        properties.setProperty("availableProcessors", String.valueOf(runtime.availableProcessors()));

        return properties;
    }

    //序列化后保存于ZK中作为节点内容数据，同时将序列后的数据转换为字节码
    public byte[] toByteArray(Object obj) {
        byte[] bytes = null;
        //将数据以byte[]形式写到内存中
        ByteArrayOutputStream out = new ByteArrayOutputStream();
        ObjectOutputStream oos = null;
        try {
            oos = new ObjectOutputStream(out);
            oos.writeObject(obj);
            oos.flush();
            //obj中数据以变成byte[]存到内存
            bytes = out.toByteArray();
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            if (oos != null) {
                try {
                    oos.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
            if (out != null) {
                try {
                    out.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
        return bytes;
    }

    //将客户端数据注册到zk
    private void createDataNode() throws IOException, KeeperException, InterruptedException {
        Properties properties = getnodeinfo();
        byte[] bytes = toByteArray(properties);
        String ip = (String) properties.get("ip");
        zooKeeper.create(hdfsnode + "/datanode_" + ip + "_", bytes, ZooDefs.Ids.OPEN_ACL_UNSAFE, CreateMode.EPHEMERAL_SEQUENTIAL);
        System.out.println("创建节点成功");
    }

    public static void main(String[] args) throws IOException, InterruptedException, KeeperException {
        if (args != null && args.length > 0) {
            curConnectString = args[0];
        }

        DataNode dataNode = new DataNode();
        dataNode.getConnect();
        dataNode.createDataNode();

        boolean flag = false;
        int choice = 2;
        while (!flag) {
            System.out.println("输入你的操作 1.创建datanode  2.退出datanode-main");
            choice = scanner.nextInt();
            switch (choice) {
                case 1:
                    System.out.println("创建datanode ");
                    dataNode.createDataNode();
                    break;
                case 2:
                    flag = true;
                    zkHelper.close();
                    scanner.close();
                    break;
                default:
                    System.out.println("没有这个操作");
            }
        }
        System.out.println("datanoe退出");
    }
}
```

③**NameNode**监听DataNode变化

```java
import lombok.SneakyThrows;
import org.apache.zookeeper.*;
import java.io.ByteArrayInputStream;
import java.io.IOException;
import java.io.ObjectInputStream;
import java.util.ArrayList;
import java.util.List;
import java.util.Properties;
import java.util.concurrent.CountDownLatch;

/**
 * 感知zk中/server节点上线及掉线变化
 */
public class NameNode {
    private static ZkHelper zkHelper;
    private static ZooKeeper zooKeeper;

    private static final String hdfsnode = "/hdfs";
    
    //保证NameNode服务不掉线，一直监听DataNode的变化
    private static CountDownLatch countDownLatch = new CountDownLatch(Integer.MAX_VALUE);

    //连接zk
    public void getConnect() throws Exception {
        zkHelper = new ZkHelper();
        zooKeeper = zkHelper.connect();

        if (zooKeeper == null || zooKeeper.getState() != ZooKeeper.States.CONNECTED) {
            System.out.println("连接zk失败  " + zkHelper.getConnectString());
            throw new Exception("连接zk失败  " + zkHelper.getConnectString());
        }

        try {
            zooKeeper.exists(hdfsnode, true);//判断是否存在hdfs节点，顺便添加监听
        } catch (Exception e) {
            zooKeeper.create(hdfsnode, "hdfs_namenode".getBytes(), ZooDefs.Ids.OPEN_ACL_UNSAFE, CreateMode.PERSISTENT);
        }
    }

    //关闭zk
    public void close() {
        if (zooKeeper == null || zooKeeper.getState() != ZooKeeper.States.CONNECTED) {
            try {
                zooKeeper.close();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }

    //获取datanode列表,此时namenode不能掉线，需要不断的获取datanode的更新信息
    public static void getDataNode(String path) throws KeeperException, InterruptedException {
        //与ZkHelper中监听事件区分开，此处建立另一个事件监听
        MyWatcher myWatcher = new MyWatcher();
        //此时绑定了监听事件
        List<String> datanode = zooKeeper.getChildren(path, myWatcher);
        datanode.forEach(System.out::println);
        
        countDownLatch.countDown();
        if (countDownLatch.getCount() == 0) {//一直让NameNoe服务不掉线
            countDownLatch = new CountDownLatch(Integer.MAX_VALUE);
        }
    }

    //反序列化获取datanode的数据信息
    public static List<Properties> getDataNodeData(List<String> datanodes) {
        List<Properties> list = new ArrayList<>();
        datanodes.forEach(datanode -> {
            MyWatcher myWatcher = new MyWatcher();
            try {
                byte[] bytes = zooKeeper.getData(datanode, myWatcher, null);//继续绑定监听
                Properties properties = (Properties) toObject(bytes);
                list.add(properties);
            } catch (Exception e) {
                e.printStackTrace();
            }
        });
        return list;
    }

    //反序列化
    public static Object toObject(byte[] bytes) {
        Object object = null;
        ObjectInputStream oi = null;
        ByteArrayInputStream bai = null;
        try {
            bai = new ByteArrayInputStream(bytes);
            oi = new ObjectInputStream(bai);
            object = oi.readObject();
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            if (oi != null) {
                try {
                    oi.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
            if (bai != null) {
                try {
                    bai.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
        return object;
    }

    public static void main(String[] args) throws Exception {
        //1.实例化服务器
        NameNode nameNode = new NameNode();
        //2.与zk建立连接
        nameNode.getConnect();
        //3.获取节点信息
        getDataNode(hdfsnode);

        countDownLatch.await();
        nameNode.close();
    }
}

class MyWatcher implements Watcher {
    @SneakyThrows
    @Override
    public void process(WatchedEvent event) {
        if (event.getType() == Event.EventType.NodeChildrenChanged) {
            //子节点修改,重新获取子节点信息
            System.out.println("datanode更新");
            NameNode.getDataNode(event.getPath());
        }
        if (event.getType() == Event.EventType.NodeDataChanged) {
            //当前节点数据修改
            System.out.println("datanode数据修改");
        }
    }
}
```

## 四、集群管理

### 1.原理

**集群机器监控**：有机器加入则在ZK中创建临时节点，一旦掉线该临时节点也被删除，其他机器也会收到其他机器上线与掉线的消息，通常用于对机器在线率有较高要求的场景，能快速对集群中的机器变化做出响应

> 以前的做法通常是通过某种手段(ping)定期检测每个机器是否在线，但这样存在两个问题，一是需要发送到每台机器，性能上有一定的消耗，而是存在一定延时

**选举Master**：所有机器创建顺序临时节点，每次选取编号最小的机器即可

> 选举Master为什么使用顺序临时节点？为什么每个节点只监听他的上一个节点？
>
> 为了防止羊群效应，分布式锁实现中具体讲解

### 2.代码实现要求

利用临时节点充当Master节点，当Master点掉线时，ZK会自动删除此节点，其它客户端监听Master节点的删除事件，当Master删除时，其它客户端争抢创建`/master`节点，**为防止主服务器假掉线(比如网络抖动)所导致的服务器信息切换所造成的性能损失，支持Master节点的可重入性**，具体实现是其它非主服务器**延时**争抢创建`/master`节点，为了提高ZK节点安全性，加入了Schema用户认证

具体执行流程是每台服务器启动时，将自己的信息注册到`/servers`下，接着查看Master节点是否存在，如存在则绑定Watcher，不存在则直接创建`/master`节点，如`/master`节点被删除，则其它监听的客户端接收到此事件后，争抢创建`/master`节点

------

以上是**保持独占式**实现方式，而原理部分讲解的**顺序有效式**才可防止羊群效应

### 3.代码实现

①**ZkHelper**帮助连接ZK

②**RunningData**工作服务器信息，将信息序列化与反序列化

```java
import lombok.Data;
import org.apache.jute.*;
import org.apache.zookeeper.server.ByteBufferInputStream;
import java.io.ByteArrayOutputStream;
import java.io.IOException;
import java.nio.ByteBuffer;

@Data
public class RunningData implements Record {
    private Long cid;
    private String name;

    @Override//序列化：ObjectOutputStream -> OutputArchive
    public void serialize(OutputArchive archive, String tag) throws IOException {
        archive.startRecord(this, tag);
        archive.writeLong(cid, "cid");
        archive.writeString(name, "name");
        archive.endRecord(this, tag);
    }

    @Override//反序列化：ObjectInputStream -> InputArchive
    public void deserialize(InputArchive archive, String tag) throws IOException {
        archive.startRecord(tag);
        this.cid = archive.readLong("cid");
        this.name = archive.readString("name");
        archive.endRecord(tag);
    }

    public static void main(String[] args) throws IOException {
        //创建对象
        RunningData data = new RunningData();
        data.setCid(1L);
        data.setName("jj");

        //序列化
        //内存数组字节流，将数据缓存在内存一次性转成byte[]
        ByteArrayOutputStream bout = new ByteArrayOutputStream();
        //Binary+xml+json都只是数据格式
        BinaryOutputArchive archive = BinaryOutputArchive.getArchive(bout);
        //将对象转成字节数组存到内存中
        data.serialize(archive, "header");
        //从内存中取一次得到byte
        byte[] bytes = bout.toByteArray();

        //反序列化
        ByteBuffer buffer = ByteBuffer.wrap(bytes);
        ByteBufferInputStream bin = new ByteBufferInputStream(buffer);
        BinaryInputArchive archive1 = BinaryInputArchive.getArchive(bin);
        RunningData data1 = new RunningData();//空对象，用于接收数据
        data1.deserialize(archive1, "create");
        System.out.println(data1);
    }
}
```

③**WorkServer**工作服务器，对于ZK来说这是客户端，用于注册节点，对于用户来说这是服务器，注册自己

```java
import com.yc.Test1Basic.ZkHelper;
import lombok.SneakyThrows;
import org.apache.jute.BinaryInputArchive;
import org.apache.jute.BinaryOutputArchive;
import org.apache.zookeeper.*;
import org.apache.zookeeper.data.Stat;
import org.apache.zookeeper.server.ByteBufferInputStream;
import java.io.ByteArrayOutputStream;
import java.nio.ByteBuffer;
import java.util.concurrent.Executors;
import java.util.concurrent.ScheduledExecutorService;
import java.util.concurrent.TimeUnit;

public class WorkServer {
    private ZooKeeper zk;

    //记录服务器状态
    private volatile boolean running = false;

    //Master节点对应zk的节点路径
    private static final String MASTER_PATH = "/master";
    //监听Master节点删除事件，当Master节点被删除时重新选举Master
    private Watcher watcher;

    //记录当前节点的基本信息
    private RunningData serverData;
    //记录集群中Master节点的基本信息
    private RunningData masterData;

    //线程池，用于延迟优化，在网络抖动时master节点掉线，当网络恢复时，任然保证当前节点被选举为master
    private ScheduledExecutorService delayExector = Executors.newScheduledThreadPool(Runtime.getRuntime().availableProcessors());

    private int delayTime = 5;

    public void setZk(ZooKeeper zooKeeper) {
        this.zk = zooKeeper;
    }

    public ZooKeeper getZk() {
        return this.zk;
    }

    //初始化当前节点数据RunningData
    public WorkServer(RunningData data) {
        //记录服务器基本信息
        this.serverData = data;
        //设置节点被删除的监听事件
        this.watcher = new Watcher() {
            @Override
            public void process(WatchedEvent event) {
                try {
                    //监听节点被删除
                    if (event.getType() == Event.EventType.NodeDeleted) {
                        //原来有master节点，且原来master节点名字与当前节点名字相同  --> 允许自己再去抢一次master
                        if (masterData != null && masterData.getName().equals(serverData.getName())) {
                            //当前节点是上一轮的master，直接抢 --> 可重入
                            takeMaster();
                        } else {
                            //否则延迟5秒再抢-->优化策略，防止网络抖动使上一个掉线的master可重入
                            delayExector.schedule(() -> {
                                takeMaster();
                            }, delayTime, TimeUnit.SECONDS);
                        }
                    }
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
        };
    }

    //检查自己是否为master
    public boolean checkMaster() {
        try {
            Stat stat = new Stat();
            //获取Master节点数据
            byte[] res = zk.getData(MASTER_PATH, watcher, stat);
            ByteBuffer buffer = ByteBuffer.wrap(res);
            ByteBufferInputStream bbis = new ByteBufferInputStream(buffer);
            BinaryInputArchive bia = BinaryInputArchive.getArchive(bbis);
            RunningData data = new RunningData();
            data.deserialize(bia, "create");
            //反序列化Master节点数据
            masterData = data;
            //与当前服务节点数据对比判断当前服务节点是不是Master节点
            if (masterData.getName().equals(serverData.getName())) {
                return true;
            }
            return false;
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
    }

    //释放master
    public void releaseMaster() throws KeeperException, InterruptedException {
        if (checkMaster()) {//当前服务是Master节点才可以释放
            zk.delete(MASTER_PATH, zk.exists(MASTER_PATH, true).getVersion());
        }
    }

    //争抢master
    public void takeMaster() {
        if (!running) {//服务器没运行，不能争抢
            return;
        }
        try {
            //序列化master节点信息
            ByteArrayOutputStream bout = new ByteArrayOutputStream();
            BinaryOutputArchive archive = BinaryOutputArchive.getArchive(bout);
            serverData.serialize(archive, "header");

            //尝试创建master临时节点，失败则抛出异常
            String r = zk.create(MASTER_PATH, bout.toByteArray(), ZooDefs.Ids.OPEN_ACL_UNSAFE, CreateMode.EPHEMERAL);
            //当前服务器争抢master节点成功
            masterData = serverData;
            System.out.println(serverData.getName() + "成为master");

            //TODO 此处只是模拟master节点掉线的情况,上线不需要
            delayExector.schedule(new Runnable() {
                @SneakyThrows
                @Override
                public void run() {
                    if (checkMaster()) {
                        releaseMaster();//释放master
                    }
                }
            }, 5, TimeUnit.SECONDS);
        } catch (KeeperException.NodeExistsException e) {
            //在创建master节点时master节点已经存在，只能获取节点信息,绑定监听
            try {
                Stat stat = new Stat();
                //获取master节点数据
                byte[] res = zk.getData(MASTER_PATH, watcher, stat);
                if (res == null) {//读取瞬间master节点宕机了，有机会再次争抢
                    takeMaster();
                } else {
                    //TODO 测试代码：获取master节点的服务器信息
                    ByteBuffer buffer = ByteBuffer.wrap(res);
                    ByteBufferInputStream bbis = new ByteBufferInputStream(buffer);
                    BinaryInputArchive bia = BinaryInputArchive.getArchive(bbis);
                    RunningData data = new RunningData();
                    data.deserialize(bia, "create");
                    masterData = data;
                }
            } catch (Exception exception) {
                exception.printStackTrace();
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
    }


    //启动服务器 1.注册到/servers下 2.判断是否有master节点,有则监听,无则创建
    public void start() throws Exception {
        if (running) {//服务器不能重复启动
            throw new Exception("服务已启动");
        }
        running = true;

        //1.在servers下创建自己的临时节点，相当于注册服务器-->序列化后存到节点中
        ByteArrayOutputStream bout = new ByteArrayOutputStream();
        BinaryOutputArchive archive = BinaryOutputArchive.getArchive(bout);
        //序列化
        serverData.serialize(archive, "header");

        //2.在servers下创建自己的服务器的节点信息，相当于服务注册，临时节点
        String r = zk.create("/servers/" + serverData.getCid(), bout.toByteArray(), ZooDefs.Ids.OPEN_ACL_UNSAFE, CreateMode.EPHEMERAL);

        //3.监听master节点删除事件的监听，第一个节点登时此/master节点不存在，则会抛出异常NoNodeException，进入catch争抢master节点
        Stat stat = new Stat();//订阅master节点删除事件
        try {
            //已经有/master，则绑定监听
            zk.getData(MASTER_PATH, watcher, stat);
        } catch (KeeperException.NoNodeException e) {
            //没有/master节点，则争抢master节点
            takeMaster();
        }
    }

    //停止服务
    public void stop() throws Exception {
        if (!running) {
            throw new Exception("服务已停止");
        }
        running = false;
        delayExector.shutdown();
        //释放master权力
        releaseMaster();
    }

    //测试
    public static void main(String[] args) throws Exception {
        RunningData data = new RunningData();
        data.setCid(1L);
        data.setName("hello");

        ZkHelper zkHelper = new ZkHelper();

        WorkServer workServer = new WorkServer(data);
        workServer.setZk(zkHelper.connect());
        workServer.start();

        Thread.sleep(100000);
    }
}
```

④**Test**测试类，当成调度器来看，模拟上线后有对各workserver,同时master每隔5s掉线的情况

```java
import com.yc.Test1Basic.ZkHelper;
import org.apache.zookeeper.ZooKeeper;
import java.io.BufferedReader;
import java.io.InputStreamReader;
import java.util.ArrayList;
import java.util.List;

public class Test {
    //启动的服务个数
    private static final int CLIENT_QTY = 10;

    public static void main(String[] args) throws Exception {
        //保存所有zk
        List<ZooKeeper> clients = new ArrayList<>();
        //保存所有服务列表
        List<WorkServer> workServers = new ArrayList<>();
        ZkHelper zkHelper = new ZkHelper();
        try {
            for (int i = 0; i < CLIENT_QTY; i++) {
                ZooKeeper zk = zkHelper.connect();
                clients.add(zk);
                //创建serverData
                RunningData runningData = new RunningData();
                runningData.setCid(Long.valueOf(i));
                runningData.setName("client-" + i);
                //创建服务
                WorkServer workServer = new WorkServer(runningData);
                workServer.setZk(zk);
                workServers.add(workServer);
                //开启服务
                workServer.start();
            }
            //让程序卡住在次数，等待键盘录入
            System.out.println("回车退出");
            new BufferedReader(new InputStreamReader(System.in)).readLine();
        } finally {
            System.out.println("服务器关闭");
            for (WorkServer workServer : workServers) {
                try {
                    workServer.stop();
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
            for (ZooKeeper zooKeeper : clients) {
                zooKeeper.close();
            }
        }
    }
}
```

## 五、分步式锁

### 1.原理

ZK实现分步式锁有两种方式**保持独占式和顺序有效式**

- **保持独占式**：将ZK中某节点看成锁，成功创建表示获取锁，删除表示释放锁
- **顺序有效式**：当客户端获取锁时在/locks下建立临时有序节点/lock-000x，每个节点监听上一个节点，当上一个节点被删除时，下一个节点自然的获取到锁，而不是争抢

> 顺序有效式可以防止羊群效应，羊群效应指如果/locks下的子节点监听的都是当前获取到抢票权的节点，一旦当前获取到锁的节点删除则每一个子节点的监听都需要重新设置，及其影响性能

### 2.代码实现要求

接下来只通过顺序有效式实现分布式锁

每个客户端抢票就相当于每个线程获取锁，获取锁就在ZK的/locks目录下创建一个临时有序的节点，创建成功后获取/locks目录下的所有临时节点，判断当前线程的节点序号是不是最小的，若是则获取锁成功，若不是则对当前节点的前一个节点添加监听事件

> 比如当前线程获取到的节点序号为/locks/seq-00000003，所有的节点列表为[/locks/seq-00000001,/locks/seq-00000002,/locks/seq-00000003]，则对/locks/seq-00000002这个节点添加一个事件监听器

锁释放则删除该临时节点，后一个节点监听到删除事件则判断自己的序号是不是最小，尝试获取锁

> 比如/locks/seq-00000001释放了，/locks/seq-00000002监听到事件，此时节点集合为[/locks/seq-00000002,/locks/seq-00000003]，则/locks/seq-00000002为最小序号节点，获取到锁

### 3.实现

①导包

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
    <version>2.7.0</version>
</dependency>
```

②**ZkHelper**帮助连接ZK

③**ZKLock**实现分布式锁

```java
import org.apache.zookeeper.*;
import org.apache.zookeeper.data.Stat;
import java.io.IOException;
import java.util.Collections;
import java.util.List;
import java.util.concurrent.CountDownLatch;

public class ZKLock {
    private ZooKeeper zooKeeper;
    private int sessionTimeOut = 2000;
    private CountDownLatch latch = new CountDownLatch(1);//zk连接等待
    private CountDownLatch waitLock = new CountDownLatch(1);//等待前一个节点被删除，然后获取锁
    private String lock = "/locks";//根节点
    private String preNode;//等待的上一个节点
    private String currentNode;//当前获取到锁的节点

    public ZKLock(String connectionStr) throws IOException, InterruptedException, KeeperException {
        //1.连接zookeeper，需要阻塞，防止主线程运行太快，监听是否连接到zookeeper，连接到给一个提示
        zooKeeper = new ZooKeeper(connectionStr, sessionTimeOut, new Watcher() {
            @Override
            public void process(WatchedEvent event) {
                if (event.getState() == Event.KeeperState.SyncConnected) {//连接到zk
                    //System.out.println("连接zk成功");
                    latch.countDown();
                }
                if (event.getType() == Event.EventType.NodeDeleted && event.getPath().equals(preNode)) {//子节点删除事件，且被删除的是前一个节点
                    waitLock.countDown();
                }
            }
        });
        //阻塞主线程，等待创建节点完成后解除阻塞
        latch.await();
        //2.看是否存在/locks节点(持久节点)，没有则创建
        Stat stat = zooKeeper.exists(lock, false);
        if (stat == null) {
            zooKeeper.create(lock, "locks".getBytes(), ZooDefs.Ids.OPEN_ACL_UNSAFE, CreateMode.PERSISTENT);
        }
    }

    public void lock() {
        try {
            //1.创建临时有序节点到/locks,记录节点--全局currentNode
            currentNode = zooKeeper.create(lock + "/lock-", null, ZooDefs.Ids.OPEN_ACL_UNSAFE, CreateMode.EPHEMERAL_SEQUENTIAL);
            //2.获取/locks下所有子节点列表
            List<String> list = zooKeeper.getChildren(lock, false);
            //3.判断子节点长度，长度=1说明只有一个节点，直接获取锁返回
            if (list.size() == 1) {
                //System.out.println("没有客户端争抢锁，直接拿到锁返回");
                return;
            }
            //4.长度>1就表示有多个子节点，取出所有子节点，排序，取出序列最小的节点获取锁
            Collections.sort(list);
            //lock-00000x
            String curNode = currentNode.substring((lock + "/").length());
            //获取当前节点在排好序的子节点list中的位置
            int index = list.indexOf(curNode);
            if (index == 0) {
                //System.out.println("没有客户端争抢锁，直接拿到锁返回");
                return;
            }

            //运行到此处表示当前服务器一定需要等待
            //5.此时找到这个节点的上一个序号节点进行监听
            preNode = lock + "/" + list.get(index - 1);
            //开启前一个节点的监听通知机制——对应zookeeper创建中的监听，知道前一个节点删除，且前一个节点是当前服务器才解除阻塞
            zooKeeper.getData(preNode, true, new Stat());
            waitLock.await();
            return;
        } catch (KeeperException e) {
            e.printStackTrace();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }

    public void unlock() {
        //1.删除当前持有锁的节点--全局currentNode
        try {
            zooKeeper.delete(currentNode, -1);
        } catch (InterruptedException e) {
            e.printStackTrace();
        } catch (KeeperException e) {
            e.printStackTrace();
        }
    }
}
```

④**TicketController**模拟抢票场景测试分布式锁

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.data.redis.core.StringRedisTemplate;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class TicketController {
    @Value("${server.port}")
    private int port;

    @Autowired
    StringRedisTemplate stringRedisTemplate;

    @RequestMapping("/secKill")
    public String secKill() throws Exception {
        //每个请求都创建一把可重入锁，分步式锁这里有多把没关系，zookeeper中的节点只有一个
        ZKLock lock = new ZKLock("localhost:2181");
        try {
            lock.lock();

            //取出票的数量
            int num = Integer.parseInt(stringRedisTemplate.opsForValue().get("goods"));
            if (num > 0) {//还有票
                stringRedisTemplate.boundValueOps("goods").decrement();
                System.out.println("服务器: " + port + " 抢票成功 " + num);
                return "服务器: " + port + " 抢票成功 " + num;
            } else {//无票
                System.out.println("服务器: " + port + " 抢票失败 ");
                return "服务器: " + port + " 抢票失败 ";
            }
        } catch (Exception e) {
            e.printStackTrace();
            return "异常";
        } finally {
            lock.unlock();
        }
    }
}
```

## 六、分步式队列

### 1.原理

ZK实现分步式队列有两种方式**同步队列和先进先出队列**，此分布式队列为无界队列

* **FIFO队列**：与分布式锁中**顺序有效式**原理一致，入列有编号，出列按编号

* **同步队列**：预先定义成员数量，等到等待队列中的数量与预先定义的数量相同时才统一按序执行

### 2.代码实现原理

队列中提供入队方法offer(T)和出队方法poll() 

* **FIFO队列**：入队时在`/queue`创建临时有序节点，出队时取出`/queue`下的所有节点并排序，取出节点编号最小的节点删除，并返回该节点存储内内容

* **同步队列**：入队时在`/queue`创建临时有序节点，同时在队列外空时使用`CounDownLatch.await()`阻塞，监听`/queue`下节点变化，当节点数量到达时`coundown()`解除阻塞

> 另一种实现方式，在`/queue`创建`/queue/num`保存队列大小n，之后每次有入队操作前先判断是否已到达队列大小，适用于大任务Task A需要很多子任务就绪才能运行的情况

### 3.代码实现

#### 3.1.FIFO队列

①**ZkHelper**帮助连接ZK

②**Order**节点中待存信息

```java
import lombok.Data;
import java.io.Serializable;

/**
 * 订单信息--待存的数据
 */
@Data
public class Order implements Serializable {
    private static final long serialVersionUID = 3674837818221468988L;
    String id;
    String name;

    public Order() {
    }

    public Order(String id, String name) {
        this.id = id;
        this.name = name;
    }
}
```

③**SimpleDistributeQueue**简单分步式队列，通过顺序持久节点实现

```java
import org.apache.zookeeper.CreateMode;
import org.apache.zookeeper.KeeperException;
import org.apache.zookeeper.ZooDefs;
import org.apache.zookeeper.ZooKeeper;
import org.apache.zookeeper.data.Stat;
import java.io.ByteArrayInputStream;
import java.io.ByteArrayOutputStream;
import java.io.ObjectInputStream;
import java.io.ObjectOutputStream;
import java.util.Collections;
import java.util.Comparator;
import java.util.List;

public class SimpleDistributeQueue<T> {//存入的数据当成泛型看
    protected ZooKeeper zk;

    protected String root;//queue节点
    protected static final String NODE_NAME = "n_";//顺序节点前缀

    //初始化
    public SimpleDistributeQueue(ZooKeeper zk, String root) {
        this.zk = zk;
        this.root = root;
    }

    //队列大小
    public int size() throws KeeperException, InterruptedException {
        return zk.getChildren(root, false).size();
    }

    //判断队列是否为空
    public boolean isEmpty() throws KeeperException, InterruptedException {
        return size() == 0;
    }

    //向队列存数据==向根节点下添加持久顺序节点
    public boolean offer(T element) throws Exception {
        //拼接顺序节点的路径
        String nodepath = root + "/" + NODE_NAME;
        try {
            //以对象流输出到内存，获取byte[]内容
            ByteArrayOutputStream bout = new ByteArrayOutputStream();
            ObjectOutputStream oout = new ObjectOutputStream(bout);
            oout.writeObject(element);
            oout.flush();

            //创建节点
            zk.create(nodepath, bout.toByteArray(), ZooDefs.Ids.OPEN_ACL_UNSAFE, CreateMode.PERSISTENT_SEQUENTIAL);
        } catch (KeeperException.NoNodeException e) {
            //没有root节点，创建一个在offer
            zk.create(root, "".getBytes(), ZooDefs.Ids.OPEN_ACL_UNSAFE, CreateMode.PERSISTENT);
            offer(element);
        } catch (Exception e) {
            throw new RuntimeException(e);
        }
        return true;
    }

    //获取节点上的数字序号
    protected String getNodeNumber(String str, String nodename) {
        int index = str.lastIndexOf(nodename);
        if (index >= 0) {
            index += NODE_NAME.length();
            return index <= str.length() ? str.substring(index) : "";
        }
        return str;
    }

    //在队列获取数据==对顺序节点排序，取出编号最小节点的数据反序列化成对象并删除改节点
    public T poll() throws KeeperException, InterruptedException {
        try {
            //获取root所有子节点
            List<String> list = zk.getChildren(root, false);
            if (list == null) {
                return null;
            }

            //排序
            Collections.sort(list, new Comparator<String>() {
                @Override
                public int compare(String lhs, String rhs) {
                    return getNodeNumber(lhs, NODE_NAME).compareTo(getNodeNumber(rhs, NODE_NAME));
                }
            });

            //循环每个排序节点
            for (String nodename : list) {
                //构造顺序节点的完整路径
                String nodefullpath = root + "/" + nodename;
                try {
                    //读取顺序节点的内容，反序列化
                    Stat stat = new Stat();
                    byte[] content = zk.getData(nodefullpath, false, stat);
                    ByteArrayInputStream bin = new ByteArrayInputStream(content);
                    ObjectInputStream oin = new ObjectInputStream(bin);

                    //删除节点
                    zk.delete(nodefullpath, zk.exists(nodefullpath, false).getVersion());

                    //返回被删节点信息，注意此处返回了，循环并不会删除所有节点
                    return (T) oin.readObject();
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
            return null;
        } catch (Exception e) {
            throw new RuntimeException(e);
        }
    }
}
```

④**Test**测试类

```java
import com.yc.Test01_API.ZkHelper;
import org.apache.zookeeper.ZooKeeper;
import java.io.IOException;

public class Test {
    public static void main(String[] args) throws IOException, InterruptedException {
        ZkHelper zkHelper = new ZkHelper();
        ZooKeeper zk = zkHelper.connect();
        
        SimpleDistributeQueue<Order> queue = new SimpleDistributeQueue(zk, "/queue");

        Order order1 = new Order("1", "apple");
        Order order2 = new Order("2", "mac");

        try {
            queue.offer(order1);
            queue.offer(order2);

            Thread.sleep(10000);

            Order u1 = queue.poll();
            Order u2 = queue.poll();

            if (order1.getId().equals(u1.getId()) && order2.getId().equals(u2.getId())) {
                System.out.println("成功");
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

#### 3.2.同步队列

在3.1.FIFO队列基础上做出改变

①**BlockingDistributeQueue**阻塞分步式队列，继承简单分步式队列，重写poll方法

```java
import com.yc.Test06_DistributeQueue.version1.SimpleDistributeQueue;
import lombok.SneakyThrows;
import org.apache.zookeeper.KeeperException;
import org.apache.zookeeper.WatchedEvent;
import org.apache.zookeeper.Watcher;
import org.apache.zookeeper.ZooKeeper;
import java.util.concurrent.CountDownLatch;

public class BlockingDistributeQueue<T> extends SimpleDistributeQueue<T> {//存入的数据当成泛型看

    //初始化
    public BlockingDistributeQueue(ZooKeeper zk, String root) {
        super(zk, root);
    }

    //在队列获取数据==对顺序节点排序，取出编号最小节点的数据反序列化成对象并删除改节点
    @Override
    public T poll() throws KeeperException, InterruptedException {
        boolean flag = true;
        while (flag) {
            final CountDownLatch countDownLatch = new CountDownLatch(1);
            final Watcher watcher = new Watcher() {
                @SneakyThrows
                @Override
                public void process(WatchedEvent event) {
                    if (event.getType() == Event.EventType.NodeChildrenChanged) {
                        countDownLatch.countDown();
                    }
                    //重新绑定监听
                    zk.getChildren(root, this);
                }
            };
            zk.getChildren(root, watcher);//绑定监听
            try {
                T node = (T) super.poll();//调用父的方法获取队列数据

                if (node == null) {
                    countDownLatch.await();
                } else {
                    return node;//直到最后节点才返回
                }
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
        return null;
    }
}
```

②**Test**测试类

```java
import com.yc.Test01_API.ZkHelper;
import com.yc.Test06_DistributeQueue.version1.Order;
import lombok.SneakyThrows;
import org.apache.zookeeper.ZooKeeper;
import java.io.IOException;
import java.util.Random;

public class Test {
    //生产线程
    static class ProducerThread implements Runnable {
        BlockingDistributeQueue<Order> queue;
        public ProducerThread(BlockingDistributeQueue<Order> queue) {
            this.queue = queue;
        }

        @SneakyThrows
        @Override
        public void run() {
            Random random = new Random();
            for (int i = 0; i < 100; i++) {
                Thread.sleep((int) Math.random() * 500);
                queue.offer(new Order(random.nextInt(100) + "", i + 1 + ""));
            }
        }
    }

    //消费线程
    static class ConsumerThread implements Runnable {
        BlockingDistributeQueue<Order> queue;
        public ConsumerThread(BlockingDistributeQueue<Order> queue) {
            this.queue = queue;
        }

        @SneakyThrows
        @Override
        public void run() {
            for (int i = 0; i < 100; i++) {
                Thread.sleep((int) Math.random() * 500);
                System.out.println(Thread.currentThread().getName() + "==" + queue.poll());
            }
        }
    }

    public static void main(String[] args) throws IOException, InterruptedException {
        ZkHelper zkHelper = new ZkHelper();
        ZooKeeper zk = zkHelper.connect();
        final BlockingDistributeQueue<Order> queue = new BlockingDistributeQueue<>(zk, "/queue");

        new Thread(new ConsumerThread(queue)).start();
        new Thread(new ConsumerThread(queue)).start();
        new Thread(new ConsumerThread(queue)).start();
        new Thread(new ProducerThread(queue)).start();
    }
}
```









