# Java的NIO

## IO概述

- `IO` 的操作方式通常分为几种：同步阻塞`BIO`、同步非阻塞`NIO`、异步非阻塞`AIO`。
  1. 在`JDK1.4` 之前，我们建立网络连接的时候采用的是`BIO` 模式。
  2. `Java NIO（New IO 或Non Blocking IO）`是从`Java 1.4` 版本开始引入的一个新的`IO API`，可以替代标准的`Java IO API`。`NIO` 支持面向缓冲区的、基于通道的`IO` 操作。`NIO` 将以更加高效的方式进行文件的读写操作。`BIO` 与`NIO` 一个比较重要的不同是，我们使用`BIO` 的时候往往会引入多线程，每个连接对应一个单独的线程；而`NIO` 则是使用单线程或者只使用少量的多线程，让连接共用一个线程。
  3. `AIO` 也就是`NIO 2`，在`Java 7` 中引入了`NIO` 的改进版`NIO 2`,它是异步非阻塞的`IO` 模型。

### 阻塞 IO(BIO)

- 阻塞`IO`（`BIO`）是最传统的一种`IO` 模型，即在读写数据过程中会发生阻塞现象，直至有可供读取的数据或者数据能够写入。
- 在`BIO` 模式中，服务器会为每个客户端请求建立一个线程，由该线程单独负责处理一个客户请求，这种模式虽然简单方便，但由于服务器为每个客户端的连接都采用一个线程去处理，使得资源占用非常大。因此，当连接数量达到上限时，如果再有用户请求连接，直接会导致资源瓶颈，严重的可能会直接导致服务器崩溃。
- 大多数情况下为了避免上述问题，都采用了线程池模型。也就是创建一个固定大小的线程池，如果有客户端请求，就从线程池中取一个空闲线程来处理，当客户端处理完操作之后，就会释放对线程的占用。因此这样就避免为每一个客户端都要创建线程带来的资源浪费，使得线程可以重用。但线程池也有它的弊端，如果连接大多是长连接，可能会导致在一段时间内，线程池中的线程都被占用，那么当再有客户端请求连接时，由于没有空闲线程来处理，就会导致客户端连接失败。

### 非阻塞IO(NIO)

- 基于`BIO` 的各种弊端，在`JDK1.4` 开始出现了高性能`IO` 设计模式非阻塞`IO`（`NIO`）。
  1. `NIO` 采用非阻塞模式，基于`Reactor` 模式的工作方式，`I/O` 调用不会被阻塞，它的实现过程是：会先对每个客户端注册感兴趣的事件，然后有一个线程专门去轮询每个客户端是否有事件发生，当有事件发生时，便顺序处理每个事件，当所有事件处理
     完之后，便再转去继续轮询。
  2. `NIO` 中实现非阻塞`I/O` 的核心对象就是`Selector`，`Selector` 就是注册各种`I/O`事件地方，而且当我们感兴趣的事件发生时，就是这个对象告诉我们所发生的事件，
  3. `NIO` 的最重要的地方是当一个连接创建后，不需要对应一个线程，这个连接会被注册到多路复用器上面，一个选择器线程可以同时处理成千上万个连接，系统不必创建大量的线程，也不必维护这些线程，从而大大减小了系统的开销。

### 异步非阻塞IO(AIO)

1. `AIO` 也就是`NIO 2`，在`Java 7` 中引入了`NIO` 的改进版`NIO 2`,它是异步非阻塞的`IO` 模型。异步`IO` 是基于事件和回调机制实现的，也就是说`AIO` 模式不需要`selector` 操作，而是是事件驱动形式，也就是当客户端发送数据之后，会主动通知服务器，接着服务器再进行读写操作。
2. `Java` 的`AIO API` 其实就是`Proactor` 模式的应用，和`Reactor` 模式类似。`Reactor` 和`Proactor` 模式的主要区别就是真正的读取和写入操作是有谁来完成的，`Reactor` 中需要应用程序自己读取或者写入数据，而`Proactor` 模式中，应用程序不需要进行实际的读写过程，它只需要从缓存区读取或者写入即可，操作系统会读取缓存区或者写入缓存区到真正的`IO` 设备。

## NIO 概述

`Java NIO` 由以下几个核心部分组成：

- `Channels`
- `Buffers`
- `Selectors`

虽然`Java NIO` 中除此之外还有很多类和组件，但`Channel，Buffer` 和`Selector` 构成了核心的`API`。其它组件，如`Pipe` 和`FileLock`，只不过是与三个核心组件共同使用的工具类。

### Channel

- 首先说一下`Channel`，可以翻译成“通道”。`Channel `和`IO` 中的`Stream`(流)是差不多一个等级的。只不过`Stream` 是单向的，譬如：`InputStream`,` OutputStream`.而`Channel` 是双向的，既可以用来进行读操作，又可以用来进行写操作。
- `NIO` 中的`Channel` 的主要实现有：`FileChannel`、`DatagramChannel`、`SocketChannel` 和`ServerSocketChannel`，这里看名字就可以猜出个所以然来：分别可以对应文件`IO`、`UDP` 和`TCP`（`Server` 和`Client`）。

### Buffer

- `NIO` 中的关键`Buffer` 实现有：`ByteBuffer`, `CharBuffer`, `DoubleBuffer`, `FloatBuffer`,`IntBuffer`, `LongBuffer`, `ShortBuffer`，分别对应基本数据类型: `byte`, `char`, `double`,`float`, `int`, `long`, `short`。

### Selector

- `Selector` 运行单线程处理多个`Channel`，如果你的应用打开了多个通道，但每个连接的流量都很低，使用Selector 就会很方便。例如在一个聊天服务器中。要使用`Selector`, 得向`Selector` 注册`Channel`，然后调用它的`select()`方法。这个方法会一直阻塞到某个注册的通道有事件就绪。一旦这个方法返回，线程就可以处理这些事件，事件的例子有如新的连接进来、数据接收等。

### Channel Buffer Selector 三者关系

1. 一个`Channel` 就像一个流，只是`Channel` 是双向的，`Channel` 读数据到Buffer，`Buffer` 写数据到`Channel`。
2. 一个`selector` 允许一个线程处理多个`channel`

## Java NIO（Channel）

### Channel 概述

`Java NIO` 的通道类似流，但又有些不同：

- 既可以从通道中读取数据，又可以写数据到通道。但流的读写通常是单向的。
- 通道可以异步地读写。
- 通道中的数据总是要先读到一个`Buffer`，或者总是要从一个`Buffer` 中写入。

### Channel 实现

- 下面是`Java NIO` 中最重要的`Channel` 的实现：
  - `FileChannel`:`FileChannel `从文件中读写数据。
  - `DatagramChannel`:`DatagramChannel` 能通过`UDP` 读写网络中的数据。
  - `SocketChannel`:`SocketChannel` 能通过`TCP` 读写网络中的数据。
  - `ServerSocketChannel`:`ServerSocketChannel` 可以监听新进来的`TCP` 连接，像`Web` 服务器那样。对每一个新进来的连接都会创建一个`SocketChannel`。

### FileChannel 介绍和示例

- `FileChannel` 类可以实现常用的`read`，`write` 以及`scatter/gather` 操作，同时它也提供了很多专用于文件的新方法。这些方法中的许多都是我们所熟悉的文件操作。

  | 方法                            | 描述                                         |
  | ------------------------------- | -------------------------------------------- |
  | `int read(ByteBuffer dst)`      | 从`Channel`中读取数据到`ByteBuffer`          |
  | `long read(ByteBuffer[] dsts)`  | 将`Channel`中的数据“分散”到`ByteBuffer[]`    |
  | `int write(ByteBuffer src)`     | 将`ByteBuffer`中的数据写入到`Channel`        |
  | `long write(ByteBuffer[] srcs)` | 将`ByteBuffer[]`中的数据聚集到"Channel"      |
  | `long position()`               | 返回此通道的文件位置                         |
  | `FileChannel position(long p)`  | 设置此通道的文件位置                         |
  | `long size()`                   | 返回此通道的文件的当前大小                   |
  | `FileChannel truncate(long s)`  | 将此通道的文件截取为给定大小                 |
  | `void force(boolean metaData)`  | 强制将所有对此通道的文件更新写入到存储设备中 |

  ~~~java
  public class FileChannelDemo {
  	public static void main(String[] args) throws IOException {
  		RandomAccessFile aFile = new
  		RandomAccessFile("d:\\atguigu\\01.txt", "rw");
  		FileChannel inChannel = aFile.getChannel();
  		ByteBuffer buf = ByteBuffer.allocate(48);
  		int bytesRead = inChannel.read(buf);
  		while (bytesRead != -1) {
  			System.out.println("读取： " + bytesRead);
  			buf.flip();
  			while (buf.hasRemaining()) {
  				System.out.print((char) buf.get());
  			}
  			buf.clear();
  			bytesRead = inChannel.read(buf);
  		}
  		aFile.close();
  		System.out.println("操作结束");
  	}
  }
  ~~~

  - `Buffer` 通常的操作
    - 将数据写入缓冲区\
    - 调用`buffer.flip()` 反转读写模式
    - 从缓冲区读取数据
    - 调用`buffer.clear()` 或`buffer.compact()` 清除缓冲区内容

### FileChannel 操作详解

#### 打开FileChannel

- 在使用`FileChannel` 之前，必须先打开它。但是，我们无法直接打开一个`FileChannel`，需要通过使用一个`InputStream`、`OutputStream` 或 `RandomAccessFile` 来获取一个 `FileChannel` 实例。下面是通过 `RandomAccessFile`打开`FileChannel` 的示例：

  ~~~java
  RandomAccessFile aFile = new RandomAccessFile("d:\\atguigu\\01.txt","rw");
  FileChannel inChannel = aFile.getChannel();
  ~~~

#### 从FileChannel 读取数据

- 调用多个`read()`方法之一从`FileChannel` 中读取数据。如：

  ~~~java
  ByteBuffer buf = ByteBuffer.allocate(48);
  int bytesRead = inChannel.read(buf);
  ~~~

  - 首先，分配一个`Buffer`。从`FileChannel` 中读取的数据将被读到`Buffer` 中。然后，调用`FileChannel.read()`方法。该方法将数据从`FileChannel` 读取到`Buffer` 中。`read()`方法返回的`int` 值表示了有多少字节被读到了`Buffer` 中。如果返回-1，表示到了文件末尾。

#### 向FileChannel 写数据

- 使用`FileChannel.write()`方法向`FileChannel` 写数据，该方法的参数是一个`Buffer`。

  ~~~java
  public class FileChannelDemo {
  	public static void main(String[] args) throws IOException {
  		RandomAccessFile aFile = new RandomAccessFile("d:\\atguigu\\01.txt", "rw");
  		FileChannel inChannel = aFile.getChannel();
  		String newData = "New String to write to file..." + System.currentTimeMillis();
  		ByteBuffer buf1 = ByteBuffer.allocate(48);
  		buf1.clear();
  		buf1.put(newData.getBytes());
  		buf1.flip();
  		while(buf1.hasRemaining()) {
  			inChannel.write(buf1);
  		}
  		inChannel.close();
  	}
  }
  ~~~

  - 注意`FileChannel.write()`是在`while` 循环中调用的。因为无法保证`write()`方法一次能向`FileChannel` 写入多少字节，因此需要重复调用`write()`方法，直到`Buffer` 中已经没有尚未写入通道的字节。

#### 关闭FileChannel

- 用完`FileChannel` 后必须将其关闭。如：

  ~~~java
  inChannel.close();
  ~~~

#### FileChannel 的position 方法

- 有时可能需要在`FileChannel` 的某个特定位置进行数据的读/写操作。可以通过调用`position()`方法获取`FileChannel` 的当前位置。也可以通过调用`position(long pos)`方法设置`FileChannel` 的当前位置。
- 这里有两个例子:
  - `long pos = channel.position();`
  - `channel.position(pos +123);`
  - 如果将位置设置在文件结束符之后，然后试图从文件通道中读取数据，读方法将返回-1 （文件结束标志）。
  - 如果将位置设置在文件结束符之后，然后向通道中写数据，文件将撑大到当前位置并写入数据。这可能导致“文件空洞”，磁盘上物理文件中写入的数据间有空隙。

#### FileChannel 的size 方法

- `FileChannel` 实例的`size()`方法将返回该实例所关联文件的大小。如:

  ~~~
  long fileSize = channel.size();
  ~~~

#### FileChannel 的truncate 方法

- 可以使用`FileChannel.truncate()`方法截取一个文件。截取文件时，文件将中指定长度后面的部分将被删除。如：

  ~~~
  channel.truncate(1024);
  ~~~

- 这个例子截取文件的前1024 个字节。

#### FileChannel 的force 方法

- `FileChannel.force()`方法将通道里尚未写入磁盘的数据强制写到磁盘上。出于性能方面的考虑，操作系统会将数据缓存在内存中，所以无法保证写入到`FileChannel` 里的数据一定会即时写到磁盘上。要保证这一点，需要调用`force()`方法。
- `force()`方法有一个`boolean `类型的参数，指明是否同时将文件元数据（权限信息等）写到磁盘上。

#### FileChannel 的transferTo 和transferFrom 方法

- 通道之间的数据传输：
- 如果两个通道中有一个是`FileChannel`，那你可以直接将数据从一个`channel` 传输到另外一个`channel`。

##### transferFrom()方法

- `FileChannel` 的`transferFrom()`方法可以将数据从源通道传输到`FileChannel` 中（译者注：这个方法在JDK 文档中的解释为将字节从给定的可读取字节通道传输到此通道的文件中）。下面是一个`FileChannel` 完成文件间的复制的例子

  ~~~java
  public class FileChannelWrite {
  	public static void main(String args[]) throws Exception {
  		RandomAccessFile aFile = new RandomAccessFile("d:\\atguigu\\01.txt", "rw");
  		FileChannel fromChannel = aFile.getChannel();
  		RandomAccessFile bFile = new
  		RandomAccessFile("d:\\atguigu\\02.txt", "rw");
  		FileChannel toChannel = bFile.getChannel();
  		long position = 0;
  		long count = fromChannel.size();
  		toChannel.transferFrom(fromChannel, position, count);
  		aFile.close();
  		bFile.close();
  		System.out.println("over!");
  	}
  }
  ~~~

  - 方法的输入参数`position` 表示从`position` 处开始向目标文件写入数据，`count` 表示最多传输的字节数。如果源通道的剩余空间小于`count` 个字节，则所传输的字节数要小于请求的字节数。此外要注意，在`SoketChannel` 的实现中，`SocketChannel` 只会传输此刻准备好的数据（可能不足`count` 字节）。因此，`SocketChannel` 可能不会将请求的所有数据(`count` 个字节)全部传输到`FileChannel` 中。

##### transferTo()方法

- `transferTo()`方法将数据从`FileChannel` 传输到其他的`channel` 中。

- 下面是一个`transferTo()`方法的例子：

  ~~~java
  public class FileChannelDemo {
  	public static void main(String args[]) throws Exception {
  		RandomAccessFile aFile = new RandomAccessFile("d:\\atguigu\\02.txt", "rw");
  		FileChannel fromChannel = aFile.getChannel();
  		RandomAccessFile bFile = new RandomAccessFile("d:\\atguigu\\03.txt", "rw");
  		FileChannel toChannel = bFile.getChannel();
  		long position = 0;
  		long count = fromChannel.size();
  		fromChannel.transferTo(position, count, toChannel);
  		aFile.close();
  		bFile.close();
  		System.out.println("over!");
  	}
  }
  ~~~

### Scatter/Gather

- `Java NIO` 开始支持`scatter`/`gather`，`scatter`/`gather` 用于描述从`Channel` 中读取或者写入到`Channel` 的操作。
- 分散（`scatter`）从`Channel` 中读取是指在读操作时将读取的数据写入多个`buffer` 中。因此，`Channel` 将从`Channel` 中读取的数据“分散（`scatter`）”到多个Buffer 中。聚集（`gather`）写入`Channel` 是指在写操作时将多个`buffer` 的数据写入同一个`Channel`，因此，`Channel` 将多个`Buffer` 中的数据“聚集（`gather`）”后发送到`Channel`。
- `scatter` / `gather` 经常用于需要将传输的数据分开处理的场合，例如传输一个由消息头和消息体组成的消息，你可能会将消息体和消息头分散到不同的`buffer` 中，这样你可以方便的处理消息头和消息体。

#### Scattering Reads

- `Scattering Reads` 是指数据从一个`channel` 读取到多个`buffer` 中

  ~~~java
  ByteBuffer header = ByteBuffer.allocate(128);
  ByteBuffer body = ByteBuffer.allocate(1024);
  ByteBuffer[] bufferArray = { header, body };
  channel.read(bufferArray);
  ~~~

- 注意`buffer` 首先被插入到数组，然后再将数组作为`channel.read()` 的输入参数。`read()`方法按照`buffer` 在数组中的顺序将从`channel` 中读取的数据写入到`buffer`，当一个`buffer` 被写满后，`channel` 紧接着向另一个`buffer` 中写。

- `Scattering Reads` 在移动下一个`buffer` 前，必须填满当前的`buffer`，这也意味着它不适用于动态消息(译者注：消息大小不固定)。换句话说，如果存在消息头和消息体，消息头必须完成填充（例如`128byte`），`Scattering Reads` 才能正常工作。

#### Gathering Writes

- `Gathering Writes` 是指数据从多个`buffer` 写入到同一个`channel`。

  ~~~java
  ByteBuffer header = ByteBuffer.allocate(128);
  ByteBuffer body = ByteBuffer.allocate(1024);
  //write data into buffers
  ByteBuffer[] bufferArray = { header, body };
  channel.write(bufferArray);
  ~~~

- `buffers` 数组是`write()`方法的入参，`write()`方法会按照`buffer` 在数组中的顺序，将数据写入到`channel`，注意只有`position` 和`limit` 之间的数据才会被写入。因此，如果一个`buffer` 的容量为`128byte`，但是仅仅包含`58byte` 的数据，那么这`58byte` 的数
  据将被写入到`channel` 中。因此与`Scattering Reads` 相反，`Gathering Writes` 能较好的处理动态消息。
