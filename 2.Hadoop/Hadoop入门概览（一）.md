# Hadoop入门概览（一）

##一.什么是Hadoop

- Hadoop是一门框架，通过使用部署在集群上的简单的程序模型去处理海量的分布式数据，包括存储和计算。

---

##二.Hadoop的组成

- Hadoop 由 HDFS ，YARN ，MapReduce三者组成

  - HDFS全称是Hadoop Distributed File System，是一个分布式文件系统
  - YARN全称是Yet Another Resource Negotiator，是Hadoop的资源管理器
  - MapReduce是计算框架，用来计算存储在Hadoop上的海量数据,他将计算框架分成Map和Reduce两个部分。

---

## 三.Hadoop的配置文件

- Hadoop的配置文件分为默认配置文件和自定义配置文件.用户一般修改自定义配置文件

- 自定义配置文件分为四类

  - **core-site.xml**:核心配置文件

    1. NameNode内部通信端口(8020,9000,9820)

    2. HDFS文件存储路径
  
    3. 配置网页登录用户
  
       ```xml
       <!-- 指定NameNode的地址 -->
           <property>
               <name>fs.defaultFS</name>
               <value>hdfs://hadoop102:8020</value>
           </property>
       
        <!-- 指定hadoop数据的存储目录 -->
           <property>
               <name>hadoop.tmp.dir</name>
               <value>/opt/module/hadoop-3.1.3/data</value>
           </property>
       
       <!-- 配置HDFS网页登录使用的静态用户为XXXXX -->
           <property>
               <name>hadoop.http.staticuser.user</name>
               <value>XXXXXXX</value>
           </property>
       </configuration>
       ```
  
  - **hdfs-site.xml**:HDFS配置文件
  
    - Hadoop web端访问地址
  
    - 文件的副本数
  
    - DN向NN汇报当前解读信息的时间间隔，默认6小时
  
    - DN扫描自己节点块信息列表的时间，默认6小时
  
    - DN和NN互相通信时间参数设置
  
      - dfs.namenode.heartbeat.recheck-interval：NameNode对DateNode回应时间设置
      - dfs.heartbeat.interval：NameNode和DateNode互相通信时间间隔
      
      ```xml
      <!-- nn web端访问地址-->
      	<property>
              <name>dfs.namenode.http-address</name>
              <value>hadoop102:9870</value>
          </property>
      	<!-- 2nn web端访问地址-->
          <property>
              <name>dfs.namenode.secondary.http-address</name>
              <value>hadoop104:9868</value>
          </property>
      
      <!-- 文件的副本数-->
          <property>
                <name>dfs.replication</name>
                <value>3</value>
          </property>
      
      <property>
      	<name>dfs.blockreport.intervalMsec</name>
      	<value>21600000</value>
      	<description>Determines block reporting interval in milliseconds.</description>
      </property>
      
      <property>
      	<name>dfs.datanode.directoryscan.interval</name>
      	<value>21600s</value>
      	<description>Interval in seconds for Datanode to scan data directories and reconcile the difference between blocks in memory and on the disk.
      	Support multiple time unit suffix(case insensitive), as described
      	in dfs.heartbeat.interval.
      	</description>
      </property>
      
      <property>
           <name>dfs.namenode.heartbeat.recheck-interval</name>
           <value>300000</value>
      </property>
      
      <property>
           <name>dfs.heartbeat.interval</name>
           <value>3</value>
      </property>
      
      
      ```
  
  - **yarn-site.xml**:YARN配置文件
  
    1. NodeManager上运行的附属服务
  
    2. ResourceManager地址(8088)
  
    3. 环境变量继承
  
       ```xml
       <configuration>
           <!-- 指定MR走shuffle -->
           <property>
               <name>yarn.nodemanager.aux-services</name>
               <value>mapreduce_shuffle</value>
           </property>
       
           <!-- 指定ResourceManager的地址-->
           <property>
               <name>yarn.resourcemanager.hostname</name>
               <value>hadoop103</value>
           </property>
       
           <!-- 环境变量的继承 -->
           <property>
               <name>yarn.nodemanager.env-whitelist</name>
               <value>JAVA_HOME,HADOOP_COMMON_HOME,HADOOP_HDFS_HOME,HADOOP_CONF_DIR,CLASSPATH_PREPEND_DISTCACHE,HADOOP_YARN_HOME,HADOOP_MAPRED_HOME</value>
           </property>
       </configuration>
       ```
  
       
  
  - **mapred-site.xml**:MapReduce配置文件
  
    1. 运行MapReduce的框架
  
    1. 配置历史服务器地址
  
    1. 配置日志的聚集
  
      1. 日志聚集的概念：应用运行完成以后，将程序运行日志信息上传到HDFS系统上。
  
    	```xml
      <configuration>
      <!-- 指定MapReduce程序运行在Yarn上 -->
        <property>
            <name>mapreduce.framework.name</name>
            <value>yarn</value>
        </property>
      </configuration>
      
      <!-- 历史服务器端地址 -->
      <property>
          <name>mapreduce.jobhistory.address</name>
          <value>hadoop102:10020</value>
      </property>
      
      <!-- 历史服务器web端地址 -->
      <property>
          <name>mapreduce.jobhistory.webapp.address</name>
          <value>hadoop102:19888</value>
      </property>
      
      <!-- 开启日志聚集功能 -->
      <property>
          <name>yarn.log-aggregation-enable</name>
          <value>true</value>
      </property>
      <!-- 设置日志聚集服务器地址 -->
      <property>  
          <name>yarn.log.server.url</name>  
          <value>http://hadoop102:19888/jobhistory/logs</value>
      </property>
      <!-- 设置日志保留时间为7天 -->
      <property>
          <name>yarn.log-aggregation.retain-seconds</name>
          <value>604800</value>
      </property>
      
      ```
  
    
  
  - 这四类文件分别存放在$HADOOP_HOME/etc/hadoop这个路径上
  
  - tips:
  
    1. Hadoop3.0x:NameNode和DateNode内部通信端口一般设置:8020，9000，9820
    2. NameNode网站访问端口号:9870
    3. yarn查看任务运行情况:8088
    4. 历史服务器通信端口: 19888
    
  
  ---
## 四.HDFS入门概览

### HDFS概念

- HDFS是一个文件系统,用于存储文件，通过目录树存储文件结构。
- nameNode上并不会存储文件块的具体位置，而是存储文件的目录树结构，由dateNode自己汇报给nameNode对应的文件块位置。

- NameNode的内存计算：
  - 每个文件快大约占用150字节，而服务器一般128GB内存，因此能存储  128 * 1024 * 1024 * 1024 / 150Byte ≈ 9.1 亿

### HDFS架构

- NameNode：是一个管理者，管理DateNode
  1. 管理HDFS的名称空间
  2. 管理副本策略
  3. 管理数据块映射信息
  4. 处理客户端读写请求
- DateNode：实际存储数据块的节点
   	1. 存储实际的数据块
   	2. 执行数据块的读/写操作

### HDFS的shell操作

 ```shell
 hadoop fs -ls: 显示目录信息
 hadoop fs -cat：显示文件内容
 hadoop fs -mkdir：创建路径
 hadoop fs -chgrp、-chmod、-chown：Linux文件系统中的用法一样，修改文件所属权限
 hadoop fs -mkdir：创建路径
 hadoop fs -cp：从HDFS的一个路径拷贝到HDFS的另一个路径
 hadoop fs -mv：在HDFS目录中移动文件
 hadoop fs -tail：显示一个文件的末尾1kb的数据
 hadoop fs -rm：删除文件或文件夹
 hadoop fs -rm -r：递归删除目录及目录里面内容
 hadoop fs -du ：统计目录下各文件大小
 hadoop fs -du -s：汇总统计目录下文件大小
 hadoop fs -count：统计文件(夹)数量
 ```

### HDFS读写流程

- HDFS写数据流程
   	1. 客户端通过Distributed FileSystem模块向NameNode请求上传文件，NameNode检查目标文件是否已存在，父目录是否存在。
   	2. NameNode返回是否可以上传。
   	3. 客户端请求第一个 Block上传到哪几个DataNode服务器上。
   	4. NameNode返回3个DataNode节点，分别为dn1、dn2、dn3。
   	5. 客户端通过FSDataOutputStream模块请求dn1上传数据，dn1收到请求会继续调用dn2，然后dn2调用dn3，将这个通信管道建立完成。
   	6. dn1、dn2、dn3逐级应答客户端。
   	7. 客户端开始往dn1上传第一个Block（先从磁盘读取数据放到一个本地内存缓存），以Packet为单位，dn1收到一个Packet就会传给dn2，dn2传给dn3；dn1每传一个packet会放入一个应答队列等待应答。
   	8. 当一个Block传输完成之后，客户端再次请求NameNode上传第二个Block的服务器。（重复执行3-7步）。
- HDFS读数据流程
  1. 客户端通过DistributedFileSystem向NameNode请求下载文件，NameNode通过查询元数据，找到文件块所在的DataNode地址。
  2. 挑选一台DataNode（就近原则，然后随机）服务器，请求读取数据。
  3. DataNode开始传输数据给客户端（从磁盘里面读取数据输入流，以Packet为单位来做校验）。
  4. 客户端以Packet为单位接收，先在本地缓存，然后写入目标文件。

### DateNode

####DataNode工作机制

  1. 一个数据块在DataNode上以文件形式存储在磁盘上，包括两个文件，一个是数据本身，一个是元数据包括数据块的长度，块数据的校验和，以及时间戳。
  2. DataNode启动后向NameNode注册，通过后，周期性（6小时）的向NameNode上报所有的块信息。
  3. 心跳是每3秒一次，心跳返回结果带有NameNode给该DataNode的命令如复制块数据到另一台机器，或删除某个数据块。如果超过10分钟没有收到某个DataNode的心跳，则认为该节点不可用。
 4. 集群运行中可以安全加入和退出一些机器。

#### 数据完整性

1. 当DataNode读取Block的时候，它会计算CheckSum。
2. 如果计算后的CheckSum，与Block创建时值不一样，说明Block已经损坏。
3. Client读取其他DataNode上的Block。
4. 常见的校验算法crc（32），md5（128），sha1（160）
5. DataNode在其文件创建后周期验证CheckSum。

#### 掉线参数时间设置

- Timeout = 2 * dfs.namenode.heartbeat.recheck-interval + 10 * dfs.heartbeat.interval

