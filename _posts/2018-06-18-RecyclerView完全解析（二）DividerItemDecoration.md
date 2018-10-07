---
layout:     post
title:      RecyclerView完全解析（二）
subtitle:   RecyclerViewDividerItemDecoration及骚操作
date:       2018-06-18
author:     yourzeromax
header-img: img/post-bg-android.jpg
catalog: true
tags:
    - Android
--- 

> 今天是端午节，本来是说能在毕业前往北京之前回家团圆的，结果因故未能离开，有些遗憾。

>  2018.10.5日，结合上文的修改对本文进行相关修改。

# 引言  
上文介绍了RecyclerView的一些基础知识以及在使用RecyclerView所必要的Adapter、LayoutManager的“模块”配置问题，那么本文将会着重介绍另外一个可以让RecyclerView显示更加个性化的重要“模块”---DividerItemDecoration。并且将会结合实际的案例来撸一个漂亮的界面。  

Decoration，意思是装饰物，足以看出它在RecyclerView中与一些非Item视图的装饰有关，一般来说都是用来作为Item分割的间隔线。如果回看上一篇博客的话，会发现没有设置ItemDecoration就会导致Item之间没有间隔线，而我们如果加上：  

```
        // 设置Item之间间隔样式
 mRecyclerView.addItemDecoration(new DividerItemDecoration(this,1));
```
则能得到下面的效果：  

![Demo](https://raw.githubusercontent.com/yourzeromax/yourzeromax.github.io/master/img/20180618/20180618-1.png) 

当然，添加一条简单的线只能证明ItemDecoration能够完成分割的功能，但是还不足以表现出它的强大，所以请继续往下看。  

# ItemDecoration的基本用法  
自定义ItemDecoration需要继承ItemDecoration，上文所使用的`new DividerItemDecoration(this,1)`其实是RecyclerView包里面已经实现好的ItemDecoration，翻看源代码会发现也是继承自ItemDecoration，它是一个没有提供抽象方法的抽象类（就是为了阻止直接创建ItemDecoration的实例），主要关注并重写它的三个方法：


方法 | 备注
---|---
getItemOffsets()|设置Item偏移
onDraw() | 绘制底层视图的方法
onDrawOver() | 绘制最上层视图的方法


这里解释一下，RecyclerView是一个ViewGroup，按照Android视图绘制机制，不光要先绘制ViewGroup自身，之后也需要绘制ViewGroup中的各子View，而对于RecyclerView来说，子View就是一个个的Item，相信大家都用过PhotoShop，其中有个很重要的概念叫做图层，而ItemDecoration有些类似图层的思想，按照绘制先后顺序，从下往上依次为：  

```
itemDecoration中的onDraw--->子Item的onDraw--->itemDecoration中的onDrawOver
```
### getItemOffsets()  
先看参数表和描述：  

参数序号|参数类型| 描述
---|---|---
1 | Rect | 可设置left、top、right、bottom的偏移量
2 | View | item的View
3 | RecyclerView | 所设置的RecyclerView本身
4 | State | RecyclerView的状态，暂时忽略  

一般来说项目中这样重写它：  

```
    @Override
    public void getItemOffsets(@NonNull Rect outRect, @NonNull View view, @NonNull RecyclerView parent, @NonNull RecyclerView.State state) {
        outRect.set(20,0,20,20);

        //以下设置也可以：
        //outRect.left = 20;
       // outRect.right = 20;
        //outRect.top = 0;
       // outRect.left = 20;
    }
```
为了查看效果，我们暂时将item_sample.xml的背景调为灰色，效果图如下：  

![Demo](https://raw.githubusercontent.com/yourzeromax/yourzeromax.github.io/master/img/20180618/20180618-2.png) 

非常简单。但这只是第一步。

### onDraw() 

参数序号|参数类型| 描述
---|---|---
1 | Canvas | RecyclerView的画布对象，接收绘画操作
2 | RecyclerView | 所设置的RecyclerView本身
3 | State | RecyclerView的状态，暂时忽略 

我们这样来重写：  

```
    @Override
    public void onDraw(@NonNull Canvas c, @NonNull RecyclerView parent, @NonNull RecyclerView.State state) {
        int childCount = parent.getChildCount();
        int left = parent.getPaddingLeft();
        int right = parent.getWidth() - parent.getPaddingRight();

        for (int i = 0; i < childCount - 1; i++) {
            View view = parent.getChildAt(i);
            float top = view.getBottom();
            float bottom = view.getBottom() + 20;
            c.drawRect(left, top, right, bottom, mPaint);
        }
    }
```

其实就是在canvas上面绘制相应的底层内容，Paint画笔对象可以在构造函数中进行初始化：  
```
    public CustomDecoration() {
        mPaint = new Paint();
        mPaint.setColor(Color.RED);
    }
```

我们来看看效果：  

![Demo](https://raw.githubusercontent.com/yourzeromax/yourzeromax.github.io/master/img/20180618/20180618-3.png) 

**注意事项：**  
这里的canvas其实是RecyclerView对象的背景画布，所以说它的绘制位置需要先获得子View的位置，然后再进行相关的绘制，可看代码思考一下。  

### onDrawOver()  
从方法执行机制和流程来说，`onDrawOver()`和`onDraw()`并没有区别，唯一的区别就是前者是再子View绘制完毕后再执行，而后者是在子View绘制完毕之前，所以才使得`onDrawOvewr`能够绘制最上层的装饰物，所以它的参数及描述都和onDraw()一致，就不再重复啦，直接看代码吧：

```
    @Override
    public void onDrawOver(@NonNull Canvas c, @NonNull RecyclerView parent, @NonNull RecyclerView.State state) {
        super.onDrawOver(c, parent, state);
        int childCount = parent.getChildCount();
        for (int i = 0; i < childCount; i++) {
            View child = parent.getChildAt(i);
            float right = child.getRight() - 20;
            float left = right - mTagWidth;
            float top = child.getTop();
            float bottom = child.getBottom() - 40;
            c.drawRect(left, top, right, bottom, mTagPaint);
        }
    }
```

再贴上效果图：  

![Demo](https://raw.githubusercontent.com/yourzeromax/yourzeromax.github.io/master/img/20180618/20180618-4.png) 

这种效果可以很轻松地联想到淘宝优惠打折的上标签，是吧？一不小心就实现了这个功能，想想还是有些激动呢。  

以上就是自定义ItemDecoration需要复写的所有方法，说起来也是很简单，但是真正的难点在于，如果去对需要的视图布局，而且需要相关的Canvas、Paint绘制基础以及Adapter获取子View位置的方法。  

# 从概念到实践---撸一个StickyHeader  
这才是本文的重点！也是一个非常炫酷的轮子，今天就带大家来新造一个。
先来看一看效果图：  



实现这些功能，我们新建一个`StickyDecoration.class`，有以下几个需要完成的地方：

- 实现数据分组，并实现Header
- 封装分组信息并实现相应方法，用于Item分组信息判断
- 提供分组信息获取接口
- 重写ItemDecoration提供的三个方法

### GroupInfo类  

```
public class GroupInfo {
    //组号
    private int mGroupID;
    // 每一个组的hader title
    private String mTitle;
    //每个Item在当前组的位置，用于判断是否为第一个；
    private int position;

    // 一个组的item数量
    private int mGroupCount;


    public GroupInfo(int groupId, String title) {
        this.mGroupID = groupId;
        this.mTitle = title;
    }

    public int getGroupID() {
        return mGroupID;
    }

    public void setGroupID(int groupID) {
        this.mGroupID = groupID;
    }

    public String getTitle() {
        return mTitle;
    }

    public void setTitle(String title) {
        this.mTitle = title;
    }

    public void setPosition(int position) {
        this.position = position;
    }

    public void setGroupCount(int count) {
        this.mGroupCount = count;
    }

    /**
     * 判断是否是组内第一个，如果是的话则绘制Header
     *
     * @return
     */
    public boolean isFirstInGroup() {
        return position == 0;
    }

    /**
     * 判断是否是组内最后一个
     *
     * @return
     */
    public boolean isLastInGroup() {
        return position == mGroupCount - 1 && position >= 0;
    }

}
```

### CallBack
由于需要根据当前position来获取item的分组信息，所以为了方便，我们可以在CustomDecoration之中加入一个接口,并且在`StickyDecoration`的构造方法之中传入一个实现好的接口：  

```
StickyDecoration.class

public class StickyDecoration extends RecyclerView.ItemDecoration{
    CallBack mCallBack;

    public StickyDecoration(CallBack mCallback) {
        this.mCallBack = mCallback;
    }

    public interface CallBack {
        void getGroupInfo(int position);
    }
}

MainActivity.class

        mRecyclerView.addItemDecoration(new StickyDecoration(new StickyDecoration.CallBack() {
            @Override
            public GroupInfo getGroupInfo(int position) {
                /**
                 * 在这里根据position来获得Item的GroupInfo描述
                 * 需要在项目中根据需求进行替换
                 */
                int groupId = position / 4;
                int index = position % 4;
                GroupInfo groupInfo = new GroupInfo(groupId,groupId+"");
                groupInfo.setPosition(index);
                return groupInfo;
            }
        }));
```

### 三个方法的重写
以上都是先行的准备，在这里才是重点，学习了如何重写三个方法过后，相信这个步骤也不会太难，但是仍然有些地方值得注意，先看代码：  

```
    @Override
    public void getItemOffsets(@NonNull Rect outRect, @NonNull View view, @NonNull RecyclerView parent, @NonNull RecyclerView.State state) {
        super.getItemOffsets(outRect, view, parent, state);
        int position = parent.getChildAdapterPosition(view);

        if (mCallback != null) {
            GroupInfo groupInfo = mCallback.getGroupInfo(position);

            //如果是组内的第一个,则需要预留一个Header的高度，否则预留分割线的高度
            if (groupInfo != null && groupInfo.isFirstInGroup()) {
                outRect.top = mHeaderHeight;
            } else {
                outRect.top = mDividerHeight;
            }
        }
    }
```
在`getItemOffsets()`方法中，主要注意的是判断是否是组内第一个item，根据返回结果来选择预留高度

`onDraw()`方法其实完全可以被`onDrawOver()`代替，StickyHeader的效果使得Header能够动态的变化，因此`onDraw()`绘制底层的视图并不适合，所以可以选择不重写。


```
    @Override
    public void onDrawOver(@NonNull Canvas c, @NonNull RecyclerView parent, @NonNull RecyclerView.State state) {
        super.onDrawOver(c, parent, state);

        int childCount = parent.getChildCount();
        for (int i = 0; i < childCount; i++) {
            View view = parent.getChildAt(i);
            int index = parent.getChildAdapterPosition(view);
            if (mCallback != null) {
                GroupInfo groupinfo = mCallback.getGroupInfo(index);
                int left = parent.getPaddingLeft();
                int right = parent.getWidth() - parent.getPaddingRight();

                //屏幕上第一个可见的 ItemView 时，i == 0;
                if (i != 0) {
                    //只有组内的第一个ItemView之上才绘制
                    if (groupinfo.isFirstInGroup()) {
                        int top = view.getTop() - mHeaderHeight;
                        int bottom = view.getTop();
                        drawHeaderRect(c, groupinfo, left, top, right, bottom);
                    }
                } else {
                    //如果是屏幕上的第一个View，则必须绘制Header，如果是当前组内的最后一个View，则它的Header需要进行特殊处理
                    int top = parent.getPaddingTop();
                    if (groupinfo.isLastInGroup()) {
                        int suggestTop = view.getBottom() - mHeaderHeight;
                        // 当 ItemView 与 Header 底部平齐的时候，判断 Header 的顶部是否小于
                        // parent 顶部内容开始的位置，如果小于则对 Header.top 进行位置更新，
                        //否则将继续保持吸附在 parent 的顶部
                        if (suggestTop < top) {
                            top = suggestTop;
                        }
                    }
                    int bottom = top + mHeaderHeight;
                    drawHeaderRect(c, groupinfo, left, top, right, bottom);
                }

            }
        }
    }
```
`onDrawOver()`方法是复写起来最复杂的，同时也是StickyHeader最为核心的内容，捋一捋逻辑：  
1. 获取子View在Adapter数据源中的位置
2. 如果不是屏幕可见第一个Item，则按照普通的处理办法来进行处理。
3. 如果既是屏幕可见第一个，又是组内最后一个，则进行特殊处理。

最后附上效果图：  

![Demo](https://raw.githubusercontent.com/yourzeromax/yourzeromax.github.io/master/img/20180618/20180618-5.gif) 

# 总结

本文介绍了自定义ItemDecoration的基本用法，重中之重则是对ItemDecoration提供的三个方法根据业务逻辑来进行重写， 之后也重新造了一个非常炫酷的StickHeader，简要分析了实现流程和逻辑。

欢迎关注   
[我的博客](https://yourzeromax.top/)