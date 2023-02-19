# JAVA的异常

## JAVA异常的类型

1. 所有异常由Throwable继承而来，但在下一层分为两个分支：分别是Error和Exception
   1. Error类层次结构描述了Java运行时系统的内部错误和资源耗尽错误。
   2. EXception分为RuntimeException分支，另一个分支包含其他异常
      - 派生于RuntimeException的异常包括以下问题：
        - 错误的强制类型转换
        - 数组访问越界
        - 访问null指针
      - 非派生于RuntimeException的异常
        - 试图超越文件末尾继续读取数据
        - 试图打开一个不存在的文件
        - 试图根据给定的字符串查找Class对象，而这个字符串表示的类并不存在
   3. Java语言规范将派生于Error类或RuntimeException类的所有异常称为非检查型（unchecked），所有其他的异常称为检查型（checked）异常。
      1. 检查型异常：编译器将会检查你是否知道这个异常并做好准备来处理后果
      2. 非检查型异常：编译器并不期望你为这些异常提供处理器。毕竟，你应该集中精力避免这些错误的发生，而不要将精力花在编写异常处理器上

## 检查型异常

1. 如果遇到了无法处理的情况，Java方法可以抛出一个异常。要在方法首部指出这个方法可能抛出一个异常，所以要修改方法首部，以反映这个方法可能抛出的检查型异常。

   public FileInputStream(String name) throws FileNotFoundException

2. 4种情况抛出异常

   1. 调用了一个抛出检查型异常的方法，例如，FileInputStream构造器
   2. 检测到一个错误，并且利用throw语句抛出一个检查型异常
   3. 程序出现错误，例如，a[-1]=0会抛出一个非检查型异常
   4. Java虚拟机或运行时库出现内部错误

3. 子类覆盖了超类的一个方法，子类方法种声明的检查型异常不能比超类方法种声明的异常更通用。

4. 如何抛出异常

   1. 语句：

   ~~~java
   throw new EOFException();
   var e = new EOFException();
   throw e;
   
   //例如
   String readDate(Scanner in) throws EOFException
   {
       while(...)
       {
           if(!in.hasNext()){
               if(n < len){
                   throw new EOFException();
               }
               .....
           }
           return s;
       }
   }
   ~~~

## 异常类

      1. 代码可能会遇到任何标准异常类都无法描述清楚的问题。在这种情况下，创建自己的异常类就是一件顺理成章的事情
      2. 方法
            1. 定义一个派生于Exception的类，或者派生于Exception的某个子类，如IOException
            2. 自定义的这个类应该包含两个构造器，一个是默认的构造器，另一个是包含详细描述信息的构造器（超类Throwable的toString方法会返回一个字符串）
    
      3. 代码：

~~~java
class FileFormatException extends IOException{
    public FileFormatException(){}
    public FileFormatException(String gripe){
        super(gripe);
    }
}
~~~

4. 常用代码

~~~java
java.lang.Throwable
Throwable()
//构造一个新的Throwable对象，但没有详细的描述信息
Throwable(String message)
//构造一个新的Throwable对象，带有指定的详细描述信息。通常，所有派生异常类都支持一个默认构造器和一个带有详细描述信息的构造器
String getMessage()
//获得Throwable对象的详细描述信息
~~~

## 捕获异常

1. 没有捕捉异常，程序就会终止，并在控制台上打印一个消息，其中包括这个异常的类型和一个堆栈轨迹
2. try/catch语句块捕捉异常
   - 如果try语句块中的任何代码抛出了catch子句种指定的一个异常类，那么
     - 程序将跳过try语句块的其余代码
     - 程序将执行catch子句中的处理器代码
   - 如果try语句块中的代码没有抛出任何异常，那么程序将跳过catch子句
   - 如果方法中的任何代码抛出了catch子句种没有声明的异常类型，那么这个方法就会立即退出
3. 如果调用了一个抛出检查型异常的方法，就必须处理这个异常，或者继续传递这个异常
   - 一般经验是捕获那些你知道如何处理的异常，而继续传播那些你不知道怎样处理的异常
   - 传播异常，必须在方法的首部添加一个throws说明符，提醒调用者这个方法可能会抛出异常
4. 可以捕获多个异常对象
   1. 异常对象可能包含有关异常性质的信息。要想获得这个对象的更多信息，可以尝试使用e.getMessage(),详细信息，使用e.getClass().getName()
   2. 同一个catch子句可以捕获多个异常类型

## 再次抛出异常与异常链

1. 可以在catch子句中抛出一个异常。通常，希望改变异常的类型时会这样做。
2. 可以把原始异常设置为新异常的“原因”

~~~java
try{
    access the database
}
catch(SQLException original){
    var e = new ServletException("database error");
    e.initCause(original);
    throw e;
}

//捕获这个异常，可以使用下面这条语句获取原始异常：
Throwable original = caughtException.getCause();
~~~

## Finally子句

1. 不管是否有异常被捕获，finally子句中的代码都会执行

2. try-with-Resources语句可以一定程度上简化finally

   1. 代码

   ~~~java
   try(var in = new Scanner(new FIleInputStream("/usr/share/dict/words"),StandardCharsets.UTF_8)){
       while(in.hasNext()){
           System.out.println(in.next());
       }
   }
   ~~~

## 分析堆栈轨迹元素

1. 堆栈轨迹（stack trace）是程序执行过程种某个特定点上所有挂起的方法调用的一个列表
2. 可以调用Throwable类的printStackTrace方法访问堆栈轨迹的文本描述信息

~~~java
var t = new Throwable();
var out = new StringWriter();
t.printStackTrace(new PrintWriter(out));
String description = out.toString();
~~~

3. StackWalker类，会生成一个StackWalker.StackFrame实例流，其中每个实例分别描述一个栈帧(stack frame)。迭代处理这些栈帧

~~~java
StackWalker walker = StackWalker.getInstance();
walker.forEach(frame -> analyze frame);
~~~

4. 常用代码

~~~java
java.lang.Throwable
    Throwable(Throwable cause)
    Throwable(String message,Throwable cause)
    //用给定的cause（原因）构造一个Throwable对象
    Throwable initCause(Throwable cause)
    //为这个对象设置原因，如果这个对象已经由原因，则抛出一个异常，返回this
    Throwable getCause()
    //获得设置为这个对象的原因的异常对象。如果没有设置原因，则返回null
    StackTraceElement[] getStackTrace()
    //获得构造这个对象时调用堆栈的轨迹
    void addSuppressed(Throwable t)
    //为这个异常增加一个"抑制的"异常
    Throwable[] getSuppressed()
    //得到这个异常的所有"抑制的"异常
    
java.lang.Exception
    Exception(Throwable cause)
    Exception(String message,Throwable cause)
    //用给定的cause(原因)构造一个Exception对象
    
java.lang.RuntimeException
    RuntimeException(Throwable cause)
    RuntimeException(String message,Throwable cause)
    //用给定的cause(原因)构造一个RuntimeException对象
~~~

## 使用异常的技巧

1. 异常处理不能代替简单的测试
2. 不要过分地细化异常
3. 充分利用异常层次结构
4. 不要压制异常
5. 在检测错误时，“苛刻”要比放任更好
6. 不要羞于传递异常 （5，6归结早抛出，晚捕获）

# 日志

1. 主要优点

   1. 可以很容易地取消全部日志记录，或者仅仅取消某个级别以下的日志，而且可以很容易地再次打开日志开关
   2. 可以很简单的禁止日志记录，因此，将这些日志代码留在程序中的开销很小
   3. 日志记录可以被定向到不同的处理器，如在控制台显示、写至文件，等等
   4. 日志记录器和处理器都可以对记录进行过滤。过滤器可以根据过滤实现器指定的标准丢弃那些无用的记录项
   5. 日志记录可以采用不同的方式格式化，例如，纯文本或XML
   6. 应用程序可以使用多个日志记录器，它们使用与包名类似的有层次结构的名字，例如，com.mycompany.myapp
   7. 日志系统的配置由配置文件控制

2. 基本日志

   1. 生成简单的日志记录，可以使用全局日志记录器(global logger)并调用其info方法:Logger.getGlobal().info("File->Open menu item selected")

3. 高级日志

   1. 定义自己的日志记录器

   ~~~java
   private static final Logger myLogger = Logger.getLogger("com.mycompany.myapp")
   ~~~

   2. 日志记录器名具有层次结构。且父与子之间将共享某些属性。

   3. 有7个日志级别：

      - SEVERE
      - WARNING
      - INFO
      - CONFIG
      - FINE
      - FINER
      - FINEST

      1. 默认情况下，只记录前3个级别

         1. 可以设置一个不同的级别：logger.setLevel(Level.FINE)
         2. 使用Level.ALL开启所有级别的日志记录，或者使用Level.OFF关闭所有级别的日志记录
         3. 所有级别都有日志记录方法

         ~~~java
         logger.warning(message);
         logger.fine(message);
         logger.log(Level.FINE,message)
         ~~~

4. 可以通过编辑配置文件来修改日志系统的各个属性。默认，位于conf/logging.properties。

   1. 日志管理器可以在虚拟机启动时虚拟化，程序中调用System.setProperty("java.util.logging.config.fife",file),接着调用LogManager.getLogManager().readConfiguration()重新初始化日志管理器

5. 日志分为日志记录器，处理器，过滤器，格式化器

6. 日志技巧

   1. 对一个简单的应用，选择一个日志记录器。
   2. 默认的日志配置会把级别等于或高于INFO的所有消息记录到控制台。用户可以覆盖这个默认配置。
   3. 将程序员想要的日志消息设定为FINE级别是一个很好的选择

7. 代码

~~~java
Logger logger = Logger.getLogger("sry");
        logger.setUseParentHandlers(false);

        ConsoleHandler consoleHandler = new ConsoleHandler();
        SimpleFormatter simpleFormatter = new SimpleFormatter();
        consoleHandler.setFormatter(simpleFormatter);
        logger.addHandler(consoleHandler);
        logger.setLevel(Level.ALL);
        consoleHandler.setLevel(Level.ALL);

        FileHandler fileHandler = new FileHandler("D:\\IdealProject\\src\\jul.log");

        //进行关联
        fileHandler.setFormatter(simpleFormatter);
        logger.addHandler(fileHandler);

        logger.severe("severe");
        logger.warning("warning");
        logger.info("info");
        logger.fine("fine");
~~~

8. 日志记录文件模式变量

| 变量 | 描述                                                         |
| ---- | ------------------------------------------------------------ |
| %h   | 系统属性 user.home的值                                       |
| %t   | 系统临时目录                                                 |
| %u   | 用于解决冲突的唯一编号（如果多个应用程序使用同一个日志文件，就应该开启append标志） |
| %g   | 循环日志的生成号                                             |
| %%   | %字符                                                        |

