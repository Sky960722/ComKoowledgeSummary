# NIO

- 以下概念来源于Java核心技术和Blog：https://jenkov.com/tutorials/java-io/outputstream.html

## 概念

- JavaNio相对于IO是一个全新，提供另一种io模型的API。
- NIO：No Blocking IO。不会阻塞的IO
- IO的使用过程中，处理的是streams和character streams。NIO则处理的是channels和buffers
- 最重要的概念：selector。一个selector可以对channels监控多个事情。

## Channels 和 Buffers

- 在NIO中，channels能从buffers中读取数据并写入，也能对buffer读取。如下图

  <img src="C:\Users\Administrator\Desktop\ComKoowledgeSummary\4.java基础\overview-channels-buffers.png" style="zoom:75%;" />

- 下面则是多个不同的 channel 和 buffer

  | FileChannel         | ByteBuffer   |
  | ------------------- | ------------ |
  | DatagramChannel     | CharBuffer   |
  | SocketChannel       | DoubleBuffer |
  | ServerSocketChannel | FloatBuffer  |
  |                     | IntBuffer    |
  |                     | LongBuffer   |
  |                     | ShortBuffer  |

## Selectors

### 概念：

- 一个selector被允许可以应对多个channel。如果你的程序需要建立多个连接，每个连接需要耗费一定的流量。那这个是极为有效的。为了使用selecot，你需要将不同的channel注册给他。一旦有事件返回，就挑选对应的channel进行处理。

  ![](C:\Users\Administrator\Desktop\ComKoowledgeSummary\4.java基础\overview-selectors.png)

### 方法

#### Creating a Selector

~~~java
Selector selector = Selector.open();
~~~

#### Registering Channels with the Selector

~~~java
channel.configureBlocking(false);

SelectionKey key = channel.register(selector, SelectionKey.OP_READ);

不同的模式
SelectionKey.OP_CONNECT
SelectionKey.OP_ACCEPT
SelectionKey.OP_READ
SelectionKey.OP_WRITE
~~~

#### Interest Set

~~~java
int interestSet = selectionKey.interestOps();

boolean isInterestedInAccept  = SelectionKey.OP_ACCEPT  == (interests & SelectionKey.OP_ACCEPT);
boolean isInterestedInConnect = SelectionKey.OP_CONNECT == (interests & SelectionKey.OP_CONNECT);
boolean isInterestedInRead    = SelectionKey.OP_READ    == (interests & SelectionKey.OP_READ);
boolean isInterestedInWrite   = SelectionKey.OP_WRITE   == (interests & SelectionKey.OP_WRITE);
~~~

#### Ready Set

~~~java
int readySet = selectionKey.readyOps();

selectionKey.isAcceptable();
selectionKey.isConnectable();
selectionKey.isReadable();
selectionKey.isWritable();
~~~

#### Channel + Selector

~~~java
Channel  channel  = selectionKey.channel();

Selector selector = selectionKey.selector();  
~~~

#### Selecting Channels via a Selector

- 通过Selector访问不通的channels。
- int select()：阻塞，直到合适的channel能被运行
- int select(long timeout)：阻塞，直到时间到了
- int selectNow：不阻塞，立马返回准别好的channel

#### 完整的例子

~~~java
Selector selector = Selector.open();

channel.configureBlocking(false);

SelectionKey key = channel.register(selector, SelectionKey.OP_READ);


while(true) {

  int readyChannels = selector.selectNow();

  if(readyChannels == 0) continue;


  Set<SelectionKey> selectedKeys = selector.selectedKeys();

  Iterator<SelectionKey> keyIterator = selectedKeys.iterator();

  while(keyIterator.hasNext()) {

    SelectionKey key = keyIterator.next();

    if(key.isAcceptable()) {
        // a connection was accepted by a ServerSocketChannel.

    } else if (key.isConnectable()) {
        // a connection was established with a remote server.

    } else if (key.isReadable()) {
        // a channel is ready for reading

    } else if (key.isWritable()) {
        // a channel is ready for writing
    }

    keyIterator.remove();
  }
}
~~~

