---
layout:     post
title:      Retrofit使用总结（二）
subtitle:   Retrofit使用方法介绍
date:       2018-07-23
author:     yourzeromax
header-img: img/post-bg-android.jpg
catalog: true
tags:
    - Android
---

# 写在前面
在上文主要介绍了Retrofit的基本用法，同时也在博客的末尾简单阐述了目前比较流行的Retrofit+RxJava+Gson框架，但是严格意义上来说，Retrofit能做的事情不止这些，这篇博客会更加深入地阐述一些其他较为重要的知识要点，主要包括以下几个方面：  

- Retrofit中提供的注解标签
- Gson以及它的converter用法
- 如何自定义converter
- RxJava以及它的CallAdapter用法
- 如何自定义callAdapter
- 其他一些补充知识

概括起来就是说本文将会完善地介绍除了@Get以外注解标签的功能，并且总结Gson和RxJava更高端的用法。话不多说，开始吧。

# Retrofit注解标签
Retrofit为我们提供的注解标签一共有22个，可以划分为三类。

##### HTTP请求方法类  
这里提供的标签都是对应的Http请求方法，接收字符串接口path和baseUrl来组成一个完整的Url：  

名称 | 描述 
---|---
@GET | 从服务器获取，可附带信息?id=123&&name=yourzeromax
@POST | 提交一个表单
@PUT | 
@DELETE | 
@PATCH | 
@HEAD | 
@OPTIONS | 
@HTTP | 
在日常使用之中，我们一般只用关注@GET和@POST

##### 字段标记类
字段表积类一般在@POST用的比较多，它主要是表明需要对表单编码或者表单参数的形式等等：  

分类| 名称 | 描述 
---|---|---
表单请求 | @FormUrlEncoded | 表示请求体是Form表单，对应Header的Content-Type
表单请求 | @Multipart | 表示请求体是一个文件上传Form表单,对应Header的Content-Type：multipart/form-data
标记 | @Streaming |表示响应体以数据流的形式返回，一般在返回数据量大的情况下添加

##### 参数描述类
参数描述类除了@Headers是作用于方法描述，用于添加请求头以外，其余都是作用于方法中的参数。  

名称 | 描述
---|---
@header | 添加不固定值的Header
@Body | 添加非表单请求体
@Field | 一般用于@POST标签下的参数，需要方法添加@FormUrlEncoded注解标签
@FieldMap | 同上，接收类型为`Map<String,String>`
@Part| 一般用于@POST标签下的参数，需要方法添加@Multipart注解标签，适用于文件上传
@PartMap|同上，接收类型为`Map<String,RequestBody>`
@Path|作用于URL拼接参数
@Query|一般用于@GET标签下的参数，数据添加在Url之后
@QueryMap|同上，接收参数`Map<String,String>`
@Url| 表明路径

> @Query、@Field和@Part这三者都支持数组和实现了Iterable接口的类型，如List，Set等。 

光是看理论，如果没有代码可看的话，估计记忆不深，那我就点上一些典型的接口吧：  



# Gson以及它的converter用法
如果没有指定Gson的话，Retrofit会默认地把网络请求返回的响应体转换成`ResponseBody`,但实际上，从Retrofit接口返回`Call<ResponseBody>`可以看出，其实是支持泛型的，换言之，我们完全可以按照自己项目中的JavaBean进行直接处理。比如

```
public interface UserInfoService {
  @GET("info/{id}")
  Call<Result<User>> getUser(@Path("id") int id);
}
```

而这种情况就需要Gson的支持：

```
Gson gson = new GsonBuilder()
      //配置Gson参数
      .create();

Retrofit retrofit = new Retrofit.Builder()
      .baseUrl("http://localhost:8080/")
      //这里就是添加Gson的默认Converter
      .addConverterFactory(GsonConverterFactory.create(gson))
      .build();
```
这时候，返回的将会得到一个我们所需要的Bean类型：User。

# 自定义Converter
为了简单说明，这里使用`Call<String>`  
Converter接口的定义只有一个方法和一个抽象工厂类：  

```
public interface Converter<F, T> {
  // 从 F(rom) 到 T(o)的转换
  T convert(F value) throws IOException;

  // 用于向Retrofit提供相应Converter的工厂，
  abstract class Factory {
    // 这里创建从ResponseBody其它类型的Converter，如果不能处理返回null
    // 主要用于对响应体的处理
    public Converter<ResponseBody, ?> responseBodyConverter(Type type, Annotation[] annotations,
    Retrofit retrofit) {
      return null;
    }

    // 在这里创建 从自定类型到ResponseBody 的Converter,不能处理就返回null，
    // 主要用于对Part、PartMap、Body注解的处理
    public Converter<?, RequestBody> requestBodyConverter(Type type,
    Annotation[] parameterAnnotations, Annotation[] methodAnnotations, Retrofit retrofit) {
      return null;
    }

    // 这里用于对Field、FieldMap、Header、Path、Query、QueryMap注解的处理
    // Retrfofit对于上面的几个注解默认使用的是调用toString方法
    public Converter<?, String> stringConverter(Type type, Annotation[] annotations,
    Retrofit retrofit) {
      return null;
    }

  }
}
```
自己定义一个转换类并实现Converter的接口：

```
public static class StringConverter implements Converter<ResponseBody, String> {

  public static final StringConverter INSTANCE = new StringConverter();

  @Override
  public String convert(ResponseBody value) throws IOException {
    return value.string();
  }
}
```
再提供一个自定义的Factory：

```
public static class StringConverterFactory extends Converter.Factory {

  public static final StringConverterFactory INSTANCE = new StringConverterFactory();

  public static StringConverterFactory create() {
    return INSTANCE;
  }

  // 从ResponseBody 到 String 的转换
  @Override
  public Converter<ResponseBody, ?> responseBodyConverter(Type type, Annotation[] annotations, Retrofit retrofit) {
    if (type == String.class) {
      return StringConverter.INSTANCE;
    }
    //其它类型我们不处理，返回null就行
    return null;
  }
}
```
之后便可以替换在Retrofit中注册新的ConverterFactory：  

```
Retrofit retrofit = new Retrofit.Builder()
      .baseUrl("http://localhost:8080/")
      .addConverterFactory(StringConverterFactory.create())
      .addConverterFactory(GsonConverterFactory.create())
      .build();
```
大功告成。

# RxJava以及它的CallAdapter用法  
RxJava并不会直接影响响应体的结构，但是它是一种处理数据流程的思想转变，Gson和Converter能够完成`Call<T>`泛型的转换，那么RxJava和CallAdapter则是完成对`Call`的转换：  

```
Retrofit retrofit = new Retrofit.Builder()
      .baseUrl("http://localhost:8080/")
      .addConverterFactory(GsonConverterFactory.create())
      .addCallAdapterFactory(RxJavaCallAdapterFactory.create())
      // 如果是RxJava2.x版本的话，用以下的语句
      .addCallAdapterFactory(RxJava2CallAdapterFactory.create()) 
      .build();
```
这时，我们便可以重新设计我们的接口返回对象：  

```
public interface UserInfoService {
  @GET("info/{id}")
  Observable<Result<User>> getUser(@Path("id") int id);
}
```
在网络请求代码中，我们可以按照标准的RxJava做法这样去使用返回的Observable对象：  

```
UserInfoService service = retrofit.create(UserInfoService.class);
service.getUser(100)
  .subscribeOn(Schedulers.io())
  .subscribe(new Subscriber<Result<User>>() {
      @Override
      public void onCompleted() {
        System.out.println("onCompleted");
      }

      @Override
      public void onError(Throwable e) {
        System.err.println("onError");
      }

      @Override
      public void onNext(Result<User> result) {
        System.out.println(result);
      }
  });

```

如果想要获取返回的Header以及对应的响应码的话，需要做以下的变化：  

```
1、用Observable<Response<T>> 代替 Observable<T> ,这里的Response指retrofit2.Response 
2、用Observable<Result<T>> 代替 Observable<T>，这里的Result是指retrofit2.adapter.rxjava.Result,这个Result中包含了Response的实例
```

# 自定义CallAdapter  
和自定义Converter有些类似，先来看看CallAdapter的接口：  

```
public interface CallAdapter<T> {

  Type responseType();

  <R> T adapt(Call<R> call);

  // 用于向Retrofit提供CallAdapter的工厂类
  abstract class Factory {

    public abstract CallAdapter<?> get(Type returnType, Annotation[] annotations,
    Retrofit retrofit);

    protected static Type getParameterUpperBound(int index, ParameterizedType type) {
      return Utils.getParameterUpperBound(index, type);
    }

    protected static Class<?> getRawType(Type type) {
      return Utils.getRawType(type);
    }
  }
}
```

同样地，我们需要自定义Adapter并且实现其中的接口：  

```
public static class CustomCallAdapter implements CallAdapter<CustomCall<?>> {

  private final Type responseType;

  // 下面的 responseType 方法需要数据的类型
  CustomCallAdapter(Type responseType) {
    this.responseType = responseType;
  }

  @Override
  public Type responseType() {
    return responseType;
  }

  @Override
  public <R> CustomCall<R> adapt(Call<R> call) {
    // 由 CustomCall 决定如何使用
    return new CustomCall<>(call);
  }
}
```
之后也需要提供一个Factory来进行注册：  

```
public static class CustomCallAdapterFactory extends CallAdapter.Factory {
  public static final CustomCallAdapterFactory INSTANCE = new CustomCallAdapterFactory();

  @Override
  public CallAdapter<?> get(Type returnType, Annotation[] annotations, Retrofit retrofit) {
    // 获取原始类型
    Class<?> rawType = getRawType(returnType);
    // 返回值必须是CustomCall并且带有泛型
    if (rawType == CustomCall.class && returnType instanceof ParameterizedType) {
      Type callReturnType = getParameterUpperBound(0, (ParameterizedType) returnType);
      return new CustomCallAdapter(callReturnType);
    }
    return null;
  }
}
```
注册那就更简单了：  

```
Retrofit retrofit = new Retrofit.Builder()
      .baseUrl("http://localhost:8080/")
      .addConverterFactory(StringConverterFactory.create())
      .addConverterFactory(GsonConverterFactory.create())
      .addCallAdapterFactory(CustomCallAdapterFactory.INSTANCE)
      .build();
```

# 补充知识  
Retrofit实际上是基于OkHttp的一层封装网络请求框架，它也提供了设置OkHttpClient的方法来对Cookie、网络请求责任链等进行自定义的功能：  

```
Retrofit retrofit = new Retrofit.Builder()
      .baseUrl("http://localhost:8080/")
      .client(new OkhttpClient())
      .addConverterFactory(StringConverterFactory.create())
      .addConverterFactory(GsonConverterFactory.create())
      .addCallAdapterFactory(CustomCallAdapterFactory.INSTANCE)
      .build();
```
OkhttpClient实例对象可以由我们自己写，具体的分析我想留在之后的OkHttp请求框架详解之中。  

至此，Retrofit的知识介绍完毕，如果后面有补充的话，再来进行相应的修改。

欢迎大家关注  
[我的博客](https://yourzeromax.top/)






