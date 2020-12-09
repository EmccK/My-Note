# Retrofit

- retrofit中的接口是通过java内置的动态代理来实现的，实现的字节码文件为`com.sun.proxy.$Proxy0`，然后再使用`classLoader`来加载
- 接口方法调用一般在不同的线程中，所以调用方法时一定要使用线程安全的`ConcurrentHashMap`保存方法，并获取方法

    - 调用过一次之后已经保存在`ConcurrentHashMap`缓存中了，下一次直接从缓存中获取方法 
- 解析注解

    - 方法本身的注解
    - 方法参数的注解
- 参数类型的适配`Converter`
- 使用反射动态切换`BaseUrl`











# 开源框架的阅读套路

- 带着问题入手
- 从最熟悉的部分切入
- 看看有哪些可扩展的设计
- 分析下运用了哪些设计模式







首先`Retrofit`是使用`Builder`建造者模式创建的，在`build`方法中实例化各种参数，然后在`Retrofit`的`create`方法中使用`动态代理`将传入的`Java.class`对象转换成一个是实例

- 首先创建`ServiceMethod`对象
- 根据`ServiceMethod`对象创建`OkHttpCall`对象
- `ServiceMethod.callAdapter.adapt(okHttpCall)`返回`CallAdaper`对象



## Retrofit使用

1. 创建一个Retrofit对象，是通过建造者模式来创建的
2. 通过Retrofit.create()方法，将我们定义的接口转化成接口实例并使用接口的方法
3. 最终的网络请求调用的都是okHttpCall使用

## Glide源码分析

1. with方法，实例化requestManager，实现生命周期的方法，是所有图片请求的管理类
2. load方法，对一些参数进行赋值
3. into方法，最终使用的还是线程池进行处理

