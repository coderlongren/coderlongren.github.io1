---
title: 交换两个数组中的元素，使连个数组SUM之差最小
date: 2018-3-1 14:18:40
categories:
	- Algorithm
tags:
	- Algorithm
---
<!-- more -->
没事在CSDN算法论坛上看到的一道题，估计好老了，是以前的华为OJ，觉得和我刚刚做过的一道leetcodeContest57 上面的一道数组交换，很像，就记录下来了。


基本是背包问题的变种，浮点数 我也不会处理了，
```c++
#include <iostream>
#include <algorithm>
using namespace std; 

void process(int a[],int b[],int n)
{    
	int temp[10];
	int i;
    for(i=0;i<n;i++)
	{
		temp[i]=a[i];
		temp[i+n]=b[i];
	}
	
    sort(temp,temp+n*2);//对数组进行排序，默认升序，从小到大
	
	cout<<endl;
	int cura=0,curb=0,suma,sumb=0;
	
	//算法基本思想：将排序后的数组，按从大到小顺序放入a，b数组中，规则是先放a，再放b，
	//直到b中的元素总和大于等于a后，再放a中，以此类推。
	
	a[cura++]=suma=temp[9];  //先将最大元素放入a中
	
	for(i=2*n-2;i>=0;i--)
	{
		if((suma>sumb&&curb < n))
		{ 
			
           	b[curb]=temp[i];
			sumb+=b[curb++];		
		}
		else if(cura<n)  //数组b中元素总和大于a中元素总和情况
		{ 
			
			a[cura]=temp[i];
			suma+=a[cura++];
		} 
	}
}
int main()
{
	int a[5];
	int b[5];
	
    int i;
	cout<<"a数组：";
	for(i=0;i<5;i++)
	{
		cin>>a[i];	
	}
	cout<<"b数组：";
	for(i=0;i<5;i++)
	{
		cin>>b[i];
	}
	
	process(a,b,5);

	cout<<"交换元素后a数组：";
	for(i=0;i<5;i++)
		cout<<a[i]<<" ";
	cout<<endl;

	cout<<"交换元素后b数组：";
	for(i=0;i<5;i++)
		cout<<b[i]<<" ";
	
	cout<<endl;
	return 0;
	
}
```
`Java版`
```java
public class twoArray {
	public static void process(int[] a, int [] b, int n) {
		int[] temp = new int[2*n];
		int i;
		// 把两个数组复制到一个数组中去
		for (int index = 0; index < n; index++) {
			temp[index] = a[index];
			temp[index + n] = b[index];
		}
		 // 从小到大 排序
		Arrays.sort(temp);
		// 将排序后的数组，按从大到小顺序放入a，b数组中，规则是先放a，再放b，
		//直到b中的元素总和大于等于a后，再放a中，以此类推
		int curA = 0; 
		int curB = 0;
		int sumA = 0; 
		int sumB = 0;
		// 最大元素 先放入a中
		a[curA++] = sumA = temp[2*n - 1];
		
		for (i = 2*n - 2; i >= 0; i--) {
			// 第二大的放进b里面
			if (sumA > sumB && curB < n) {
				b[curB] = temp[i];
				sumB += b[curB++];
			}
			else if (curA < n) {
				a[curA] = temp[i];
				sumA += a[curA++];
			}
		}
			
		
	}
	public static void main(String[] args) {
		int[] a = {1,3,5,7,8};
		int[] b = {2,3,5,4,10};
		process(a, b, a.length);
		for (int i : a) {
			System.out.print(i + " ");
		}
		System.out.println();
		for (int i : b) {
			System.out.print(i + " ");
		}

		/*** result
		*10 5 5 3 1 
		*8 7 4 3 2
		**/ 
	}
}	


```