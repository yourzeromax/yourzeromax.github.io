---
layout:     post
title:      RecyclerView完全解析（一）
subtitle:   RecyclerView的基础知识
date:       2018-06-15
author:     yourzeromax
header-img: img/post-bg-android.jpg
catalog: true
tags:
    - Android
---

> 临近毕业，心中感慨万分，毕业论文也写的差不多了，有空来撸一下对RecyclerView的总结。

> 2018.10.5日，重新上传代码，同时修改博客中部分内容错误

# 引言
RecyclerView是什么？这个问题一搜一大把，总之它就是Google提供给大家用来替换ListView或者是GridView的一个新控件，虽然在最开始接触Android开发，学习郭神的《第一行代码》中RecyclerView部分的时候，感觉它的使用非常麻烦，并且容易出错，但是随着开发经验的不断增加，才发现RecyclerView真的是一个神级控件，尤其是在拜读了相关的源码过后，对Google的工程师们佩服的更加五体投地。    


# RecyclerView的优点

## 功能解耦彻底
既然RecyclerView是ListView的替代品，那么就不得不提一下它们的对比。如果读者是一名Android初学者，请相信我，RecyclerView一定比ListView好用，因为它的设计能够达到完全解耦的目的，换言之，它将它的视图表现形式分割成几个不同的“模块”，再统一进行组装：  

- Adapter进行Item视图的控制
- LayoutManager进行Item排列顺序控制
- ItemAnimator进行Item动画控制
- ItemDecoration进行Item间隔样式控制
- ItemTouchHelper控制Item拖拽或侧滑表现

看起来RecyclerView的内容比ListView多很多，但实际上在运用的过程中，RecyclerView的使用逻辑非常清晰，并且能够做到很好的二次封装和样式修改，本系列博客也主要是总结RecyclerView使用过程中会出现的所有情况。  

## 复用优化  
Recycler一词代表着“回收，复用”，从名字上就可以看出RecyclerView之中一定使用了View复用的机制，它的复用机制比ListView更加复杂，同时也更加高效，更加适合我们用到自己的项目之中，但是并非所有复用都是百利无一害的，复用机制虽然能够缓解Android的渲染压力，但是也可能会给我们带来一些意想不到的麻烦：**View错位等问题**，在之后的博客之中会详细分析这些情况并且给出解决方案，初学者可暂时不用考虑。

# 如何使用RecyclerView  
首先应该导入包：

```
    implementation 'com.android.support:recyclerview-v7:28.0.0'
```
我记得哪个sdk版本以前是集成到v7兼容包的，之后需要单独导入。  

RecyclerView可以看做成一个ViewGroup，因此，xml添加它也并没有什么差别：  


```
activity_main.xml

<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">

    <android.support.v7.widget.RecyclerView
        android:id="@+id/my_recycler_view"
        android:layout_width="match_parent"
        android:layout_height="match_parent" />

</LinearLayout>
```

然后可以按照下面的代码分别添加RecyclerView需要的“模块”：

```
//初始化RecyclerView
mRecyclerView = (RecyclerView) findViewById(R.id.my_recycler_view);

// 设置adapter
mRecyclerView.setAdapter(mAdapter);
// 设置布局管理器
mRecyclerView.setLayoutManager(mLayoutManager);

// 设置Item添加和移除的动画
mRecyclerView.setItemAnimator(new DefaultItemAnimator());
// 设置Item之间间隔样式
mRecyclerView.addItemDecoration(mDividerItemDecoration);

```
当然上述的“模块”中，LayoutManager和Adapter是必须的，如果不添加，RecyclerView将不能正常显示，其余的不添加也不影响RecyclerView显示，但是就没有相应的功能。这样的解耦方式能够让我们着重于各类“模块”的独立开发，非常高效，同时也易于维护和替换。 

## Adapter
Adapter是RecyclerView最为重要的“模块”，不管采用内部匿名来的方式还是采用继承的方式，它又必须实现几个重要的方法：  

方法  | 描述
---|---
getItemCount() |  返回RecyclerView的Item数量 
onCreateViewHolder(ViewGroup viewGroup, int position)| 返回每个Item的Viewholder对象
onBindViewHolder(ViewHolder ViewHolder, int position)| 利用上文返回的viewholder进行视图初始化

RecyclerView只关心显示哪个Item而不会去关心该如何去显示Item的内容，它把这项任务分摊给了Adapter进行，因此Adapter除了需要数据源以外，还需要一个item的视图，我们可以单独定义一个xml文件： 

```
item_sample.xml

<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="60dp"
    android:orientation="vertical">

    <TextView
        android:id="@+id/tv_id"
        android:layout_width="match_parent"
        android:layout_height="wrap_content" />

    <TextView
        android:id="@+id/tv_name"
        android:layout_width="match_parent"
        android:layout_height="wrap_content" />
</LinearLayout>
```  
查看MyAdapter源码：  


```
public class MyAdapter extends RecyclerView.Adapter<MyAdapter.MyViewHolder> {
    Context mContext;
    List<Student> students;
    int layoutId;


    public MyAdapter(Context context, List<Student> students, int layoutId) {
        this.mContext = context;
        this.students = students;
        this.layoutId = layoutId;
    }

    @NonNull
    @Override
    public MyViewHolder onCreateViewHolder(@NonNull ViewGroup viewGroup, int i) {
        View view = LayoutInflater.from(mContext).inflate(layoutId, viewGroup, false);
        return new MyViewHolder(view);
    }

    @Override
    public void onBindViewHolder(@NonNull MyViewHolder myViewHolder, int i) {
        Student student = students.get(i);
        myViewHolder.tvId.setText(student.id+"");
        myViewHolder.tvName.setText(student.name);
    }

    @Override
    public int getItemCount() {
        return students.size();
    }

     static class MyViewHolder extends RecyclerView.ViewHolder {
        TextView tvId, tvName;

        private MyViewHolder(@NonNull View itemView) {
            super(itemView);
            this.tvId = itemView.findViewById(R.id.tv_id);
            this.tvName = itemView.findViewById(R.id.tv_name);
        }
    }
}

MainActivity.class 

    /**
     * 装填模拟数据
     */
    private void initLists(int num) {
        for (int i = 0; i < num; i++) {
            lists.add(new Student(20144200 + i, "name " + i));
        }
    }

    /**
     * 装填适配器Adapter
     */
    private void initAdapter() {
        mAdapter = new MyAdapter(this, students, R.layout.item_sample);
    }
```


**特别提示：**

`onCreateViewHolder()`中的第一个参数则是RecyclerView本身，第二个参数代表当前item的位置，它需要返回一个ViewHolder对象，这个对象需要继承自`RecyclerView.ViewHolder`,一般将ViewHolder作为Adapter的内部类（见demo代码）。  

`onBindViewHolder()`中的第一个参数就是`onCreateViewHolder()`所返回对应位置的Viewholder，这个方法之中需要对视图进行相应的初始化，比如设置监听器，设计TextView内容等等。

除了上述三个必须实现的方法以外，还有很多可以选择的方法，之后的博客会依次介绍。

## LayoutManager  
LayoutManager主要控制Item是如何排列的，前文已经列出如何为RecyclerView指定一个LayoutManager，那么这里将强调能够提供哪些LayoutManager以及它们的排列表现形式，RecyclerViewView一共为我们提供三种不同的LayoutManager：  

类名 | 描述
---|---
LinearLayoutManager | 线性排列，横向，纵向
StaggeredGridLayoutManager | 瀑布流排列
GridLayoutManager | 表格排列 

### LinearLayoutManager

虽然提供了两种构造方法，但是在实际使用之中，一般使用  

```
new LinearLayoutManager( Context context, int orientation, boolean reverseLayout)
```  

第一个参数为上下文环境，第二个参数为布局显示方式(HORIZONAL,VERTICAL)，第三个参数为布尔值是否反转,一般默认为false，表示依次排序，如果为true，则表示按照反顺序排序。  

### StaggeredGridLayoutManager
瀑布流排列其实是一种交错排列，

```
new StaggeredGridLayoutManager(int spanCount,int orientation)
```
第一个参数是有多少列，第二个参数代表布局显示方式(HORIZONAL,VERTICAL)。  

### GridLayoutManager
表格排列，类似于GridLayout布局，GridLayoutManager更像是LinearLayoutManager和StaggeredGridLayoutManager相互作用的一个布局管理器，可以设置列数，又可以设置显示方向以及设置页面的加载数据是否反转；  

```
new GridLayoutManager( Context context, int spanCount, int orientation, boolean reverseLayout)
```

LayoutManager也是可以自定义的，但是往往上述自带的三种管理器已经能够满足项目的需要，因此也不再赘述。 

# 获得当前Item位置  
RecyclerView同时管理很多个Item，并且一个数据源对应着一个Item，有时候需要根据数据源来对相应位置的item进行处理，可以用以下代码来获取：  

```
            View view = mRecyclerView.getChildAt(i);
            int index = mRecyclerView.getChildAdapterPosition(view);
```
其中i指的是当前屏幕可见的位置。


# 总结 
本文介绍了RecyclerView诞生的背景和用途，同时也简单地介绍了一下REcyclerView的优点，虽然RecyclerView用起来看似代码复杂，但是它的解耦性使得业务逻辑非常清晰，在针对不同“模块”的使用方法，也进行了必要的代码和文字结合。最后也详细分析了Adapter和LayoutManager这两个必须的“模块”的使用方法，其余功能可以查看接下来的博客。  

> 2018.10.5日更新，使用最新上传的RecyclerViewDemo代码  

[Github代码地址](https://github.com/yourzeromax/RecyclerViewDemo)

欢迎关注   
[我的博客](https://yourzeromax.top/)



