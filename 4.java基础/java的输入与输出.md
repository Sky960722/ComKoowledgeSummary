# JAVA的输入与输出

## InputStream，OutputStream，Reader和Writer

1. 流的分类 

   1. 按操作数据单位不同分为：字节流(8 bit)，字符流(16 bit) 
   2. 按数据流的流向不同分为：输入流，输出流 
   3. 按流的角色的不同分为：节点流，处理流

2. | (抽象基类) | 字节流       | 字符流 |
   | ---------- | ------------ | ------ |
   | 输入流     | InputStream  | Reader |
   | 输出流     | OutputStream | Writer |

### java.io.InputStream

1. InputStream是一个抽象类，只有一个抽象方法——abstract int read()。这个方法将读入一个字节，并返回读入的字节，或者在遇到输入源结尾时返回-1。

   ~~~java
   public abstract int read() throws IOException;
   ~~~

2. InputStream还有若干个非抽象的方法，这些方法都调用了抽象的read方法，因此，各个子类都只需覆盖这一个方法

   ~~~java
   //读入的值将置于b中从off开始的位置。返回实际读入的字节数，或者开始读取的第一个字节正好是文件末尾，返回-1
   public int read(byte b[], int off, int len) throws IOException {
   		//判断当前传入的字节数组是否是null
           if (b == null) {
               throw new NullPointerException();
           } else if (off < 0 || len < 0 || len > b.length - off) {//判断off和len是否是有效位置。off必然大于0，len也必然大于0，同时off也必然小于b.length,不然数组会越界
               throw new IndexOutOfBoundsException();
           } else if (len == 0) { //len = 0，则表明只读取了0个字节，直接返回0即可
               return 0;
           }
   		//先读取一个字节，如果当前字节已到末尾，则返回-1，不然赋值给b[off]
           int c = read();
           if (c == -1) {
               return -1;
           }
           b[off] = (byte)c;
   
           int i = 1;//这个i是返回读取的字节数
           try {
           	//for循环顺序读取，直到读到结尾，退出
               for (; i < len ; i++) {
                   c = read();
                   if (c == -1) {
                       break;
                   }
                   b[off + i] = (byte)c;
               }
           } catch (IOException ee) {
           }
           return i;
       }
       
       private static final int MAX_SKIP_BUFFER_SIZE = 2048;
       
       //这个方法是在输入流中跳过n个字节，返回实际跳过的字节数
       public long skip(long n) throws IOException {
   		//remaining是读取的字节流，初始值是n，只要输入流的字节总量超过n个字节量，则后续的计算中，remaining必然等于0
           long remaining = n;
           // nr是每次执行read(byte b[], int off, int len)这个方法返回实际读入的字节数
           int nr;
   
           if (n <= 0) {
               return 0;
           }
   		
   	   //设置缓冲池的大小，一般来说n，但是最多不能超过2048bytes，相当于2kb
           int size = (int)Math.min(MAX_SKIP_BUFFER_SIZE, remaining);
           byte[] skipBuffer = new byte[size];
           //当前还能读的数据，remaining要始终大于0，保证读取的数据量没有超过n
           while (remaining > 0) {
               //每次读取的字节填入到skipBuffer中，同时Math.min确保最后一次读取的字节正好是n的量
               nr = read(skipBuffer, 0, (int)Math.min(size, remaining));
               //说明输入流已经到了末尾，直接退出循环
               if (nr < 0) {
                   break;
               }
               // 读取完后，每次更新remaining的值
               remaining -= nr;
           }
   	    //返回实际读取的字节数
           return n - remaining;
       }
   ~~~

3. Inputstream方法

   ~~~java
   java.io.InputStream
   abstract int read() //从数据中读入一个字节，并返回该字节
   int read(byte[] b) //读入一个字节数组，并返回实际读入的字节数，或者再碰到输入流的结尾时返回-1
   void close() //关闭这个输入流
   void mark(int readlimit) //在输入流的当前位置打一个标记
   void reset() //返回到最后一个标记，随后对read的调用将重新读入这些字节
   boolean markSupported() //如果这个流支持打标机，则返回true
   ~~~

4. InputStream和OutputStream在调用完后，需要调用close方法来关闭。

4. 

### java.io.OutputStream

   1. 和InputStream类似，是一个抽象类，有一个抽象方法abstract void write(int b)

   2. OutputStream还有若干个非抽象的方法，这些方法都调用了抽象的write方法，因此，各个子类都只需覆盖这一个方法

      ~~~java
      public void write(byte b[], int off, int len) throws IOException {
              if (b == null) {
                  throw new NullPointerException();
              } else if ((off < 0) || (off > b.length) || (len < 0) ||
                         ((off + len) > b.length) || ((off + len) < 0)) {
                  throw new IndexOutOfBoundsException();
              } else if (len == 0) {
                  return;
              }
              for (int i = 0 ; i < len ; i++) {
                  write(b[off + i]); //循环遍历当前b数组，并写出
              }
          }
      ~~~

   3. ~~~java
      java.io.OutputStream
      abstract void write(int n) //写出一个字节的数据
      void write(byte[] b) 
      void write(byte[] b,int off,int len) //写出所有字节或者某个范围的字节到数组b中
      void close() //冲刷并关闭输出流
      void flush() //冲刷输出流，将缓存的数据发送到目的地
      ~~~

   ### java.io.Reader

1. 对于Unicode文本，Unicode格式的字符底层存储是占1~2个字符，因此用Read和Writer类合适
   1. char类型是2个字节。而Unicode字符超过了65536。现代的UTF-16编码格式，每个字符用16位表示，通常称这16位为代码单元，而辅助字符编码为一对连续的代码单元。也就是2~4个字节。因此char类型准确来说描述的UTF-16的代码单元。
   2. 码点是指与一个编码表中的某个字符对应的代码值。
   3. Character.isSupplementaryCodePoint(int codePoint) 可以判断当前码点是否在增补字符范围内，也就是在两个字节以内，能被char基本类型解析。

2. Reader类是抽象类，有一个抽象方法public int read()，返回一个Unicode码元

### java.io.Writer

1. 和上述类似

### 4个附加的接口：Closeable、Flushable、Readble和Appendable

1. Closeable接口
   1. 方法：public void close() throws IOException;
   2. InputStream、OutputStream、Reader和Writer都实现了Closeable接口
2. Flushable接口
   1. 方法：void flush() throws IOException;
   2. OutputStream和Writer都实现了Flushable接口
3. Readable接口
   1. 方法：int read(java.nio.CharBuffer cb)
   2. Reader类实现了Readable接口
4. Appendable接口
   1. 方法：Appendable append(char c) 
   2. 只有Writer实现了Appendable


## 常用的流

| 分类     | 字节输入流          | 字节输出流           | 字符输入流        | 字符输出流         |
| -------- | ------------------- | -------------------- | ----------------- | ------------------ |
| 访问文件 | FileInputStream     | FileOutputStream     | FileReader        | FileWriter         |
| 缓冲流   | BufferedInputStream | BufferedOutputStream | BufferReader      | BufferWriter       |
| 转换流   |                     |                      | InputStreamReader | OutputStreamWriter |
| 打印流   |                     | PrintStream          |                   | PrintWriter        |
| 特殊流   | DataInputStream     | DataOutputStream     |                   |                    |
| 对象流   | ObjectInputStream   | ObjectOutputStream   |                   |                    |

# File类

1. java.io.File类：文件和文件目录路径的抽象表示形式，与平台无关
2.  File类方法
   1.   获取功能
      1. public String getAbsolutePath()：获取绝对路径 
      2. public String getPath() ：获取路径 
      3. public String getName() ：获取名称 
      4.  public String getParent()：获取上层文件目录路径。若无，返回null 
      5. public long length() ：获取文件长度（即：字节数）。不能获取目录的长度。 
      6. public long lastModified() ：获取最后一次的修改时间，毫秒值 
      7. public String[] list() ：获取指定目录下的所有文件或者文件目录的名称数组 
      8. public File[] listFiles() ：获取指定目录下的所有文件或者文件目录的File数组  
   2. 重命名功能 
      1.  public boolean renameTo(File dest):把文件重命名为指定的文件路径
   3. 判断功能
      1. public boolean isDirectory()：判断是否是文件目录 
      2.  public boolean isFile() ：判断是否是文件 
      3. public boolean exists() ：判断是否存在 
      4. public boolean canRead() ：判断是否可读 
      5.  public boolean canWrite() ：判断是否可写 
      6.  public boolean isHidden() ：判断是否隐藏
   4. 创建功能
      1. public boolean createNewFile() ：创建文件。若文件存在，则不创建，返回false 
      2.  public boolean mkdir() ：创建文件目录。如果此文件目录存在，就不创建了。 如果此文件目录的上层目录不存在，也不创建。 
      3. public boolean mkdirs() ：创建文件目录。如果上层文件目录不存在，一并创建 注意事项：如果你创建文件或者文件目录没有写盘符路径，那么，默认在项目 路径下。
   5. 删除功能
      1. public boolean delete()：删除文件或者文件夹

# Path

 1. ~~~java
    java.nio.file.Paths
    static Path get(String first,String ..more) //通过连接给定的字符串创建一个路径
    
    Path resolve(Path other)
    Path resolver(String other) //如果other是绝对路径，那么就返回other；否则，返回通过连接this和other获得的路径
    
    Path resolveSlibling(Path other)
    Path resolveSibling(String other) //如果other是绝对路径，就返回other；否则，返回通过连接this的父路径和other获得的路径
    
    Path relativize(Path other) //返回用this进行解析，相对于other的相对路径
    
    Path normalize() //移除诸如.和..等冗余的路径元素
    
    Path toAbsolutePath() //返回与该路径等价的绝对路径
    
    Path getParent() //返回父路径，或者在该路径没有父路径时，返回null
    
    Path getFileName() //返回该路径的最后一个部件，或者在该路径没有任何部件时，返回null
    
    Path getRoot() //返回该路径的根路径
    
    toFile() //从该路径中创建一个File对象
    
    ~~~
