---
layout:     post
title:      Android 同一个TextView中多彩显示文字
subtitle:   String、Spannable、Buidler的使用技巧
date:       2018-08-13
author:     yourzeromax
header-img: img/post-bg-android.jpg
catalog: true
tags:
    - Android
---

# 写在前面
最近，在公司的项目中需要将一段文字分别涂上两种不同的颜色，最笨重的解决办法就是用多个TextView相互进行拼接显示，但是不光让业务逻辑变得繁杂，也让代码可靠性降低，因此本文介绍两种可以实现在同一个TextView下展示不同色彩的方法，如下图所示：  
![图1 TextView示意图](https://raw.githubusercontent.com/yourzeromax/yourzeromax.github.io/master/img/20180813/20180813-1.png)

其中涉及到的是SpannableString等相关的类知识，当然String是一个既基础也复杂的对象类，所以在文章开头会阐述String、StringBuffer和StringBuilder的应用对比，话不多说，开始吧。  

# String、StringBuffer、StringBuilder
在Java和Android中，对于字符串来说常用的除了String以外，还有StringBuffer和StringBuilder这两种，这也是校招必备的知识要点之一，所以先用一张表来谈谈这三者的区别：  

名字 | 说明|线程安全
---|---|---
String | 不可变序列|---
StringBuffer| 可变序列|安全
StringBuilder| 可变序列|非线程安全  
由此可见，三者最大的不同点主要在于两个方面，一个是否可变序列、另一个则关乎于是否为线程安全，什么叫不可变序列呢？看下列代码：

```
   String str = "hello ";
        String str1 = str + "word";
        if (str == str1) {
            Log.d(TAG, "str与str1为同一对象：" + true);
        } else {
            Log.d(TAG, "str与str1为同一对象：" + false);
        }
        
        StringBuilder stringBuilder =new StringBuilder("hello ");
        StringBuilder stringBuilder1 = stringBuilder.append("word");
        if (stringBuilder == stringBuilder1) {
            Log.d(TAG, "builder与builder为同一对象：" + true);
        } else {
            Log.d(TAG, "stringBuilder与stringBuilder为同一对象：" + false);
        }
```
最终输出结果为：

```
str与str1为同一对象：false
builder与builder为同一对象：true
```  
从结果可以看出，String和StringBuilder对象分别拼接了一个字符串，但是String对象得到的却不是同一个对象，而StringBuilder依旧是原来的对象---不可变序列String在添加字符串时会重新创建一个新的对象，而可变序列StringBuilder和StringBuffer则能够使用同一个对象。
## String补充知识
实际上，在拼接String对象时，Java虚拟机内部会对代码进行运行优化：  
```
//实际代码
String str = "I love";
    for(int i = 0;i<1000;i++){
    str+="java";
}
//默认优化代码：
  String str="";
    for(int i=0; i<1000; i++){
        StringBuilder sb = new StringBuilder(str);
        sb.append("java");
        str=sb.toString();
        }
```    
除外，看一下代码运行结果：  

```

        String str1 = "hello world";
        String str2 = new String("hello world");
        String str3 = "hello world";
        String str4 = new String("hello world");
         
        System.out.println(str1==str2);
        System.out.println(str1==str3);
        System.out.println(str2==str4);
        
   // 运行结果：
   // false
   // true
   // false
``` 
在class文件中有一部分来存储编译期间生成的字面常量以及符号引用,这部分叫做class文件常量池，在运行期间对应着方法区的运行时常量池。  
也就是说str1之所以与str3相等，是因为他们都指向了同一个常量。这算是String对象的唯一一个优势。

## 性能比较与使用总结
虽然String与StringBuffer和StringBuilder相比起来几乎全落下风，但是也不至于让开发者在实际使用时感到恐慌，其实大多数时候都可以直接使用String，但是要注意以下的使用场景：  

性能排名：StringBuilder > StringBuffer > String  
1. 当字符串相加操作少时，使用String。
2. 当字符串相加操作多时，使用StringBuilder。
3. 当字符串相加操作多，并且涉及多线程，使用StringBuffer。
  
# SpannableString及相关类 

从这里开始才介绍标题的实现办法，
实现多彩TextView的显示最常用的办法就是SpannableString，当然也有SpannableStringBuidler(没有Buffer)。  
在最开始学习的时候不太容易记住每个对象和每个方法的作用是什么，导致每一次都要翻看Api，这样的开发效率极其低下，在这里总结Spannable的相关用法，帮助记忆。首先，Spannable的使用流程有四步：  

1. 创建SpannableString(Builder)对象。
2. 创建需要的Span样式格式。
3. 调用第一步得到对象的setSpan()方法，完成字符串装载。
4. 调用TextView.setText()显示。    

是不是很简单？（废话，把重点都省略掉了。）再来分析下上面的四个步骤：首先前两步都是创建相关的对象，后两步才是将对象真正调用起来，第四步不用解释:

```
        ForegroundColorSpan foregroundColorSpan = new ForegroundColorSpan(Color.RED);
        //BackgroundColorSpan span = new BackgroundColorSpan(Color.GRAY);
        
        SpannableString spannableString = new SpannableString("shaw是一头猪");
        spannableString.setSpan(foregroundColorSpan,0,4, Spannable.SPAN_EXCLUSIVE_EXCLUSIVE);
        tvSpannable.setText(spannableString);
```

SpannableString在创建之后便不能修改字符串，而SpannableStringBuilder依旧能够通过append（）方法来添加字符串。所以大多数开发者都会选择Buidler来使用;在获得相应的字符串对象后，都会有一个setSpan()方法，这个方法非常重要非常重要非常重要,我们来剖析一下它的四个参数：  

序号 | 参数类型|参数名|描述
---|---|---|---
1 | Object| what|设置Span样式的对象
2 | int |  start| 开始字符下标(包含)
3 | int |  end  | 结束字符下标(不包含)
4 | int |  flags| SPannable扩展flag 

## Span样式类  

首先第一个参数是设置Span的样式，它的对象类别有很多，最为常用的有如下几种：  
名称 | 类名|参数|描述
---|---|---|---
背景色 | BackgroundColorSpan| Color|设置背景颜色
前景色 | ForegroundColorSpan| Color|设置前景颜色（字体颜色）
字体风格 | StyleSpan |  Typeface  | 斜体、加粗等
图片 | ImageSpan |  Drawable| 添加图片替换文字
缩放大小 | RelativeSizeSpan |  float| 设置相对其他文字的大小  

除了这些常用的，还有很多意想不到的效果，这里就不一一举例了，可以去Android官网查一查相关的Api，总之，第一个参数只要传相关的XXXSpan类对象，就代表设置了相关的风格变化。

## 其余三个参数的作用  
start参数和end参数如字面意思所示，需要提的是start是包含了该下标，而end则没有包含该下标，什么意思呢？举个栗子，字符串的下标从0开始，“shaw是一头猪”这一个字符串总共有0-7的下标，如果start和end分别为0，4，则指代的是shaw这0，1，2，3的下标字符串。虽然比较绕口，但是逻辑还是很简单的，多多思考便能明白。  

最后一个参数是标识参数，它的取值总共有四种：

名称 | 描述
---|---
Spannable.SPAN_EXCLUSIVE_EXCLUSIVE|前后都不包括，即在指定范围的前面和后面插入新字符都不会应用新样式 
Spannable.SPAN_EXCLUSIVE_INCLUSIVE|前面不包括，后面包括。即仅在范围字符的后面插入新字符时会应用新样式
Spannable.SPAN_INCLUSIVE_EXCLUSIVE|前面包括，后面不包括。
Spannable.SPAN_INCLUSIVE_INCLUSIVE|前后都包括。  
这个参数主要是对新键入的字符有作用，换言之对EditText使用SpannableString，如果我们设置Spannable.SPAN_EXCLUSIVE_INCLUSIVE，在选中的span下标前面和后面都输入文字，前面的文字没有任何效果，后面的则不同，添加上相同的Span特效，（前面不应用特效，后面应用特效），其它几个Flags参数的含义看表格的描述就能明白。    

[heybox://news?news_id=162571](heybox://news?news_id=162571&link_id=5359177&url=https://api.xiaoheihe.cn/maxnews/app/detail/162571)  
  
[视频 heybox://video?link_id=5344673](heybox://video?link_id=5344673)
    
[游戏 heybox://opengame?appid=578080&game_type=pc](heybox://opengame?appid=578080&game_type=pc)    
  
[游戏专辑 heybox://gameAlbum?id=63](heybox://gameAlbum?id=63)  
  
[帖子 heybox://link?link_id=5357618](heybox://link?link_id=5357618)

[百科 heybox://web?url=https://api.xiaoheihe.com?a=b&title=强袭胸甲](heybox://web?url=https://api.xiaoheihe.com?a=b&title=强袭胸甲)    
  
[专题活动 heybox://web?url=https://api.xiaoheihe.com?a=b&title=2018%20活动](heybox://web?url=https://api.xiaoheihe.com?a=b&title=2018活动)
    
[Roll heybox://rollRoom?link_id=35465](heybox://rollRoom?link_id=35465)  

