---
layout: post
title: algorithms03-Elementary-Sort
categories: algs4
---

这一讲的内容是几个基础的排序方法，选择排序，插入排序，希尔排序以及之前提到过的Knuth Shuffle。基本都是学过的内容，复习巩固一下，这篇博文主要总结java中Comparable接口的应用以及希尔排序。

### Comparable接口

在程序设计中，**回调(Callback)**是一个很有用的概念，它指的是“对一段可执行代码的引用”，通俗地讲，一个用户程序client调用了服务程序中的某个方法，这个方法又*回过头调用client的一个**函数***，这就被称为回调。比较常见的例子就是排序函数了，例如使用C的qsort函数时，要以函数指针的方法传给它一个比较函数，换个角度看，这不就正是qsort函数“回过头来调用”了我们自定义的比较函数吗，这就是所谓回调。

各种语言实现回调的方式有所差异，如上文提到的，C是用函数指针实现回调的。而在java中，我们**使用接口来实现回调**。还是以排序函数（称为sort）为例，sort所接受的参数（一般是要排序的数列）实现Comparable接口，这样sort就可以反过来调用参数的cmpareTo函数，进行比较，实现了回调。来看具体的代码：

{% highlight java %}
private static class Item implements Comparable<Item> {
	private int i;
	private String str;
	
	public Item(int i, String str) {
		this.i = i;
		this.str = str;
	}
	
	@Override
	public int compareTo(Item o) {
		// TODO Auto-generated method stub
		if(this.i > o.i)
			return 1;
		else if(this.i < o.i)
			return -1;
		return 0;
	}
}
{% endhighlight %}

其中Comparable采用泛型的目的是指明比较函数接受的参数（即Comparable<Item>尖括号中的类型与compareTo(Item o)函数接受的参数类型必须是一样的。具体进行回调的代码（即sort函数）见下面的的实例代码。


### 选择排序（SelectionSort）

选择排序应该是最简单的排序了吧，不过还是有可以总结的地方的。先上代码，方便分析：

{% highlight java %}
public static void SelectionSort(Comparable[] a){
	for (int i = 0; i < a.length; i++) {
		int min = i;
		for (int j = i+1; j < a.length; j++) {
			if(a[j].compareTo(a[min]) < 0)
				min = j;
		}
		swap(a, i, min);
	}
			
}
{% endhighlight %}

我们从**不变性(invariants)**的角度进行分析，选择排序算法执行过程中保持的不变性是：扫描过的部分（具体到代码中就是０到ｉ的部分）始终是递增的，并且没有扫描过的部分所有的数都比扫描过的部分的数大（这与下面要说的插入排序不同）。选择排序的原理就是不断重复这样的过程（每次选出未扫描过的最小数，与ｉ进行交换）直到最后全部的数组都扫描过，所有都是有序的。

一次完整的选择排序共需~$$N^2/2$$次比较操作和N次交换操作，时间复杂度为~$$N^2$$，**注意，不管是序列的情况如何，即使是已经有序的序列，复杂度仍然是~$$N^2$$，这与插入排序不同**


### 插入排序（InsertSort）

同样是先上代码：
{% highlight java %}
public static void InsertSort(Comparable[] a) {
	for (int i = 0; i < a.length; i++) {
		for (int j = i; j > 0; j--) {
			if(a[j].compareTo(a[j-1]) < 0)
				swap(a, j, j-1);
			else 
				break;
		}
	}
}
{% endhighlight %}

插入排序维护的不变性是：ｉ左边的所有元素均为递增。即只有选择排序维护的不变性的第一部分，**它不保证在ｉ左边的元素一定比右边小**。我想这也是插入排序不是每种情况都有~$$N^2$$的复杂度的原因。

平均来说，对于一个顺序为随机的序列，插入排序需要~$$N^2/4$$次比较和~$$N^2/4$$次交换（最差情况将两个４改成２），表面上看，插入排序凭空比选择排序多出了那么多的交换操作，而涉及访存的操作操作又是最耗时的，那么插入排序一定比选择排序慢得多了？其实不然，**在序列基本有序（“逆序数”为c*N这个级别，ｃ为常数）时，插入排序可以在线性时间内完成！**，实际上，可以证明，**插入排序的交换操作数等于序列的逆序数（比较数等于交换数＋Ｎ－１）**。

这是个很重要的结论，这表明在序列基本有序的时候插入排序是十分快的，而实际上我们经常碰到这样“基本有序”的数列。不仅如此，我们还可以主动去“制造”这样基本有序的序列，为高效的插入排序创造条件，这是我们的第一个想法；除此之外，插入排序一个一个地交换太浪费时间，能否跨步走呢？这是我们的第二个想法。一个快得多的排序方式：希尔排序，正是基于这两个很自然的想法。


### 希尔排序（ShellSort） 

希尔排序相较于前两个方法要复杂一些，它是先对序列进行若干次h-sort，即跨度为ｈ的插入排序，通过这种方式来将数组变得“基本有序”。注意，这里的ｈ是越来越小的，到最后变成1-sort，就是我们的插入排序，然而这时的序列以及很“有序”了，插入排序进行得非常快。可以想见，希尔排序整体速度是很快的，因为对于前面ｈ比较大的插入排序，实际上进行操作的数组只有N/h（不要看到这里认为“这不还是没什么改变吗，ｈ是常数啊”，注意ｈ不是常数，刚开始的ｈ是很大的，一般至少N/3）,而对于后面ｈ较小的，注意此时序列已经基本有序了。

ｈ的序列一般采用Knuth的3x+1序列，即1,4,13,40,121,364...，这时的希尔排序在最差情况下的时间复杂度是Ｏ（$$N^{3/2}$$），而在平均情况下，它的时间复杂度只有大约nlog(n)，精确的表述仍未被发现。

下面是希尔排序的代码：

{% highlight java %}
public static void sort(Comparable[] a) {
	int N = a.length;
	int h = 1;
	while(h < N/3)
		h = 3*h + 1;
	while (h > 0) {
		for (int i = h; i < N; i++)
			for(int j = i; j >= h && a[j].compareTo(a[j-h]) < 0; j -= h)
				swap(a, j, j-h);
		h = h/3;
	}
}
{% endhighlight %}