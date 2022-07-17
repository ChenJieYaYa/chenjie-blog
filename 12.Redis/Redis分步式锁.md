# Redis分步式锁















Redis分布式锁带来的系列问题及解决
1.Redisson
分布式锁可能存在锁过期释放，业务没执行完的问题
能不能将锁的过期时间设置得长点来解决此问题呢？显然是不太好的，业务的执行时间是不确定的
Redisson解决该问题，给获得锁得线程开启定时守护线程，每隔一段时间检查锁是否存在，存在则延长锁的过期时间，防止锁过期提前释放

![在这里插入图片描述](https://img-blog.csdnimg.cn/9c6d100d6d734c509b67388040c89c6c.png)

2.Redlock算法
线程一在Redis的master节点上拿到了锁，但是加锁的key还没同步到slave节点，恰好这时master节点发生故障，一个slave节点就会升级为master节点，线程二就可以获取同个key的锁啦，但线程一也已经拿到锁了，锁的安全性就没了

![在这里插入图片描述](https://img-blog.csdnimg.cn/71e1d0d80acb426d80091c8699415a43.png)

Redlock解决这个问题，即部署多个Redis master以保证它们不会同时宕掉，并且这些master节点是完全相互独立的，相互之间不存在数据同步，实现步骤如下

![img](https://img-blog.csdnimg.cn/2048da489d09419599923363cc9dcc98.png)