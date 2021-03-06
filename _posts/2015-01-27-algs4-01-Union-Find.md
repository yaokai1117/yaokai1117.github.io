---
layout: default
title: algorithms01-Union-Find
categories: algs4
---

今天开始学习Coursera上的一个算法课程　*Algorithms Part I* ,参考书[Algorithms, 4th Edition](http://algs4.cs.princeton.edu/home/)。在这个过程中我大概会按每一讲来对应地更新我的blog。今天是第一讲：Union-Find.


### Union-Find的算法及其优化

#### 1.Dynamic connectivity问题
所谓Dynamic connectiveity，实际上是我们很熟悉的一个概念，简单的说集合上定义的一个*等价关系*（自反性，传递性，对称性)能够对集合进行分划,这是在线性代数课程上就接触过的（实际上在很多地方都遇到过，这是个很普适的概念，甚至我热学课的老师在讲热力学第零定律推出温度定义时也提到了这个）。我们要做的就是构造一个ADT来描述这种关系，一个可行的办法就是我们接下来要说的Union-Find。

这个问题有很广的应用，我在这学期学习数据结构时，有一次用Kruskal算法求最小生成数，需要表示图的连通片，就是一个这一类的问题。我当时用的方法是最基础的算法，复杂度为n^2的，事实上这个问题可以有更好（好得多）的算法，let's have a look:)

**说明**：因为课程中有些地方不知怎么翻译，碰巧发现这一讲的内容用图论语言来描述十分适合，所以本文中相关概念均用图论语言描述（如讲components称为连通片）。

#### 2. Union-Find
首先明确这个ADT应该能做哪些事情,也就是UF的API了: 

~~~ 
    void union(p, q)            //将p, q相连接，实际上也将它们所在连通片连接在一起  
    int find(p)                 //返回ｐ所在连通片的辨识标志  
    boolean connected(p, q)     //判断p, q是否连通  
~~~
一个很自然的想法是，使用一个长度为Ｎ的一维数组id[N]表示Ｎ个元素，当且仅当id[p] == id[q]时，p与q相连通，同一连通片中的元素在数组中的值相同。这样做find操作和connected操作都只需要常数时间，然而每次union操作却要花费Ｏ(n)的时间，因为遍历整个数组是不看避免的，我们希望能对其进行优化。

* **第一层优化：树型结构实现Quick-Union**。仍然是数组id[N]，换一种角度，将每个连通片看成一棵树（从图论的角度来看，这是很直观的），id[p]的值是p的parent，对于树根ｒ有id[r]==r。find返回ｐ所在树的根，即用根来作为每个连通片的标志。这样，union(p,q)只要先进行find(p)和find(q)找到p、q的根pr、qr，再让pr成为qr的孩子，即id[pr]=qr，就可以完成union操作。时间复杂度为Ｏ(tree height),相较于之前的Ｏ(N)
要好了很多。值得注意的是这时find操作和connected操作也需要Ｏ(tree height)的时间，当然这是值得的。  

* **第二层优化：使树趋于“平衡”**。经过第一层优化后的算法其时间复杂度降到了Ｏ(tree height)，但算法对tree height完全没有采取限制措施，换言之，最差情况下，一颗“歪脖子树”的tree height可以达到N,这时算法的时间复杂度又退化回Ｏ(n)了。因此第二步优化（称为weighted quick-union)的目的就是为了使树更“平衡”一些。只需在进行union时加一步比较操作，让规模更小的树并进规模较大的树中（注意，这里的规模指的是顶点个数而不是高度），就会有很大的改善。可以证明，加上这一步操作后树的高度至多为log(N)+1，时间复杂度进一步优化为Ｏ(log(N))。 
{% highlight java %}
public void union(int p, int q) {
        int rootP = find(p);
        int rootQ = find(q);
        if (rootP == rootQ) return;

        // make smaller root point to larger one
        if (sz[rootP] < sz[rootQ]) { 
            id[rootP] = rootQ; 
            sz[rootQ] += sz[rootP]; 
        }
        else { 
            id[rootQ] = rootP; 
            sz[rootP] += sz[rootQ]; 
        }
    }

{% endhighlight %} 

* **第三层优化：使树趋于“平坦”**。既然UF中这棵树“越矮越好”，我们为什么不想办法让它更矮一点（最好高度只有２）呢？出于这样的考虑，第三步优化（称为path compression,路径压缩)的目的是使树平坦化。如果每次find操作在找到根后载回过头来再循环一次，将一路上遇到的所有节点的parent都变改为树根，那么最终得到的树高度会非常小，但是这样操作十分麻烦，且无必要。实际上，我们只需在find的循环中加一句id[p]=id[id[p]]，路径就可以被压缩到一个很理想的状况，此时所有操作的时间复杂度都可以达到接近常数时间了。
{% highlight java %}
public int find(int p) {
        if (p < 0 || p >= id.length) throw new IndexOutOfBoundsException();
        while (p != id[p]) {
            id[p] = id[id[p]];    // path compression by halving
            p = id[p];
        }
        return p;
    }
{% endhighlight %}

每一步优化的复杂度比较：

|algorithm           | constructor | union       | find        |
|:------------------:|:-----------:|:-----------:|:-----------:|
|quick-find          |N            |N            |1            |
|quick-union         |N            |tree height  |tree height  |
|weightedquick-union |N            |lgN          |lgN          |
|weighted QU with path compression|N|close to 1  |close to 1   |
|impossible          |N            |1            |1            |    
  
这位教授在视频中提到了一个很有意思的观点，是我之前没有听到过的。一个复杂度为n^2的算法为什么不够快呢？这似乎是个没什么意义的问题，可能会被回答以“常识”之类的答案，这位教授给出的解答却是，因为它跟不上计算机技术发展的速度。假设计算机处理器的速度快了十倍，在这期间它的存储能力也扩大了十倍，即问题的规模从ｎ扩大到了10ｎ，那么，一个时间复杂度为ｎ^2的算法做同一件事情反而会慢十倍。反而言之，要使一个n^2的算法不至于随着计算机发展显得“越来越慢”，处理器的发展速度至少要在问题规模发展速度的平方这个数量级上。*我们并不是（大多数情况下不是）因为一个算法解决某个问题太慢而认为这个复杂度不行，而是无法容忍随着技术的发展解决一个问题需要的时间反而更多*。这种说法本身的准确性姑且先不去深究，但这个观点的确很有启发性，放在一门讲算法的课程的开头也的确十分合适，它很有助于我理解“渐进复杂度关心的是**增长的趋势**”这一概念。


### 课后作业：Percolation
这一周的课后作业是Union-Find的一个具体应用。一个N-by-N的正方形格子，开始是所有的格子都是blocked，最上层可以看做有水，通过不断随机地open一些格子，最终水可以流到最下层，称为穿滤percolates。其中包含了一个很有趣的数学现象，即当实验的规模增大，达到穿滤所需open的格数占总格数的比固定在0.593附近。关于这次作业的详细内容详见[这里](http://coursera.cs.princeton.edu/algs4/assignments/percolation.html)  
[我的实现代码(github)](https://github.com/yaokai1117/algs4.git)  
![per1](/images/percolates.png) ![per2](/images/percolation-threshold100.png)

### 2015-02-03更新
今天才知道这个数据结构有个名字叫*"并查集森林"*，详见[这里](http://zh.wikipedia.org/wiki/%E5%B9%B6%E6%9F%A5%E9%9B%86)  
看来以后还是要多谷歌呀。
