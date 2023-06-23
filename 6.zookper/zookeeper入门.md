# Zookeeper入门

## 概述

- Zookeeper是一个开源的分布式的，为分布式框架提供协调服务的Apache项目
- Zookeeper从设计模式角度来理解：是一个基于观察者模式设计的分布式服务管理框架，它负责存储和管理大家都关心的数据，然后接受观察者的注册，一旦这些数据的状态发生变化，Zookeeper就将负责通知已经在Zookeeper上注册的那些观察者做出相应的反应
- 大概工作流程
  1. 服务端启动时去注册信息（创建都是临时节点）
  2. 获取到当前在线服务器列表，并且注册监听
  3. 服务器节点下线
  4. 客户端收到服务器节点上下线事件通知
- 观察者设计模式
  - 观察者模式（Observer Pattern）是一种行为型设计模式，它在面向对象编程中被广泛应用。当对象间存在一对多关系时，则使用观察者模式。比如，当一个对象被修改时，则会自动通知依赖它的对象

## Zookeeper特点

1. Zookeeper：一个跟随者（Leader），多个跟随者（Follower）组成的集群
2. 集群中只要有半数以上节点存活，Zookeeper集群就能正常服务，所以Zookeeper适合安装奇数台服务器
3. 全局数据一致：每个Server保存一份相同的数据副本，CLient无论连接到哪个Server，数据都是一致的
4. 更新请求顺序执行，来自同一个Client的更新请求按其发送顺序依次执行
5. 数据更新原子性，一次数据更新要么成功，要么失败
6. 实时性，在一定时间范围内，Client能读到最新数据

## 数据结构

- Zookeeper 数据模型的结构与Unix文件系统类似，整体上可以看作是一颗树，每个节点称做一个ZNode。每一个ZNode默认能够存储 1MB 的数据，每个 ZNode 都可以通过 其路径唯一标识

## 应用场景

- 提供的服务包括：统一命名服务、统一配置管理、统一集群管理、服务器节点上下线、软负载均衡等
  - 统一命名服务
    - 在分布式环境下，经常需要对应用/服务进行统一命名，便于识别。例如：IP不容易记住，而域名容易记住
  - 统一配置管理
    1. 分布式环境下，配置文件同步非常常见
       1. 一般要求一个集群中，所有节点的配置信息是一致的，比如Kafka集群
       2. 对配置文件修改后，希望能够快速同步到各个节点上
    2. 配置管理可交由Zookeeper实现
       1. 可将配置信息写入Zookeeper上的一个Znode
       2. 各个客户端服务器监听这个Znode
       3. 一旦Znode中的数据被修改，Zookeeper 将通知各个客户端服务器
    3. 统一集群管理
       1. 分布式环境中，实时掌握每个节点的状态是必要的
          1. 可根据节点实时状态做出一些调整
       2. Zookeeper可以实现实时监控节点状态变化
          1. 可将节点信息写入Zookeeper上的一个ZNode
          2. 监听这个ZNode可获取它的实时状态变化
    4. 服务器动态上下限
       - 客户端能实时洞察到服务器上下线的变化
         1. 服务端启动时去注册信息（创建都是临时节点）
         2. 客户端获取到当前在线服务器列表，并且注册监听
         3. 服务器节点下线
         4. 客户端获取到节点上下线事件通知
    5. 软负载均衡
       - 在Zookeeper中记录每台服务器的访问数，让访问数最少的服务器去处理最新的客户端请求

# Zookeeper安装

### 本地模式

1. 安装JDK
2. 官网下载jar包
3. 拷贝 apache-zookeeper-xxx-bin.tar.gz 安装包到指定目录
4. 解压并修改名称
5. 配置修改（单机模式）
   1. 将 ./zookeeper/conf 这个路径下的 zoo_sample.cfg 修改为 zoo.cfg
   2. 打开 zoo.cfg 文件，修改 dataDir 路径
   3. 在 /opt/module/zookeeper/这个目录上创建 zkData 文件夹
6. 启动Zookeeper

~~~shell
bin/zkServer.sh start
~~~

7. 查看进程是否启动 进程名(QuorumPeerMain)

~~~shell
jps
~~~

8. 查看状态

~~~shell
bin/zkServer.sh status
~~~

9. 启动客户端

~~~shell
bin/zkCli.sh
~~~

10. 推出客户端

~~~
quit
~~~

11. 停止 Zookeeper

~~~shell
bin/zkServer.sh stop
~~~

### 配置参数解读

- Zookeeper中的配置文件 zoo.cfg 中参数含义解读
  - tickTime = 2000：通信心跳时间，Zookeeper是服务器与客户端心跳时间，单位毫秒
  - initLimit = 10：LF初始通信时限，Leader 和 Follower 初始连接  时能容忍的最多心跳数 （tickTime 数量）
  - syncLimit = 5：LF同步通信时限。Leader 和 Follower之间通信时间如果超过 syncLimit * tickTime，Leader 认为 Follower死掉，从服务器列表中删除 Follower
  - dataDir：保存Zookeeper中的数据
    - 注意：默认的tmp目录，容易被Linux系统定期删除，所以一般不用默认的tmp目录
  - clientPort = 2181：客户端连接端口，通常不做修改

### Zookeeper 集群配置和操作

1. 首先做出对应的集群规划

2. 在某台集群对应的 dataDir 目录下 创建一个 myid 文件，在文中添加与 server 对应的编号

3. 拷贝配置好的 zookeeper 到其他机器上，并分别修改对应集群的 myid

4. 配置 zoo.cfg 文件

   1. 重命名 /opt/module/zookeeper-3.5.7/conf 这个目录下的 zoo_sample.cfg 为 zoo.cfg
   2. 打开 zoo.cfg 文件
   3. 配置参数解读

   ~~~shell
   server.A=B:C:D
   ~~~

   - A是一个数字，表示这个是第几号服务器
     - 集群模式下配置一个文件 myid，这个文件在 dataDir 目录下，这个文件里面有一个数据就是 A 的值，Zookeeper 启动时读取此文件，拿到里面的数据与 zoo.cfg  里面的配置信息比较从而判断到底是哪个 server
   - B是这个服务器的地址
   - C是这个服务器Follower与集群中的Leader服务器交换信息的端口
   - D是Leader服务器挂了，需要一个端口来重新进行选举，选出一个新的 Leader，而这个端口就是用来执行选举时 服务器相互通信的端口

5. 同步 zoo.cfg 文件 到各个集群

6. 启动集群

# 选举机制

### 第一次启动（5台机子为参考例子）

- SID：服务器ID，用来标识一台Zookeeper集群中的机器，每台机器不能重复，和 myid 一致
- ZXID：事务ID，ZXID 是一个事务ID，用来标识一次服务器状态的变更，在某一时刻，集群中的每台机器的 ZXID 不一定完全一致，这和 Zookeeper 服务器对于客户端 "更新请求" 的处理逻辑有关
- Epoch：每个 Leader 任期的代号。没有 Leader时 同一轮 投票过程中的逻辑时钟值是相同的。每投完一次票 这个数据就会增加

1. 服务器 1 启动，发起一次选举。服务器 1投自己一票。此时服务器1票数一票，不够半数以上（3票），选举无法完成，服务器 1状态保持为 LOOKING
2. 服务器2启动，再发起一次选举，服务器 1和2分别投自己一票并交换选票信息：此时服务器1发现服务器2的 myid 比自己目前投票推举的（服务器1）大，更改选票为推举服务器2。此时服务器1为0票，服务器 2 为2票，没有半数以上结果，选举无法完成，服务器 1，2状态保持
3. 服务器 3 启动，发起一次选举。此时服务器1和2都会更改选票为服务器3.此次投票结果：服务器1为0票，服务器2为0票，服务器3为3票。此时服务器3的票数已经超过半数，服务器3当选Leader。服务器1，2更改状态为FOLLOWING，服务器3更改状态为LEADING
4. 服务器4启动，发起一次选举。因为此时集群已有Leader，服务器4投票给服务器3，并更改状态为 FOLLOWING
5. 服务器5启动，同4一样

- 总结
- 每参与一次投票，都会将手中的票数给当前SID最大的机器，直到当前上线的机器超过集群总数的一半，后面上线的机器上线依旧会发起选举，但因为有Leader，自动把票给 Leader

### 非第一次启动

1. Zookeeper集群中的一台服务器出现以下两种情况之一时，就会开始进入 Leader 选举：
   - 服务器初始自动化
   - 服务器运行期间无法和Leader保持连接
2. 而当一台机器进入Leader选举流程时，当前集群可能会处于以下两种状态
   - 集群中本来就已经存在一个Leader
     - 对于第一种已经存在Leader的情况，机器试图去选举Leader时，会被告知当前服务器的Leader信息，对于该机器来说，仅仅需要和Leader机器建立连接，并进行状态同步即可
   - 集群中确实不存在Leader
     - 选举Leader 规则：
       1. EPOCH大的直接胜出
       2. EPOCH相同，事务id大的胜出
       3. 事务id相同，服务器id大的胜出

# 客户端命令行操作

## 启动客户端

~~~
bin/zkCil.sh -server hadoop102:2181
~~~

## 命令行语法

| 命令行基本语法 | 功能描述                                                     |
| -------------- | ------------------------------------------------------------ |
| help           | 显示所有操作命令                                             |
| Is path        | 使用 ls 命令来查看当前 znode 的子节点[可监听]<br />-w 监听子节点变化<br />-s 附加次级信息 |
| create         | 普通创建<br />-s 含有序列<br />-e 临时(重启或者超时消失)     |
| get path       | 获得节点的值[可监听]<br />-w 监听节点内容变化<br />-s 附加次级信息 |
| set            | 设置节点的具体值                                             |
| stat           | 查看节点状态                                                 |
| delete         | 删除节点                                                     |
| deleteall      | 递归删除节点                                                 |

## 次级信息字段含义

1. czxid：创建节点的事务 zxid
   - 每次修改 ZooKeeper 状态都会产生一个 ZooKeeper 事务ID。事务 ID 是ZooKeeper 中所有修改总的次序。每次修改都有唯一的 zxid，如果 zxid1 小于 zxid2，那么 zxid1 在 zxid2 之前发生
2. ctime：znode 被创建的毫秒数 （从1970年开始）
3. mzxid：znode 最后更新的事务 zxid
4. mtime：znode 最后修改的毫秒数（从1970年开始）
5. pZxid：znode最后更新的子节点 zxid
6. cversion：znode 子节点变化号，znode 子节点修改次数
7. dataversion：znode 数据变化号
8. aclVersion：znode 访问控制列表的变化号
9. ephemeralOwner：如果是临时节点，这个是 znode拥有着的 session id。如果不是临时节点 则是0
10. dataLength：znode的数据长度
11. numChildren：znode 子节点数量

# 节点类型

- 持久（Persistent）：客户端和服务器端断开连接后，创建的节点不删除

  - 持久化目录节点
    - 客户端与Zookeeper断开连接后，该节点依旧存在
  - 持久化顺序编号目录节点
    - 客户端与 Zookeeper 断开连接后，该节点依旧存在，只是 Zookeeper 给该节点名称进行顺序编号
- 短暂（Ephemeral）：客户端和服务器端断开连接后，创建的节点自己删除

  - 临时目录节点
    - 客户端与Zookeeper断开连接后，该节点被删除
    - 客户端与Zookeeper断开连接后，该节点被删除，只是Zookeeper给该节点名称进行顺序编号

# 监听器

- 客户端注册监听它关心的目录节点，当目录节点发生变化（数据改变、节点删除、子目录节点增加删除）时，Zookeeper会通知客户端。监听机制保证Zookeeper保存的任何的数据的任何改变都能快速的响应到监听了该节点的应用程序。

## 原理详解

1. 首先有一个 main()线程
2. 在 main线程中创建了Zookeeper客户端，这时就会创建两个线程，一个负责网络连接通信（connect），一个负责监听（listener）。
3. 通过 connect 线程将注册的监听事件发送给 Zookeeper
4. 在 Zookeeper 的注册监听器列表中将注册的监听事件添加到列表中
5. Zookeeper监听到有数据或路径变化，就会将这个消息发送给 listener 线程
6. listener 线程内部调用了 process() 方法

## 常见的监听

1. 监听节点数据的变化
   - get path[watch]
2. 监听子节点增减的变化
   - ls path [watch]

3. 注意：注册一次，只能监听一次。想再次监听，需要再次注册

# 客户端API操作

## 环境搭建

1. 添加 pom 文件

~~~xml
<dependencies>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>RELEASE</version>
        </dependency>
        
        <dependency>
            <groupId>org.apache.logging.log4j</groupId>
            <artifactId>log4j-core</artifactId>
            <version>2.8.2</version>
        </dependency>

        <dependency>
            <groupId>org.apache.zookeeper</groupId>
            <artifactId>zookeeper</artifactId>
            <version>3.5.7</version>
        </dependency>
        
 </dependencies>
~~~

2. 在 src/amin/resources 目录下，新建一个文件，命名为 "log4j.properties"

~~~
log4j.rootLogger=INFO, stdout
log4j.appender.stdout=org.apache.log4j.ConsoleAppender
log4j.appender.stdout.layout=org.apache.log4j.PatternLayout
log4j.appender.stdout.layout.ConversionPattern=%d %p [%c] - %m%n
log4j.appender.logfile=org.apache.log4j.FileAppender
log4j.appender.logfile.File=target/spring.log
log4j.appender.logfile.layout=org.apache.log4j.PatternLayout
log4j.appender.logfile.layout.ConversionPattern=%d %p [%c] - %m%n
~~~

3. 创建包名和类名，并编写代码

## 创建 ZooKeeper 客户端并注册监听事件

~~~java
private String connectString = "hadoop102:2181,hadoop103:2181,hadoop104:2181"; //服务器地址
private int sessionTest = 2000; //会话关闭时间
private ZooKeeper zkClient; 

@Before
public void init() throws IOException, InterruptedException, KeeperException {
        //获取客户端对象,需要ip地址端口号，会话连接时间，和监听方法
         zkClient = new ZooKeeper(connectString, sessionTest, new Watcher() {
             //收到事件通知后的回调函数 
            @Override
            public void process(WatchedEvent watchedEvent) {

                System.out.println("==========================");
                List<String> children = null;
                try {
                     //获取节点，并决定是否要再次启动监听
                    children = zkClient.getChildren("/", false);
                } catch (KeeperException e) {
                    e.printStackTrace();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
   
                for (String child : children) {
                    System.out.println(Thread.currentThread().getName()+":"+child);
                }
            }
        });
    }

	@Test
    public void getChildren() throws InterruptedException, KeeperException {
        //获取节点
        List<String> children = zkClient.getChildren("/", false);

        for (String child : children) {
            System.out.println( Thread.currentThread().getName() + ":"+child) ;
        }

        //防止主线程结束进程
        Thread.sleep(Long.MAX_VALUE);
    }
~~~

## 创建子节点

~~~java
 @Test
   public void create() throws InterruptedException, KeeperException {
       //参数1：要创建的节点的路径
       //参数2：节点数据
       //参数3：节点权限
       //参数3：节点的类型
        String nodeCreated = zkClient.create("/atguigu", "ss.avi".getBytes(), ZooDefs.Ids.OPEN_ACL_UNSAFE, CreateMode.PERSISTENT);
   }
~~~

## 获取子节点并监听节点变化

~~~java
@Test
    public void getChildren() throws InterruptedException, KeeperException {
        //获取节点
        List<String> children = zkClient.getChildren("/", false);

        for (String child : children) {
            System.out.println( Thread.currentThread().getName() + ":"+child) ;
        }

        //防止主线程结束进程
        Thread.sleep(Long.MAX_VALUE);
    }
~~~

## 判断 Znode 是否存在

~~~java
@Test
public void exist() throws InterruptedException, KeeperException {

    Stat stat = zkClient.exists("/atguigu3", false);

    System.out.println(stat ==null?"not exist" :"exist");
}
~~~

# 客户端向服务端写数据流程

## 写流程之写入请求直接发送给 Leader 节点

1. Client write ZK Server Leader
2. Leader 通知 Follower 写数据，等待 Follower 回应
3. 超过半数的 Follower 回应，并回应给 Client

## 写流程之写入请求发送给follower节点

1. Client 向 follower 发送写请求
2. follower 向 leader 发送写请求
3. leader 告诉 follower 可以写
4. follower 向 leader 回应 写完
5. leader 通知 其他 folloer 写，直到超过半数ack 回应
6. 回应第一个 follower，并让它回应给 client，告诉他写入成功
7. 回应剩余没有写入的 follower 节点，让他们写入

# 服务器动态上下限监听案例

## 需求

1. 某分布式系统中，主节点有多台，可以动态上下线，任意一台客户端都能实时感知到主节点服务器的上下线

## 需求分析

1. 服务器端 向 Zookeeper 集群注册信息，创建临时节点
2. 客户端获取当前在线服务器列表，并且注册监听
3. 服务器节点下线
4. 客户端 收到 服务器节点上下线 事件通知

## 具体实现

1. 服务端向 Zookeeper 注册代码

~~~shell
public class DistrubuteServer {

    private String connectString = "hadoop102:2181,hadoop103:2181,hadoop104:2181";
    private int sessionTimeout = 2000;
    private ZooKeeper zk;

    public static void main(String[] args) throws InterruptedException, KeeperException, IOException {

        DistrubuteServer server = new DistrubuteServer();
        //1 获取zk连接
        server.getConnect();

        //2 注册服务器到zk集群
        server.regist(args[0]);

        //3 启动业务逻辑（睡觉）

        server.business();
    }

   //具体业务场景
    private void business() throws InterruptedException {

        Thread.sleep(Long.MAX_VALUE);
    }

   //2.服务器上线，并告知 客户端 创建 /servers/+ 服务器名字，临时带序号节点
    private void regist(String hostname) throws InterruptedException, KeeperException {

        String create = zk.create("/servers/" + hostname, hostname.getBytes(), ZooDefs.Ids.OPEN_ACL_UNSAFE,
                CreateMode.EPHEMERAL_SEQUENTIAL);

        System.out.println(hostname + " is online");

    }

   //获得 zookeeper 客户端
    private void getConnect() throws IOException {
        zk = new ZooKeeper(connectString, sessionTimeout, new Watcher() {
            @Override
            public void process(WatchedEvent watchedEvent) {

            }
        });

    }

}
~~~

2. 客户端代码

~~~java
public class DistributeClient {

    private ZooKeeper zk;
    private String connectString = "hadoop102:2181,hadoop:103,2181,hadoop104:2181";
    private int sessionTimeout = 2000;

    public static void main(String[] args) throws InterruptedException, KeeperException, IOException {
        DistributeClient client = new DistributeClient();

        //1 获取zk连接
        client.getConnect();

        System.out.println("监听开始");
        //2 监听/serves下面子节点的增加和删除
        client.getServerList();

        // 3 业务逻辑（睡觉）
        client.business();
    }

    private void business() throws InterruptedException {
        Thread.sleep(Long.MAX_VALUE);
    }

   //监听/servers 底下的子节点变化，同时注册监听，有变化就获取每个节点的主机名称信息
    private void getServerList() throws InterruptedException, KeeperException {
        List<String> children = zk.getChildren("/servers", true);

        ArrayList<String> servers = new ArrayList<>();
        for (String child : children) {

            byte[] data = zk.getData("/servers/" + child, false, null);

            servers.add(new String(data));

        }

        //打印服务器列表信息
        System.out.println(servers);
    }


   //获取zk连接，同时注册监听，
    private void getConnect() throws IOException {

        zk = new ZooKeeper(connectString, sessionTimeout, new Watcher() {
            @Override
            public void process(WatchedEvent watchedEvent) {

                try {
                    getServerList();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } catch (KeeperException e) {
                    e.printStackTrace();
                }
            }
        });

    }
}

~~~

# ZooKeeper分布式锁案例

## 定义

- 分布式锁：比如说“进程1”在使用该资源的时候，会先去获得锁，“进程1”获得锁以后会对该资源保持独占，这样其他进程就无法访问该资源，“进程1”用完该资源以后就将锁释放掉，让其他进程来获得锁，那么通过这个锁机制，我们就能保证分布式系统中多个进程能够有序的访问该临界资源

##  原生Zookeeper 实现分布式锁案例

- 思路
  1. 客户端访问 /locks,服务段接受到请求后，在 /locks节点下创建一个临时顺序节点
  2. 判断自己是不是当前节点下最小的节点：是，获取到锁，不是，对前一个节点进行监听
  3. 获取到锁，处理完业务后，delete节点释放锁，然后下面的节点将收到通知，重复第二步判断
- 分布式锁实现

~~~java
public class DistributedLock {

    private String connectString = "hadoop102:2181,hadoop103:2181,hadoop104:2181";
    private int sessionTimeout = 2000;
    private ZooKeeper zk;



    private CountDownLatch connectLatch = new CountDownLatch(1);
    private CountDownLatch waitLatch = new CountDownLatch(1);

    private String waitPath;
    private String currentMode;

    public DistributedLock() throws IOException, InterruptedException, KeeperException {
        //获取连接
        zk = new ZooKeeper(connectString, sessionTimeout, new Watcher() {
            @Override
            public void process(WatchedEvent watchedEvent) {

                //connectLatch 如果连接上zk 可以释放
                if (watchedEvent.getState() == Event.KeeperState.SyncConnected) {
                    connectLatch.countDown();
                }

                //waitLatch 需要释放
                if (watchedEvent.getType() == Event.EventType.NodeDeleted && watchedEvent.getPath().equals(waitPath)) {
                    waitLatch.countDown();
                }

            }
        });

        //等待zk正常连接后，往下走程序
        connectLatch.await();

        //判断根节点/locks是否存在
        Stat stat = zk.exists("/locks", false);

        if (stat == null) {
            zk.create("/locks", "locks".getBytes(), ZooDefs.Ids.OPEN_ACL_UNSAFE, CreateMode.PERSISTENT);
        }


    }

    //对zk加锁
    public void zklock() {
        //创建对应的临时带序号节点
        try {
            currentMode = zk.create("/locks/" + "seq-", null, ZooDefs.Ids.OPEN_ACL_UNSAFE,
                    CreateMode.EPHEMERAL_SEQUENTIAL);

            Thread.sleep(10);

            //判断创建的节点是否是最小的序号节点，如果是获取到锁；如果不是，监听到序号前一个节点

            List<String> children = zk.getChildren("/locks", false);

            //如果children只有一个值，那就直接获取锁；如果有多个节点，需要判断，谁最小
            if (children.size() == 1) {
                return;
            } else {
                Collections.sort(children);

                //获取节点名称
                String thisNode = currentMode.substring("/locks/".length());
                //通过节点获取该节点在children集合的位置
                int index = children.indexOf(thisNode);

                //判断
                if (index == -1) {
                    System.out.println("数据异常");
                } else if (index == 0) {
                    return;
                } else {
                    //需要监听 他前一个节点变化
                    waitPath = "/locks/" + children.get(index - 1);
                    zk.getData(waitPath, true, null);

                    //等待监听
                    waitLatch.await();

                    return;
                }

            }

        } catch (KeeperException e) {
            e.printStackTrace();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

    }

    //解锁
    public void unZkLokc() {
        try {
            zk.delete(currentMode, -1);
        } catch (InterruptedException e) {
            e.printStackTrace();
        } catch (KeeperException e) {
            e.printStackTrace();
        }
    }
}
~~~

- 分布式锁测试

~~~java
public class DistributedLockTest {

    public static void main(String[] args) throws IOException, InterruptedException, KeeperException {

        final DistributedLock lock1 = new DistributedLock();

        final DistributedLock lock2 = new DistributedLock();

        new Thread(new Runnable() {
            @Override
            public void run() {

                try {
                    lock1.zklock();
                    System.out.println("线程1 启动.获取到锁");
                    Thread.sleep(5*1000);

                    lock1.unZkLokc();
                    System.out.println("线程1 释放锁");
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }).start();


        new Thread(new Runnable() {
            @Override
            public void run() {

                try {
                    lock2.zklock();
                    System.out.println("线程2 启动.获取到锁");
                    Thread.sleep(5*1000);

                    lock2.unZkLokc();
                    System.out.println("线程2 释放锁");
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }).start();
    }
}
~~~

## CountDownLatch

- 是一个同步工具类，它允许一个或多个线程等待，直到其他线程运行完成后再执行[1](https://bing.com/search?q=CountDownLatch+如何使用)[。它主要有两个方法：`countDown()` 和 `await()`。`countDown()` 方法用于使计数器减一，其一般是执行任务的线程调用，`await()` 方法则使调用该方法的线程处于等待状态，其一般是主线程调用

## Curator 框架

- 原生的 Java API 开发存在的问题

  1. 会话连接是异步的，需要自己去处理。比如使用 CountDownLatch
  2. Watch需要重复注册，不然就不能生效
  3. 开发的复杂性还是比较高的
  4. 不支持多节点删除和创建。需要自己去递归

- Curator 是一个专门解决分布式锁的框架，解决了原生 Java API 开发分布式遇到的问题

- 实操案例

  - 添加依赖

  ~~~xml
  <dependencies>
         
          <dependency>
              <groupId>org.apache.curator</groupId>
              <artifactId>curator-recipes</artifactId>
              <version>4.3.0</version>
          </dependency>
  
          <dependency>
              <groupId>org.apache.curator</groupId>
              <artifactId>curator-client</artifactId>
              <version>4.3.0</version>
          </dependency>
  
      </dependencies>
  ~~~

  - 代码实现

  ~~~java
  public class CuratorLockTest {
  
      public static void main(String[] args){
  
          //创建分布式锁1
          InterProcessMutex locks1 = new InterProcessMutex(getCuratorFramework(), "/locks");
  
          //创建分布式锁2
          InterProcessMutex locks2 = new InterProcessMutex(getCuratorFramework(), "/locks");
  
          new Thread(new Runnable() {
              @Override
              public void run() {
                  try {
                      locks1.acquire();
                      System.out.println("线程1 获取到锁");
                      Thread.sleep(100*1000);
                      locks1.acquire();
                      System.out.println("线程1 再次获取到锁");
  
                      Thread.sleep(100*1000);
                      locks1.release();
                      System.out.println("线程1 释放锁");
  
                      locks1.release();
                      System.out.println("线程1 再次释放锁");
                  } catch (Exception e) {
                      e.printStackTrace();
                  }
              }
          }).start();
  
  
          new Thread(new Runnable() {
              @Override
              public void run() {
                  try {
                      locks2.acquire();
                      System.out.println("线程2 获取到锁");
  
                      locks2.acquire();
                      System.out.println("线程2 再次获取到锁");
  
                      Thread.sleep(5*1000);
                      locks2.release();
                      System.out.println("线程2 释放锁");
  
                      locks2.release();
                      System.out.println("线程2 再次释放锁");
                  } catch (Exception e) {
                      e.printStackTrace();
                  }
              }
          }).start();
      }
  
      private static CuratorFramework getCuratorFramework() {
          //重试策略，初试时间 3 秒，重试 3 次
          ExponentialBackoffRetry policy = new ExponentialBackoffRetry(3000, 3);
  		
  	   // 通过工厂创建 Curator
          CuratorFramework client = CuratorFrameworkFactory
                  .builder()
                  .connectString("hadoop102:2181,hadoop103:2181,hadoop104:2181")
                  .connectionTimeoutMs(2000)
                  .sessionTimeoutMs(2000)
                  .retryPolicy(policy)
                  .build();
  
          //启动客户端
          client.start();
  
          System.out.println("Zookeeper 启动成功");
          return client;
      }
  }
  ~~~

# 其他

## 生产集群安装多少zk合适

- 安装奇数台

- 生产经验

  - 10台服务器：3台 zk;
  - 20台服务器：5台 zk;
  - 100台服务器：11台 zk;
  - 200台服务器：11 台zk;
  - 服务器台数多：好处，提高可靠性；坏处：提高通信延时

  



