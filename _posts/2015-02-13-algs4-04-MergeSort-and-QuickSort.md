---
layout: post
title: algorithms04-MergeSort-and-QuickSort
categories: algs4
---

这周课程的内容是两个应用非常广的排序算法:MergeSort和QuickSort,相较于上一篇博文中提到的算法，这两个算法的速度要快得多（非平凡算法），我很担心一篇博文的篇幅能否叙述完，let's have a look.


### Comparator的使用

在上一篇博文中我们简单总结了一下Comparable接口的用法，这一次是Comparator，虽然看起来很像，但是两者有很大的区别。Comparator与Ｃ中函数指针的用法更接近了，不过它是直接传一个对象过去，可以将Comparator理解为“一种比较的方法”（与之对应的，Comparable更像是赋予实现对象一个“可以进行比较的属性”），使用Comparator可以以多种顺序进行比较，只要对应地实现多个Comparator对象即可，一般某个对象的某一个Comparator用大写字母命名（见下，public static final嘛）。
{% highlight java %}
public class Student {
	public static final Comparator<Student> BY_ID = new ById();
	public static final Comparator<Student> BY_NAME = new ByName();
	
	private int id;
	private String name;
	
	public Student (int id, String name) {/**/}
	
	private static class ById implements Comparator<Student>{
		@Override
		public int compare(Student o1, Student o2) {
			return o1.id - o2.id;
		}
	}	
	private static class ByName implements Comparator<Student> {
		@Override
		public int compare(Student o1, Student o2) {
			return o1.name.compareTo(o2.name);
		}
	}
}
{% endhighlight %}


### 归并排序（MergeSort）

归并排序是分治法的一个典型应用。假设我们有两个已经排好序的数列，要将它们合并为一个有序数列，那么线性时间的算法是显然的，“两个指针，一趟扫描”。归并排序的实现就是基于此，我们将要排序的数列分成两半，递归地分别对这两个子数列进行排序，然后进行归并。来看代码：

{% highlight java %}
import java.util.Comparator;
public class MergeSort {
	
	private static final int CUTOFF = 7;

	private static void merge(Object[] a, Object[] aux, int low, int mid, int high, Comparator c) {
		for (int k = low; k <= high; k++)
			aux[k] = a[k];
		
		int i = low, j = mid + 1;
		for (int k = low; k <= high; k++){
			if (i > mid)
				a[k] = aux[j++];
			else if (j > high)
				a[k] = aux[i++];
			else if (c.compare(aux[j], aux[i]) < 0)
				a[k] = aux[j++];
			else 
				a[k] = aux[i++];
		}
	}
	
	public static void sort(Object[] a, Object[] aux, int low, int high, Comparator c) {
		
//		if(low >= high)
//			return;
		
		if (high <= low + CUTOFF - 1){			// First step of optimizing, using insertion sort for tiny subarrays
			InsertSort.sort(a, low, high, c);
			return;
		}
		
		int mid = (low + high) / 2;
		sort(a, aux, low, mid, c);
		sort(a, aux, mid+1, high, c);
		if (c.compare(a[mid], a[mid+1]) >= 0)	// Second step of optimizing, stopped
			return;
		merge(a, aux, low, mid, high, c);
	}
}
{% endhighlight %}

对长度为Ｎ的序列进行归并排序需要进行NlogN次比较操作和6NlogN次数组访问操作，因此有NlogN的时间复杂度，比上篇博文中的算法要快了很多。空间使用方面，归并排序需要cN的额外内存空间（也有in-place的算法，但因为太复杂而不实用）。值得注意的是，（未经优化的）归并排序**在所有情况下的复杂度都是NlogN**,这与快排有很大不同。它与快排的另外一点区别是，归并排序是稳定的，而快排不是（顺带一提上篇博文中三种算法中只有插入排序是稳定的）。

归并排序可以进行一些优化。首先，当子数列较小时，使用归并排序有点大材小用了，并不合算，所以当要处理的数组长度小于等于７时，改用插入排序是一个不错的做法（见代码中注释Step1处），当然，直接无视这一类情况，最后整体来一遍插入排序也是很好的做法；然后，考虑到归并排序处理已经排好了的序列也需要NlogN的时间，我们可以避免这一情况，见Step2处，如果判断出两个分数组已经排好序了，就不进行归并了。

归并排序有另一种被称为bottom-up的实现方法，代码简洁且不需要递归（然而在许多情况下比原来的实现方式稍稍慢一点），它是先从跨度为一归并起，然后跨度为２,以此类推。代码：
{% highlight java %}
public static void sortBU(Object[] a, Comparator c) {
	int N = a.length;
	Object[] aux = new Object[N]; 
	for (int size = 1; size < N; size = 2*size)
		for (int low = 0; low < N-size; low += 2*size)
			merge(a, aux, low, low+size-1, Math.min(low+2*size-1, N-1), c);
}
{% endhighlight %}

利用归并排序可以实现计算一个数列的逆序数的NlogN算法，一边归并，一边计数，每个“排在后面”的元素的并入都带来mid-i+1的逆序数增加。


#### 快速排序（QuickSort）

接下来就是大名鼎鼎的快排了，这是应用及其广泛的算法，快如其名。快排同样是基于分治法，与归并不同，快排是先处理再递归，首先选一个数组元素作为枢轴，然后通过分划使比这个元素大的都移到它右边，比它小的都移到右边。然后递归对左右两个数列进行操作，最终完成对整个数组的排序。代码：
{% highlight java %}
private static int partition(Object[] a, int low, int high, Comparator c) {
	int i = low, j = high + 1;
	while (true){
		while (c.compare(a[++i], a[low]) < 0)
			if (i == high) break;
		while (c.compare(a[--j], a[low]) > 0)
			if (j == low) break;
		
		if (j <= i)
			break;
		swap(a, i, j);
	}
	swap(a, low, j);
	return j;
}

public static void sort(Object[] a, int low, int high, Comparator c) {
	
//		if (low >= high)
//			return;
	
	if (high <= low + CUTOFF - 1){								//using insertion sort for tiny subarrays
		InsertSort.sort(a, low, high, c);
		return;
	}
	
	int pivot = (int)((high-low + 1)*Math.random()) + low;		//random pivot, avoid "the worst condition"
	swap(a, low, pivot);
	
	pivot = partition(a, low, high, c);
	sort(a, low, pivot-1, c);
	sort(a, pivot+1, high, c);
}
{% endhighlight %}

快排的平均时间复杂度也是NlogN，只需要常数的额外空间，是in-place算法，而且要比归并快一些。简单分析一下，每次对整个数组进行分划（由若干个小分划组成）需要线性时间，而平均情况下这样的操作一共要有logN次，所以复杂度是NlogN。

然而通常的以第一个元素为枢轴的快排有一个很致命的缺点：当待排序的数组已经有序或者逆序的时候，会退化为冒泡排序，复杂度一下子变为$$N^2$$,这是因为这种情况下每次分划只向前进一位，需要1+2+3+...+(n-1)+n~$$\frac{1}{2}n^2$$次比较。解决这个问题有几种可行的思路，一个是快排前将数组shuffle一次，这是教程中的做法；另一种是去一个０～Ｎ的随机数，将其与第一个调换，然后在快排，这是很常用的随机化快排，也是这篇博文中代码所用的方法；第三种是取第一个、最后一个和中位数三个数字，然后再中间大小的数。

这三种方法可以避免（极大地减小几率）worst case的出现，然而，当数组中有大量相同键时，它们还是无能为力，复杂度还是会跌到$$N^2$$。这时我们就需要快排的一个变种：三路分划快排。即将数组分划为三个部分，小于ｖ的，等于ｖ的，和大于ｖ的。我们使用lt,i和gt三个变量，算法执行过程中维持的不变性是：lt左边的都小于ｖ，gt右边的都大于ｖ，从lt到i-1都是等于ｖ的，最终退出循环时有gt==v-1，这时lt和gt就将数组分成了我们想要的三个部分了。另外，这种方法实现的快排代码意外地简洁，且不需递归，来看代码：
{% highlight java %}
public static void sort3Way(Object[] a, int low, int high, Comparator c) {	
	if (low >= high)
		return;
	
	int i = low + 1;
	int lt = low, gt = high;
	Object v = a[low];
	while (i <= gt){
		int cmp = c.compare(a[i], v);
		if (cmp < 0)
			swap(a, i++, lt++);
		else if (cmp > 0)
			swap(a, i, gt--);
		else 
			i++;
	}
	
	sort3Way(a, low, lt - 1, c);
	sort3Way(a, gt + 1, high, c);
}
{% endhighlight %}



