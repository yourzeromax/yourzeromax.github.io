---
layout:     post
title:      Android自定义属性及xmlns全面解析
subtitle:   领略Android自定义属性的魅力
date:       2018-08-30
author:     yourzeromax
header-img: img/post-bg-android.jpg
catalog: true
tags:
    - Android
---     

 # 写在前面  
 
 halo，大家好~又到了每周一次的分享时间(放屁！)，每天从出门到上班大约有四十多分钟的公交，也利用这段时间刷刷别人的技术博客，成长了很多，这周重新温习了一下Android自定义View和属性，在这个过程中，对下列xml布局中的语法产生了好奇：
```
<LinearLayout 
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools" >
```  
xmlns几乎在每一个xml文件最外层控件都有，但是它有什么作用呢？于是乎便有了想写这篇博客的冲动。  
因此，本文的重点内容就是简单阐述一下如何自定义View和它的属性，并且再来讲讲xmlns的用途。  
# 自定义View  
众所周知，View和ViewGroup承载着Android视图的任务，并且ViewGroup继承自View，Google为了降低开发者的入门门槛，提供了很多已经封装好了的View控件，比如Button、TextView、ImageView等等，对于ViewGroup来说，也有LinearLayout、FrameLayout、RelativeLayout等能够包含其他View的组控件。  
无论哪一种控件，它都直接或者间接地继承自View类，虽然Google所提供的控件非常丰富，但是在实际的项目中，往往需要自己去特定一些View控件，终归有以下四种情况：  
1. 继承View
2. 继承ViewGroup
3. 继承已有的View
4. 继承已有的ViewGroup
 
本文主要讲述第一种情况，至于后面三种情况，就需要深入到View的绘制流程之中学习，万变不离其宗。   

onMeasure  
onLayout  
onDraw

这三座大山是View绘制的精髓，文章暂时不涉及View的绘制；如果只是创建一个普通的自定义View，则只用关心onDraw方法的重写，还是直接依照步骤上代码看看吧：

## 创建MyView的构造函数：  

```
    public MyView(Context context) {
        this(context, null);
    }

    public MyView(Context context, @Nullable AttributeSet attrs) {
        this(context, attrs, 0);
    }

    public MyView(Context context, @Nullable AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
        TypedArray typedArray = context.obtainStyledAttributes(attrs, R.styleable.MyView, defStyleAttr, 0);
        s = typedArray.getString(R.styleable.MyView_Text);
        typedArray.recycle();
    }
```  

构造函数可以当成模板使用，其中看到有三个参数的构造函数的情况有些复杂，什么是TypeArray？可以理解成xml布局中属性的集合，在自定义属性中，也是通过这个对象来获取我们需要的自定义值。
```
context.obtainStyledAttributes（）  
方法接受四个参数，其中我们关心的是第二个参数，它是一个id值，  
构成方法是：  
R.styleable.XXX
```    
## 创建自定义属性类别
R.styleable.XXX就是我们需要自己定义的属类别性，在res/values/中新建attrs.xml文件，并添加以下代码：  

```
<resources>
  <declare-styleable name="MyView">
       <attr name="Text" format="string"/>
       <attr name="TextColor" format="color"/>
       <attr name="TextDrawable" format="reference"/>
       <attr name="TextSize" format="integer"/>
       <attr name="TextAllow" format="boolean"/>
   </declare-styleable>
</resources>
```

declare-styleable标签包裹自定义的属性类别，也就是上文中XXX所需要替代的值，在该标签下可以包含多个<attr>标签值，每一个attr都代表能够包含的属性，而name表示的是xml中自定义的属性名，format表示该属性的取值类型，可以包含string、integer、boolean、color、refrence等，分别表示字符串、int值、布尔值、颜色id、对象引用（drawable等）。  

## xml中添加自定义属性
我们在xml中为自定义View添加自定义属性，可以如下所示：  

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    tools:context=".activities.ViewActivity">

    <com.yourzeromax.test_new.views.MyView
        android:id="@+id/mv_test"
        android:layout_width="40dp"
        android:layout_height="40dp"
        app:Text="123" 
        app:TextAllow="true"
        app:TextColor="@color/colorAccent"
        app:TextSize="13"
        app:TextDrawable="@drawable/drable_list"/>
    
</LinearLayout>
```  

我们再回来看一下View的构造函数中的代码：

```
     TypedArray typedArray = context.obtainStyledAttributes(attrs, R.styleable.MyView, defStyleAttr, 0);
      String  s = typedArray.getString(R.styleable.MyView_Text);
        typedArray.recycle();
```  

其中s的值便是我们在xml中指定的字符串值，在本例子中，s="123"，这样便完成了xml属性到代码属性的转换，不信可以自己试试。  
> Tips:在使用完TyoedArray对象以后，一定要记得recycle（）回收，避免内存泄漏的发生。  

# xmlns的作用    

xml布局中使用最多的是android:id、andorid:layout_width等形式，在我们自定义的xml中，添加了app:Text、app:TextAllow等这样的写法，其实冒号前面的单词并不重要，重要的是后面引号中的内容，前面的单词可以任意取名，我们在最开始指定了这样一句代码：  

```
xmlns:app="http://schemas.android.com/apk/res-auto"
```
包括系统自动为我们添加的这一句代码：  

```
xmlns:android="http://schemas.android.com/apk/res/android"
```  
正是这两句代码的存在，才使得我们能够使用android、app这样的标签，为什么呢？

xmlns是xml namespace的缩写形式，换言之，这条代码就是为我们指定了app或者android的命名空间，一般情况下，我们引用“apk/res-auto”就让app能够访问res下所有资源的能力，当然如果想让其只能访问某个res资源，只需要引用“包名/apk/res/具体属性文件夹”即可。有了这些语句，就能够让我们使用已经制定好的属性集，懂了吧？是不是特别简单

## xmlns:tools的用法  
在布局中，任意xml标签下添加代码：  

```
xmlns:tools="http://schemas.android.com/tools"
```  
之后，我们便能使用android为我们提供的tools，先举个开发过程中的案例：  

> 有时候在布局一个xml的过程中需要查看某个TextView的文字效果，于是指定了android:Text =
> "客户们都是笨蛋"；这时，产品上线，结果忘了删除这行代码，数据加载也没有重新设置text内容。  
  
这种场面是不是就很尴尬了？使用tools标签就能够避免这样的尴尬，我们在xml布局中加入tools:text="客户们都是笨蛋"，发现预览中是这样的：  
![](https://raw.githubusercontent.com/yourzeromax/yourzeromax.github.io/master/img/20180830/20180830-1.png)

但实际上，如果运行的话，是看不到这条text的。
除此之外，tools:标签几乎支持所有android:的同名标签，比如tools:src、tools:textSize等，可以自己进行探索。  
总之，tools的作用就是能够方便开发者在开发过程中提前预览xml布局而对之后的运行结果不产生影响，挺方便的一个工具。  

欢迎关注
[我的博客](www.yourzeromax.top)
