---
title: copyClass
date: 2018-01-03 23:26:52
categories:
	- Java
tags:
	- Java
	- JDK
---
摘要：深浅拷贝
![](/images/love1.jpg)
<!-- more -->

# jaav 克隆（深浅拷贝）
## java中对象的创建 
clone顾名思义就是复制， 在Java语言中， clone方法被对象调用，所以会复制对象。所谓的复制对象，首先要分配一个和源对象同样大小的空间，在这个空间中创建一个新的对象。那么在java语言中，有几种方式可以创建对象呢？ 
1 使用new操作符创建一个对象 2 使用clone方法复制一个对象 
那么这两种方式有什么相同和不同呢？ new操作符的本意是分配内存。程序执行到new操作符时， 首先去看new操作符后面的类型，因为知道了类型，才能知道要分配多大的内存空间。分配完内存之后，再调用构造函数，填充对象的各个域，这一步叫做对象的初始化，构造方法返回后，一个对象创建完毕，可以把他的引用（地址）发布到外部，在外部就可以使用这个引用操纵这个对象。而clone在第一步是和new相似的， 都是分配内存，调用clone方法时，分配的内存和源对象（即调用clone方法的对象）相同，然后再使用原对象中对应的各个域，填充新对象的域， 填充完成之后，clone方法返回，一个新的相同的对象被创建，同样可以把这个新对象的引用发布到外部。 
## 复制对象 ||  复制引用
在java中，这样的代码很常见
```
    Person p = new Person(23, "zhang");  
    Person p1 = p;  
      
    System.out.println(p);  
    System.out.println(p1);  
```
当Person p1 = p;执行之后， 是创建了一个新的对象吗？ 首先看打印结果： com.pansoft.zhangjg.testclone.Person@2f9ee1ac
com.pansoft.zhangjg.testclone.Person@2f9ee1ac
那么下面这样的代码呢？ 
```
    public class Person implements Cloneable{  
          
        private int age ;  
        private String name;  
          
        public Person(int age, String name) {  
            this.age = age;  
            this.name = name;  
        }  
          
        public Person() {}  
      
        public int getAge() {  
            return age;  
        }  
      
        public String getName() {  
            return name;  
        }  
          
        @Override  
        protected Object clone() throws CloneNotSupportedException {  
            return (Person)super.clone();  
        }  
    }  
```
可已看出，打印的地址值是相同的，既然地址都是相同的，那么肯定是同一个对象。p和p1只是引用而已，他们都指向了一个相同的对象
在浅拷贝中，对于基本类型的数据进行复制，还好，直接复制一份就行了。  
而对于像String这样的 引用类型就会出现两种情形了 。
由于age是基本数据类型， 那么对它的拷贝没有什么疑议，直接将一个4字节的整数值拷贝过来就行。但是name是String类型的， 它只是一个引用， 指向一个真正的String对象，那么对它的拷贝有两种方式： 直接将源对象中的name的引用值拷贝给新对象的name字段， 或者是根据原Person对象中的name指向的字符串对象创建一个新的相同的字符串对象，将这个新字符串对象的引用赋给新拷贝的Person对象的name字段。这两种拷贝方式分别叫做浅拷贝和深拷贝。深拷贝和浅拷贝的原理如下图所示： 
![示意图](http://img.blog.csdn.net/20180101142921199?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzM3OTc5Mjg=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

下面通过代码进行验证。如果两个Person对象的name的地址值相同， 说明两个对象的name都指向同一个String对象， 也就是浅拷贝， 而如果两个对象的name的地址值不同， 那么就说明指向不同的String对象， 也就是在拷贝Person对象的时候， 同时拷贝了name引用的String对象， 也就是深拷贝。验证代码如下：
```
    Person p = new Person(23, "zhang");  
    Person p1 = (Person) p.clone();  
      
    System.out.println("p.getName().hashCode() : " + p.getName().hashCode());  
    System.out.println("p1.getName().hashCode() : " + p1.getName().hashCode());  
      
    String result = p.getName().hashCode() == p1.getName().hashCode()   
            ? "clone是浅拷贝的" : "clone是深拷贝的";  
      
    System.out.println(result);  
```
打印结果为： p.getName().hashCode() : 115864556
p1.getName().hashCode() : 115864556
clone是浅拷贝的
打印的hashcode值是一样的，说明他们使用的是一份内存。
所以，clone方法执行的是浅拷贝。
那么我们如何实现深度拷贝呢？

---
## 覆盖Object中的clone方法， 实现深拷贝 
现在为了要在clone对象时进行深拷贝， 那么就要Clonable接口，覆盖并实现clone方法，除了调用父类中的clone方法得到新的对象， 还要将该类中的引用变量也clone出来。如果只是用Object中默认的clone方法，是浅拷贝的，再次以下面的代码验证： 
```
    static class Body implements Cloneable{  
        public Head head;  
          
        public Body() {}  
      
        public Body(Head head) {this.head = head;}  
      
        @Override  
        protected Object clone() throws CloneNotSupportedException {  
            return super.clone();  
        }  
          
    }  
    static class Head /*implements Cloneable*/{  
        public  Face face;  
          
        public Head() {}  
        public Head(Face face){this.face = face;}  
          
    }   
    public static void main(String[] args) throws CloneNotSupportedException {  
          
        Body body = new Body(new Head());  
          
        Body body1 = (Body) body.clone();  
          
        System.out.println("body == body1 : " + (body == body1) );  
          
        System.out.println("body.head == body1.head : " +  (body.head == body1.head));  
          
          
    }  
```
在以上代码中， 有两个主要的类， 分别为Body和Face， 在Body类中， 组合了一个Face对象。当对Body对象进行clone时， 它组合的Face对象只进行浅拷贝。打印结果可以验证该结论： body == body1 : false
body.head == body1.head : true
## 改进之后的 拷贝
```
package com.coderlong.copyClass;
/**
* @author 作者 : coderlong
* @version 创建时间：2018年1月1日 下午2:36:06
* 类说明: 
*/
public class Body  implements Cloneable{
	Head head;
	public Body(){
		
	}
	public Body(Head head){
		this.head = head;
	}
	
	@Override
	protected Object clone() throws CloneNotSupportedException {
		// TODO Auto-generated method stub
		// 如果想进行深拷贝的话
		Body newbody = (Body)super.clone();
		newbody.head = (Head)head.clone();
		return newbody;
	}

	static class Head implements Cloneable{
		public Face face;
//		@Override
		protected Object clone() throws CloneNotSupportedException {
			// TODO Auto-generated method stub
			return super.clone();
		}
		public Head(){
			
		}
		public Head(Face face){
			this.face = face;
		}
	}
	public static void main(String[] args) throws CloneNotSupportedException {
		// TODO Auto-generated method stub
		Body body1 = new Body(new Head());
		Body body2 = (Body)body1.clone();
		System.out.println(body1 == body2);
		System.out.println(body1.head == body2.head);
	}
}

```
打印结果为： body == body1 : false
body.head == body1.head : false

由此可见， body和body1内的head引用指向了不同的Head对象， 也就是说在clone Body对象的同时， 也拷贝了它所引用的Head对象， 进行了深拷贝
### 应该还不是完全彻底地深度拷贝 
未完 待续。。。。。。
