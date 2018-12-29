---
layout:     post
title:      开源项目ZMRecyclerViewAdapter
subtitle:   一种方便的RecyclerViewAdapter
date:       2018-12-29
author:     yourzeromax
header-img: img/post-bg-github-cup.jpg
catalog: true
tags:
    - Android
---

# 写在前面

这个开源项目早就写了，但是最近闲下来整理了一下，总的来说，这是一个能够替换RecyclerViewAdapter的开源控件，能够实现少数几行代码就丰富RecycelrView的表现。封装的非常巧妙，调用极少的函数。

# ZMRecyclerViewAdapter

![Build](https://img.shields.io/badge/Build-v1.1.1-blue.svg)   ![](https://jitpack.io/v/yourzeromax/ZMRecyclerViewAdapter.svg)

### 项目地址
[ZMRecyclerViewAdaper](https://github.com/yourzeromax/ZMRecyclerViewAdapter/blob/master/README.md)

### Gradle集成方法

```
//build.gradle

	allprojects {
		repositories {
			...
			maven { url 'https://www.jitpack.io' }
		}
	}
	
	dependencies {
	        implementation 'com.github.yourzeromax:ZMRecyclerViewAdapter:v1.1.1'
	        implementation  'com.android.support:recyclerview-v7:28.0.0'
	        }
```

为了方便，并未集成RecyclerView，需要自己在项目中配置。

### 功能介绍

项目一共提供了三种Adapter的使用：
通用Adapter，MultiAdapter以及能够自由添加Header和Footer的Adatper，基本涵盖了常用的RecyclerView场景，非常不错的一款开源作品。

### 使用方法 

在使用之前，需要装载数据，在demo之中可以看到：
```
 List<Student> dataList=StudentUtils.getMultiStudentsData(20);
 
 //StudentUtils.java
     public static List<Student> getMultiStudentsData(int dataLength) {
        List<Student> data = new ArrayList<>();
        for (int i = 0; i < dataLength; i++) {
            Student student = new Student("yuzhimou  " + i, 20 + i, 20144444);
            if (i % 2 == 0) {
                student.setMulti(true);
            } else {
                student.setMulti(false);
            }
            data.add(student);
        }
        return data;
    }
```

##### ZMCommonAdapter

```
    private void initRecyclerView() {
        recyclerView.setLayoutManager(new LinearLayoutManager(this));
        recyclerView.setAdapter(new ZMCommonAdapter<Student>(this, dataList, R.layout.item_student) {
            @Override
            public void convert(CommonViewHolder viewHolder, Student data) {
                viewHolder.setText(R.id.tv_name, data.getName());
                viewHolder.setText(R.id.tv_id, String.format(Locale.ENGLISH, "%1$d", data.getId()));
            }
        });
    }
```

##### ZMMultiAdapter

```
 private void initRecyclerView() {
        recyclerView.setLayoutManager(new LinearLayoutManager(this));
        recyclerView.setAdapter(new ZMMultiAdapter<Student>(this, dataList) {
            @Override
            public void convert(CommonViewHolder viewHolder, Student data) {
                viewHolder.setText(R.id.tv_name, data.getName());
                viewHolder.setText(R.id.tv_id, String.format(Locale.ENGLISH, "%1$d", data.getId()));
            }

            @Override
            public int getLayoutId(int position, Student data) {
                if (data.isMulti()) {
                    return R.layout.item_student_multi;
                }
                return R.layout.item_student;
            }
        });
    }
```

##### ZMHeaderFooterAdapter

```
    private void initRecyclerView() {
        recyclerView.setLayoutManager(new LinearLayoutManager(this));

        ZMCommonAdapter<Student> mAdapter = new ZMCommonAdapter<Student>(this, dataList, R.layout.item_student) {
            @Override
            public void convert(CommonViewHolder viewHolder, Student data) {
                viewHolder.setText(R.id.tv_name, data.getName());
                viewHolder.setText(R.id.tv_id, String.format(Locale.ENGLISH, "%1$d", data.getId()));
            }
        };
        ZMHeaderFooterAdapter<Student> mHeaderFooterAdapter = new ZMHeaderFooterAdapter<>(mAdapter);
        View mHeader = LayoutInflater.from(this).inflate(R.layout.item_header, recyclerView, false);
        View mFooter = LayoutInflater.from(this).inflate(R.layout.item_footer, recyclerView, false);
        mHeaderFooterAdapter.addHeaderView(R.layout.item_header, mHeader);
        mHeaderFooterAdapter.addFooterView(R.layout.item_footer, mFooter);
        recyclerView.setAdapter(mHeaderFooterAdapter);
    }
```
`ZMHeaderFooterAdapter`使用起来就比较复杂了，简单来说有以下几个步骤：
1. 建立一个通用的Adapter，并以此构造一个HeaderFooterAdaoter
2. 创建Header和Footer实例
3. 和其他Adapter用法一样


# 总结 
经过这样封装，是不是让RecyclerView的使用变得特别简单？之后再补充上源代码解析吧，欢迎大家star~

