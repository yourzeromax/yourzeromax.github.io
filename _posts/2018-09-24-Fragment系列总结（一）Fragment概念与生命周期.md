---
layout:     post
title:      Fragment系列总结（一）Fragment概念与生命周期
subtitle:   坐下！Fragment的基本操作！
date:       2018-09-24
author:     yourzeromax
header-img: img/post-bg-android.jpg
catalog: true
tags:
    - Android
---    

# 写在前面  
Fragment是Google在Android3.0新加的东西，它的功能和作用如同名字一样，代表着一块块碎片，而这些碎片则可以灵活地嵌入到各Activity之中。  
其他关于Fragment的介绍，相信大家看各种博客和书籍都了解得非常多了，所以在此介绍一些关于Fragment独特的见解，在刚开始的时候，我对Fragment的理解也只停留在书籍介绍的部分，但是随着开发经验的积累，才真正明白Google工程师们的伟大...  
Android整个框架其实是一个比较经典的MVC开发模型，抛开Model和View容易理解以外，虽然没有iOS之中各种XXXController命名方法这么直观，但是Activity其实一直都承担着控制器的角色，如果按照一种十分完美的项目工程来说，Fragment的引入，能够减轻臃肿的Activity业务控制代码，对不同区域View的控制任务进行划分，分别移到到不同的Fragment之中，而Activity原则上只需要管理这些Fragment以及极少的业务控制部分。这种开发方式不仅使得Android能够解决屏幕碎片化的问题，也让各部分的业务逻辑开发变得浅显易懂。  
总而言之：Fragment其实是承担Acitivity中不同View区域逻辑的控制器，而Activity将通过FragmentManager对所有的Fragment进行统一管理。
# Fragment基础概念
Fragment的问世经过了很多年，而这些年Google的工程师们也一直在改进Fragment的代码，目前的现状就是除了SDK包中自带的Fragment以外，还提供了存在于V4兼容包内的Fragment，一般情况下，大多数开发者们都会使用v4包下的Fragment，这几乎成为主流。随着包路径所带来的不同以外，Activity中的Fragment管理类的获取方式也从`getFragmentManager()`变成了`getSupportFragmentManager()`。  
静态Fragment使用方法不谈，动态Fragment的操作将会以事务（Transaction）的形式完成，代码如下： 

```
FramgentB fragmentB = new FragmentB();
FragmentManager mFragmentManager = getSupportFragmentManager();
FragmentTransaction mFragmentTransaction = mFragmentManager.beginTransaction();
mFragmentTransaction.add(R.id.fragment_constrainer,fragmentB).commit();
``` 
开发过程中往往会采用链的形式进行调用，详情可以查看本文末尾提供的代码。Activity对于Fragment的操作主要包括七种：add、remove、replace、attach、detach、show、hide。  
## add(int id,Fragment fragment,String tag)
向特定View区域添加一个Fragment，此方法接收三个参数,第一个是Activity中在xml定义的ViewGroup(一般是FrameLayout)的Id,第二个参数是需要添加的Fragment，第三个参数是为Fragment指定一个Tag标识，方便之后的查找。如果查看过add()的源码就会发现，在成功添加后，Fragment还会将第一个参数作为自己的id来进行标识（非唯一），换言之，我们可以通过id和tag来寻找到该fragment。
## remove(Fragment fragment)
移除Fragment

## replace(int id,Fragment fragmentB)
这个方法等于先remove掉该id的fragment，然后再add一个fragmentB

## show(Fragment fragment)和hide(Fragment fragment)
显示/隐藏视图

## detach(Fragment fragment)和attach(Fragment fragment)
剥离/连接视图
差别在下文讲述

## findFragmentById(int id)和findFragmentByTag(String tag)
这两个方法类似于findViewById()的逻辑来对Activity管理的Fragment进行查找，采用依次遍历的方式，将符合id或者tag字段的fragment进行返回。那么，问题来了：如果一个framelayout的id添加了很多个Fragment，怎么知道返回的是哪一个Fragment？运行demo试试呗。

好了，除此之外，也没什么基础概念需要提醒大家了。  

# Activity和Fragment的关系
开局一张图送大家：  

![Fragment与Activity关系](https://user-gold-cdn.xitu.io/2018/9/27/1661ae2d016ca2c0?w=1118&h=862&f=png&s=106007)  
此图蕴含着Activity和Fragment的关系，按照前文的思想，我们来假设一个场景：

临近春节，一个叫activity的老板圈了一块场子要开展销会，他觉得如果整个场子都自己来管理的话有点混乱，于是他邀请了几个商家分别叫fragmentA、fragmentB,fragmentC,分别划分一块地给他们经营，如何经营他管不着，他只需要通过fragmentmanager这个管家来管好这几个叫fragment的商家就好了。

虽然情景有些僵硬，但是activity就是这样将自己的责任和显示区域交给fragment。而对于我们开发者来讲，fragment所管理的显示区域就应该通过oncreateView()和onViewCreated()这两个方法来进行构造：  

```

    @Nullable
    @Override
    public View onCreateView(@NonNull LayoutInflater inflater, @Nullable ViewGroup container, @Nullable Bundle savedInstanceState) {
        View view = inflater.inflate(R.layout.fragment_a,container,false);
        return view;
    }

    @Override
    public void onViewCreated(@NonNull View view, @Nullable Bundle savedInstanceState) {
        super.onViewCreated(view, savedInstanceState);
        Log.d(TAG, "onViewCreated: ");
    }
```  

`onCreateView()`一般用来初始化需要显示的视图，只需要将它进行返回就好，它不必被添加到所所传入的container对象(add中所提供id的ViewGroup)中，其实如果被添加了，反而会报出View重复添加的错误。
`onViewCreate()`则被用来完成剩余的View初始化工作，会紧跟着被调用。

# Fragment生命周期  
按照惯例，我也先来上一张经典的生命周期图：  

![Fragment生命周期](https://user-gold-cdn.xitu.io/2018/9/27/1661ad177faa96b8)  

如果按照正常的Framgent使用逻辑来看的话，Fragment将会按照上图的顺序进行执行，但是问题往往不止这么简单，比如说，发生attach、hide、show等操作时，Fragment的生命周期该如何调用呢？

何为生命周期？我的理解是到了特定的时候（或特定的事件）去调用特定的方法，Activity的生命周期是由系统进行管理和调用的，但是Fragment的生命周期则是由托管的Activity进行统一的调用和管理。总之，Fragment依赖Activity，而Activity持有Fragment的引用和管理权限，一般来讲，这种特殊身份都是由Activity所持有的内部FragmentManager对象而缔造的（它的内部维护着持有各Fragment引用的列表）。《Android权威编程指南》一书中提到了一个很重要的概念：
> 在activity处于运行状态时，添加fragment会发生什么呢?这种情况下，FragmentManager立 即驱赶fragment，调用一系列必要的生命周期方法，快速跟上activity的步伐(与activity的最新状 态保持同步)。  

或许在亲自运行过demo后，能够深有体会，不再赘述。  

# 不同操作情况下的生命周期   
之前看过很多博客，都有提到在不同transaction操作Fragment状态 情况下，都会不同程度地对当前Fragment的生命周期产生影响，但是某些博主并没有阐述清楚，而且凭借主观臆测就下结论，极其容易给读者们造成误导，我在保证不出错的情况下，还是建议大家能够结合上传的github源码进行实际情况运行分析，亲自得到结果，那么，在这一小节，我会展示Fragment生命周期变化的场景。

## add()和remove()对应生命周期
如果进行add操作，将会完整地执行fragment生命周期的前半部分，如下：

```
    @Override
    public void onClick(View view) {
        switch (view.getId()) {
            case R.id.btn_fragmentA:
                mFragmentTransaction.add(R.id.fl_contrainer, mFragmentA).commit();
                break;
            case R.id.btn_fragmentB:
                mFragmentTransaction.add(R.id.fl_contrainer, mFragmentB).commit();
                break;
            case R.id.btn_fragmentC:
                mFragmentTransaction.add(R.id.fl_contrainer, mFragmentC).commit();
                break;
        }
    }
```

![](https://user-gold-cdn.xitu.io/2018/10/2/16634ef243cb9d09?w=1068&h=289&f=png&s=45065)

在进行了add并commit事务过后，紧接着执行remove，会依次调用以下生命周期：

![](https://user-gold-cdn.xitu.io/2018/10/2/16634f4b1d7ee539)

且每add不一样的Fragment，这个Fragment也会按照自己定义的生命周期去执行，这并没有什么难度，但是假设如果在FragmentA的oncreateView过程中去addFragmentB呢？直接看结果吧：  

![](https://user-gold-cdn.xitu.io/2018/10/2/16634f982ee677aa?w=990&h=449&f=png&s=77739)

（相关代码在demo中被注释掉）如果你还记得我在上文小字中提到的“驱赶”一类的结论，你就会发现，这种奇葩的add情况，就是快速地初始化完Fragment，尽可能地赶上Activity地生命周期，是不是很奇妙？ 
当然，这里需要提醒的是，Fragment之中也会存在着相应的子Fragment管理类，获取方法为getChildFragmentManager()，其余操作和Activity的Fragment管理类相同，切记。  

## replace()生命周期  
既然是replace的操作，必然和两个Fragment有关，demo中是FragmentA先被add，然后被FragmentB替换掉：

![](https://user-gold-cdn.xitu.io/2018/10/2/1663502128f3100e?w=1023&h=596&f=png&s=87288) 

陪大家一起分析一下，从这个log可以看出两个问题：

1.被替换的Fragment A将走完所有的生命周期，即被销毁。  
2.Fragment B的创建周期调用要早于FragmentA的销毁周期调用。 

首先，第一点，被替换的Fragment最终会被销毁，第二点，为什么FragmentB的创建要早于FragmentA的销毁呢？其实startActivity()也有类似的道理(先创建新Activity，再销毁旧Activity)，这是因为创建后再销毁，可以避免出现视图空白。  

## show()和hide()的生命周期  
show和hide的前提是该Fragment已经被添加并且关联，在此操作的基础上，我们来研究它的生命周期变化：  

hide:
![](https://user-gold-cdn.xitu.io/2018/10/2/166350a909d69cb3?w=838&h=219&f=png&s=14074)  
show:
![](https://user-gold-cdn.xitu.io/2018/10/2/166350a909d69cb3?w=838&h=219&f=png&s=14074)  
(字呢！！！！！)
我没逗大家 = =，之所以没有字，那是因为.....
show和hide操作压根就对Fragment生命周期没影响！！！！！！！！！  
show和hide操作压根就对Fragment生命周期没影响！！！！！！！！！  
show和hide操作压根就对Fragment生命周期没影响！！！！！！！！！

重要的事情说三遍，over  

## detach和attach生命周期  
detach会销毁Fragment的视图，但是并不会销毁实例

![](https://user-gold-cdn.xitu.io/2018/10/2/166351062ffd18fa?w=1024&h=187&f=png&s=25357)  

同理，attach只是会恢复Fragment的视图，并不会执行完整的create流程：  

![](https://user-gold-cdn.xitu.io/2018/10/2/16635125095a4b9e?w=979&h=231&f=png&s=32904)  

惊不惊喜，意不意外？？？

好了，基本的知识就介绍到这，更多的骚操作，可以自己查看代码。  

# 总结
Fragment的生命周期是关于Fragment最为重要的知识点之一，在实际的coding过程中，也极容易在这一块内容翻跟头，所以需要十分重视，本文先简要地总结了一下Fragmnet所支持的基本操作，然后对各种操作下的生命周期变化情况进行了详细的讲解。  

> 友情提示：运行github给出的代码，点击不同的按钮可能会出现崩溃的情况，这是因为前置操作并未完成或者其他一些不允许的操作发生，建议大家自行查看崩溃日志，加深印象！

[Github代码](https://github.com/yourzeromax/Test_Fragment)

欢迎关注
[我的博客](https://yourzeromax.top/)

