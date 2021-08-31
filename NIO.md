# NIO
* Java NIO 系统的核心在于：通道和缓冲区
* 通道表示打开到 IO 设备（文件、套接字等）的连接，若需要使用 NIO 系统，需要获取用于连接 IO 设备的通道以及用于容纳数据的缓冲区
* 总结：Channel 负责传输，Buffer 负责存储
* 参考：[尚硅谷Java NIO线程教程(180分钟掌握New IO/IO API)](https://www.bilibili.com/video/BV14W411u7ro?from=search&seid=16718589256289697611)


## 1. 缓冲区
### (1). 获取缓冲区
* 缓冲区就是数组，用于存储不同类型的数据
* 根据数据类型不同（boolean 除外），提供了相应类型的缓冲区：ByteBuffer、IntBuffer、、、、、、
* 管理方式几乎一致，通过 allocate() 获取

### (2). 存取数据
* put()：存
* get()：取

### (3). 缓冲区中的四个核心属性
* capacity：容量，表示缓冲区中最大存储数据容量
* limit：界限，表示缓冲区中可以操作数据大小
* position：位置，表示缓冲区中正在操作数据位置
* mark：标记，表示记录当前 position 的位置，可通过 reset() 恢复到 mark 的位置
* **0 <= mark <= position <= limit <= capacity**

### (4). 直接缓冲区与非直接缓冲区
* 非直接缓冲区：通过 allocate() 方法分配缓冲区，将缓冲区建立在 JVM 内存中（直接内存）
* 直接缓冲区：通过 allocateDirect() 方法分配，将缓冲区建立在物理内存中，可以提高效率
![直接缓冲区与非直接缓冲区](https://cdn.jsdelivr.net/gh/qtxsnwwb/image-hosting@master/20210823/直接缓冲区与非直接缓冲区.457x02zb85g0.png)


## 2. 通道
> Channel 本身不能直接访问数据，只能与Buffer交互

### (1). 通道的主要实现类
```java
java.nio.channels.Channel 接口：
	FileChannel、SocketChannel、ServerSocketChannel、DatagramChannel
```
> FileChannel：从文件中读取数据
> DatagramChannel：从 UDP 网络中读 / 写数据
> SocketChannel：从 TCP 网络中读 / 写数据
> ServerSocketChannel：允许你监听来自 TCP 的连接，就像服务器一样。每个连接都会有一个 SocketChannel 产生

### (2). 获取通道
* getChannel()
    * 本地IO：FileInputStream / FileOutputStream
    * 网络IO：Socket、ServerSocket、DatagramSocket
* JDK 1.7 中 NIO 2 针对各个通道提供了静态方法 open()
* JDK 1.7 中 NIO 2 的 Files 工具类的 newByteChannel()

### (3). 数据传输
* 利用通道 + 缓冲区完成
* 使用直接缓冲区完成（内存映射文件）

### (4). 通道之间的数据传输
* transferFrom()、transferTo()
* 也是以直接缓冲区方式完成

### (5). 分散与聚集
* 分散读取：将通道中的数据分散到多个缓冲区（按缓冲区顺序填入）
* 聚集写入：将多个缓冲区中的数据聚集到通道中（按缓冲区顺序，写入 position 和 limit 之间的数据到 Channel）

### (6). 字符集：Charset
* 编码：字符串 ——> 字节数组
* 解码：字节数组 ——> 字符串



## 3. NIO 阻塞与非阻塞
* 传统的 IO 流都是**阻塞式**的。也就是说，当一个线程调用 read() 或 write() 时，**该线程被阻塞**，直到有一些数据被读取或写入，该线程在此期间不能其他任务。因此，在完成网络通信进行 IO 操作时，由于线程会阻塞，所以**服务器必须为每个客户端都提供一个独立的线程进行处理**，当服务器需要大量客户端时，性能急剧下降。
* Java NIO 是**非阻塞式**的。当线程从某通道进行读写数据时，若没有数据可用时，该线程可以执行其他任务。**线程通常将非阻塞 IO 的空闲时间用于在其他通道上执行 IO 操作**。所以**单独的线程可以管理多个输入和输出通道（借助选择器实现）**。因此，NIO 可以让服务器端使用一个或有限几个线程来同时处理连接到服务器端的所有客户端。



## 4. NIO 选择器（Selector）
* 也称多路复用器，用于检查一个或多个 NIO Channel 的状态是否处于可读、可写。如此可以实现单线程管理多个 channels，也就是可管理多个网络链接。待 channel 处于可读 / 可写状态，再传输给服务器端。
* 使用 Selector 的好处是使用更少的线程就可以处理通道，避免了线程上下文切换带来的开销
* 可监听的事件类型：
    1. SelectionKey.OP_READ：监听读
    2. SelectionKey.OP_WRITE：监听写
    3. SelectionKey.OP_CONNECT：监听连接
    4. SelectionKey.OP_ACCEPT：监听接收
