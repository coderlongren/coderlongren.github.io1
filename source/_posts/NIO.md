---
title: 三言两语之 NIO
date: 2018-3-23 23:02:17
categories:
	- Java
tags:
	- NIO
---
## 老是说，异步非阻塞，同步非阻塞，异步IO，NIO，同学你能分清吗？

NIO的selector是NIO实现非阻塞的关键之处。  
仅用单个线程来处理多个Channels的好处是，只需要更少的线程来处理通道。事实上，可以只用一个线程处理所有的通道。对于操作系统来说，线程之间上下文切换的开销很大，而且每个线程都要占用系统的一些资源（如内存）。因此，使用的线程越少越好。

## Selector的创建
	Selector selector = Selector.open();
## 向selector注册通道channel
	channel.configureBlocking(false);
	SelectionKey key = channel.register(selector,
	Selectionkey.OP_READ);
通道触发了一个事件意思是该事件已经就绪。
这四种事件用SelectionKey的四个常量来表示：
    SelectionKey.OP_CONNECT
    SelectionKey.OP_ACCEPT
    SelectionKey.OP_READ
    SelectionKey.OP_WRITE

当向Selector注册Channel时，register()方法会返回一个SelectionKey对象。这个对象包含了一些你感兴趣的属性：

    interest集合
    ready集合
    Channel
    Selector

### interest集合
	int interestSet = selectionKey.interestOps();

### ready集合
eady 集合是通道已经准备就绪的操作的集合。在一次选择(Selection)之后，你会首先访问这个ready set。可以这样访问ready集合：
	int readySet = selectionKey.readyOps();
可以用像检测interest集合那样的方法，来检测channel中什么事件或操作已经就绪。但是，也可以使用以下四个方法，它们都会返回一个布尔类型：
1. selectionKey.isAcceptable();
2. selectionKey.isConnectable();
3. selectionKey.isReadable();
4. selectionKey.isWritable();
### 通过selectionKey来访问 channel 和 selector
	Channel  channel  = selectionKey.channel();
	Selector selector = selectionKey.selector();


还可以通过Selector选择通道   
一旦向selector注册一个或多个channel 就可以调用重载的seclect()方法，返回你感兴趣的事件，
select()阻塞到至少有一个通道在你注册的事件上就绪了。
select(long timeout)和select()一样，除了最长会阻塞timeout毫秒(参数)。
selectNow()不会阻塞，不管什么通道就绪都立刻返回

selectedKeys() 一旦调用了select()方法，并且返回值表明有一个或更多个通道就绪了，然后可以通过调用selector的selectedKeys()方法，访问已选择键集中的就绪通道。
Set selectedKeus = selector.selectedKeys();

用完selector之后调用他的close（）方法即可关闭此通道。  
```java
Selector selector = Selector.open();
channel.configureBlocking(false);
SelectionKey key = channel.register(selector, SelectionKey.OP_READ);
while(true) {
  int readyChannels = selector.select();
  if(readyChannels == 0) continue;
  Set selectedKeys = selector.selectedKeys();
  Iterator keyIterator = selectedKeys.iterator();
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
```
## Java NIO和IO的主要差别
	IO         |      NIO
	面向流     |      面向缓冲
	阻塞IO     |      非阻塞IO
	无         |      选择器

### 面向流和面向缓冲有什么区别啊？ 
IO面向流，就是说每次从流中读取一个或者多个字节， 一直到都区所有的字节，他们没有被缓存在任何地方，而且不能前后移动流里面的数据，如果要实现前后移动数据，还是需要缓存到一个缓冲区。 
NIO呢， 就是每次读取，就读取到一个缓冲区中(buffer), 他有position, limit,等属性，所以我们可以前后移动数据，还可以转换读写模式，岂不是很方便。
### 阻塞和非阻塞IO到底什么区别？
IO流是阻塞的，每当他调用read（） 或者write（）方法是，该线程会被阻塞，直到她读到了数据，或者数据完全写入了，这个线程在此期间什么都不能干，即被阻塞了。  NIO使一个线程从某通道发送请求读取数据，但是它仅能得到目前可用的数据，如果目前没有数据可用时，就什么都不会获取。而不是保持线程阻塞，所以直至数据变的可以读取之前，该线程可以继续做其他的事情。 非阻塞写也是如此。一个线程请求写入一些数据到某通道，但不需要等待它完全写入，这个线程同时可以去做别的事情。 线程通常将非阻塞IO的空闲时间用于在其它通道上执行IO操作，所以一个单独的线程现在可以管理多个输入和输出通道（channel）。

### Selector
selector允许一个单独的线程来监视多个输入通道，你可以注册多个通道使用一个选择器，然后使用一个单独的线程来“选择”通道：这些通道里已经有可以处理的输入，或者选择已准备写入的通道。这种选择机制，使得一个单独的线程很容易来管理多个通道。

### 对数据处理有什么不同呢？
IO的话，估计会一行一行的读向下面这样的代码:
```java
BufferedReader reader = new BufferedReader(new InputStreamReader(input));
String nameLine   = reader.readLine();
String ageLine    = reader.readLine();
String emailLine  = reader.readLine();
String phoneLine  = reader.readLine();
```
很多人和我一样，看到这个程序，觉得没什么问题啊，第一行读到了什么，第二行读到了什么。。。。。。  
可是正式如此准确的预测，你每次都知道每一步都读到了什么数据，所以在多线程中造成了IO的致命弱点---一旦，中间的某一个操作阻塞了，那么你不得不等待他完成，才能继续执行，那么百万级别以上的服务器肯定是不成立的。  
NIO的处理方式：  

	ByteBuffer buffer = ByteBuffer.allocate(48);
	int bytesRead = inChannel.read(buffer);
你不能确定的直到每一次read之后，buffer里面究竟读取了多少字符，也不知道是否已经读取完了，好吧，你的确不不知道，所以：   
NIO可让您只使用一个（或几个）单线程管理多个通道（网络连接或文件），但付出的代价是解析数据可能会比从一个阻塞流中读取数据更复杂。  
如果需要管理同时打开的成千上万个连接，这些连接每次只是发送少量的数据，例如聊天服务器，实现NIO的服务器可能是一个优势。  
OK，那么接下来的几天，我打算模仿Netty(NIO的一个框架，对NIO繁琐的API进行了很好的封装)，自己实现一个NIO的聊天服务器. 造一个好一点的轮子。