---
title: Transient
date: 2018-1-12 20:18:40
categories:
	- java
tags:
	- 个人
	- 开发
	- JDK
---
重新
<!-- more -->
## 1.transient的作用以及用法
我们都知道一个对象只要实现了Serilizable接口，这个对象就可以被序列化，java的这种序列化模式为开发者提供了很多便利，我们可以不必关系具体序列化的过程，只要这个类实现了Serilizable接口，这个类的所有属性和方法都会自动序列化。

然而在实际开发过程中，我们常常会遇到这样的问题，这个类的有些属性需要序列化，而其他属性不需要被序列化，打个比方，如果一个用户有一些敏感信息（如密码，银行卡号等），为了安全起见，不希望在网络操作（主要涉及到序列化操作，本地序列化缓存也适用）中被传输，这些信息对应的变量就可以加上transient关键字。换句话说，这个字段的生命周期仅存于调用者的内存中而不会写到磁盘里持久化。

总之，java 的transient关键字为我们提供了便利，你只需要实现Serilizable接口，将不需要序列化的属性前添加关键字transient，序列化对象的时候，这个属性就不会序列化到指定的目的地中。

示例code如下：
``` 
package 序列化;

import java.io.FileInputStream;
import java.io.FileNotFoundException;
import java.io.FileOutputStream;
import java.io.IOException;
import java.io.ObjectInputStream;
import java.io.ObjectOutputStream;

/**
* @author 作者 : coderlong
* @version 创建时间：2018年1月12日 上午11:51:21
* 类说明: 
*/
public class TestTransient {

	public static void main(String[] args) throws FileNotFoundException, IOException {
		// TODO Auto-generated method stub
		User user = new User();
        user.setUsername("王雅珂");
        user.setPasswd("123456");
 
        System.out.println("read before Serializable: ");
        System.out.println("username: " + user.getUsername());
        System.err.println("password: " + user.getPasswd());
        
        ObjectOutputStream os = new ObjectOutputStream(new FileOutputStream("E:/Ehcache/user.txt"));
        os.writeObject(user); // 把user对象 序列化
        os.flush();
        os.close();
        System.out.println("user对象已经序列化了");
        ObjectInputStream in = new ObjectInputStream(new FileInputStream("E:/Ehcache/user.txt"));
        try {
			user = (User) in.readObject();
		} catch (ClassNotFoundException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		} // 从流中读取User的数据
        
        in.close();

        System.out.println("\nread after Serializable: ");
        System.out.println("username: " + user.getUsername());
        System.err.println("password: " + user.getPasswd());
   
	}

}


read before Serializable: 
password: `123456`
username: 王雅珂
user对象已经序列化了

read after Serializable: 
username: 王雅珂
password: `null`
```
## 2. transient使用小结
1）一旦变量被transient修饰，变量将不再是对象持久化的一部分，该变量内容在序列化后无法获得访问。

2）transient关键字只能修饰变量，而不能修饰方法和类。注意，本地变量是不能被transient关键字修饰的。变量如果是用户自定义类变量，则该类需要实现Serializable接口。

3）被transient关键字修饰的变量不再能被序列化，一个静态变量不管是否被transient修饰，均不能被序列化。

```
package 序列化;

import java.io.FileInputStream;
import java.io.FileNotFoundException;
import java.io.FileOutputStream;
import java.io.IOException;
import java.io.ObjectInputStream;
import java.io.ObjectOutputStream;

/**
* @author 作者 : coderlong
* @version 创建时间：2018年1月12日 下午1:08:49
* 类说明: 
*/
public class TestTransient2 {

	public static void main(String[] args) throws FileNotFoundException, IOException {
		// TODO Auto-generated method stub
		// TODO Auto-generated method stub
				User user = new User();
		        user.setUsername("王雅珂");
		        user.setPasswd("123456");
		 
		        System.out.println("read before Serializable: ");
		        System.out.println("username: " + user.getUsername());
		        System.err.println("password: " + user.getPasswd());
		        
		        ObjectOutputStream os = new ObjectOutputStream(new FileOutputStream("E:/Ehcache/user.txt"));
		        os.writeObject(user); // 把user对象 序列化
		        os.flush();
		        os.close();
		        System.out.println("user对象已经序列化了");
		        user.setUsername("wangyake");
		        ObjectInputStream in = new ObjectInputStream(new FileInputStream("E:/Ehcache/user.txt"));
		        try {
					user = (User) in.readObject();
				} catch (ClassNotFoundException e) {
					// TODO Auto-generated catch block
					e.printStackTrace();
				} // 从流中读取User的数据
		        
		        in.close();

		        System.out.println("\nread after Serializable: ");
		        System.out.println("username: " + user.getUsername());
		        System.err.println("password: " + user.getPasswd());

	}

}
//  运行结果
password: `123456`
read before Serializable: 
username: 王雅珂
user对象已经序列化了

read after Serializable: 
username: wangyake
password: `null`

```
## 3. transient使用细节——被transient关键字修饰的变量真的不能被序列化吗？
在上一段代码 ：
```
package 序列化;

import java.io.Externalizable;
import java.io.FileInputStream;
import java.io.FileOutputStream;
import java.io.IOException;
import java.io.ObjectInput;
import java.io.ObjectInputStream;
import java.io.ObjectOutput;
import java.io.ObjectOutputStream;
import java.io.ObjectStreamException;
import java.util.jar.Attributes.Name;

import org.junit.experimental.theories.Theories;

/**
* @author 作者 : coderlong
* @version 创建时间：2018年1月12日 下午1:26:25
* 类说明: 
*/
public class ExternalizableTest implements Externalizable {
	private transient String Name = "是的，我也将会被序列化";
	@Override
	public void writeExternal(ObjectOutput out) throws IOException {
		// TODO Auto-generated method stub
		out.writeObject(Name);
	}

	@Override
	public void readExternal(ObjectInput in) throws IOException, ClassNotFoundException {
		// TODO Auto-generated method stub
		Name = (String)in.readObject();
	}
	
	
	 public static void main(String[] args) throws Exception {
		 ExternalizableTest ex = new ExternalizableTest();
		 ObjectOutputStream out = new ObjectOutputStream(new FileOutputStream("E:/Ehcache/user.txt"));
		 out.writeObject(ex);
		 out.flush();
		 out.close();
		 
		 ObjectInputStream in = new ObjectInputStream(new FileInputStream("E:/Ehcache/user.txt"));
		 ex = (ExternalizableTest) in.readObject();
	     System.out.println(ex.Name);
	     in.close();
 
	 }

}

```
输出结果是 `是的，我也将会被序列化`  
为什么呢？不是说类的变量被transient关键字修饰以后将不能序列化了吗？  
在Java中，对象的序列化可以通过实现两种接口来实现，若实现的是Serializable接口，则所有的序列化将会自动进行，若实现的是Externalizable接口，则没有任何东西可以自动序列化，需要在writeExternal方法中进行手工指定所要序列化的变量，这与是否被transient修饰无关。  
所以 Name依旧被序列化之后反序列化了。:blush:




