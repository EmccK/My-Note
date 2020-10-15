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

