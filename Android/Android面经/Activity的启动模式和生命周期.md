Activity的启动模式和生命周期(特殊情况下生命周期)

- 生命周期

    - 正常启动一个Activity：`onCreate->onStart->onResume`
    - 在当前Activity下打开一个占据半个屏幕的Dialog后，又关闭：`onCreate->onStart->onResume->onPause->onResume`
    - 在当前Activity下打开一个完全覆盖当前Activity的Activity：`onCreate->onStart->onResume->onPause->onStop->onRestart->onStart->onResume`
    - 如果因为内存太小，之前的Activity被回收：`onCreate->onStart->onResume->onPause->onStop->onCreate->onStart->onResume`
    - 如果Activity的配置发生了变化，一般也会重新创建Activity，如果不想重新创建Activity，则需要在Manifest中配置`android:configChanges="orientation"`
        - 常用的的有`locale（一般指语言发生变化），orientation（屏幕方向发生了变化），screenSize（屏幕尺寸发生了变化，一般指旋转屏幕），keyboardHidden（键盘的可访问性发生了变化）`
    - onCreate
        - Activity创建时都会调用这个方法
    - onStart
        - Activity创建前最后的准备函数
    - onResume
        - Activity完全创建出来并显示出来与用户交互
    - onPause
        - Activity失去焦点
    - onStop
        - Activity完全隐藏
    - onRestart
        - Activity从完全隐藏到显示调用
    - onDestroy
        - Activity销毁调用此函数

- 启动模式

    - **Standard**

        标准模式，是Activity默认的启动模式，不论Activity栈中有没有相同的Activity，每次启动都会创建一个新Activity。

    - **SingleTop**

        栈顶复用模式，如果要启动的新Activity当前位于栈顶，则此Activity不会被重新创建，而是会调用onNewIntent，否则重新创建新实例。

        对应的标志位：FLAG_ACTIVITY_SINGLE_TOP

    - **SingleTask**

        栈内复用模式，如果当前Activity栈内有要启动的Activity，则会将要启动的Activity出栈，Activity前面的所有Activity也会出栈。

        对应的标志位：FLAG_ACTIVITY_NEW_TASK		FLAG_ACTIVITY_CLEAR_TOP配合之前的标志位使用，清除栈上面的Activity。
    
    - **SingleInstance**
    
        单实例模式，是一种加强版的SingTask模式，除了具有SingleTask的所有特性外，具有此模式的Activity只能单独位于一个任务栈中，后序的所有请求均不会创建新的示例，除非这个任务栈被系统销毁了。
        
    - **TaskAffinity**					任务栈，在Activity中的参数，指定确定的任务栈，不能和包名相同，否则相当于没有指定，如果启动Activity时没有这个任务栈，则创建相应的任务栈，然后创建Activity。
    
- IntentFilter匹配规则

    Intent同时匹配action、category、data才算完全匹配，一个Activity中可以有多个IntentFilter，只要匹配到其中一个就可以启动相应的Activity。

    - action
        - 只要匹配到其中一个action即可，区分大小写
    - category
        - Intent中可以没有category，当没有category时，系统默认会添加`android.intent.category.DEFAULT`
        - 如果有的话，必须加上`android.intent.category.DEFAULT`
        - Intent中的category必须是已经定义过的category
    - data
        - 有两部分组成，mimeType和URI。mimeType指媒体类型，URI包含的数据比较多。
        - `<scheme>://<host>:<port>/[<path>|<pathPrefix>|<pathPattern>]`
        - 实际的例子：
        - `content://com.example.project:200/folder/subfolder/etc`
        - `http://www.baidu.com:8`