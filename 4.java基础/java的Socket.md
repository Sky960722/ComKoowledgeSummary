# JAVA的Socket

## Socket

1. Socket--套接字，负责启动该程序内部和外部之间的通信

2. ~~~java
   java.net.Socket
   Socket(String host,int port) //构建一个套接字，用来连接给定的主机和端口
   InputStream getInputStream() 
   OutPutStream getOutputStream() //获取可以从套接字中读取数据的流，以及可以向套接字写出数据的流
   ~~~

3. 套接字超时

   ~~~java
   java.net.Socket
   Socker() //创建一个还未被连接的套接字
   void connect(SocketAddress address) //将该套接字连接到给定的地址
   void connect(SocketAddress address,int timeoutInMilliseconds) //将套接字连接到给定的地址。如果在给定的时间内没有响应，则返回
   void setSoTimeout(int timeoutInMilliseconds) //设置该套接字上读请求的阻塞时间
   boolean isConnected() //如果该套接字已被连接，则返回true
   boolean isClosed() //如果套接字已经被关闭，则返回true
   ~~~

## InetAddress

1. 一个主机地址由4个字节组成（在IPv6中是16个字节）

2. ~~~java
   java.net.InetAddress
   static InetAddress getByName(String host)
   static InetAddress getAllByName(String host) //为给定的主机名创建一个InetAddress对象，或者一个包含了该主机名所对应的所有因特网地址的数组
   static InetAddress getLocalHost() //为本地主机创建一个InetAddress对象
   byte[] getAddress() //返回一个包含数字型地址的字节数组
   String getHostAddress() //返回一个由十进制数组成的字符串
   String getHostName() //返回主机名
   ~~~

## ServerSocket

1. 服务端Socket，一旦启动了服务器程序，便会等待某个客户端连接到它的端口。

2. ~~~java
   java.net.ServerSocket
   ServerSocker(int port) //创建一个监听端口的服务器套接字
   Socket accept() //等待连接
   void close() //关闭服务器套接字
   ~~~

3. 半关闭：套接字连接的一端可以终止其输出，同时仍旧可以接受来自另一端的数据

4. ~~~java
   java.net.Scoket
   void shutdownOutput() //将输出流设为 流结束
   void shutdownInput() //将输入流设为 流结束
   boolean isOutputShutdown() //如果输出已被关闭，则返回true
   boolean isInputShutdown() //如果输入已被关闭，则返回true
   ~~~

5. 可中断套接字：为了中断套接字操作，读取或写出数据时，线程不在阻塞

6. ~~~java
   java.net.InetSocketAddress
   InetSocketAddress(String hostname,int port) //用给定的主机和端口参数创建一个地址对象
   boolean isUnresolved() //如果不能解析该地址对象，则返回true
   
   java.nio.channels.SocketChannel
   static SocketChannel open(SocketAddress address) //打开一个套接字通道，并将其连接到远程地址
   
   java.nio.channels.Channels
   static InputStream newInputStream(ReadableByteChannel channel) //创建一个输入流，用以从指定的通道读取数据
   static OutputStream newOutputStream(WritableByteChannel channl) //创建一个输出流，用以向指定的通道写入数据
   ~~~
