---
title: NIO缓冲区的一点理解
date: 2019-05-01 15:18:40
categories:
	- Java
tags:
	- Java
---
关于pageCache, send_file, 磁盘读写的复习
<!-- more -->

Kafka号称可以再一秒内，实现百万消息的写入，Kafka权威指南中提到Kafka的broker使用了NIO里面mmp方式的页缓存。
## 传统的文件读取写入，
都需要经历四次数据拷贝， 两次系统调用(read, write)
1. 操作系统从磁盘读取数据（通常是DMA方式）到内核空间的 pagecache
2. 应用程序读取内核空间的数据到用户空间的缓冲区
3. 应用程序将数据(用户空间的缓冲区)写回内核空间到套接字缓冲区(内核空间)
4. 操作系统将数据从套接字缓冲区(内核空间)复制到通过网络发送的NIC缓冲区(这里不太理解， 对应的网卡缓冲区)

### 缓冲区操作
进程执行 I/O 操作,归结起来,也就是向操作系统发出请求,让它要么把缓冲区里的数据排干 (写),要么用数据把缓冲区填满(读)。进程使用这一机制处理所有数据进出操作。操作系统内部处理这一任务的机制,其复杂程度可能超乎想像,但就概念而言,却非常直白易懂
数据从外部磁盘向运行中的进程的内存区域移动的过程。进程使用read()系统调用,要求其缓冲区被填满。内核随即向磁盘控制硬件发出命令,要求其从磁盘读取数据。磁盘 控制器把数据直接写入内核内存缓冲区,这一步通过 DMA 完成,无需主 CPU 协助。一旦磁盘控 制器把缓冲区装满,内核即把数据从内核空间的临时缓冲区拷贝到进程执行read()调用时指定的缓冲区。  
这里忽略了很多的细节问题，大部分哎计算机系统结构中都讲到的， 用户空间就是常规的进程(对应JVM进程), 驻守在用户空间, 当进程接收到了IO操作时候， 他会执行一个open(), or read(), or write()等系统调用，等待内核把数据通过DMA的方式拷贝到内核缓冲， 在拷贝到应用进程缓冲。 
	为什么，要多执行一次内核空间拷贝到用空间，因为一般的硬件是无法直接访问用户空间的，  

## 零拷贝
使用 零拷贝技术 可以有效的减少一次数据拷贝的过程
具体到fileChannel里面就是 
```java
public abstract long transferTo(long position, long count,
        WritableByteChannel target)
throws IOException;
```
和
```java
public abstract long transferFrom(ReadableByteChannel src,
      long position, long count)
throws IOException;

```
这两个FileChannel里面的API可以提供Linux里面sendFile系统调用的功能， 将一个通道的数据直接 传输到 Socket通道中。
这里形参，是WritableByteChannel， 和 ReadableByteChannel通道，而SocketChannel实现了可读和可写的抽象接口。  

直接的通道传输不会更新与某个 FileChannel 关联的 position 值。请求的数据传输将从position参数指定的位置开始,传输的字节数不超过 count 参数的值。实际传输的字节数会由 方法返回,可能少于请求的字节数。  
对于传输数据来源是一个文件的transferTo方法,如果position+count的值大于文件的size值,传输会在文件尾的位置终止。假如传输的目的地是一个非阻塞模式的 socket 通道,那么 当发送队列(send queue)满了之后传输就可能终止,并且如果输出队列(output queue)已满的话 可能不会发送任何数据。类似地,对于 transferFrom()方法:如果来源 src 是另外一个 FileChannel 并且已经到达文件尾,那么传输将提早终止;如果来源 src 是一个非阻塞 socket 通道,只有当前处于队列中的数据才会被传输(可能没有数据)。由于网络数据传输的非确定性,阻塞模式的 socket 也可能会执行部分传输.....   
```java
private static void testCatFile(String file) throws Exception {
    WritableByteChannel target = Channels.newChannel(System.out);
    // Sysout.out是一个rintStream extends FilterOutputStream
    FileChannel channel = new FileInputStream(file).getChannel();
    channel.transferTo(0, channel.size(), target);
}
```
