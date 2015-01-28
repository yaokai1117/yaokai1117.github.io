---
layout: post
title: Reviewing java before using it to learn DS and algorithms
---

由于我现在在Coursera上学习的课程 *Algorithms Part I* 使用java作为实现语言，我需要对java的一些基本知识做一些复习，尤其是java中OOP的部分，熟悉它对于数据结构的设计和使用有很多帮助。我主要参考的书籍是[Introduction to Programming in Java](http://introcs.cs.princeton.edu/java/home/).


### Java中的OOP复习

既然是临场复习，就只写具体实现了，只简略的借助Introduction to Programming in Java这本书进行一下回顾。

#### 一个class的构成
我在上面提到的书中找到了一张很好的图来简单明了地说明java中的class实现：
![class-in-java](/images/class-in-java-anatomy.png)

#### 如何设计自己的class（与课程其实关系不大，但看了之后觉得很有启发，决定写上来）

1. 首先设计API，要使问题尽量明确，考虑到各种可能的inputs，同时要注意*"It's easy to add methods to an existing API, but you can never remove them without breaking existing clients"*,避免多余的设计，“*when in doubt, leave it out*";    

2. 然后进行封装（Encapsulation），使用private关键字。用户只应该接触到他们需要的信息和被允许的操作，至于具体的实现，应对用户隐藏；    

3. 保持不可变性（Immutability），使用final关键字。这主要是指class中的一些instance variables（见上图），这样使代码更易于维护，同时避免一些意外的发生。尤其要注意的是，在使用数组之类的可变对象(mutable objects)时，final只保证引用指向同一个实例对象，而不保证对象的内容不变。
    
4. 保持代码的健壮性，使用exception和assert。但assert不要用在public方法中，因为会被disable，一般只用在测试开发中！
  
       
#### Java中的继承与借口

[两种方法的比较](http://introcs.cs.princeton.edu/java/36inheritance/)
    
这里还是只写具体实现了。先说继承，子类拥有父类所用实例变量和方法，还可以有自己的方法，关键字是extends；然后是接口，接口的定义与类相似，只要将class改为interface,且只能为public（当然啦，不然怎么起”接口“的作用呢），接口的实现是使用关键字implements，注意，一个类可以实现多个接口。
    
    
### 在Java中使用数据结构

#### 泛型(Generics)与自动装盒(Autoboxing)

Java中的泛型与C++中的模版语法很类似，如下：
{% highlight java %}
Stack<String> stack = new Stack<String>();
stack.push("Test");
...
String next = stack.pop(); 
{% endhighlight %}

而自动装盒则涉及到Java中的类型问题。Java中主要分两种类型，基本类型（primitive types)和引用类型（reference types)，基本类型有八个，他们都有其封装类。而泛型中使用的必须是引用类型（即必须是个对象），所以在使用时需要进行转换。自动装盒则是Java在赋值、传参、表达式中会自动进行基本类型及其封装类之间的转换，例如下面的代码：
{% highlight java %}
Stack<Integer> stack = new Stack<Integer>();
stack.push(17);        // auto-boxing (int -> Integer)
int i = stack.pop();   // auto-unboxing (Integer -> int)
{% endhighlight %}

#### java.util包中的数据结构

java.util包中已经包含了我们常用的几个数据结构，了解这些结构会使我们的算法学习方便许多。这些数据结构有：
LinkedList, ArrayList, Stack, Set, Hashtable, HashMap等，具体的API...用到再说:)


    

