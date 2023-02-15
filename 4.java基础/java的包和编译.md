# JAVA包和编译

## 包和路径

 1. javac这个编译可以编译.java文本，java源文件中没有package语句，则是无名包。通常情况下，需要将源文件与完整包名放到匹配的子目录，或者通过-cp加载类路径。

 2. 虚拟机加载是通过包名和类路径找到对应的类，如果java启动没有包名.类名，会报错。除非没有包，则直接当前目录启动。

 3. 类路径分为三类，用:分割：/home/user/classdir:.:/home/user/archives/archive.jar

    	1. 基目录  /home/user/classdir
    	2. 当前目录 (.)
    	3. JAR文件 /home/user/archives/archive.jar 

 4. 设置类路径有三种方式

    	1. -classpath 

    ~~~shell
    java -classpath /home/user/classdir:.:/home/user/archives/archive.jar MyProg
    ~~~

    2. -cp

    ~~~shell
    java -cp /home/user/classdir:.:/home/user/archives/archive.jar MyProg
    ~~~

    3. --class-path 

    ~~~shell
    java --class-path  /home/user/classdir:.:/home/user/archives/archive.jar MyProg
    ~~~

 5. JAR文件

    	1. jar制作JAR，这个文件是压缩且带有格式的，如果要指定程序入口，可以带e。JAR文件类似tar包，带有文件格式，因此，可以将多.class文件合并成一个文件，并能通过包名.类名找到类。
    	2. 启动jar包，直接通过 java -jar MyProgram.jar 执行

 6. 参数选项

| javac | 编译.java文件                                                |
| ----- | ------------------------------------------------------------ |
| java  | 执行.class或者jar程序                                        |
| jar   | 制作JAR文件                                                  |
| c     | 创建新的或者空的存档文件并加入文件                           |
| C     | 临时改变目录,例如 jar cvf jarFileName.jar -C classes *.class |
| e     | 创建入口点，配合 java -jar 使用 例如 jar cvfe test.jar reflection.MethodTableTest  ./reflection/*.class   java -jar test.jar |
| f     | 指定jar文件名作为第二个命令行参数。                          |
| v     | 生成详细的输出结果                                           |
| x     | 解压文件                                                     |



