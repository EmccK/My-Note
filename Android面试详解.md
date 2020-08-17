# Activity面试详解

## 一、Activity生命周期

### 1. activity的四种状态

`running`/`paused`/`stopping`/`killed`

### 2. activity生命周期分析

1. `Acitivity启动`->`onCreate()`->`onStart()`->`onResume()`

每一个Activity在创建时都会进入onCreate()状态，此时可以进行一些初始化的操作，Activity处于不可见的状态。

onStart()状态处于可见，但是不可操作以及交互的状态。

onResume()状态处于可见并且可以与用户进行交互。

2. `点击Home键回到主页（Activity不可见）`->`onPause()`->`onStop()`

onPause()表示Activity处于停止状态，不可被触摸，对应onResume()方法。

onStop()一般会在onPause()方法执行完之后调用，表明整个Activity已经停止，或者完全被覆盖，这个时候Activity完全不可见，是处于后台运行的，如果系统内存紧张，Activity有可能会被回收掉。

3. `当我们再次回到原Activity时`->`onRestart()`->`onStart()`->`onResume()`

onRestart()表示Activity从不可见到可见的状态

4. `退出当前Activity时`->`onPause()`->`onStop()`->`onDestroy()`

onDestroy()表明当前Activity正在被销毁，在这里面可以做一些回收工作以及资源的释放。

### 3.android进程优先级

`前台`/`可见`/`服务`/`后台`/`空` 



----

-----



# Fragment面试详解

## 一、Fragment为什么被称为第五大组件

### 1.Fragment为什么被称为第五大组件

Fragment在使用频率上不输于其它四大组件，同时拥有自己的生命周期。

### 2. Fragment加载到Activity的两种方式

1. 添加Fragment到Activity的布局文件当中

2. 动态在Activity中添加fragment

    ```java
    FragmentTransaction transaction = getSupportFragmentManager().beginTransaction();
            transaction.replace(R.id.fragment_container, new TestFragment()).commit();
    ```

### 3. FragmentPagerAdapter和FragmentStatePagerAdapter区别

由于FragmentStatePagerAdapter在每次切换ViewPager的时候它是回收内存的，它适合页面比较多的情况，因为页面比较多的时候会更耗内存，所以说它会回收内存。

而FragmentPagerAdapter在切换的时候，并没有回收内存，它只是把UI分离，所以适合页面比较少的情况。



------



## 二、Fragment的生命周期

### 1. Fragment生命周期

![](http://blog.img.wangdankai.cn/fragment.png)

### 2. Fragment从启动到销毁生命周期的全过程

<img src="http://blog.img.wangdankai.cn/20200423182040.png" style="zoom:80%;" />



-------



## 三、Fragment之间的通信

### 1. 在Fragment中调用Activity中的方法  getActivity

### 2. 在Activity中调用Fragment中的方法 接口回调

### 3. 在Fragment中调用Fragment中的方法 findFragmentById



-----



## 四、Fragment管理器：FragmentManager

**`replace、add、remove`**



----

-----



# Service面试详解

## 一、service基础

### 1. Service是什么

Service(服务)是一个一种可以在后台执行长时间运行操作而没有用户界面的应用组件。

Service可由其他用户组件来启动，服务一旦被启动后，将一直在后台运行，即使启动它的组件已经被销毁了也不会受影响。

**Service和Broadcast都是运行在主线程中，都不能做长时间的耗时操作。**

### 2. Service和Thread的区别

**A. 定义**

Thread是程序执行的最小单元，是分配CPU的最小单位，子线程可以执行一些耗时的操作。

Service是Android中的一种特殊机制，Service是运行在主线程，绝对不能运行耗时操作

**B. 实际开发**

**C. 应用场景**

音乐、天气预报数据统计啊这些的要用Service



-----



## 二、启动Service的两种方式以及区别

### 1.startService

1. 定义一个类继承`Service`
2. 在`Manifest.xml`文件中配置该`Service`
3. 使用Context的`startService(Intent)`方法启动该Service
4. 不再使用时，调用`stopService(Intent)`方法停止该服务

### 2. bindService

1. 创建BindService服务端，继承自Service并在类中，创建一个实现IBinder接口的实例对象并提供公共方法给客户端调用
2. 从onBind()回调方法返回此Binder实例
3. 在客户端中，从onServiceConnected()回调方法接收Binder，并使用提供的方法调用绑定服务。



----

----



# Broadcast Receiver面试详解

## 一、广播

### 1. 广播定义

在Android中，Broadcast是一种广泛运用在应用程序之间传输信息的机制，Android中我们要发送的广播内容是一个Intent，这个Intent中可以携带我们要传送的数据。

广播实现了不同程序之间的数据传输与共享。

### 2. 广播的使用场景

**A.同一个app具有多个进程的不同组件之间的消息通信**

**B.不同app之间的组件之间消息通信**

### 3. 广播的种类

- Normal Broadcast: Context.sendBroadcast
- System Broadcast: Context.sendOrderedBroadcast
- Local Broadcast: 只在自身APP内传播



-----



## 二、实现广播-receiver

### 1. 静态注册：注册完成就一直运行

是在manifest文件中把广播接收者写在里面

当Activity被销毁了，仍然能收到广播，即使应用进程被杀掉也能收到广播

### 2. 动态注册：跟随activity的生命周期

是在代码中调用registerReceiver这个方法来进行注册的

会受activity生命周期的影响，activity销毁了，广播接收者也就失效了



-----



## 三、内部实现机制

1. 自定义广播接收者BroadcastReceiver，并复写onReceive()方法;
2. 通过Binder机制向AMS(Activity Manager Service)进行注册;
3. 广播发送者通过Binder机制向AMS发送广播;
4. AMS查找符合相应条件(IntentFilter/Permission等)的BroadcastReceiver，将广播发送到BroadcaseReceiver(一般情况下是Activity)相应的消息循环队列中;
5. 消息循环队列执行拿到此广播，回调BroadcastReceiver中的onReceive()方法



-----

----



# Handler面试详解

## 一、什么是handler

`handler`通过发送和处理`Message`和`Runnable`对象来关联相对应线程的`MessageQueue`

1. 可以让对应的`Message`和`Runnable`在未来的某个时间点进行相应处理
2. 让自己想要处理的耗时操作放在子线程，让更新UI的操作放在主线程



-----



## 二、handler的使用方法

1. `post(runnable)`
    - 创建Handler类
    - 创建子线程，执行耗时操作等
    - 执行完成之后执行`Handler.post(runnable)`在runnable执行刷新UI操作
2. `sendMessage(message)`
    - 创建Handler类，重写`handleMessage`方法
    - 在`handleMessage`方法中根据Message的参数，执行不同的刷新UI的操作
    - 创建子线程，执行耗时操作等
    - 执行完成，获取Message类，为Message的what、arg1、arg2或obj等参数赋相应的值
    - 最后执行`handler.sendMessage(msg)`方法



-----



## 三、handler机制的原理

<img src="http://blog.img.wangdankai.cn/20200506102030.png" style="zoom:67%;" />

<img src="http://blog.img.wangdankai.cn/20200506102147.png" style="zoom:67%;" />



-----



## 四、handler引起的内存泄漏以及解决办法

原因：静态内部类持有外部类的匿名引用，导致外部activity无法释放

解决办法：

- handler内部持有外部activity的弱引用
- 把handler改为静态内部类
- mHandler.removeCallback()



-----

----



# AsyncTask面试详解

## 一、什么是AsyncTask

它本质上就是一个封装了线程池和handler的异步框架

AsyncTask它本质上封装了线程池和handler，这个AsyncTask框架主要是用来执行异步任务的，所以说由于它内部集成了handler，所以说它能够方便的在UI线程和工作线程也就是子线程当中灵活的切换



------



## 二、AsyncTask的使用方法

### 1. 三个参数

```java
AsyncTask<Integer,Integer,String>
```

第一个参数：在执行AsyncTask所要传入的参数

第二个参数：在后台执行任务时，需要显示当前的进度

第三个参数：当任务执行完毕后，如果需要对结果进行返回，返回的类型

### 2. 5个方法

```java
public class UpdateInfoAsyncTask extends AsyncTask<Integer, Integer, String> {

    private TextView mTextView;
    private ProgressBar mProgressBar;

    public UpdateInfoAsyncTask(TextView textView, ProgressBar progressBar) {
        this.mTextView = textView;
        this.mProgressBar = progressBar;
    }

    @Override
    protected void onPreExecute() {
        //耗时操作还没做之前进行一些初始化操作
        //是在UI线程中调用的
        //一般显示进度条之类的
        mTextView.setText("开始执行异步线程");
    }

    @Override
    protected String doInBackground(Integer... integers) {
        //执行耗时操作
        //此函数在onPreExecute调用完之后调用
        //执行的结果会通过该函数返回，然后被传递到onPostExecute方法中去
        //还可以通过执行 publishProgress(Integer),不断的显示后台所进行的计算
        int i;
        for (i = 0; i <= 100; i += 10) {
            publishProgress(i);
        }
        return i + integers[0].intValue() + "";
    }

    @Override
    protected void onProgressUpdate(Integer... values) {
        //在publishProgress调用之后调用此方法
        int value = values[0];
        mProgressBar.setProgress(value);
    }

    @Override
    protected void onPostExecute(String s) {
        //后台计算完成之后调用
        //将计算结果通过参数传进来
        mTextView.setText("异步操作执行结束" + s);
    }
```

```java
//构造函数是传入一个TextView和ProgressBar，然后execute()函数传入参数
new UpdateInfoAsyncTask(((TextView)findViewById(R.id.text_view)),((ProgressBar)findViewById(R.id.progress_bar))).execute(1);
```



-----



## 三、AsyncTask内部原理

1. AsyncTask的本质是一个静态的线程池，AsyncTask派生出的子类可以实现不同的异步任务，这些任务都是提交到静态的线程池中执行。
2. 线程池中的工作线程执行`doInBackground(mParams)`方法执行异步任务
3. 当任务状态改变之后，工作线程会向UI线程发送消息，AsyncTask内部的InternalHandler响应这些消息，并调用相关的回调函数

一句话概括：内部封装了线程池，通过Handler来发送消息，在UI线程和子线程当中传递



------



## 四、AsyncTask的注意事项

### 一、内存泄漏

与Handler的原因类似，解决方法也是类似的

### 二、生命周期

如果不调用AsyncTask.cancel方法，就不会被销毁

必须在Activity的onDestroy()方法中调用cancel方法

### 三、结果丢失

由于屏幕旋转或者后台被杀，导致AsyncTask中的弱Activity引用无效，导致结果丢失



----



### 四、并行or串行

默认使用的是串行，如果要使用并行，则调用`executeOnExecutor`方法

串行较稳定一些，并行不够稳定



-----

-----



# View的绘制机制

## 一、view树的绘制流程

`measure->layout->draw`

树的递归过程



----



## 二、measure

树的递归过程

<img src="http://blog.img.wangdankai.cn/20200506111650.png" style="zoom:67%;" />

1. ViewGroup.LayoutParams参数

    用来指定视图高度和宽度的参数

    可以设定三种类型的值：固定值、match_parent、wrap_content

2. MeasureSpec参数

    是一个三十二位的int值

    前两位是测量模式，后30位表示的是在这种测量模式下的尺寸的大小

3. onMeasure方法

    measure->onMeasure->setMeasureDimension

    最终调用setMeasureDimension()，必须被调用，否则会调异常



----



## 三、layout

树的递归过程

<img src="http://blog.img.wangdankai.cn/20200506111650.png" style="zoom:67%;" />



----



## 四、draw

两个容易混淆的方法

1. invalidate()

    如果视图没有发生变化，就不会调用Layout方法

2. requestLayout()

    当布局发生变化时，会调用此方法



-----

----



# ListView面试详解

## 一、什么是ListView

ListView就是一个数据集合以动态滚动的方式展示到用户界面上的View



----



## 二、ListView适配器模式

<img src="http://blog.img.wangdankai.cn/20200506114325.png" style="zoom:50%;" />



----



## 三、ListView的recycleBin机制

<img src="http://blog.img.wangdankai.cn/20200506120407.png" style="zoom:67%;" />



----



## 四、ListView的优化

convertview重用/viewHolder

- 三级缓存
- 监听滑动事件
- 尽量少做耗时操作