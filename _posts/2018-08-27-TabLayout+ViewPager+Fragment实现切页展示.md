---
layout:     post
title:      TabLayout+ViewPager+Fragment实现切页展示
subtitle:   最为常用的界面三驾马车使用方法
date:       2018-08-30
author:     yourzeromax
header-img: img/post-bg-android.jpg
catalog: true
tags:
    - Android
---     

# 写在前面  
![网易云音乐](https://raw.githubusercontent.com/yourzeromax/yourzeromax.github.io/master/img/20180827/20180827-1.jpg)  
目前大多数的APP都采用的是几个Tab标签以及多个界面滑动的形式来提供多层次的交互体验，最为常用的做法就是采用TabLayout+ViewPager+Fragment的方式，最近在公司项目中遇到类似的界面，也看了各个论坛很多份博客，但是发现都没有完全把这种方法的坑填完，因此写下这篇博客，一方面是对知识的总结，另一方面也能让其他开发者们少走一些弯路，博客内容主要分为四个章节：

 1. TabLayout+ViewPager+Fragment的简单用法总结。
 2. 所使用的两种PagerAdapter的差别分析及选择。
 3. 懒加载策略。
 4. 卡顿及性能优化建议。

一般情况下上面四个章节的内容足以应付过来，但是往往在一些特殊的情况下，仍然会遇到一些不能解决的问题，这时就需要深入到源码之中来具体问题具体分析。话不多说，接下来将进行使用总结。  

# TabLayout+ViewPager+Fragment的用法
首先，需要引入工具包：  

```
    implementation 'com.android.support:design:27.1.1'
    implementation 'com.android.support:support-v4:27.1.1'
```

用法其实非常简单，有点类似于RecyclerView，其中主要关心四个对象：Tablayout、ViewPager、PagerAdapter、Fragment。前两个就跟普通的View控件一样，可以直接通过XML来进行布局以及在onCreate获取相应的实例：  

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    tools:context=".activities.TabLayoutActivity">

    <android.support.design.widget.TabLayout
        android:id="@+id/tl_tabs"
        android:layout_width="match_parent"
        android:layout_height="40dp" />

    <android.support.v4.view.ViewPager
        android:id="@+id/vp_content"
        android:layout_width="match_parent"
        android:layout_height="match_parent" />
</LinearLayout>
``` 
Fragment建议采用v4兼容包下的，我们所需要使用的Fragment是需要自己来实现，但是和普通的Fragment没什么区别，因此也就省略了Fragment的创建步骤，而PagerAdapter有两种实现可以使用，具体会在下一小节介绍，TabLayout+ViewPager+Fragment方法的使用流程：  

 1. 创建存储多个Fragment实例的列表
 2. 创建PagerAdapter实例并关联到Viewpager中
 3. 将ViewPager关联到Tablayout中
 4. 根据需求改写Tablayout属性*

最后一步不是必须的，为了更加清楚地描述这个调用流程，贴上一个示意图：  
![这里写图片描述](https://raw.githubusercontent.com/yourzeromax/yourzeromax.github.io/master/img/20180827/20180827-2.png)  
  
  贴上代码：  
  

```
public class TabLayoutActivity extends AppCompatActivity implements MyFragment.OnFragmentInteractionListener {
    TabLayout tabLayout;
    ViewPager viewPager;
    
    List<Fragment> fragments = new ArrayList<>();
    List<String> titles = new ArrayList<>();

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_tab_layout);

        tabLayout = findViewById(R.id.tl_tabs);
        viewPager = findViewById(R.id.vp_content);

        fragments.add(MyFragment.newInstance("11111", "11111"));
        fragments.add(MyFragment.newInstance("22222", "22222"));
        fragments.add(MyFragment.newInstance("33333", "33333"));
        fragments.add(MyFragment.newInstance("44444", "44444"));
        fragments.add(MyFragment.newInstance("55555", "55555"));
        titles.add("fragment1");
        titles.add("fragment2");
        titles.add("fragment3");
        titles.add("fragment4");
        titles.add("fragment5");
        viewPager.setAdapter(new FragmentStatePagerAdapter(getSupportFragmentManager()) {
            @Override
            public Fragment getItem(int position) {
                return fragments.get(position);
            }

            @Override
            public int getCount() {
                return fragments.size();
            }

            @Override
            public void destroyItem(ViewGroup container, int position, Object object) {
                super.destroyItem(container, position, object);
            }

            @Nullable
            @Override
            public CharSequence getPageTitle(int position) {

                return titles.get(position);
            }
        });

        tabLayout.setupWithViewPager(viewPager);
    }

    @Override
    public void onFragmentInteraction(Uri uri) {

    }
}

```   
getPageTitle(int position)函数是返回当前TabLayout的标签标题的，当然，也可以不通过PagerAdapter中的这个函数返回，采用下面的这种方式也可行(有多少个就addTab多少次)：  

```
 tabLayout.addTab(tabLayout.newTab().setText("tab 1"));
```

# PagerAdapter  
PagerAdapter是一个抽象类，它有两个实现子类供我们使用，分别是FragmentStatePagerAdapter和FragmentPagerAdapter。创建这两个类的实例需要传入一个FragmentManager对象，像代码那样处理就行了，从类名就可以看出来它俩的最大差别就在“State-状态”上，什么意思呢？指的是所包含存储的Fragment对象的状态是否保存。看源码可以发现，FragmentStatePagerAdapter中比FragmentPagerAdapter多维护着两个列表：  
```
    private ArrayList<Fragment.SavedState> mSavedState = new ArrayList<Fragment.SavedState>();
    private ArrayList<Fragment> mFragments = new ArrayList<Fragment>();
``` 
而这两个列表带来的最大差别则体现在void destroyItem(ViewGroup container, int position, Object object)这个函数之中，看下FragmentStatePagerAdapter的函数源码：  

```
    @Override
    public void destroyItem(ViewGroup container, int position, Object object) {
        Fragment fragment = (Fragment) object;

        if (mCurTransaction == null) {
            mCurTransaction = mFragmentManager.beginTransaction();
        }
        if (DEBUG) Log.v(TAG, "Removing item #" + position + ": f=" + object
                + " v=" + ((Fragment)object).getView());
        while (mSavedState.size() <= position) {
            mSavedState.add(null);
        }
        mSavedState.set(position, fragment.isAdded()
                ? mFragmentManager.saveFragmentInstanceState(fragment) : null);
        mFragments.set(position, null);

        mCurTransaction.remove(fragment);
    }
``` 
再看下FragmentPagerAdapter的这个函数：  

```
   @Override
    public void destroyItem(ViewGroup container, int position, Object object) {
        if (mCurTransaction == null) {
            mCurTransaction = mFragmentManager.beginTransaction();
        }
        if (DEBUG) Log.v(TAG, "Detaching item #" + getItemId(position) + ": f=" + object
                + " v=" + ((Fragment)object).getView());
        mCurTransaction.detach((Fragment)object);
    }
``` 
具体情况就不再往下分析啦，还有一个坑等下再说。  
ViewPager还有一个比较重要的函数是：  

```
viewPager.setOffscreenPageLimit(int limit);
```
这个方法默认值为1，Google在开发ViewPager时，考虑到如果滑动的时候才创建Fragment实例时会带来一定程度的卡顿，因此为ViewPager设置了缓存机制，而上述函数则是设置缓存Fragment的数量，示意图如下：  
![这里写图片描述](https://raw.githubusercontent.com/yourzeromax/yourzeromax.github.io/master/img/20180827/20180827-3.png)
也就是说，limit的值代表着还要缓存当前Fragment左右各limit个Fragment，一共会创建2*limit+1个Fragment。超出这个limit范围的Fragment就会被销毁，而上述两种PagerAdapter的差别就是销毁的流程不同！  
这里就不放Log图给大家看，直接告诉大家，FragmentPagerAdapter在销毁Fragment时不会调用onDestroy（）方法，而带了State的Adapter则会调用Fragment的onDestroy()方法，换言之，前者仅仅是销毁了Fragment的View视图而没有销毁Fragment这个对象，但是后者则彻彻底底地消灭了Fragment对象，这是很重要的知识要点哦~！也是下面谈性能优化和懒加载的前提条件。

本小节最后，告诉大家一个关于如何选择PagerAdapter的结论：

> FragmentPagerAdapter适用于Fragment比较少的情况，它会把每一个Fragment保存在内存中，不用每次切换的时候，去保存现场，切换回来在重新创建，所以用户体验比较好。而对于Fragment比较多的情况，需要切换的时候销毁以前的Fragment以释放内存，就可以使用FragmentStatePagerAdapter。  

暂时不懂这句话的含义没关系，请接着往下面看。   
 
# 懒加载策略  
Android的View绘制流程是最消耗CPU时间片的操作，尤其是在ViewPager缓存Fragment的情况下，如果在View绘建的同时还进行多个Fragment的数据加载，那用户体验简直是爆炸（不仅浪费流量，而且还造成不必要的卡顿）。。。因此，需要对Fragment们进行懒加载策略。什么是懒加载？就是被动加载，当Fragment页面可见时，才从网络加载数据并显示出来。那什么时候Fragment可见呢？Fragment之中有这样一个函数：  

```
  @Override
    public void setUserVisibleHint(boolean isVisibleToUser) {
        super.setUserVisibleHint(isVisibleToUser);
        doYourJobs();
    }
``` 
当Fragment的可见状态发生变化时就会调用这个函数，boolean参数isVisibleToUser代表当前的Fragment是否可见。  
如果这么简单地调用函数就能实现懒加载的话，那也没什么好说的，但是这里又有一个巨坑，则是因为这个setUserVisibleHint函数是游离在Fragment生命周期之外的，它的执行有可能早于onCreate和onCreateView，然而既然要时间数据的加载，就必须要在onCreateView创建完视图过后才能使用，不然就会返回空指针崩溃，懒加载的重点也是在这儿，那么我们来分析，实行懒加载必须满足哪些条件呢？  

```
1.View视图加载完毕，即onCreateView（）执行完成
2.当前Fragment可见，即setUserVisibleHint（）的参数为true
3.初次加载，即防止多次滑动重复加载
```
有了这两个条件过后，便能够正常执行懒加载过程，我们在Fragment全局变量之中增加对应的三个标志参数并赋上初始值：  

```
boolean mIsPrepare = false;		//视图还没准备好
boolean mIsVisible= false;		//不可见
boolean mIsFirstLoad = true;	//第一次加载
``` 
当然在onCreateView中确保了View已经准备好时，将mPrepare置为true，在setUserVisibleHint中确保了当前可见时，mIsVisible置为true，第一次加载完毕后则将mIsFirstLoad置为false，避免重复加载。  

```
@Override
public void onViewCreated(View view, @Nullable Bundle savedInstanceState) {
    super.onViewCreated(view, savedInstanceState);
    mIsPrepare = true;
    lazyLoad();
}

@Override
public void setUserVisibleHint(boolean isVisibleToUser) {
    super.setUserVisibleHint(isVisibleToUser);
    //isVisibleToUser这个boolean值表示:该Fragment的UI 用户是否可见
    if (isVisibleToUser) {
        mIsVisible = true;
        lazyLoad();
    } else {
        mIsVisible = false;
    }
}
``` 
最后，贴上懒加载的lazyLoad()代码：

> 一定要记住，只要标志位改变，就要进行lazyLoad()函数的操作

 

```
private void lazyLoad() {
    //这里进行三个条件的判断，如果有一个不满足，都将不进行加载
    if (mIsPrepare || mIsVisible||!mIsFirstLoad) {
    return;
    }
        loadData();
        //数据加载完毕,恢复标记,防止重复加载
        mIsFirstLoad = false;
    }
    
  private void loadData() {
    //这里进行网络请求和数据装载
    }

``` 
当然，在最后，如果Fragment销毁的话，还应该将三个标志位进行默认值初始化：  

```
   @Override
    public void onDestroyView() {
        super.onDestroyView();
        mIsFirstLoad=true;
        mIsPrepare=false;
        mIsVisible = false;
    }
``` 
为什么在onDestroyView中进行而不是在onDestroy中进行呢？这又要提到之前Adapter的差异，onDestroy并不一定会调用，读者可以思考思考为什么。  
   
# 卡顿及性能优化建议  
Fragment的加载最为耗时的步骤主要有两个，一个是Fragment创建（尤其是创建View的过程），另一个就是读取数据填充到View上的过程。懒加载能够解决后者所造成的卡顿，但是针对前者来说，并没有效果。  
Google为了避免用户因翻页而造成卡顿，采用了缓存的形式，但是其实缓不缓存，只要该Fragment会显示，都会进行Fragment创建，都会耗费相应的时间，换言之，缓存只不过将本应该在翻页时的卡顿集中在启动该Activity的时候一起卡顿。  
# 优化方案一：设置缓存页面数    
`viewPager.setOffscreenPageLimit(int limit)` 能够有效地一次性缓存多个Fragment，这样就能够解决在之后每次切换时不会创建实例对象，看起来也会流畅。但是这样的做法，最大的缺点就是容易造成第一次启动时非常缓慢！如果第一次启动时间满足要求的话，就使用这种简单地办法吧。

# 优化方案二：避免Fragment的销毁  
不管是FragmentStatePagerAdapter还是FragmentPagerAdapter，其中都有一个方法可以被覆写：  

```
            @Override
            public void destroyItem(ViewGroup container, int position, Object object) {
               // super.destroyItem(container, position, object);
            }
```  
把中间的代码注释掉就行了，这样就可以避免Fragment的销毁过程，一般情况下能够这样使用，但是容易出现一个问题，我们再来看看FragmentStatePagerAdapter的源码：  

```
    @Override
    public void destroyItem(ViewGroup container, int position, Object object) {
        Fragment fragment = (Fragment) object;

        if (mCurTransaction == null) {
            mCurTransaction = mFragmentManager.beginTransaction();
        }
        if (DEBUG) Log.v(TAG, "Removing item #" + position + ": f=" + object
                + " v=" + ((Fragment)object).getView());
        while (mSavedState.size() <= position) {
            mSavedState.add(null);
        }
        mSavedState.set(position, fragment.isAdded()
                ? mFragmentManager.saveFragmentInstanceState(fragment) : null);
        mFragments.set(position, null);

        mCurTransaction.remove(fragment);
    }
``` 
看到没？这个过程之中包含了对FragmentInstanceState的保存！这也是FragmentStatePagerAdapter的精髓之处，如果注释掉，一旦Activity被回收进入异常销毁状态，Fragment就无法恢复之前的状态，因此这种方法也是有纰漏和局限性的。FragmentPagerAdapter的源代码就留给大家自己去研究分析，也会发现一些问题的哦。

# 优化方案三：避免重复创建View

优化Viewpager和Fragment的方法就是尽可能地避免Fragment频繁创建，当然，最为耗时的都是View的创建。所以更加优秀的优化方案，就是在Fragment中缓存自身有关的View，防止onCreateView函数的频繁执行，我就直接上源码了：

```
public class MyFragment extends Fragment {
	View rootView;
	
    @Override
    public View onCreateView(LayoutInflater inflater, ViewGroup container,
                             Bundle savedInstanceState) {
        if (rootView == null) {
            rootView = inflater.inflate(R.layout.fragment_my, container, false);
        }
        return rootView;

   @Override
    public void onDestroyView() {
        super.onDestroyView();
        Log.d(TAG, "onDestroyView: " + mParam1);
        mIsFirstLoad=true;
        mIsPrepare=false;
        mIsVisible = false;
        if (rootView != null) {
            ((ViewGroup) rootView.getParent()).removeView(rootView);
        }
}
``` 
onCreateView中将会对rootView进行null判断，如果为null，说明还没有缓存当前的View，因此会进行过缓存，反之则直接利用。当然，最为重要的是需要在`onDestroyView() ` 方法中及时地移除rootView，因为每一个View只能拥有一个Parent，如果不移除，将会重复加载而导致程序崩溃。    

> 其实ViewPager+Fragment的方式，ViewPager中显示的就是Fragment中所创建的View，Fragment只是一个控制器，并不会直接显示于ViewPager之中，这一点容易被忽略。  

暂时想到的优化方案就只有这么多了。  
# 总结  
本文主要讲述两个部分的知识：**三驾马车实现切页展示的基础方法**以及**如何优化性能表现和避免卡顿**。其中，对于ViewPager+Fragment体系的卡顿原因进行了分析，也主要有两个方面：**创建Framgent实例（创建View）**和**数据加载导致卡顿**。后者卡顿通过懒加载的形式能够完美解决，而前者因实例创建引起的卡顿则提出了三种不同的优化选择，应该说，每一种方案都有利有弊，并没有绝对的好与不好，在项目运用中，还是得根据需求和实际情况来进行选择，当然，要从内存泄漏、卡顿时间、容错率等多个方面来综合考量。不过话说回来，最优的优化方案还是尽可能的精简自己的View布局。

总之，Fragment是Android中最为重要的知识点之一，我在总结本博客的过程之中也有很大的收获，多看源码了解问题的根源过后再对症下药，不失为一种程序员的基本素养。  

公司项目，多有不便，博客源码之后再补充，还请谅解。   
 
欢迎关注我的博客   
   [yourzeromax主页](http://yourzeromax.top/)




