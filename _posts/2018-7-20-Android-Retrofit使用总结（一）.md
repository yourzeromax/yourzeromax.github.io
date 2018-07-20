---
layout:     post
title:      Retrofit使用总结（一）
subtitle:   Retrofit使用方法介绍
date:       2018-07-20
author:     yourzeromax
header-img: img/post-bg-android.jpg
catalog: true
tags:
    - Android
---

> 好久不见

# 写在前面
好久没有更新自己的博客了，或许是由于最近刚毕业就入职，稍微有些松懈了吧。来到了新的环境，也并不是这么快就融入的，还好情况一天比一天要好。
最近开始着手于自己家的Android项目，今天也和大家分享一下Retrofit网络框架库的使用。

# 相关资源
 Gradle配置引包
```
compile 'com.squareup.retrofit2:converter-gson:2.3.0'
compile 'com.squareup.retrofit2:adapter-rxjava:2.3.0'
compile 'com.squareup.retrofit2:retrofit:2.3.0'
compile 'io.reactivex:rxandroid:1.2.1'
```
[github地址](https://github.com/square/retrofit)  

[Retrofit官方网站](http://square.github.io/retrofit/)
# 什么是Retrofit？
 Retrofit是一款由Square公司开源的著名框架，按照官网的文档来说，这款框架是“专门用于Android和Java的一种类型安全的Http客户端框架”。不懂这句话没关系，反正知道Retrofit能够完成网络连接就好啦。  
 
 Retrofit能够将后端所构造的HTTP API转化成一个Java类型的接口，便于前端程序员理解和使用。
 
 # 如何使用Retrofit？
 首先在使用之前当然需要compile一下相关的库，Retrofit的传统使用方法主要包含三个步骤：
1.  建立一个接口类并封装好相关的方法
2.  构造Retrofit实例并实现上述的接口对象。
3.  调用对象中的相应方法，处理返回数据。

看起来是不是特别简单？那接下来就一边哔哔一边拿代码解释吧。  

## 接口类的创建及注意事项  
先上代码：
```
public interface HttpInterface {
    @GET("users/{user}/repos")
    Call<List<Repo>> listRepos(@Path("user") String user);
}
```
这是一个标准的Retrofit请求接口类的创建，学过计算机网络的人都应该知道网络请求有GET和POST两种方式，在上文代码中的@GET便是指定相应的请求方式，网络连接需要URL的形式，而GET标签后的String路径则是指代URL中的文件路径和文件名，URL的统一格式为：

```
协议类型://<主机名>:<端口>/<路径>/文件名
```
比如：

```
https://api.github.com/users/yourzeromax/repos
```
可以对比一下（端口是可以默认缺省），然后就能明白GET标签后指代的是什么。标签下的{user}和参数传递中@Path("user")指定的参数也是形成替换关系的。至此完成Retrofit请求的第一步。 

## 构造Retrofit实例并创建接口对象  
Retrofit的实例对象是采用的建造者模式：  

```
Retrofit retrofit = new Retrofit.Builder()
    .baseUrl("https://api.github.com/")
    .build();

GitHubService service = retrofit.create(GitHubService.class);
```
baseUrl()方法能够构造一次请求中的协议头和主机名,使得在抽象类中的方法能够直接指定该主机下的访问资源，这是非常便捷的。  

接下来获取抽象接口类的实例化对象：
```
HttpInterface service = retrofit.create(HttpInterface.class);
```  
类似于反射加载的形式创建出实例对象，这个对象便是网络请求操作的载体，至此完成了第二步。  
## 发起请求并处理返回数据  
使用上文得到的service对象来做一次GET请求：  

```
Call<List<Repo>> repos = service.listRepos("octocat");
```  
这里获得一个Call对象，正如字面意思一样，所返回的对象封装着网络请求返回的数据，要处理这些数据，就应该使用call.enqueue()的方式，来发送处理请求，如下：  

```
call.enqueue(new Callback<List<Repo>>() {
    @Override
    public void onResponse(Call<List<Repo>> call, Response<List<Repo>> response) {
        try {
            System.out.println(response.toString());
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    @Override
    public void onFailure(Call<List<Repo>> call, Throwable t) {
        t.printStackTrace();
    }
});
```  
上述采用的是异步的方式处理数据，一般来说网络请求的发起是在主线程的，采用异步的形式是非常重要的，但是在实际的使用过程中，这样的编程风格难免会显得羞涩一些，Retrofit也充分考虑到这种情况，为开发者们准备了Gson和RxJava的框架库，也就是说，目前最为常用的网络请求方式是Retrofit+RxJava+Gson的形式，感慨于编程的智慧。  
# Retrofit+RxJava+Gson  
一下子丢出三个框架名字，估计初学者会受不了，什么叫RxJava？Gson的作用又是什么？  
① 简单来说，线程异步调动在Android开发中是一件非常平常的事情，RxJava是一款优雅的异步处理框架，它的使用方法采用链式的形式，非常直观，具有极高的艺术性，建议大家移步此文进行学习：  
[如何使用RxJava？](http://gank.io/post/560e15be2dca930e00da1083)   

②G前网络流行的数据传输方式是Json格式，但是Java是一个面向对象编程的语言，如何将Json格式的数据转换成Java对象也是一门大学问，Gson是由Google公司开源的一款Json和Java对象相互转换的框架库，它的使用也非常简单：  

```
        Gson gson = new Gson();
        Student student = new Student();
        student.setName("yuzhimou");
        student.setNick("shaw");
        student.setAge(22);
        String jsonStr = gson.toJson(student);   //对象转Json
        Student stu = gson.fromJson(jsonStr); //Json转对象
``` 
Retrofit作为目前最为火热的网络请求开源框架库，充分考虑到开发者的感受，在它的使用过程中也融入了Gson和RxJava两种框架。如果要使用这种融合的请求方式，需要先对之前的抽象接口进行相应的改动，将返回对象改为Observable：  
```
public interface HttpInterface {
  @GET("users/{user}/repos")
    Observable<List<Repo>> listRepos(@Path("user") String user);
}
```
之后便好理解啦，再修改相应的Retrofit构造，增加RxJavaCallAdapterFactory和GsonConverterFactory：
```
Retrofit retrofit = new Retrofit.Builder()
      .baseUrl("http://localhost:4567/")
      .addConverterFactory(GsonConverterFactory.create())
      .addCallAdapterFactory(RxJavaCallAdapterFactory.create())
      // 针对rxjava2.x
      .addCallAdapterFactory(RxJava2CallAdapterFactory.create()) 
      .build();
```
按照RxJava的书写方式，来调整相应的代码如下：  

```
HttpInterface service = retrofit.create(BlogService.class);
service.listRepos("yourzeromax")
  .subscribeOn(Schedulers.io())
  .subscribe(new Subscriber<Result<List<Blog>>>() {
      @Override
      public void onCompleted() {
        System.out.println("onCompleted");
      }

      @Override
      public void onError(Throwable e) {
        System.err.println("onError");
      }

      @Override
      public void onNext(Result<List<Blog>> blogsResult) {
        System.out.println(blogsResult);
      }
  });
```  
在onNext()中处理返回的数据，这时会惊奇地发现，所返回的Json数据直接转换成了Result<List<Blog>>对象，再调用相关的getter方法就能读取需要的返回数据。而且丝毫不用担心线程调度的问题，只用开心地关注该关注的地方就完事。  
Retrofit的使用就是这么简单。  
# 总结  
本文主要介绍了Retrofit的基本使用方法，虽然非常简单，但是还没有介绍相关的标签内容，并且在实际的业务开发之中，需要自己去定义相关的Coverter和Adapter，这也是下一篇博客的重点内容。


