# `Flink`客户端提交任务

- 前面讲过`flink`作业提交的流程。现在开始深化这部分内容。首先，需要一个`flink`作业和对应的`jar`包。用来提交任务

## Flink作业

~~~java
package org.apache.flink;

import org.apache.flink.api.common.functions.FlatMapFunction;
import org.apache.flink.api.java.tuple.Tuple2;
import org.apache.flink.streaming.api.datastream.DataStreamSink;
import org.apache.flink.streaming.api.datastream.DataStreamSource;
import org.apache.flink.streaming.api.datastream.KeyedStream;
import org.apache.flink.streaming.api.datastream.SingleOutputStreamOperator;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.util.Collector;

public class SocketWordCount {
    public static void main(String[] args) throws Exception{
        /**
         * 创建StreamExecutionEnvironment
         */
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        /** 读取socket数据 */
        DataStreamSource<String> fileStream =   env.socketTextStream("127.0.0.1",9999);
        /** 将数据转成小写 */
        SingleOutputStreamOperator<String> mapStream = fileStream.map(String :: toLowerCase);
        /** 按照空格切分字符串*/
        SingleOutputStreamOperator<Tuple2<String,Integer>> flatMapStream = mapStream.flatMap(new Split());
        /** 分组聚合*/
        KeyedStream<Tuple2<String,Integer>,String> keyStream = flatMapStream.keyBy(value -> value.f0);
        /** 聚合*/
        SingleOutputStreamOperator<Tuple2<String,Integer>> sumStream = keyStream.sum(1);
        /** 打印 */
        DataStreamSink<Tuple2<String,Integer>> sink = sumStream.print();
        /** 执行任务 */
        env.execute("WordCount");
    }

    public static class Split implements FlatMapFunction<String, Tuple2<String,Integer>> {
        @Override
        public void flatMap(String element, Collector<Tuple2<String, Integer>> collector) throws Exception {
            String [] eles = element.split(" ");
            for(String chr : eles){
                collector.collect(new Tuple2<>(chr,1));
            }
        }
    }
}
~~~

- 这个通过maven进行打包。需要对应的打包插件。这些不多说了。

- 然后，直接看 flink 客户端提交命令 flink。会获取对应的提交命令：

  ~~~shell
  flink run -c org.example.SocketWordCount flink-tutorial-1.0-SNAPSHOT.jar
  ~~~

  

  ~~~shell
  java    
    -Dlog.file=/root/flink/flink-dist/target/flink-1.15.0-bin/flink-1.15.0/log/flink-root-client-admin-1.log -Dlog4j.configuration=file
    :/root/flink/flink-dist/target/flink-1.15.0-bin/flink-1.15.0/conf/log4j-cli.properties 
    -Dlog4j.configurationFile=file:/root/flink/flink-dist/target/flink-1.15.0-bin/flink-1.15.0/conf/log4j-cli.properties 
    -Dlogback.configurationFile=file:/root/flink/flink-dist/target/flink-1.15.0-bin/flink-1.15.0/conf/logback.xml 
    -classpath /root/flink/flink-dist/target/flink-1.15.0-bin/flink-1.15.0/lib/flink-cep-1.15.0.jar:
      /root/flink/flink-dist/target/flink-1.15.0-bin/flink-1.15.0/lib/flink-connector-files-1.15.0.jar:/root/flink/flink-dist/target/flink-1.15.0-bin/flink-1.15.0/lib/flink-csv-1.15.0.jar:/root/flink/flink-dist/target/flink-1.15.0-bin/flink-1.15.0/lib/flink-json-1.15.0.jar:/root/flink/flink-dist/target/flink-1.15.0-bin/flink-1.15.0/lib/flink-scala_2.12-1.15.0.jar:/root/flink/flink-dist/target/flink-1.15.0-bin/flink-1.15.0/lib/flink-shaded-zookeeper-3.5.9.jar:/root/flink/flink-dist/target/flink-1.15.0-bin/flink-1.15.0/lib/flink-table-api-java-uber-1.15.0.jar:/root/flink/flink-dist/target/flink-1.15.0-bin/flink-1.15.0/lib/flink-table-planner-loader-1.15.0.jar:/root/flink/flink-dist/target/flink-1.15.0-bin/flink-1.15.0/lib/flink-table-runtime-1.15.0.jar:/root/flink/flink-dist/target/flink-1.15.0-bin/flink-1.15.0/lib/log4j-1.2-api-2.17.1.jar:/root/flink/flink-dist/target/flink-1.15.0-bin/flink-1.15.0/lib/log4j-api-2.17.1.jar:/root/flink/flink-dist/target/flink-1.15.0-bin/flink-1.15.0/lib/log4j-core-2.17.1.jar:/root/flink/flink-dist/target/flink-1.15.0-bin/flink-1.15.0/lib/log4j-slf4j-impl-2.17.1.jar:/root/flink/flink-dist/target/flink-1.15.0-bin/flink-1.15.0/lib/flink-dist-1.15.0.jar::: org.apache.flink.client.cli.CliFrontend run -c org.example.SocketWordCount flink-tutorial-1.0-SNAPSHOT.jar
  ~~~

- `CliFrontend` 是 Flink 客户端的统一入口，主要职责包括：

  - **解析命令行参数**
     识别 `run`、`list`、`cancel`、`savepoint` 等子命令。
  - **构建执行环境**
     例如 `RunOptions`（保存作业主类、JAR 路径、并行度等信息）。
  - **封装用户程序**
     使用 `PackagedProgram` 将用户 JAR 与 `mainClass` 封装成可执行对象。
  - **提交作业**
     将 `JobGraph` 发送给集群（Standalone、YARN、K8s）上的 JobManager。

## CliFrontend解析

- 首先查看`main`方法

  ~~~java
  public static void main(final String[] args) {
          int retCode = INITIAL_RET_CODE;
          try {
              retCode = mainInternal(args);
          } finally {
              System.exit(retCode);
          }
  }
  
  @VisibleForTesting
      static int mainInternal(final String[] args) {
          EnvironmentInformation.logEnvironmentInfo(LOG, "Command Line Client", args);
  
          // 1. find the configuration directory
        	//导入配置文件路径
          final String configurationDirectory = getConfigurationDirectoryFromEnv();
  
          // 2. load the global configuration
          // 导入 配置
          final Configuration configuration =
                  GlobalConfiguration.loadConfiguration(configurationDirectory);
  
          // 3. load the custom command lines
          // 导入对应的命令行：flink有对应的 集群，yarn，k8s等。所以需要导入。等提交的时候判断
          final List<CustomCommandLine> customCommandLines =
                  loadCustomCommandLines(configuration, configurationDirectory);
  
          int retCode = INITIAL_RET_CODE;
          try {
              final CliFrontend cli = new CliFrontend(configuration, customCommandLines);
              CommandLine commandLine =
                  	//获取命令行
                      cli.getCommandLine(
                              new Options(),
                              Arrays.copyOfRange(args, min(args.length, 1), args.length),
                              true);
              Configuration securityConfig = new Configuration(cli.configuration);
              DynamicPropertiesUtil.encodeDynamicProperties(commandLine, securityConfig);
              SecurityUtils.install(new SecurityConfiguration(securityConfig));
              //根据命令行进行提交
              retCode = SecurityUtils.getInstalledContext().runSecured(() -> cli.parseAndRun(args));
          } catch (Throwable t) {
              final Throwable strippedThrowable =
                      ExceptionUtils.stripException(t, UndeclaredThrowableException.class);
              LOG.error("Fatal error while running command line interface.", strippedThrowable);
              strippedThrowable.printStackTrace();
          }
          return retCode;
      }
  ~~~

- 主要看 `loadCustomCommandLines`方法。`cli.getCommandLine`方法和 `cli.parseAndRun(args)` 方法。

- loadCustomCommandLines

- **作用**：动态加载不同部署模式的命令行解析器

  - 返回一个 `List<CustomCommandLine<?>>`
  - 不同模式对应的实现类：
    - `DefaultCLI` → Standalone 集群
    - `YarnSessionCLI` → YARN 集群
    - `KubernetesSessionCli` → K8s 集群
  - 提交作业时会根据参数和配置匹配合适的 `CustomCommandLine` 来决定提交逻辑。

- `getCommandLine`

  **作用**：使用 Apache Commons CLI 解析用户输入的参数

  - 输入：
    - `Options`（定义支持的命令行选项，比如 `-c`, `-m`, `-p` 等）
    - `args`（命令行输入）
  - 输出：
    - `CommandLine` 对象（键值对形式存储参数）
  - **注意**：这里会解析出 main class、jar 路径、并行度、保存点路径等核心信息。

- `parseAndRun`

  **作用**：根据用户输入的第一个参数（子命令）执行对应方法

  - 流程：
    1. 取出第一个参数（如 `run`、`list`、`cancel`）
    2. 匹配到对应的方法：
       - `run` → `CliFrontend.run()`
       - `list` → `CliFrontend.list()`
       - `cancel` → `CliFrontend.cancel()`
    3. 进入后续的作业构建与提交逻辑（构造 `PackagedProgram` → 生成 `JobGraph` → 提交到集群）

### loadCustomCommandLines

~~~java
public static List<CustomCommandLine> loadCustomCommandLines(
            Configuration configuration, String configurationDirectory) {
    	//初始化一个 CustomCommandLine 数组
        List<CustomCommandLine> customCommandLines = new ArrayList<>();
       //通用客户端
        customCommandLines.add(new GenericCLI(configuration, configurationDirectory));

        //	Command line interface of the YARN session, with a special initialization here
        //	to prefix all options with y/yarn.
        final String flinkYarnSessionCLI = "org.apache.flink.yarn.cli.FlinkYarnSessionCli";
        try {
            customCommandLines.add(
                	//yarn客户端
                    loadCustomCommandLine(
                            flinkYarnSessionCLI,
                            configuration,
                            configurationDirectory,
                            "y",
                            "yarn"));
        } catch (NoClassDefFoundError | Exception e) {
            final String errorYarnSessionCLI = "org.apache.flink.yarn.cli.FallbackYarnSessionCli";
            try {
                LOG.info("Loading FallbackYarnSessionCli");
                customCommandLines.add(loadCustomCommandLine(errorYarnSessionCLI, configuration));
            } catch (Exception exception) {
                LOG.warn("Could not load CLI class {}.", flinkYarnSessionCLI, e);
            }
        }

        //	Tips: DefaultCLI must be added at last, because getActiveCustomCommandLine(..) will get
        // the
        //	      active CustomCommandLine in order and DefaultCLI isActive always return true.
    	// 默认客户端
        customCommandLines.add(new DefaultCLI());

        return customCommandLines;
    }
~~~

- `flink`客户端目前默认有3个客户端。通过一个数组包装返回。

  - `GenericCLI`
  - `flinkYarnSessionCLI`
  - `DefaultCLI`

- `CliFrontend` 会加载全局配置（`Configuration`）并绑定可用的 **命令行客户端实现**（`CustomCommandLine` 列表），形成一个支持多种集群模式的客户端入口。

-  `cli`（即具体的 `CliFrontend` 实例）则基于当前配置和已注册的客户端，实现参数解析与任务提交，最终返回解析好的用户命令行对象（`CommandLine`）。

- ~~~java
  final CliFrontend cli = new CliFrontend(configuration, customCommandLines);
              CommandLine commandLine =
                      cli.getCommandLine(
                              new Options(),
                              Arrays.copyOfRange(args, min(args.length, 1), args.length),
                              true);
  ~~~

### `cli.parseAndRun(args)`

~~~java
public int parseAndRun(String[] args) {

        // check for action
        if (args.length < 1) {
            CliFrontendParser.printHelp(customCommandLines);
            System.out.println("Please specify an action.");
            return 1;
        }

        // get action
        String action = args[0];

        // remove action from parameters
        final String[] params = Arrays.copyOfRange(args, 1, args.length);

        try {
            // do action
            switch (action) {
                    //  private static final String ACTION_RUN = "run";
                    //ACTION_RUN 就是 run 。因此直接看这个方法
                case ACTION_RUN:
                    run(params);
                    return 0;
                case ACTION_RUN_APPLICATION:
                    runApplication(params);
                    return 0;
                case ACTION_LIST:
                    list(params);
                    return 0;
                case ACTION_INFO:
                    info(params);
                    return 0;
                case ACTION_CANCEL:
                    cancel(params);
                    return 0;
                case ACTION_STOP:
                    stop(params);
                    return 0;
                case ACTION_SAVEPOINT:
                    savepoint(params);
                    return 0;
                case ACTION_CHECKPOINT:
                    checkpoint(params);
                    return 0;
                case "-h":
                case "--help":
                    CliFrontendParser.printHelp(customCommandLines);
                    return 0;
                case "-v":
                case "--version":
                    String version = EnvironmentInformation.getVersion();
                    String commitID = EnvironmentInformation.getRevisionInformation().commitId;
                    System.out.print("Version: " + version);
                    System.out.println(
                            commitID.equals(EnvironmentInformation.UNKNOWN)
                                    ? ""
                                    : ", Commit ID: " + commitID);
                    return 0;
                default:
                    System.out.printf("\"%s\" is not a valid action.\n", action);
                    System.out.println();
                    System.out.println(
                            "Valid actions are \"run\", \"run-application\", \"list\", \"info\", \"savepoint\", \"stop\", or \"cancel\".");
                    System.out.println();
                    System.out.println(
                            "Specify the version option (-v or --version) to print Flink version.");
                    System.out.println();
                    System.out.println(
                            "Specify the help option (-h or --help) to get help on the command.");
                    return 1;
            }
        } catch (CliArgsException ce) {
            return handleArgException(ce);
        } catch (ProgramParametrizationException ppe) {
            return handleParametrizationException(ppe);
        } catch (ProgramMissingJobException pmje) {
            return handleMissingJobException();
        } catch (Exception e) {
            return handleError(e);
        }
    }
~~~

~~~java
 protected void run(String[] args) throws Exception {
        LOG.info("Running 'run' command.");

        final Options commandOptions = CliFrontendParser.getRunCommandOptions();
        final CommandLine commandLine = getCommandLine(commandOptions, args, true);

        // evaluate help flag
        if (commandLine.hasOption(HELP_OPTION.getOpt())) {
            CliFrontendParser.printHelpForRun(customCommandLines);
            return;
        }

        final CustomCommandLine activeCommandLine =
                validateAndGetActiveCommandLine(checkNotNull(commandLine));

        final ProgramOptions programOptions = ProgramOptions.create(commandLine);

        final List<URL> jobJars = getJobJarAndDependencies(programOptions);

        final Configuration effectiveConfiguration =
                getEffectiveConfiguration(activeCommandLine, commandLine, programOptions, jobJars);

        LOG.debug("Effective executor configuration: {}", effectiveConfiguration);

        try (PackagedProgram program = getPackagedProgram(programOptions, effectiveConfiguration)) {
            executeProgram(effectiveConfiguration, program);
        }
    }

public void invokeInteractiveModeForExecution() throws ProgramInvocationException {
        FlinkSecurityManager.monitorUserSystemExitForCurrentThread();
        try {
            //这一步就是调用的jar包里传的类名以及他的main方法。
            callMainMethod(mainClass, args);
        } finally {
            FlinkSecurityManager.unmonitorUserSystemExitForCurrentThread();
        }
    }
~~~

- 之前任务提交的时候，已经讲解了一部分提交任务的流程。这里就不再细看了。
- 主要也想看的是flink集群如何接收jar包和对应的环境。然后初始化task。并启动任务对应的task。和如何运用netty进行数据传输。

