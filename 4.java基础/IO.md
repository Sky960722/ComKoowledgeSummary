# IO

- 以下概念来源于Java核心技术和Blog：https://jenkov.com/tutorials/java-io/outputstream.html

## 概念

​	JAVA IO 是一个用于读写data的api。JAVA IO API在JAVA IO 包下。IO包主要由 输入（INPUT（Source））、输出（OUTPUT（Destination））和流（Streams、Reader）组成。

- 典型的输入和输出
  - Files
  - Pipes
  - Network Connections
  - In-memory Buffers（arrays）
  - System.in,System.out,System.err
- 流则有四个大类：InputStream,OutputStream,Reader和Writer。并且底下有非常多的子类，这些子类有各自不同的目的。这些不同的目的概括一下：
  - 访问文件
  - 访问网络
  - 访问内存
  - 线程间通信
  - 读写文本文件
  - 流转换
  - 读写数字
  - 读写类

## 类型清单

|                | Byte Based                          |                                   | Character Based                 |                 |
| -------------- | ----------------------------------- | --------------------------------- | ------------------------------- | --------------- |
|                | Input                               | Output                            | Input                           | Output          |
| Basic          | InputStream                         | OutputStream                      | InputStreamReader               | OutputWriter    |
| Arrays         | ByteArrayInputStream                | ByteArrayOutputStream             | CharArrayReader                 | CharArrayWriter |
| Files          | FileInputStream RandomAccessFile    | FileOutputStream RandomAccessFile | FileReader                      | FileWriter      |
| Pipes          | PipedInputStream                    | PipedOutputStream                 | PipedReader                     | PipedWriter     |
| Buffering      | BufferedInputStream                 | BufferedOutputStream              | BufferedReader                  | BufferedWriter  |
| Filtering      | FilterInputStream                   | FilterOutputStream                | FilterReader                    | FilterWriter    |
| Parsing        | PushbackInputStream StreamTokenizer |                                   | PushbackReader LineNumberReader |                 |
| Strings        |                                     |                                   | StringReader                    | StringWriter    |
| Data           | DataInputStream                     | DataOutputStream                  |                                 |                 |
| Data-Formatted |                                     | PrintStream                       |                                 | PrintWriter     |
| Objects        | ObjectInputStream                   | ObjectOutputStream                |                                 |                 |
| Utilities      | SequenceInputStream                 |                                   |                                 |                 |

## InputStream/OutputStream

### 概念

- InputStream表示的是一个顺序的字节流，可以从InputStream读取字节。

- OutputStream用来将流写出。

- 两者都是抽象类。真正工作的流都是基于他们的实现类

  | InputStream          | OutputStream          |
  | -------------------- | --------------------- |
  | ByteArrayInputStream | ByteArrayOutputStream |
  | FileInputStream      | FileOutputStream      |
  | PipedInputStream     | PipedOutputStream     |
  | BufferedInputStream  | BufferedOutputStream  |
  | FilterInputStream    | FilterOutputStream    |
  | PushBackInputStream  | DataOutputStream      |
  | DataInputStream      | PrintStream           |
  | ObjectInputStream    | ObjectOutputStream    |
  | SequenceInputStream  |                       |

### InputStream方法

#### read

- 返回-1，则表面读到结尾

  ~~~java
  int data = inputStream.read();
  while(data != -1) {
      // do something with data variable
  
      data = inputStream.read(); // read next byte
  }
  ~~~

#### read(byte[])

- int read(byte[])

- int read(byte[], int offset, int length)

~~~sql
InputStream inputstream = new FileInputStream("c:\\data\\input-text.txt");

byte[] data      = new byte[1024];
int    bytesRead = inputstream.read(data);

while(bytesRead != -1) {
  doSomethingWithData(data, bytesRead);

  bytesRead = inputstream.read(data);
}
inputstream.close();
~~~

#### readAllBytes()

- 尽可能的读取

  ~~~java
  byte[] fileBytes = null;
  try(InputStream input = new FileInputStream("myfile.txt")) {
     fileBytes = input.readALlBytes();
  }
  ~~~

#### Closing an InputStream

~~~java
InputStream inputstream = new FileInputStream("c:\\data\\input-text.txt");

int data = inputstream.read();
while(data != -1) {
  data = inputstream.read();
}
inputstream.close();

try( InputStream inputstream = new FileInputStream("file.txt") ) {

    int data = inputstream.read();
    while(data != -1){
        data = inputstream.read();
    }
}
~~~

### OutputStream方法

#### write(byte)

~~~java
OutputStream outputStream = new FileOutputStream("c:\\data\\output-text.txt");

while(hasMoreData()) {
  int data = getMoreData();
  outputStream.write(data);
}
outputStream.close();
~~~

#### write(byte[])

~~~java
OutputStream outputStream = new FileOutputStream("/usr/home/jakobjenkov/output.txt");

byte[] sourceBytes = ... // get source bytes from somewhere.

int bytesWritten = outputStream.write(sourceBytes, 0, sourceBytes.length);
~~~

#### flush()

~~~java
outputStream.flush();
~~~

#### Close an OutputStream

~~~java
OutputStream outputStream = new FileOutputStream("c:\\data\\output-text.txt");

while(hasMoreData()) {
    int data = getMoreData();
    outputStream.write(data);
}
outputStream.close();

try( OutputStream outputStream = new FileOutputStream("c:\\data\\output-text.txt")) {
    while(hasMoreData()) {
        int data = getMoreData();
        outputStream.write(data);
    }
}
~~~



## 文件读写相关的类

### File

#### 目的

​	java.io.File 的目的是访问底层文件系统。它能提供能做以下5件事（File并不能读取文件，如果要读取文件，需要用FileInputStream,FileOutputStream或者RandomStream）

- 检查文件或文件夹是否存在
- 产生一个不存在的文件夹
- 文件长度的读取
- 重命名或移动文件
- 删除文件
- 检查该路径是文件还是文件夹
- 列出该文件夹下所有的文件

##### 产生文件

~~~java
File file = new File("c:\\data\\input-file.txt");
~~~

##### 检查文件是否存在

~~~java
File file = new File("c:\\data\\input-file.txt");

boolean fileExists = file.exists();

File file = new File("c:\\data");

boolean fileExists = file.exists();
~~~

##### 产生一个文件夹

~~~java
File file = new File("c:\\users\\jakobjenkov\\newdir");

boolean dirCreated = file.mkdir();
~~~

##### 文件长度

~~~java
File file = new File("c:\\data\\input-file.txt");

long length = file.length();
~~~

##### 重命名或移动文件

~~~java
File file = new File("c:\\data\\input-file.txt");

boolean success = file.renameTo(new File("c:\\data\\new-file.txt"));
~~~

##### 删除文件或文件夹

~~~java
File file = new File("c:\\data\\input-file.txt");

boolean success = file.delete();
~~~

##### 递归删除文件夹

~~~java
public static boolean deleteDir(File dir){
    File[] files = dir.listFiles();
    if(files != null){
        for(File file : files){
            if(file.isDirectory()){
                deleteDir(file);
            } else {
                file.delete();
            }
        }
    }
    return dir.delete();
}
~~~

##### 检查该路径是文件还是文件夹

~~~java
File file = new File("c:\\data");

boolean isDirectory = file.isDirectory();
~~~

##### 列出该文件夹下所有的文件

~~~java
File file = new File("c:\\data");

String[] fileNames = file.list();

File[]   files = file.listFiles();
~~~

### RandomAccessFile

#### 目的

​	RandomAccessFile允许你随意导航文件并在指定的位置读取或者写入。也可以替换文件中的一部分。

##### 产生一个RandomAccessFile

~~~java
RandomAccessFile file = new RandomAccessFile("c:\\data\\file.txt", "rw");
~~~

###### 访问类型

| Mode | 描述                   |
| ---- | ---------------------- |
| r    | 读取                   |
| w    | 写入                   |
| rwd  | 读写同步               |
| rws  | 读写同步（元数据同步） |

##### 导航指定位置

~~~java
RandomAccessFile file = new RandomAccessFile("c:\\data\\file.txt", "rw");

file.seek(200);
~~~

##### 获取文件导航位置

~~~java
long position = file.getFilePointer();
~~~

##### 读取字节

~~~java
RandomAccessFile file = new RandomAccessFile("c:\\data\\file.txt", "rw");

int aByte = file.read();
~~~

##### 读取字节到数组

~~~java
RandomAccessFile randomAccessFile = new RandomAccessFile("data/data.txt", "r");

byte[] dest      = new byte[1024];
int    offset    = 0;
int    length    = 1024;
int    bytesRead = randomAccessFile.read(dest, offset, length);
~~~

##### 写入字节

~~~java
RandomAccessFile file = new RandomAccessFile("c:\\data\\file.txt", "rw");

file.write(65); // ASCII code for A
~~~

##### 写入字节数组

~~~java
RandomAccessFile file = new RandomAccessFile("c:\\data\\file.txt", "rw");

byte[] bytes = "Hello World".getBytes("UTF-8");

file.write(bytes);

RandomAccessFile file = new RandomAccessFile("c:\\data\\file.txt", "rw");

byte[] bytes = "Hello World".getBytes("UTF-8");

file.write(bytes, 2, 5);
~~~

##### 回收该对象

~~~java
RandomAccessFile file = new RandomAccessFile("c:\\data\\file.txt", "rw");

file.close();

try(RandomAccessFile file = new RandomAccessFile("c:\\data\\file.txt", "rw")){

    //read or write from the RandomAccessFile

}
~~~

### FileInputStream/FileOutputStream

目的：读写文本，返回的字节

#### FileInputStream构造

~~~java
String path = "C:\\user\\data\\thefile.txt";
File   file = new File(path);

FileInputStream fileInputStream = new FileInputStream(file);
~~~

#### 读取/读取字节数组

~~~java
FileInputStream fileInputStream =
    new FileInputStream("c:\\data\\input-text.txt");


int data = fileInputStream.read();
while(data != -1) {
    // do something with data variable

    data = fileInputStream.read(); // read next byte
}

FileInputStream fileInputStream = new FileInputStream("c:\\data\\input-text.txt");

byte[] data      = new byte[1024];
int    bytesRead = fileInputStream.read(data, 0, data.length);

while(bytesRead != -1) {
  doSomethingWithData(data, bytesRead);

  bytesRead = fileInputStream.read(data, 0, data.length);
}
~~~

#### FileInputStream关闭

~~~java
FileInputStream fileInputStream = new FileInputStream("c:\\data\\input-text.txt");

int data = fileInputStream.read();
while(data != -1) {
  data = fileInputStream.read();
}
fileInputStream.close();

try( FileInputStream fileInputStream = new FileInputStream("file.txt") ) {

    int data = fileInputStream.read();
    while(data != -1){
        data = fileInputStream.read();
    }
}
~~~

#### FileOutputStream构造

~~~java
String path = "C:\\users\\jakobjenkov\\data\\datafile.txt";
File   file = new File(path);

FileOutputStream output = new FileOutputStream(file);

OutputStream output = new FileOutputStream("c:\\data\\output-text.txt", true); //append

OutputStream output = new FileOutputStream("c:\\data\\output-text.txt", false); //overwrite
~~~

#### 写入

~~~java
OutputStream outputStream = new FileOutputStream("c:\\data\\output-text.txt");

outputStream.write(123);
~~~

#### 字节数组写入

~~~java
OutputStream outputStream = new FileOutputStream("c:\\data\\output-text.txt");

byte bytes =  new byte[]{1,2,3,4,5};

outputStream.write(bytes);
~~~

#### 刷新缓冲区

- 用Java的FileOutputStream将数据写出到磁盘，首先会先进入内存，稍后在写入磁盘。在FileOutputStream关闭前，一定要确保内存中的数据全部写入到磁盘中。在调用close()方法前，调用这个方法。

~~~java
OutputStream outputStream = new FileOutputStream("c:\\data\\output-text.txt");

byte bytes =  new byte[]{1,2,3,4,5};

outputStream.write(bytes);

outputStream.flush()
~~~

#### 关闭FileOutputStream

~~~java
OutputStream outputStream = new FileOutputStream("c:\\data\\output-text.txt");

outputStream.write(123);

outputStream.close();

try( OutputStream outputStream = new FileOutputStream("c:\\data\\output-text.txt") ) {
    outputStream.write(123);
}
~~~

### FileReader/FileWriter

#### 目的

- 直接通过字符的方式读取文件

##### 读取字符

~~~java
FileReader fileReader = new FileReader("c:\\data\\input-text.txt");

int data = fileReader.read();
while(data != -1) {
  data = fileReader.read();
}
~~~

##### 读取字符串

~~~java
FileReader fileReader = new FileReader("c:\\data\\input-text.txt");

char[] destination = new char[1024];

int charsRead = fileReader.read(destination, 0, destination.length);
~~~

##### 关闭FileReader

~~~java
fileReader.close();

try(FileReader fileReader =
    new FileReader("c:\\data\\text.txt")){

    int data = fileReader.read();
    while(data != -1) {
        System.out.print((char) data));
        data = fileReader.read();
    }
}
~~~

##### FileOutputStream构造

~~~java
String path = "C:\\users\\jakobjenkov\\data\\datafile.txt";
File   file = new File(path);

FileOutputStream output = new FileOutputStream(file);
~~~

##### 追加/覆写模式

~~~java
OutputStream output = new FileOutputStream("c:\\data\\output-text.txt", true); //append

OutputStream output = new FileOutputStream("c:\\data\\output-text.txt", false); //overwrite
~~~

##### 写入字符

~~~java
OutputStream outputStream = new FileOutputStream("c:\\data\\output-text.txt");

outputStream.write(123);
~~~

##### 写入字符串

~~~java
OutputStream outputStream = new FileOutputStream("c:\\data\\output-text.txt");

byte bytes =  new byte[]{1,2,3,4,5};

outputStream.write(bytes);
~~~

##### 刷新缓冲区

~~~java
OutputStream outputStream = new FileOutputStream("c:\\data\\output-text.txt");

byte bytes =  new byte[]{1,2,3,4,5};

outputStream.write(bytes);

outputStream.flush()
~~~

关闭FileOutputStream

~~~java
OutputStream outputStream = new FileOutputStream("c:\\data\\output-text.txt");

outputStream.write(123);

outputStream.close();

try( OutputStream outputStream = new FileOutputStream("c:\\data\\output-text.txt") ) {
    outputStream.write(123);
}
~~~

## ByteArrayInputStream/ByteArrayOutputStream/BufferedReader/BufferedWriter

### 概念

- 为了提升效率，java从流中读取数据或者写出数据，是需要从用户空间到内核空间（操作系统空间），再从内核空间调用函数，与磁盘进行交互。这个过程数据重复拷贝了3次，并且还涉及到进程的上下文切换。
- 如果每次都调用InputStream.read或者OutputStream.writer，会导致效率低下。因此，直接将数据大批量写入到内存中，等内存满了，或者程序写完/读完。再一次执行上述这个过程。
- 除了构造方法略有差异以外，其余方法一致。

### 构造方法

~~~java
byte[] bytes = ... //get byte array from somewhere.

int offset = 20;
int length = 45;

ByteArrayInputStream byteArrayInputStream = new ByteArrayInputStream(bytes, offset, length);

~~~

