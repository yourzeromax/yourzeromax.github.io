---
layout:     post
title:      Java IO流理解指南（一）流概念与File类
subtitle:   深入理解Java中IO流的使用技巧
date:       2018-04-02
author:     yourzeromax
header-img: img/post-bg-android.jpg
catalog: true
tags:
    - Java
---

>本文是系列文章第一篇，主要介绍数据流的概念和File类的解读

# 写在前面
近来，有学弟在学习Android的过程中和我交流说：Java I/O流的使用好难，并且流的概念非常不好理解。想起我当时学习Java和Android的时候，在流的概念上也是走了太多的弯路，于是想蹭着这个机会，写一篇这样的文章。  

# 流概念简述  
应该说，在实际的应用场合之中，会涉及到与文件之类的交互，通俗的来讲，就是将数据写入文件或者是从文件中读取数据，不同的语言对于这一个需求所采取的方法不同，但是思想都是大同小异，在Java语言中，就采用的是一种叫做“流”的概念，也就是I/O流。
## 什么是流  
如同字面意思，流这个字可以组词：水流、流量等。Java中的流便指的是数据像水一样可以到处流动，所谓I/O流，就是指的是输入输出的数据流，接下来我会大量采用比方的形式，来帮助大家深入理解流的概念。比如下图（找不到图，大家想象一下自家的水龙头，从自来水厂经过水管运输再经过水龙头流出来）。

## 流入和流出
“水往高处走”，数据也是一样，在数据流这个概念中，就有一个流入方向和流出方向的概念，比如上面描述的场景，自来水厂是流入方向，而水龙头是流出方向。但是，我想说的是，流入和流出不是绝对的。比如人类和一台计算机交互过程中，当我们用键盘打字时，流入对象是键盘，流出对象是电脑或者它的CPU：  

![](https://raw.githubusercontent.com/yourzeromax/yourzeromax.github.io/master/img/20180402/20180402-1.png)

当电脑处理完数据后，用显示屏展示给我们看时，这时流入流出对象又是颠倒过来的：  

![](https://raw.githubusercontent.com/yourzeromax/yourzeromax.github.io/master/img/20180402/20180402-2.png)


## 流的分类  
从实际生活中体验到流的概念后，我们再回到Java中的流，流可以从两个层面上进行分类：

层面 | 类别A| 类别B
---|---|---
流动方向 | 输入流| 输出流
数据类型 | 字节流| 字符流    

然后两两进行组合出现四种流：字节输入流、字节输出流、字符输入流、字符输出流...有人看到这里就懵逼了，哈哈，不用怕，其实是有规律的，我们姑且分为字节流和字符流吧，输入和输出可以镜像记忆的，之后会详细介绍。   

# 内容提纲  
本文旨在帮助大家理解Java I/O流，除了上文对流概念的简单介绍以外，由于在实际开发应用中，用的最多的是和文件进行交互，所以会先简要介绍一下File类的知识，然后会分为字节流和字符流进行详解。现在，我亲自画一个丑丑的模型给大家看看：

![](https://raw.githubusercontent.com/yourzeromax/yourzeromax.github.io/master/img/20180402/20180402-3.png)

上图的关系就很明白了，水源可能是河、自来水厂、地下水等，出水口可能是水龙头、
淋浴等。那么我们将这套模型类比到java中来看，就可以得出下图（三种流只是为了方便理解才列出来，可以想象流就是一个水管，一头流入，一头流出）：  

![](https://raw.githubusercontent.com/yourzeromax/yourzeromax.github.io/master/img/20180402/20180402-4.png)  

# File类
Java是一个典型的OOP语言（面向对象），相信大家在学习Java的第一天就知道java擅长把事物抽象成类，File类就是描述计算机文件的类，在Java程序中，一个File对象就代表着一个实实在在存储于电脑中的文件（或者文件夹）。既然这是一个抽象类的描述，那么接下来就会介绍怎样获得一个File类对象，在获取到File对象后，就会介绍一个对象所应当具有的方法行为（这个File对象应该有什么样的操作）。

> Java中把文件和文件夹都定义为File对象，这一点需要注意。

## File对象的获取方法
构造方法共四种，如下表：

Java语言 |  说明
---|---
File(File parent, String child); | 通过给定的父抽象路径名和子路径名字符串创建一个新的File对象
File(String pathname)； | 通过将给定路径名字符串转换成抽象路径名来创建一个新 File 对象
File(String parent, String child)；  | 根据 parent 路径字符串和 child 路径名字符串创建一个新 File 对象
File(URI uri)；  | 将给定的 file: URI 转换成一个抽象路径名来创建一个新的 File 对象  
获取方法有且只有上面四种，一般最常用的是前三种，最后一种不怎么使用，是不是很简单？  
## 方法行为  
本节将介绍File对象的常用方法，这些方法根据用途可以分为三个方面：判断、查询、操作。

### 判断  

Java语言 | 方法描述
---|---
public boolean isAbsolute()| 测试此抽象路径名是否为绝对路径名。
public boolean canRead() | 测试应用程序是否可以读取此抽象路径名表示的文件。
public boolean canWrite() | 测试应用程序是否可以写入此抽象路径名表示的文件。
public boolean exists() | 测试此抽象路径名表示的文件或目录是否存在。
public boolean isDirectory() | 测试此抽象路径名表示的文件是否是一个目录。
public boolean isFile() | 测试此抽象路径名表示的文件是否是一个标准文件。

### 查询  

Java语言 | 方法描述
---|---
public String getName()| 返回由此抽象路径名表示的文件或目录的名称。
public String getParent() |  返回此抽象路径名的父路径名的路径名字符串，如果此路径名没有指定父目录，则返回 null。
public File getParentFile() | 返回此抽象路径名的父路径名的抽象路径名，如果此路径名没有指定父目录，则返回 null。
public String getPath()| 将此抽象路径名转换为一个路径名字符串。
public long lastModified() | 返回此抽象路径名表示的文件最后一次被修改的时间。
public long length() | 返回由此抽象路径名表示的文件的长度。
public String[] list()| 返回由此抽象路径名所表示的目录中的文件和目录的名称所组成字符串数组。
public File[] listFiles() | 返回一个抽象路径名数组，这些路径名表示此抽象路径名所表示目录中的文件。
public File[] listFiles(FileFilter filter)|返回表示此抽象路径名所表示目录中的文件和目录的抽象路径名数组，这些路径名满足特定过滤器。
public String toString()|返回此抽象路径名的路径名字符串。

### 操作

Java语言 | 方法描述
---|---
public boolean delete() | 删除此抽象路径名表示的文件或目录。
public void deleteOnExit() | 在虚拟机终止时，请求删除此抽象路径名表示的文件或目录。
public boolean mkdir()|创建此抽象路径名指定的目录。
public boolean mkdirs()|创建此抽象路径名指定的目录，包括创建必需但不存在的父目录。
public boolean renameTo(File dest)|重新命名此抽象路径名表示的文件。
public boolean setLastModified(long time)|设置由此抽象路径名所指定的文件或目录的最后一次修改时间。
	
将File对象方法进行分类是为了方便大家记忆的，我列出来的都是比较常规的用法，当然大家不用死记硬背，熟能生巧就ok，实在记不了用法，可以回来看看这篇文章。

至此，第一篇文章结束，如果大家有不懂的地方，欢迎留言或者发我邮件。  

[主页](http://www.yourzeromax.top)
	










