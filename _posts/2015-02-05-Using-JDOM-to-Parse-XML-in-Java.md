---
layout: default
title: Using JDOM to parse XML in java
categories: java
---

今天来写一点使用JDOM在java中解析XML文件的方法。由于试着参加了我们学校今年的igem（还不知道能不能进入最终的队伍，筛人方法太诡异，sad...），需要看获奖作品的java源码，写这篇博文的缘由也是因为在所看的项目中大量涉及到使用JDOM进行XML文件的处理。


### 什么是XML  

>XML指 *可扩展标记语言（EXtensible Markup Language）*，它是一种标记语言，很类似html。
>XML 的设计宗旨是传输数据，而非显示数据。
>XML 标签没有被预定义。您需要自行定义标签。
>XML 被设计为具有自我描述性。

以上摘自w3school对XML的描述。XML被广泛的运用于数据的传输，它的语法非常简单，一个XML文档主要由元素和属性构成，它是以树结构组织起来的，必须具有一个根节点（在我们接下来生成XML文档时会体会到这一点）。来看一个同样来着w3school的具体示例：
{% highlight xml %}
<?xml version="1.0" encoding="UTF-8"?>
<bookstore>
<book category="COOKING">
  <title lang="en">Everyday Italian</title> 
  <author>Giada De Laurentiis</author> 
  <year>2005</year> 
  <price>30.00</price> 
</book>
<book category="CHILDREN">
  <title lang="en">Harry Potter</title> 
  <author>J K. Rowling</author> 
  <year>2005</year> 
  <price>29.99</price> 
</book>
<book category="WEB">
  <title lang="en">Learning XML</title> 
  <author>Erik T. Ray</author> 
  <year>2003</year> 
  <price>39.95</price> 
</book>
</bookstore>
{% endhighlight %}
  
#### 值得注意的几点是:

* XML标签**必须**被正确地闭合。与html不同，不闭合的标签被XML认为是语法错误。

* 属性必须加引号，且**不宜多用**,滥用属性不符合XML的宗旨，也影响XML文件的可拓展性。一个被广泛认可的理念是*”元数据（有关数据的数据）应当存储为属性，而数据本身应当存储为元素“*

* 命名时，使用下划线'_'，而非减号'-'或'.'以及':'，否则一些情况下会引起错误。

* 类似与于html语言，XML以\<!-- This is a comment -->的风格进行注释。

* 实体引用的问题，即一些特殊字符如'<'等不能直接用，需要使用实体引用。如用"&amp;amp;"代替"＆"


### 使用JDOM在java中解析XML文件  
 
在java中使用XML文件主要有四种方法DOM、SAX、JDOM和DOM4J，这四种方法的比较在[这篇文章](http://www.cnblogs.com/lanxuezaipiao/archive/2013/05/17/3082949.html)有所说明。因为里面有些内容我还不甚理解，就不在这里再讨论了，既然要看的项目中使用的是JDOM，并且我比较下来JDOM的语法也是最清晰的（简化了DOM的不少东西，同时大量使用了java集合类），所以就先了解JDOM吧。

（顺便说一句，无意中发现上一段中所提博文的作者是我们学校的一个研究生学姐，以前还看过她的一篇有关面试的文章，世界真小啊。这位学姐拿了百度和阿里的offer，最终去了阿里，也是很厉害。）

首先引两句上述博文中的话来简单说明一下JDOM(Java-based Document Object Model)的特点：

>JDOM与DOM主要有两方面不同。首先，JDOM仅使用具体类而不使用接口。这在某些方面简化了API，但是也限制了灵活性。第二，API大量使用了Collections类，简化了那些已经熟悉这些类的Java开发者的使用。  
> JDOM自身不包含解析器。它通常使用SAX2解析器来解析和验证输入XML文档（尽管它还可以将以前构造的DOM表示作为输入）。它包含一些转换器以将JDOM表示输出成SAX2事件流、DOM模型或XML文本文档。JDOM是在Apache许可证变体下发布的开放源码。

接下来说说用JDOM解析XML的简单流程，首先，如上文所说，JDOM本身不包含解析器，所以第一步是创建一个SAXBuilder，来生成我们所需要的文档对象（document对象）；第二步就是document对象的创建，或者说将XML文件转化为java能“看得懂”的document对象，可以用java.io中的FILE对象来作为build()函数的参数，也可以使用InputStream；第三步，是获取XML文件中的根元素，就是上文中强调的那个。因为XML是以树结构组织起来的，所以这一步配合第四步，获得并遍历一个元素的children，实际上就能完成对整个XML文件的操纵。

后续的步骤，如对元素进行修改增删，以及生成一个新的XML文档，看着API来都是很容易的，下面是我的实验代码，实现了对XML文档的读取和生成。其中第一部分的输入文件就是上文中用于举例的XML文件，输出文件见最下面。

{% highlight java %}
public class testJDOM {
	public static void main(String args[]) throws JDOMException, IOException {
		
		/*
		 * this part is to parse a exited XML document
		 */
		SAXBuilder builder = new SAXBuilder();				//Step 1: make a builder
		String xmlPath = "src/learn/xml/test.xml";									
		File inputFile = new File(xmlPath);
		Document document = builder.build(inputFile);		//Step 2: build a document
		Element bookstore = document.getRootElement();		//Step 3: get the root element
		List<Element> booklist = bookstore.getChildren();	//Step 4: get children 
		for (Element book : booklist ) {
			System.out.println(book.getAttributeValue("category") + " : ");
			List<Element> bookmetalist = book.getChildren(); 
			for (Element bookmeta : bookmetalist) {
				System.out.println(bookmeta.getName() + " : " + bookmeta.getText());
			}
			System.out.println();
		}
		
		/*
		 * this part is to write things to a new XML file
		 */
		Document myDocument = new Document();
		String xmlPath2 = "src/learn/xml/test2.xml";
		
		Element rootElement = new Element("root");
		
		Element element = new Element("student");
		
		Element nameElement = new Element("name");
		nameElement.setText("yaokai");
		Element sexElement = new Element("sex");
		sexElement.setText("male");
		Element descriptionElement = new Element("description");
		descriptionElement.setText("yaokai is a student of School of Computer Science and Technology, USTC.");
		
		element.addContent(nameElement);
		element.addContent(sexElement);
		element.addContent(descriptionElement);
		
		rootElement.addContent(element);
		myDocument.setRootElement(rootElement);
		
		XMLOutputter myOutputter = new XMLOutputter();
		OutputStream myOStream = new FileOutputStream(xmlPath2);
		myOutputter.output(myDocument, myOStream);
	}
}
{% endhighlight %}

第二部分的输出文件：
{% highlight xml %}
<?xml version="1.0" encoding="UTF-8"?>
<root><student><name>yaokai</name><sex>male</sex><description>yaokai is a student of School of Computer Science and Technology, USTC.</description></student></root>

{% endhighlight %}
  


