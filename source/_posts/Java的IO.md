---
title: Java的 I/O机制
date: 2019-03-10 23:15:41
tags: Java
---
Java 的 I/O 大概可以分成以下几类：
*   磁盘操作：File
*   字节操作：InputStream 和 OutputStream
*   字符操作：Reader 和 Writer
*   对象操作：Serializable：序列化就是将一个对象转换成字节序列，方便存储和传输。
*   网络操作：Socket
*   新的输入/输出：NIO

Java I/O 使用了装饰者模式来实现
<!--more-->

[深入分析 Java I/O 的工作机制](https://www.ibm.com/developerworks/cn/java/j-lo-javaio/index.html)
[Linux 网络 I/O 模型简介（图文）](https://blog.csdn.net/anxpp/article/details/51503329)

# UNIX网络编程对I/O模型的分类

Linux 的内核将所有外部设备都看做一个文件来操作（一切皆文件），对一个文件的读写操作会调用内核提供的系统命令，返回一个file descriptor（fd，文件描述符）。而对一个socket的读写也会有响应的描述符，称为socket fd（socket文件描述符），描述符就是一个数字，指向内核中的一个结构体（文件路径，数据区等一些属性）。

根据UNIX网络编程对I/O模型的分类，UNIX提供了5种I/O模型。

进程是无法直接操作I/O设备的，其必须通过系统调用请求内核来协助完成I/O动作，而内核会为每个I/O设备维护一个buffer。

![image](http://490.github.io/images/20190314_213445.png)

整个请求过程为： 用户进程发起请求，内核接受到请求后，从I/O设备中获取数据到buffer中，再将buffer中的数据copy到用户进程的地址空间，该用户进程获取到数据后再响应客户端。

## 阻塞I/O模型

最常用的I/O模型，默认情况下，所有文件操作都是阻塞的。

比如I/O模型下的套接字接口：在进程空间中调用recvfrom，其系统调用直到数据包到达（比如，还没有收到一个完整的UDP包）且被复制到应用进程的缓冲区中或者发生错误时才返回，而在用户进程这边，整个进程会被阻塞。当内核一直等到数据准备好了，它就会将数据从内核中拷贝到用户内存，然后内核返回结果，用户进程才解除block的状态，重新运行起来。

进程在调用recvfrom开始到它返回的整段时间内都是被阻塞的，所以叫阻塞I/O模型。所以，blocking IO的特点就是在IO执行的两个阶段都被block了。

![image](http://490.github.io/images/20190315_075932.png)

## 非阻塞I/O模型

当用户进程调用recvfrom时，系统不会阻塞用户进程，而是立刻返回一个ewouldblock错误，从用户进程角度讲 ，并不需要等待，而是马上就得到了一个结果。用户进程判断标志是ewouldblock时，就知道数据还没准备好，于是它就可以去做其他的事了，于是它可以再次发送recvfrom，一旦内核中的数据准备好了。并且又再次收到了用户进程的system call，那么它马上就将数据拷贝到了用户内存，然后返回。

当一个应用程序在一个循环里对一个非阻塞调用recvfrom，我们称为轮询。应用程序不断轮询内核，看看是否已经准备好了某些操作。这通常是浪费CPU时间，但这种模式偶尔会遇到。

![image](http://490.github.io/images/20190315_075950.png)

## I/O复用模型

单个process就可以同时处理多个网络连接的IO。它的基本原理就是select/epoll这个function会不断的轮询所负责的所有socket，当某个socket有数据到达了，就通知用户进程。

Linux提供select/poll，进程通过将一个或多个fd（文件描述符）传递给select或poll系统调用，阻塞在select操作上，这样，select/poll可以帮我们侦测多个fd是否处于就绪状态。

select/poll是顺序扫描fd是否就绪，而且支持的fd数量有限，因此它的使用受到了一些制约。

Linux还提供一个epoll系统调用，epoll使用基于事件驱动方式代替顺序扫描，因此性能更高。当有fd就绪时，立即回调函数rollback。

![image](http://490.github.io/images/20190315_080655.png)

I/O多路复用就是通过一种机制，一个进程可以监视多个描述符，一旦某个描述符就绪（一般是读就绪或者写就绪），能够通知程序进行相应的读写操作。但select，pselect，poll，epoll本质上都是同步I/O，因为他们都需要在读写事件就绪后自己负责进行读写，也就是说这个读写过程是阻塞的，而异步I/O则无需自己负责进行读写，异步I/O的实现会负责把数据从内核拷贝到用户空间。

与多进程和多线程技术相比，I/O多路复用技术的最大优势是系统开销小，系统不必创建进程/线程，也不必维护这些进程/线程，从而大大减小了系统的开销。

[IO多路复用之select、poll、epoll详解](https://www.cnblogs.com/jeakeven/p/5435916.html)

当用户进程调用了select，那么整个进程会被block，而同时，内核会“监视”所有select负责的socket，当任何一个socket中的数据准备好了，select就会返回。这个时候用户进程再调用read操作，将数据从内核拷贝到用户进程。

这个图和blocking IO的图其实并没有太大的不同，事实上，还更差一些。因为这里需要使用两个system call (select 和 recvfrom)，而blocking IO只调用了一个system call (recvfrom)。但是，用select的优势在于它可以同时处理多个connection。（多说一句。所以，如果处理的连接数不是很高的话，使用select/epoll的web server不一定比使用multi-threading + blocking IO的web server性能更好，可能延迟还更大。select/epoll的优势并不是对于单个连接能处理得更快，而是在于能处理更多的连接。）

在IO multiplexing Model中，实际中，对于每一个socket，一般都设置成为non-blocking，但是，如上图所示，整个用户的process其实是一直被block的。只不过process是被select这个函数block，而不是被socket IO给block。

## 信号驱动I/O模型

首先开启套接口信号驱动I/O功能，并通过系统调用sigaction执行一个信号处理函数（此系统调用立即返回，进程继续工作，非阻塞）。当数据准备就绪时，就为改进程生成一个SIGIO信号，通过信号回调通知应用程序调用recvfrom来读取数据，并通知主循环函数处理树立。

![image](http://490.github.io/images/20190315_080801.png)

## 异步I/O

告知内核启动某个操作，并让内核在整个操作完成后（包括数据的复制）通知进程。

信号驱动I/O模型通知的是何时可以开始一个I/O操作，异步I/O模型有内核通知I/O操作何时已经完成。

![image](http://490.github.io/images/20190315_080817.png)


# I/O多路复用技术

I/O编程中，需要处理多个客户端接入请求时，可以利用多线程或者I/O多路复用技术进行处理。

正如前面的简介，I/O多路复用技术通过把多个I/O的阻塞复用到同一个select的阻塞上，从而使得系统在单线程的情况下可以同时处理多个客户端请求。

与传统的多线程模型相比，I/O多路复用的最大优势就是系统开销小，系统不需要创建新的额外线程，也不需要维护这些线程的运行，降低了系统的维护工作量，节省了系统资源。

主要的应用场景：

- 服务器需要同时处理多个处于监听状态或多个连接状态的套接字。
- 服务器需要同时处理多种网络协议的套接字。
- 支持I/O多路复用的系统调用主要有select、pselect、poll、epoll。

而当前推荐使用的是epoll，优势如下：

- 支持一个进程打开的socket fd不受限制。
- I/O效率不会随着fd数目的增加而线性下将。
- 使用mmap加速内核与用户空间的消息传递。
- epoll拥有更加简单的API。



![image](http://490.github.io/images/20190315_081717.png)

![image](http://490.github.io/images/20190315_081722.png)


![image](http://490.github.io/images/20190315_081725.png)






# Java NIO

NIO，就是 New IO，是非阻塞 IO。支持面向缓冲区，基于通道的 IO 操作。

传统的 IO 是面向流的，并且是单向的。如果要进行双工的通信，要建立两个流，一个输入流，一个输出流。

NIO 是面向缓冲区的，是双向的，通信双方建立一个通道，然后把数据存放在缓冲区中，然后缓冲区在通道中流动进行数据的传输。打个比方，通道 —— 铁路，缓冲区 —— 火车，数据 —— 人，一堆人想要从 A 地到达 B 地，首先需要把人装上火车，然后火车在铁路上从 A 地跑到 B 地，实现将人从 A 地运到 B 地；同样的道理，一堆数据如果想要从 A 地运到 B 地，需要先把数据装入缓冲区，然后将缓冲区从 A 地跑到 B 地，从而实现将数据从 A 地运到 B 地。



## 缓冲区 Buffer

首先，有这些个 Buffer 种类：

- ByteBuffer
- CharBuffer
- ShortBuffer
- IntBuffer
- LongBuffer
- FloatBuffer
- DoubleBuffer

*简而言之就是，8 个基本类型，除了 reference 都有相应类型的缓冲区在。*

### 缓冲区中的核心属性

```java
// Invariants: mark <= position <= limit <= capacity
private int mark = -1;    // 可以记录当前 position 的位置，然后调用 reset()，可以把 position 恢复到这个位置
private int position = 0; // 缓冲区中正在操作的数据的位置
private int limit;        // 缓冲区中可以操作数据的大小，limit 后的数据是不能进行读写的
private int capacity;     // 缓冲区中最大存储数据的容量，一旦声明不能改变 
```

> **Note：** 抽象类虽然自身不可以实例化，但是其子类覆盖了所有的抽象方法后，是可以实例化的，所以抽象类的构造函数，适用于给其子类对象进行初始化的。
>
> 所以对于 `ByteBuffer.allocate()` 方法，实际上是新建了一个 ByteBuffer 抽象类的子类 HeapByteBuffer 对象，HeapByteBuffer 类实现了 ByteBuffer 的所有抽象方法，所以我们可以通过调用 ByteBuffer 抽象类的构造函数来初始化 HeapByteBuffer 对象。

### 直接缓冲区与非直接缓冲区

|              | 非直接缓冲区              | 直接缓冲区                                         |
| ------------ | ------------------------- | -------------------------------------------------- |
| **分配方法** | `allocate()`              | `allocateDirect()`                                 |
| **特点**     | 将缓冲区建立在 JVM 内存中 | 将缓冲区建立在物理内存 (直接内存) 中，可以提高效率 |

可以通过 `isDirect()` 来判断当前的缓冲区是不是直接缓冲区。

**为什么使用直接缓冲区可以提高效率呢？**

正常情况下，如果你想将一些数据写到物理磁盘上，你需要先将数据从 JVM 内存中 copy 到内核地址空间，因为内核才真正具有控制计算机硬件资源的功能，用户态运行的上层应用程序只能通过系统调用来让内核态的资源来帮助将数据写入硬盘。

![image](http://490.github.io/images/20190317_073241.png)

当要将一个超大的文件写到硬盘上时，这个 copy 的操作就显得很费时了。

![image](http://490.github.io/images/20190317_073249.png)

而使用直接内存，我们可以操作物理磁盘在内存中的内存映射文件，来直接将物理硬盘上的数据加载进内存，或者将内存中的写入硬盘中，而这个映射，其实是一个物理地址到逻辑地址的转换（或者逆过程）。

![image](http://490.github.io/images/20190317_073255.png)

### 常用方法代码示例

```java
public class BufferDemo {
    private ByteBuffer buffer = ByteBuffer.allocate(1024);
    private ByteBuffer directBuffer = ByteBuffer.allocateDirect(1024);

    /**
     * 打印 buffer 的 4 大核心属性
     */
    @Test
    public void test0() {
        System.out.println(buffer.position());
        System.out.println(buffer.limit());
        System.out.println(buffer.capacity());
        System.out.println("---------------------");
    }

    /**
     * Test put, get, flip, rewind,
     */
    @Test
    public void test1() {
        String str = "abcde";
        /* put：向缓冲区中写入数据 */
        buffer.put(str.getBytes());
        System.out.println(buffer.position()); // 5
        System.out.println(buffer.limit());    // 1024
        System.out.println(buffer.capacity()); // 1024
        System.out.println("---------------------");

        /* flip：将缓冲区从写状态切换到读状态 */
        buffer.flip();
        System.out.println(buffer.position()); // 0
        System.out.println(buffer.limit());    // 5
        System.out.println(buffer.capacity()); // 1024
        System.out.println("---------------------");

        /* get：从缓冲区中读数据出来 */
        byte[] tmp = new byte[2];
        buffer.get(tmp);
        System.out.println("get result: " + new String(tmp));
        System.out.println(buffer.position());  // 2
        System.out.println(buffer.limit());     // 5
        System.out.println(buffer.capacity());  // 1024
        System.out.println(buffer.remaining()); // 3，看看还有多少元素可读
        System.out.println("---------------------");

        /* mark：记录当前 position 的位置到 mark 变量 */
        /* reset：令 position = mark */
        buffer.mark();
        buffer.get(tmp);
        System.out.println("get result: " + new String(tmp));
        System.out.println("before reset: position = " + buffer.position()); // 4
        buffer.reset();
        System.out.println("after reset: position = " + buffer.position());  // 2
        System.out.println("---------------------");

        /* rewind：令 position = 0，就是个倒带的操作 */
        buffer.rewind();
        System.out.println(buffer.position());  // 0
        System.out.println(buffer.limit());     // 5
        System.out.println(buffer.capacity());  // 1024
        System.out.println(buffer.remaining()); // 5
        System.out.println("---------------------");
        
        /* compact：从读状态切换回写状态，可以接着上回写的地方继续往下写 */
        buffer.compact();
        System.out.println(buffer.position());  // 5
        System.out.println(buffer.limit());     // 1024
        System.out.println(buffer.capacity());  // 1024
        System.out.println(buffer.remaining()); // 1019
        System.out.println("---------------------");
                
        /* clear，实际上并没有将数据真的清除，只有当新的数据把旧的数据覆盖了，旧的数据才真的被清除 */
        buffer.clear();
        System.out.println(buffer.position());  // 0
        System.out.println(buffer.limit());     // 1024
        System.out.println(buffer.capacity());  // 1024
        System.out.println(buffer.remaining()); // 1024
        System.out.println("---------------------");

        /* isDirect：判断缓冲区是不是直接内存缓冲区 */
        System.out.println(buffer.isDirect());       // false
        System.out.println(directBuffer.isDirect()); // true
        System.out.println("---------------------");
    }
}
```



## 通道 Channel

用于连接两个节点。在 NIO 中负责缓冲区中数据的传输，它本身是不能存储数据的。

### 主要实现类

- `java.nio.channels.Channel` 接口
  - 本地传输：
    - FileChannel
  - 网络传输：
    - SocketChannel
    - ServerSocketChannel
    - DatagramChannel

### 获取通道

Java 中以下类拥有 `getChannel()` 方法，可以通过这个方法获取对应的通道。

- 本地 IO
	- FileInputStream / FileOutputStream
	- RandomAccessFile
- 网络 IO
	- Socket
	- ServerSocket
	- DatagramSocket

除此之外，JDK 1.7 中的 AIO 还提供了以下两种获取通道的方式：

- 为各个通道提供了静态方法 `open()`
- `Files.newByteChannel()`

### 通道之间的数据传输

- `transferFrom()`
- `transferTo()`

### 分散读取与聚集写入

就是以前是用一个缓冲区协助通道传输数据，Scatter 和 Gather 就是用一堆缓冲区（缓冲区数组）协助通道传输数据。

| 分散读取 Scattering Reads：按照缓冲区的顺序，将从 Channel 中读取的数据依次将 Buffer 填满。 | 聚集写入 Gathering Writes：按照缓冲区的顺序，写入position 和 limit 之间的数据到 Channel 。 |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| ![image](http://490.github.io/images/20190317_073314.png)                          | ![image](http://490.github.io/images/20190317_073321.png)                          |

### 代码示例

```java
public class ChannelDemo {
    /**
     * 利用通道完成文件的复制（非直接缓冲区）
     */
    @Test
    public void copyFile1() {
        FileInputStream in = null;
        FileOutputStream out = null;
        FileChannel inChannel = null;
        FileChannel outChannel = null;
        try {
            in = new FileInputStream("1.png");
            out = new FileOutputStream("2.png");
            inChannel = in.getChannel();
            outChannel = out.getChannel();
            ByteBuffer buffer = ByteBuffer.allocate(1024);
            while (inChannel.read(buffer) != -1) {
                buffer.flip();
                outChannel.write(buffer);
                buffer.clear();
            }
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            try {
                if (in != null) {
                    in.close();
                }
                if (out != null) {
                    out.close();
                }
                if (inChannel != null) {
                    inChannel.close();
                }
                if (outChannel != null) {
                    out.close();
                }
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }

    /**
     * 使用直接缓冲区完成文件的复制(内存映射文件)，通过 Channel 的 map 方法获取的缓冲区就是直接缓冲区
     */
    @Test
    public void copyFile2() {
        FileChannel inChannel = null;
        FileChannel outChannel = null;
        try {
            inChannel = FileChannel.open(Paths.get("1.png"), StandardOpenOption.READ);
            outChannel = FileChannel.open(Paths.get("2.png"), StandardOpenOption.WRITE,
                    StandardOpenOption.READ, StandardOpenOption.CREATE);
            MappedByteBuffer inBuffer = inChannel.map(
                FileChannel.MapMode.READ_ONLY, 0, inChannel.size());
            MappedByteBuffer outBuffer = outChannel.map(
                FileChannel.MapMode.READ_WRITE, 0, inChannel.size());

            byte[] dst = new byte[inBuffer.limit()];
            inBuffer.get(dst);
            outBuffer.put(dst);
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            if (inChannel != null) {
                try {
                    inChannel.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
            if (outChannel != null) {
                try {
                    outChannel.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
    }

    /**
     * 通道之间的数据 transfer (直接缓冲区)
     */
    @Test
    public void copyFile3() {
        FileChannel inChannel = null;
        FileChannel outChannel = null;
        try {
            inChannel = FileChannel.open(Paths.get("1.png"), StandardOpenOption.READ);
            outChannel = FileChannel.open(Paths.get("2.png"), StandardOpenOption.WRITE,
                    StandardOpenOption.READ, StandardOpenOption.CREATE);
            // 以下两行效果相等
            inChannel.transferTo(0, inChannel.size(), outChannel);
            outChannel.transferFrom(inChannel, 0, inChannel.size());
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            if (inChannel != null) {
                try {
                    inChannel.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
            if (outChannel != null) {
                try {
                    outChannel.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
    }
}
```



## 阻塞与非阻塞



### NIO 完成网络通信的三个核心

- 通道（Channel）：负责连接。
	- SocketChannel
	- ServerSocketChannel
	- DatagramChannel
	- Pipe.SinkChannel
	- Pipe.SourceChannel
- 缓冲区（Buffer）：负责数据存取。
- 选择器（Selector）：是 SelectableChannel 的多路复用器，用于监控 SelectableChannel 的 IO 状况。



## DatagramChannel





## 管道 Pipe

Java NIO 的 Pipe 可以在两个线程之间建立单项的数据连接，管道有一个 `Pipe.SinkChannel` 和 一个 `Pipe.SourceChannel`，数据会被写到 `Pipe.SinkChannel`，然后从 `Pipe.SourceChannel` 中读取。

### 代码示例

```java
public class NioPipeDemo {
    @Test
    public void test() throws IOException {
        Pipe pipe = Pipe.open();
        Pipe.SinkChannel sinkChannel = pipe.sink();

        String str = "我是管道发来的数据";
        ByteBuffer byteBuffer = ByteBuffer.allocate(1024);
        byteBuffer.put(str.getBytes());
        byteBuffer.flip();
        while (byteBuffer.hasRemaining()) {
            sinkChannel.write(byteBuffer);
        }

        Pipe.SourceChannel sourceChannel = pipe.source();
        byteBuffer.clear();
        int len = sourceChannel.read(byteBuffer);
        System.out.println(new String(byteBuffer.array(), 0, len));

        sinkChannel.close();
        sourceChannel.close();
    }
}
```

















