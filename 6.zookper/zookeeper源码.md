# Zookeeper源码解析--第一部分

## 启动shell脚本

1. zookeeper-3.5.7/bin/zkServer.sh 是启动脚本

~~~shell
zkServer.sh
## 获取当前脚本名称 大部分场景下和$0效果相同 zkServer.sh
ZOOBIN="${BASH_SOURCE-$0}"
## 获取当前脚本的非目录部分 .
ZOOBIN="$(dirname "${ZOOBIN}")"
## 获取当前脚本的绝对路径 ./zookeeper-3.5.7/bin
ZOOBINDIR="$(cd "${ZOOBIN}"; pwd)"


if [ -e "$ZOOBIN/../libexec/zkEnv.sh" ]; then
  . "$ZOOBINDIR"/../libexec/zkEnv.sh
else
  ## 执行zkEnv.sh 脚本命令
  . "$ZOOBINDIR"/zkEnv.sh
fi

## 这个是start开始启动
case $1 in
start)
    echo  -n "Starting zookeeper ... "
    --zookeeper的pid文本，存在就说明启动
    if [ -f "$ZOOPIDFILE" ]; then
      ## kill -0 该程序存在，则直接退出 
      if kill -0 `cat "$ZOOPIDFILE"` > /dev/null 2>&1; then
         echo $command already running as process `cat "$ZOOPIDFILE"`.
         exit 1
      fi
    fi
    ## 入口是QuorumPeerMain $ZOOCFG 入参的参数 配置文件 zoo.cfg 在zkEnv.sh里面获取
    nohup "$JAVA" $ZOO_DATADIR_AUTOCREATE "-Dzookeeper.log.dir=${ZOO_LOG_DIR}" \
    "-Dzookeeper.log.file=${ZOO_LOG_FILE}" "-Dzookeeper.root.logger=${ZOO_LOG4J_PROP}" \
    -XX:+HeapDumpOnOutOfMemoryError -XX:OnOutOfMemoryError='kill -9 %p' \
    -cp "$CLASSPATH" $JVMFLAGS $ZOOMAIN "$ZOOCFG" > "$_ZOO_DAEMON_OUT" 2>&1 < /dev/null &
    
    
ZOOMAIN="-Dcom.sun.management.jmxremote -Dcom.sun.management.jmxremote.local.only=$JMXLOCALONLY org.apache.zookeeper.server.quorum.QuorumPeerMain"

zkEnv.sh:
##不变
ZOOBINDIR="${ZOOBINDIR:-/usr/bin}" 
##./zookeeper-3.5.7
ZOOKEEPER_PREFIX="${ZOOBINDIR}/.."

##获取$ZOOCFGDIR变量，路径是./zookeeper-3.5.7/conf
if [ "x$ZOOCFGDIR" = "x" ]
then
  if [ -e "${ZOOKEEPER_PREFIX}/conf" ]; then
    ZOOCFGDIR="$ZOOBINDIR/../conf"
  else
    ZOOCFGDIR="$ZOOBINDIR/../etc/zookeeper"
  fi
fi

## 获取ZOOCFG值，在conf底下的zoo_sample.cfg
if [ "x$ZOOCFG" = "x" ]
then
    ZOOCFG="zoo.cfg"
fi

ZOOCFG="$ZOOCFGDIR/$ZOOCFG"

~~~

2. 查看源代码 rg.apache.zookeeper.server.quorum.QuorumPeerMain ，参数是zoo.cfg

~~~java
public class QuorumPeerMain {
  public static void main(String[] args) {
      QuorumPeerMain main = new QuorumPeerMain();
        try {
            //这个方法初始化，进去这个方法
            main.initializeAndRun(args);
        }
  }
  
  protected void initializeAndRun(String[] args)
        throws ConfigException, IOException, AdminServerException
    {
        QuorumPeerConfig config = new QuorumPeerConfig();
        // 把配置文件配置进去
        if (args.length == 1) {
            config.parse(args[0]);
        }

        // Start and schedule the the purge task
        DatadirCleanupManager purgeMgr = new DatadirCleanupManager(config
                .getDataDir(), config.getDataLogDir(), config
                .getSnapRetainCount(), config.getPurgeInterval());
        purgeMgr.start();

        if (args.length == 1 && config.isDistributed()) {
            runFromConfig(config);
        } else {
            LOG.warn("Either no config or no quorum defined in config, running "
                    + " in standalone mode");
            // there is only server in the quorum -- run as standalone
            ZooKeeperServerMain.main(args);
        }
    }
    
    
    
}

//配置QuorumMaj这个类的内容
public class QuorumPeerConfig {
    public void parse(String path) throws ConfigException {
        LOG.info("Reading configuration from: " + path);
       
        try {
            File configFile = (new VerifyingFileFactory.Builder(LOG)
                .warnForRelativePath()
                .failForNonExistingPath()
                .build()).create(path);
                
            Properties cfg = new Properties();
            FileInputStream in = new FileInputStream(configFile);
            try {
                cfg.load(in);
                configFileStr = path;
            } finally {
                in.close();
            }
            //配置传进来的cfg
            parseProperties(cfg);
        } 
    }
    
    public void parseProperties(Properties zkProp){
        ...;
         if (dynamicConfigFileStr == null) {
             //初始化QuorumMaj
            setupQuorumPeerConfig(zkProp, true);
            if (isDistributed() && isReconfigEnabled()) {
                // we don't backup static config for standalone mode.
                // we also don't backup if reconfig feature is disabled.
                backupOldConfig();
            }
        }
    }
    
    void setupQuorumPeerConfig(Properties prop, boolean configBackwardCompatibilityMode)
            throws IOException, ConfigException {
        //这个方法解析zoo.cfg的server.2=Linux:2888:3888
        quorumVerifier = parseDynamicConfig(prop, electionAlg, true, configBackwardCompatibilityMode);
        //当前主机的myid
        setupMyId();
        setupClientPort();
        setupPeerType();
        checkValidity();
    }
    
    //这个接口是当前所有服务器的信息
    public interface QuorumVerifier {
    long getWeight(long id);
    boolean containsQuorum(Set<Long> set);
    long getVersion();
    void setVersion(long ver);
    Map<Long, QuorumServer> getAllMembers();
    Map<Long, QuorumServer> getVotingMembers();
    Map<Long, QuorumServer> getObservingMembers();
    boolean equals(Object o);
    String toString();
}
    
    //这个是当前存储的每台服务器的信息
     public static class QuorumServer {
        public InetSocketAddress addr = null;
		//选举地址
        public InetSocketAddress electionAddr = null;
        //Follower和leader的通信地址 
        public InetSocketAddress clientAddr = null;
        
        public long id;

        public String hostname;
        
        public LearnerType type = LearnerType.PARTICIPANT;
        
        private List<InetSocketAddress> myAddrs;
     }
    
    
    public static QuorumVerifier parseDynamicConfig(Properties dynamicConfigProp, int eAlg, boolean warnings,
	   boolean configBackwardCompatibilityMode) throws IOException, ConfigException {
       boolean isHierarchical = false;
        for (Entry<Object, Object> entry : dynamicConfigProp.entrySet()) {
            String key = entry.getKey().toString().trim();                    
            if (key.startsWith("group") || key.startsWith("weight")) {
               isHierarchical = true;
            } else if (!configBackwardCompatibilityMode && !key.startsWith("server.") && !key.equals("version")){ 
               LOG.info(dynamicConfigProp.toString());
               throw new ConfigException("Unrecognised parameter: " + key);                
            }
        }
        //初始化当前所有服务器的信息，并生产动态配置文件，默认所有的服务器都是参与者，可以参与
        QuorumVerifier qv = createQuorumVerifier(dynamicConfigProp, isHierarchical);
               
        int numParticipators = qv.getVotingMembers().size();
        int numObservers = qv.getObservingMembers().size();
        if (numParticipators == 0) {
            if (!standaloneEnabled) {
                throw new IllegalArgumentException("standaloneEnabled = false then " +
                        "number of participants should be >0");
            }
            if (numObservers > 0) {
                throw new IllegalArgumentException("Observers w/o participants is an invalid configuration");
            }
        } else if (numParticipators == 1 && standaloneEnabled) {
            // HBase currently adds a single server line to the config, for
            // b/w compatibility reasons we need to keep this here. If standaloneEnabled
            // is true, the QuorumPeerMain script will create a standalone server instead
            // of a quorum configuration
            LOG.error("Invalid configuration, only one server specified (ignoring)");
            if (numObservers > 0) {
                throw new IllegalArgumentException("Observers w/o quorum is an invalid configuration");
            }
        } else {
            if (warnings) {
                if (numParticipators <= 2) {
                    LOG.warn("No server failure will be tolerated. " +
                        "You need at least 3 servers.");
                } else if (numParticipators % 2 == 0) {
                    LOG.warn("Non-optimial configuration, consider an odd number of servers.");
                }
            }
            /*
             * If using FLE, then every server requires a separate election
             * port.
             */            
           if (eAlg != 0) {
               for (QuorumServer s : qv.getVotingMembers().values()) {
                   if (s.electionAddr == null)
                       throw new IllegalArgumentException(
                               "Missing election port for server: " + s.id);
               }
           }   
        }
        return qv;
    }
}

// 初始化当前议员会议
public QuorumMaj(Properties props) throws ConfigException {
        for (Entry<Object, Object> entry : props.entrySet()) {
            String key = entry.getKey().toString();
            String value = entry.getValue().toString();

            if (key.startsWith("server.")) {
                int dot = key.indexOf('.');
                long sid = Long.parseLong(key.substring(dot + 1));
                //获取配置文件的所有服务器的sid和对应的value，并初始化成每一台服务器信息，默认是  LearnerType.PARTICIPANT
                QuorumServer qs = new QuorumServer(sid, value);
                allMembers.put(Long.valueOf(sid), qs);
                if (qs.type == LearnerType.PARTICIPANT)
                    votingMembers.put(Long.valueOf(sid), qs);
                else {
                    observingMembers.put(Long.valueOf(sid), qs);
                }
            } else if (key.equals("version")) {
                version = Long.parseLong(value, 16);
            }
        }
        //初始化投票人数，只有超过半数才会开始投票
        half = votingMembers.size() / 2;
    }
}
~~~

