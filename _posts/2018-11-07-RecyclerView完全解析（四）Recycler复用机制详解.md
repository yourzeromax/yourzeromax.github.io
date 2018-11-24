---
layout:     post
title:      RecyclerView完全解析（四）
subtitle:   回顾总结Recycler的复用机制
date:       2018-11-07
author:     yourzeromax
header-img: img/post-bg-android.jpg
catalog: true
tags:
    - Android
--- 

# 写在前面 

> 我自己都记不清本系列的上一篇文章是多久写的了

言之，对于RecyclerView的一般使用来说并不用去研究内部的复用机制，因为它已经封装的十分完美，但是往往优秀的Android需求会有各种千奇百怪的开发需求，就比如说一个RecyclerView列表中每个Item还嵌套了RecyclerView....

这时候如果不去研究复用机制，不采取一定的优化措施的话，App的卡顿程度将呈现明显上升的趋势，所以这也是本文讲述的重点。当然，我自己也没有把RecyclerView的一整套复用机制吃透，本文也是读取了大量他人的博客所总结出来的结果，先贴这位大佬的分析文章，写的相当精彩：  
 
 [基于情景分析RecyclerView的回收复用机制原理](https://www.cnblogs.com/dasusu/p/7746946.html)  
 
 讲道理，我并不认为我写的这篇文章能够概括所有的复用原理，但是至少本文能够为读者理清楚一些关键的要点（很多博客都未阐述清楚，甚至有一些误导），还是推荐在阅读这文章之前或者之后都去看看相关的博客。  
 
 文章内容分为三个板块：  
1.  关键知识要点介绍
2.  `ViewHolder`的复用原理
3.  `ViewHolder`的回收原理和流程  

  
# 关键知识要点  

### 类和相关数据结构  
`Recycler`、`RecycledViewPool`和`ViewCacheExtension`   
`Recycler`是其中最重要的内部类，与复用有关的数据结构如下：  

变量|	作用
---|---
mChangedScrap|	与RecyclerView分离的ViewHolder列表
mAttachedScrap|	未与RecyclerView分离的ViewHolder列表
mCachedViews|	ViewHolder缓存列表
mViewCacheExtension	|开发者可以控制的ViewHolder缓存的帮助类
mRecyclerPool|	ViewHolder缓存池

`RecyclerView`复用的并非单个`View`，而是整个`ViewHolder`,换言之，以上的这些数据结构都是存储的`ViewHolder` 
其中，前三个都存在于`Recycler`这个内部类之中，查看源码便可知：  
```
//RecyclerView.class
    public final class Recycler {
        final ArrayList<ViewHolder> mAttachedScrap = new ArrayList<>();
        ArrayList<ViewHolder> mChangedScrap = null;

        final ArrayList<ViewHolder> mCachedViews = new ArrayList<ViewHolder>();
        ······
        }
```
这三个呢，`mChangedScrap`和`mAttachedScrap`在我的理解之下更多的是和RecyclerView自身测量、布局等流程有关（RecyclerView在显示之前会经历很多次重新移除ViewHolder、再重新添加的过程），我们一般不用去考虑，当然，它俩的区别在于：  

```
        //RecyclerView.java
        void scrapView(View view) {
            final ViewHolder holder = getChildViewHolderInt(view);
            if (holder.hasAnyOfTheFlags(ViewHolder.FLAG_REMOVED | ViewHolder.FLAG_INVALID)
                    || !holder.isUpdated() || canReuseUpdatedViewHolder(holder)) {
                if (holder.isInvalid() && !holder.isRemoved() && !mAdapter.hasStableIds()) {
                    throw new IllegalArgumentException("Called scrap view with an invalid view."
                            + " Invalid views cannot be reused from scrap, they should rebound from"
                            + " recycler pool." + exceptionLabel());
                }
                holder.setScrapContainer(this, false);
                mAttachedScrap.add(holder);
            } else {
                if (mChangedScrap == null) {
                    mChangedScrap = new ArrayList<ViewHolder>();
                }
                holder.setScrapContainer(this, true);
                mChangedScrap.add(holder);
            }
        }
```
if()中的判断条件已经指出了，值得注意的是，`Viewholder`可能不会显示，甚至是隐藏，但是并不代表它从RecyclerView中被dettach掉！  

至于`mCachedViews`这个数据结构，很多人在博客中讲的云里雾里的...说它是什么状态没有被改变但是被移出视图，未被回收的View集合，其实没必要这么复杂，直接一句话就能说清楚了：

> `mCachedViews`将会把滑动过程中刚移出的视图存储，下次反向滑动时能够直接被使用。

`ViewCacheExtension`一般不怎么用，直接丢了吧。 `mRecyclerPool`顾名思义，是一个能够存储可复用View的池子，它可以被多个RecyclerView共享，并且RecyclerView内部的代码也会自动判断是否进行创建。  

因此可以看出，真正与RecyclerVIew复用有关系的数据结构就只有后面的三种，而抛开没用过的，也就只有两种了。

# 源码解析
从RecyclerView内部工作流程来看，源码解析可以分成两个步骤进行讲解，一个是复用流程、一个是回收流程。

### 复用流程  
各位看官先别急，`RecycerView`的源码非常多，没必要全部去了解，所以我告诉大家，复用机制的调用入口是`getViewForPosition()`方法  
```
//Recycler.class
    public View getViewForPosition(int position) {
            return getViewForPosition(position, false);
        }

    View getViewForPosition(int position, boolean dryRun) {
        return tryGetViewHolderForPositionByDeadline(position, dryRun, FOREVER_NS).itemView;
        }
```

所以复用机制的重点就是在`tryGetViewHolderForPositionByDeadline()`方法之中：  

```
        ViewHolder tryGetViewHolderForPositionByDeadline(int position,
                boolean dryRun, long deadlineNs) {
                     ...
            // 0) If there is a changed scrap, try to find from there
            if (mState.isPreLayout()) {
                holder = getChangedScrapViewForPosition(position);
                fromScrapOrHiddenOrCache = holder != null;
            }
            
            
            // 1) Find by position from scrap/hidden list/cache
            if (holder == null) {
                holder = getScrapOrHiddenOrCachedHolderForPosition(position, dryRun);
                     ...
            }
                     ...
            
            // 2) Find from scrap/cache via stable ids, if exists
                if (mAdapter.hasStableIds()) {
                    holder = getScrapOrCachedViewForId(mAdapter.getItemId(offsetPosition),
                            type, dryRun);
                     ...
                }
                
                
            // 3) views from mViewCacheExtension
                if (holder == null && mViewCacheExtension != null) {
                    // We are NOT sending the offsetPosition because LayoutManager does not
                    // know it.
                    final View view = mViewCacheExtension
                            .getViewForPositionAndType(this, position, type);
                    if (view != null) {
                        holder = getChildViewHolder(view);
                    ...
                    }
                    ...
                    
                    
            // 4)
                if (holder == null) { // fallback to pool
                   ...
                    holder = getRecycledViewPool().getRecycledView(type);
                   ...
                }
                ...
                
                
            // 5)
                if (holder == null) {
                    ...
                    holder = mAdapter.createViewHolder(RecyclerView.this, type);
                    ...
                }
                
                }
            
            // 6)
                if (mState.isPreLayout() && holder.isBound()) {
                // do not update unless we absolutely have to.
                holder.mPreLayoutPosition = position;
            } else if (!holder.isBound() || holder.needsUpdate() || holder.isInvalid()) {
                    ...
                final int offsetPosition = mAdapterHelper.findPositionOffset(position);
                bound = tryBindViewHolderByDeadline(holder, offsetPosition, position, deadlineNs);
            }

            final ViewGroup.LayoutParams lp = holder.itemView.getLayoutParams();
            final LayoutParams rvLayoutParams;
            if (lp == null) {
                rvLayoutParams = (LayoutParams) generateDefaultLayoutParams();
                holder.itemView.setLayoutParams(rvLayoutParams);
            } else if (!checkLayoutParams(lp)) {
                rvLayoutParams = (LayoutParams) generateLayoutParams(lp);
                holder.itemView.setLayoutParams(rvLayoutParams);
            } else {
                rvLayoutParams = (LayoutParams) lp;
            }
            rvLayoutParams.mViewHolder = holder;
            rvLayoutParams.mPendingInvalidate = fromScrapOrHiddenOrCache && bound;
            return holder;
        }
```
具体的代码非常长，我也帮大家省略了没必要去仔细深究的代码片段（当然，还是值得亲自去查看源码，了解一些），并且列出了关键的步骤，其中0-1步可以简单地理解为`RecyclerView`在布局过程中的几个过程，步骤2是判断是否Cache池里面有相应的holder对象，步骤3是从`mViewCacheExtension`中寻找对象，步骤4是从`RecycledViewPool`中寻找，如果前面的步骤都没有需要的holder对象，才会调步骤5，用Adapter的`createViewHolder`方法，去创建一个holder。最后的步骤6则是根据不同的ViewHolderFlags情况（是否可直接复用等等）来决定是否执行`tryBindViewHolderByDeadline()`方法，而它则是著名`onBindViewHolder()`方法的唯一入口。懂我的意思吧？

> 如果有使用图片三级缓存的经历，那么就会发现这个过程非常类似！它会一步一步地去寻找自己需要的holder对象，只有在上一个步骤不满足自己的需求时，才会进行次一级的搜索，这种代码思想非常值得我们学习，也可以说是我在看源码过程之中最大的收获！ 

#### getRecycledViewPool()  
每一个步骤都调用了对应的函数，还是值得去看一看，这里举`getRecycledViewPool()`，这个相对比较常用的方法来说一说：  
```
        //Recycler.class
        RecycledViewPool getRecycledViewPool() {
            if (mRecyclerPool == null) {
                mRecyclerPool = new RecycledViewPool();
            }
            return mRecyclerPool;
        }
        
                void setRecyclerView(RecyclerView recyclerView) {
            if (recyclerView == null) {
                mRecyclerView = null;
                mChildHelper = null;
                mWidth = 0;
                mHeight = 0;
            } else {
                mRecyclerView = recyclerView;
                mChildHelper = recyclerView.mChildHelper;
                mWidth = recyclerView.getWidth();
                mHeight = recyclerView.getHeight();
            }
            mWidthMode = MeasureSpec.EXACTLY;
            mHeightMode = MeasureSpec.EXACTLY;
        }
```
这也是RecyclerVIewPool可以公用的原因。再次为Google的工程师们点个赞！  

### 回收流程  
既然有复用，那么必定有回收。RecyclerView的回收工作入口是：`recycleViewHolderInternal()`  ,请看简化后的代码：
```
    void recycleViewHolderInternal(ViewHolder holder) {
    //省略很多判断条件
    ...
    
         if (forceRecycle || holder.isRecyclable()) {
          // 1) 判断mCachedViews是否装满
                if (mViewCacheMax > 0
                        && !holder.hasAnyOfTheFlags(ViewHolder.FLAG_INVALID
                        | ViewHolder.FLAG_REMOVED
                        | ViewHolder.FLAG_UPDATE
                        | ViewHolder.FLAG_ADAPTER_POSITION_UNKNOWN)) {
                    // Retire oldest cached view
                    int cachedViewSize = mCachedViews.size();
                    if (cachedViewSize >= mViewCacheMax && cachedViewSize > 0) {
                        recycleCachedViewAt(0);
                        cachedViewSize--;
                    }

                    int targetCacheIndex = cachedViewSize;
                    if (ALLOW_THREAD_GAP_WORK
                            && cachedViewSize > 0
                            && !mPrefetchRegistry.lastPrefetchIncludedPosition(holder.mPosition)) {
                        // when adding the view, skip past most recently prefetched views
                        int cacheIndex = cachedViewSize - 1;
                        while (cacheIndex >= 0) {
                            int cachedPos = mCachedViews.get(cacheIndex).mPosition;
                            if (!mPrefetchRegistry.lastPrefetchIncludedPosition(cachedPos)) {
                                break;
                            }
                            cacheIndex--;
                        }
                        targetCacheIndex = cacheIndex + 1;
                    }
                    mCachedViews.add(targetCacheIndex, holder);
                    cached = true;
                }
                if (!cached) {
                    addViewHolderToRecycledViewPool(holder, true);
                    recycled = true;
                }
            } else {
                ...
            }
            mViewInfoStore.removeViewHolder(holder);
            if (!cached && !recycled && transientStatePreventsRecycling) {
                holder.mOwnerRecyclerView = null;
            }
        }
    }
    
    
    void recycleCachedViewAt(int cachedViewIndex) {
        ...
            ViewHolder viewHolder = mCachedViews.get(cachedViewIndex);
            if (DEBUG) {
                Log.d(TAG, "CachedViewHolder to be recycled: " + viewHolder);
            }
            addViewHolderToRecycledViewPool(viewHolder, true);
            mCachedViews.remove(cachedViewIndex);
        }
```

回收机制中，最最最重要的除了之前很多个被省略的判断条件以外，就是判断是否mCacheViews是否被装满，然后将mCacheViews中的第一个视图通过`addViewHolderToRecycledViewPool()`方法移到`RecyclerViewPool`之中，存放新的缓存视图。当然，如果后者都已经被装满了，就会丢弃，因为也用不到这么多缓存视图，对吧？ 可以看出，回收机制还是相对比较简单的。

# 总结
`RecyclerView`的回收机制被设计地非常经典，它的工作流程可以被分为复用和回收两个步骤，同时，在整个流程之中，最为重要的数据结构有两个：`mCachedViews`和`RecyclerViewPool`，设计思想采用了类似图片三级缓存的原理，能够在保证完成功能的同时，保持住最佳的性能表现。最后，我还是建议大家带着问题去看一下源码和本文开始推荐的博客文章，进一步加深印象，然后你就发现，其实复用机制原来这么简单！

# 题外话 
一般来说，查看源码有三个功能：1.应付面试官。2.学习源码的设计模式。3.从源码的角度分析问题和进行优化。如果是应届生，多看看常用的源码就行了，不用过分钻研；但是对于已经工作的程序员来说，如果不能汲取源码的设计思想，那么这个源码看的挺失败的~当然，项目中遇到问题了，深入源码设计找到问题的根本也是非常重要的能力之一，如果是优化的话，至少从目前来看，大多数发现可以优化的地方都是看公司前辈写的代码或者是博客才知道的~哈哈哈。

> 我才不会告诉大家，这篇博客我写了半个月。

