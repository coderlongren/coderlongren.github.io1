---
title: IO流的一点总结
date: 2018-6-20 14:18:40
categories:
	- JDK
tags:
	- JDK
---
摘要：IO流总结
<!-- more -->
## Java输入流
输入流的基本组件是InputStream类, 继承他的类大概有  
	InputStream
	 |
	 +--FileInputStream 文件输入流
	 |
	 +--ByteArrayInputStream  
	 |
	 +--PipedInputStream
	 |
	 +--FilterInputStream
	 |
	 +--BufferedInputStream 
	 |
	 +--PushbackInputStream 
	 |
	 +--DataInputStream 
	 |
	 +--ObjectInputStream

继承Inputstream抽象类，都能够使用如下的方法：
1. read()读取一个字节并将读取的字节作为int返回。当到达输入流的结尾时，它返回-1。
2. read(byte[] buffer)，读取最大值直到指定缓冲区的长度。它返回在缓冲区中读取的字节数。如果到达输入流的结尾，则返回-1。
3. read(byte [] buffer，int offset，int length)读取最大值到指定长度字节 数据从偏移索引开始写入缓冲区。它返回读取的字节数或-1，如果到达输入流的结束。
4. close() 关闭输入流
5. available() 返回可以从此输入流读取但不阻塞的估计字节数。

### FileInputStream
顾名其意：  
FileInputStream fin  = new FileInputStream(srcFile);

### BufferedInputStream
缓冲流：  
BufferedInputStream通过缓冲数据向输入流添加功能。
它维护一个内部缓冲区以存储从底层输入流读取的字节
。
```java
public static void main(String[] args) {
	String srcFile = "test.txt";
	try (BufferedInputStream bis = new BufferedInputStream(new FileInputStream(
	    srcFile))) {
		  byte byteData;
		  while ((byteData = (byte) bis.read()) != -1) {
		    System.out.print((char) byteData);
		  }
		} catch (Exception e2) {
		  e2.printStackTrace();
	}
}
```
### PushBackInputStream
回退流，

### DataInputStream
DataInputStream可以从输入流中读取Java基本数据类型值。  
DataInputStream类包含读取数据类型值的读取方法。  
例如，要读取int值，它包含一个readInt()方法;  
读取char值，它有一个readChar()方法等。  
它还支持使用readUTF()方法读取字符串。

## Java输出流
在抽象超类OutputStream中定义了三个重要的方法:
1. write() 将字节写入输出流。  
它有三个版本，允许我们一次写一个字节或多个字节。
2. flush() 用于将任何缓冲的字节刷新到数据源。
3. close()方法关闭输出流。
### FileOutputStream 文件输出流
一般的 为了注意打开的文件不存在，可以包含在try-catch快中， 不过也可以使用API提供的重载构造函数，在文件不存在的时候，新建文件
FileOutputStream fos   = new FileOutputStream(destFile, true);

###  打印流 PrintStream
大引流可以把我们需要的数据打印到file中去，是输出流的具体装饰器
PrintStream可以以合适的格式打印任何数据类型值，基本或对象。
PrintStream可以将数据写入输出流不抛出IOException。
简单使用：  
```java
String path = "testPrintStream.txt";
	PrintStream printStream = new PrintStream(path);
	try {
		printStream.print("test1 \r\n");
		printStream.print("test2 \r\n");
		printStream.print("test3");
		printStream.flush(); // 刷新输出流
		System.out.println("file was writted in " + new File(path).getAbsolutePath());
		
	} catch (Exception e) {
		// TODO: handle exception
	}
```
### DataOutputStream
DataOutputStream可以将Java基本数据类型值写入输出流。  
DataOutputStream类包含一个写入数据类型的写入方法。  它支持使用writeUTF（String text）方法将字符串写入输出流。

### ObjectOutputStream
用于Java的序列化操作，很简单就不说了。

## 可外化对象序列化
同样是支持序列化/反序列化对象的类，不过在具体的序列化过程逻辑，需要自己实现
需要实现Externalizable接口
```java
public class Student1 implements Externalizable{
	private String name = "yake";
	public Student1() {
		
	}
	public Student1(String name) {
		this.name = name;
	}
	@Override
	public String toString() {
		// TODO Auto-generated method stub
		return "name:" + this.name;
	}
	@Override
	public void writeExternal(ObjectOutput out) throws IOException {
		// TODO Auto-generated method stub
			out.writeObject(this.name);
	}
	@Override
	public void readExternal(ObjectInput in) throws IOException, ClassNotFoundException {
		// TODO Auto-generated method stub
		this.name = (String)in.readObject();
	}
}
```
## 字符流
顶层的抽象类：
Reader  
Writer  
例子都举烂了，自己写吧，少年。


