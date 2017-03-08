---
layout: default
title: algorithms02-Stack-and-Queue
categories: algs4
---

算法课程的第二讲，是比较熟悉的栈和队列。所以这里只总结几点新的知识，主要是数组的resize，java中的loitering,java中的for-each,以及这一周的编程作业。


### 数组的resize

使用数组来实现栈的代码十分简洁，平均速度也较链表实现要快一些，然而，数组实现的一个很大弊端是数组的长度是固定的，如果我们想要client免于提供数组的长度的话，就必须实现数组的resize。  

**这里一下子就体现出国产教材和国外教材的差距了呀**,我在学校里学的的数据结构（用的严蔚敏的那本教材）在处理这个问题时，采用的是初始长度100（还是1000,不记得了）,每次超出就加10。现在看来真是**愚不可及**，按这样的方法，如果要向栈中压入ｎ个元素(n足够大)，耗费的时间讲达到 $$n^2$$ （花在resize上的时间有100+110+120+...+n,有$$n^2／２$$之多）,对于压栈这么基本的操作，这是不能容忍的，更关键的是，这个操作很容易就能做的更好，达到线性时间完成。

可行的方法是，当数组满了的时候，创建一个长度为原来两倍的数组，然后将元素复制过去。这样的花的时间是n+(2+4+8+16+...+n) ~ 3n,大大节约了时间。这样，每次压栈操作的均摊时间就能达到常数这个级别了。让我回想起之前看Ｃ++模版时发现其中的vector也是采用了这种方法，但是当时不了解原因，现在看起来真是精彩的一个做法。


### java中的loitering

所谓loitering，是java中内存泄露的一种情况，指在一个对象已经不在被需要的时候仍持有它的指针（引用）。  

java中采用了自动垃圾回收的机制，一般认为这是相较于Ｃ和Ｃ++的一大进步，但是，java中的内存泄露仍然是一个很大的问题，经常因为一些很难发现的细节而导致内存泄露。lotering是一个相对简单的情况，至少在课堂上碰到的这种情况从原理上还是比较容易理解的。就以栈的数组实现为例，当一个元素退栈时（return a[--n])，要将数组中留给它的引用的地方清为null（a[n]=null)。因为java的垃圾回收器只在没有引用指向它时，才会将它回收。

memrory leak是一个（一系列）很复杂也很需要关注的问题，尤其是Ｃ和java这些语言上。我在google搜索loitering时看到了很多更复杂的内容和实例，看来以后不可避免地要和这个问题打很多交道。


### java中的for-each
for-each是java中的一个非常好用的特性，它给我们进行遍历操作操作提供了一个很方便的方法。例如，要遍历打印一个字符串数组中的内容，只需要这样一条语句：  
{% highlight java %}
for (String s : array)
    System.out.println(s);
{% endhighlight %}  
要在自己设计的数据结构中实现这么一个功能，必须实现java的Iterable接口，这个接口是十分简单的，只要包含一个返回Iterator的函数就可以了：　　
{% highlight java %}
public interface Iterable<Item>{
    Iterator<Item> iterator();
}
{% endhighlight %}  
而在Iterator中必须实现hasNext,next,和remove方法  
{% highlight java %}
public interface Iterator<Item>{
    boolean hasNext();
    Item next();
    void remove();
}
{% endhighlight %}
然后，使用foreach语句时，java就会按照如下方式进行遍历：
{% highlight java %}
Iterator<String> i = myObject.iterator();
while (i.hasNext()){
    String s = i.next();
    System.out.println(s);
}
{% endhighlight %}

### 这一周的编程作业

这一周的编程作业主要是关于栈和队列的实现的，看似是两条简单题，实际上由于对时空复杂度的样限制，很多地方还是颇有难度的。

* **第一题**：Deque双向队列。这不是什么复杂的概念，在严蔚敏的书上也提到过。因为限制了每个操作都必须在常数时间内完成，所以我采用了双向链表（一开始非常低级的用了单链表，太低级的错误了，以后**写程序之前一定要先想清楚**），另外链表操作的边界情况很值得注意。

* **第二题**:RandomizedQueue随机出队的队列。这倒是挺新鲜的结构，一开始还被要求弄迷糊了，走了许多弯路，实际上稍微改一下resizing的算法就可以。由于要随机读取肯定要用数组实现，至于数组最大的弱点：删除操作，还是用老办法，先标记，resizing时一并去掉。
{% highlight java %}
private void resize(int capacity) {
		Item[] tmpItems = (Item[]) new Object[capacity];
		int rmNum = 0;
		for(int i = 0; i < size; i++){
			if(rqItems[i] == null){
				rmNum++;                        //记录跳过的已删除的items数目
				continue;
			}
			tmpItems[i - rmNum] = rqItems[i];
		}
		size = size - vacancy;                   //vacancy表示之前删去的元素数，即留下的空缺     
		vacancy = 0;
		rqItems = tmpItems;
	}
{% endhighlight %}

* **第三题**Subset子集。实际上就是第二题的一个简单应用。不过这里倒是涉及到另一个算法**Knuth－Shuffle**,这是Knuth大神的一个洗牌算法，并不复杂，简单来说是：从Ｎ开始向下，每个数（ｋ）和０～ｋ之间的随机数互换。这样如果产生随机数是常数时间的话，一趟Knuth－Shuffle可以在线性时间内完成。
下面是Knuth－Shuffle的java代码,引自algs4的booksite：
{% highlight java %}  
 public static void shuffle(Object[] a) {
        int N = a.length;
        for (int i = 0; i < N; i++) {
            // choose index uniformly in [i, N-1]
            int r = i + (int) (Math.random() * (N - i));
            Object swap = a[r];
            a[r] = a[i];
            a[i] = swap;
        }
 }
{% endhighlight %}


按照惯例附上[作业代码](https://github.com/yaokai1117/algs4/tree/master/RandomizedQueueAndDeques/src)
