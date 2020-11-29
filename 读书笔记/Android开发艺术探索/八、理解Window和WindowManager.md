# 理解Window和WindowManager

## 一、概述

`Window`表示一个窗口的概念，一般在使用类似悬浮窗的时候，会用到`Window`来实现。

`Window`是一个抽象类，它的具体实现是`PhoneWindow`。而创建`Window`需要通过`WindowManager`来实现。

`WindowManager`是外界访问`Window`的入口，`Window`的具体实现位于`WindowManagerService`中，`WindowManager`和`WindowManagerService`的交互一个IPC过程。

Android中所有的视图都是通过`Window`来呈现的，不管是`Activity`、`Dialog`还是`Toast`，它们的视图实际上都是**附加**在`Window`上的，因此**`Window`实际上是`View`的直接管理者。**

## 二、Window和WindowManager

下面的代码是使用WindowManager添加一个Window的过程。

```java
mWindowManager = getWindowManager();
mFloatingButton = new Button(this);
mFloatingButton.setText("button");
mLayoutParams = new LayoutParams(
LayoutParams.WRAP_CONTENT,
LayoutParams.WRAP_CONTENT,
0, 0, PixelFormat.TRANSPARENT);
mLayoutParams.flags = LayoutParams.FLAG_NOT_TOUCH_MODAL | LayoutParams.FLAG_NOT_FOCUSABLE;
mLayoutParams.gravity = Gravity.START | Gravity.TOP;
mLayoutParams.x = 0;
mLayoutParams.y = 0;
mLayoutParams.type = LayoutParams.TYPE_APPLICATION_OVERLAY;
mWindowManager.addView(mFloatingButton, mLayoutParams);
```

上述代码可以将一个Button添加到屏幕坐标为(0, 0)的位置上。`WindowManager.LayoutParams`中的`flags`和`type`这两个参数比较重要。

**`Flags`**参数表示`Window`的属性，它有很多选项，通过这些选项可以控制`Window`的显示特性，这里主要介绍几个常用的选项，剩下的请查看[官方文档](https://developer.android.com/reference/android/view/WindowManager.LayoutParams#flags)。

- `FLAG_NOT_FOCUSABLE`：表示`Window`不需要获取焦点，也不需要接收各种输入事件，此标记会同时启用`FLAG_NOT_TOUCH_MODAL`，最终事件会直接传递给下层的具有焦点的`Window`。
- `FLAG_NOT_TOUCH_MODAL`：在此模式下，系统会将当前`Window`区域以外的单击事件传递给底层的`Window`，当前`Window`区域以内的单击事件则自己处理。这个标记很重要，一般来说都需要开启此标记，否则其他`Window`将无法收到单击事件。
- `FLAG_SHOW_WHEN_LOCKED`：开启此模式可以让Window显示在锁屏的界面上。

**`Type`**参数表示Window的类型，Window有三种类型，分别是`应用Window`、`子Window`和`系统Window`。

- `应用类Window`对应着一个Activity。

- `子Window`不能单独存在，它需要附属在特定的父Window之中，比如常见的一些Dialog就是一个子Window。

- `系统Window`是需要声明权限在能创建的`Window`，比如`Toast`和系统状态栏这些都是系统`Window`。

Window是分层的，每个Window都有对应的`z-ordered`，层级大的会覆盖在层级小的Window的上面。在三类Window中，应用Window的层级范围是1~99，子Window的层级范围是1000 ~ 1999，系统Window的层级范围是2000~2999，这些层级范围对应着`WindowManager.LayoutParams`的type参数。

如果需要使用系统Window，则需要声明权限`<uses-permission android:name="android.permission.SYSTEM_ALERT_WINDOW" />`，因为系统类型的Window是需要检查权限的。

`WindowManager`所提供的功能很简单，常用的只有三个方法，即添加View、更新View和删除View，这三个方法定义在`ViewManager`中，而`WindowManager`继承了`ViewManager`。

```java
public interface ViewManager
{
    public void addView(View view, ViewGroup.LayoutParams params);
    public void updateViewLayout(View view, ViewGroup.LayoutParams params);
    public void removeView(View view);
}
```

`WindowManager`操作`Window`的过程更像是在操作`Window`中的`View`。我们常见的拖动的`Window`效果，设置`onTouchListener`，然后在`ACTION_MOVE`中，设置`View`的`X`和`Y`坐标，然后使用`WindowManager`更新`View`。

```java
public boolean onTouch(View v, MotionEvent event) {
    int rawX = (int) event.getRawX();
    int rawY = (int) event.getRawY();
    switch (event.getAction()) {
        case MotionEvent.ACTION_MOVE: {
            mLayoutParams.x = rawX;
            mLayoutParams.y = rawY;
            mWindowManager.updateViewLayout(mFloatingButton, mLayoutParams);
            break;
        }
    }
    return false;
}
```

## 三、Window的内部机制

`Window`是一个抽象的概念，每一个`Window`都对应着一个`View`和一个`ViewRootImpl`，`Window`和`View`通过`ViewRootImpl`来建立联系，**因此`Window`并不是实际存在的，它是以`View`的形式存在。**

从`WindowManager`的定义也可以看出，它提供的三个接口方法`addView`、`updateViewLayout`以及`removeView`都是针对`View`的，这说明`View`才是`Window`存在的实体。

在实际使用中无法直接访问`Window`，对`Window`的访问必须通过`WindowManager`。

为了分析Window的内部机制，这里从Window的添加、删除以及更新说起。

### Window的添加过程

`Window`的添加过程需要通过`WindowManager`的`addView`来实现，`WindowManager`是一个接口，它的真正实现是`WindowManagerImpl`类。在`WindowManagerImpl`中`Window`的三大操作如下：

```java
@Override
public void addView(@NonNull View view, @NonNull ViewGroup.LayoutParams params) {
    applyDefaultToken(params);
    mGlobal.addView(view, params, mContext.getDisplayNoVerify(), mParentWindow,
            mContext.getUserId());
}

@Override
public void updateViewLayout(@NonNull View view, @NonNull ViewGroup.LayoutParams params) {
    applyDefaultToken(params);
    mGlobal.updateViewLayout(view, params);
}

private void applyDefaultToken(@NonNull ViewGroup.LayoutParams params) {
    // Only use the default token if we don't have a parent window.
    if (mDefaultToken != null && mParentWindow == null) {
        if (!(params instanceof WindowManager.LayoutParams)) {
            throw new IllegalArgumentException("Params must be WindowManager.LayoutParams");
        }

        // Only use the default token if we don't already have a token.
        final WindowManager.LayoutParams wparams = (WindowManager.LayoutParams) params;
        if (wparams.token == null) {
            wparams.token = mDefaultToken;
        }
    }
}

@Override
public void removeView(View view) {
    mGlobal.removeView(view, false);
}
```

其中`mGlobal`是一个`WindowManagerGlobal`类，可以发现，`WindowManagerImpl`并没有直接实现`Window`的三大操作，而是全部交给了`WindowManagerGlobal`来处理，`WindowManagerGlobal`以单例模式返回`WindowManagerGlobal`实例。`WindowManagerImpl`这种工作模式是典型的桥接模式，将所有的操作委托给`WindowManagerGlobal`来实现。

`WindowManagerGlobal`的`addView`方法主要分为如下几步：

#### 1. 检查参数是否合法，如果是子Window那么还需要调整一些布局参数

```java
if (view == null) {
    throw new IllegalArgumentException("view must not be null");
}
if (display == null) {
    throw new IllegalArgumentException("display must not be null");
}
if (!(params instanceof WindowManager.LayoutParams)) {
    throw new IllegalArgumentException("Params must be WindowManager.LayoutParams");
}

final WindowManager.LayoutParams wparams = (WindowManager.LayoutParams) params;
if (parentWindow != null) {
    parentWindow.adjustLayoutParamsForSubWindow(wparams);
}
```

#### 2. 创建ViewRootImpl并将View添加到列表中

在`WindowManagerGlobal`内部有如下几个列表比较重要：

```java
private final ArrayList<View> mViews = new ArrayList<View>();
private final ArrayList<ViewRootImpl> mRoots = new ArrayList<ViewRootImpl>();
private final ArrayList<WindowManager.LayoutParams> mParams = new ArrayList<WindowManager.LayoutParams>();
private final ArraySet<View> mDyingViews = new ArraySet<View>();
```

在上面的声明中，`mViews`存储的是所有`Window`所对应的`View`，`mRoots`存储的是所有`Window`所对应的`ViewRootImpl`，`mParams`存储的是所有`Window`所对应的布局参数，而`mDyingViews`则存储了那些正在被删除的`View`对象，或者说是那些已经调用`removeView`方法但是删除操作还未完成的`Window`对象。在`addView`中通过如下方式将`Window`的一系列对象添加到列表中：

```java
root = new ViewRootImpl(view.getContext(), display);
view.setLayoutParams(wparams);
mViews.add(view);
mRoots.add(root);
mParams.add(wparams);
```

#### 3. 通过ViewRootImpl来更新界面并完成Window的添加过程

这个步骤由`ViewRootImpl`的`setView`方法来完成，`View`的绘制过程是由`ViewRootImpl`来完成的，这里当然也不例外，在`setView`内部会通过`requestLayout`来完成异步刷新请求。在下面的代码中，`scheduleTraversals`实际是`View`绘制的入口：

```java
public void requestLayout() {
    if (!mHandlingLayoutInLayoutRequest) {
        checkThread();
        mLayoutRequested = true;
        scheduleTraversals();
    }
}
```

接着会通过`WindowSession`最终完成`Window`的添加过程。在下面的代码中，`mWindowSession`的类型是是`IWindowSession`，它是一个`Binder`对象，真正的实现类是`Session`，也就是`Window`的添加过程是一次IPC调用。

```java
try {
    mOrigWindowType = mWindowAttributes.type;
    mAttachInfo.mRecomputeGlobalAttributes = true;
    collectViewAttributes();
    res = mWindowSession.addToDisplay(mWindow, mSeq, mWindowAttributes, getHostVisibility(), mDisplay.getDisplayId(), mAttachInfo.mContentInsets, mInputChannel);
} catch (RemoteException e) {
    mAdded = false;
    mView = null;
    mAttachInfo.mRootView = null;
    mInputChannel = null;
    mFallbackEventHandler.setView(null);
    unscheduleTraversals();
    setAccessibilityFocus(null, null);
    throw new RuntimeException("Adding window failed", e);
}
```

在`Session`内部会通过`WindowManagerService`来实现`Window`的添加，代码如下所示：

```java
public int addToDisplay(IWindow window, int seq, WindowManager.LayoutParams attrs, int viewVisibility, int displayId, Rect outContentInsets, InputChannel outInputChannel) {
    return mService.addWindow(this, window, seq, attrs, viewVisibility, displayId, outContentInsets, outInputChannel);
}
```

如此以来，`Window`的添加请求就交给`WindowManagerService`去处理了，在`WindowManagerService`内部会为每一个应用保留一个单独的`Session`。

`调用WindowManager的addView` -> `WindowManagerImpl的addView方法` -> `WindowManagerGlobal单例类的addView方法` -> `真正的方法（1.判断参数是否合法2.创建ViewRootImpl，并添加进列表3.调用ViewRootImpl的`**`setView`**`完成添加的过程）`

而`setView`方法（`requestLayout`->`scheduleTraversals` -> `WindowSession` -> `WindowManagerService.addToDisplay`）是一次IPC调用。

这就是`Window`添加的整体流程。

### Window的删除过程

`Window`的删除过程和添加过程一样，都是先通过`WindowManagerImpl`后，再进一步通过`WindowManagerGlobal`来实现的。下面是`WindowManagerGlobal`的`removeView`的实现。

```java
public void removeView(View view, boolean immediate) {
    if (view == null) {
        throw new IllegalArgumentException("view must not be null");
    }

    synchronized (mLock) {
        int index = findViewLocked(view, true);
        View curView = mRoots.get(index).getView();
        removeViewLocked(index, immediate);
        if (curView == view) {
            return;
        }

        throw new IllegalStateException("Calling with view " + view
                + " but the ViewAncestor is attached to " + curView);
    }
}
```

`removeView`的逻辑很清晰，首先通过`findViewLocked`来查找待删除的`View`的索引，这个查找就是对建立的数组进行遍历，然后调用`removeViewLocked`来做进一步的删除

```java
private void removeViewLocked(int index, boolean immediate) {
    ViewRootImpl root = mRoots.get(index);
    View view = root.getView();

    if (root != null) {
        root.getImeFocusController().onWindowDismissed();
    }
    boolean deferred = root.die(immediate);
    if (view != null) {
        view.assignParent(null);
        if (deferred) {
            mDyingViews.add(view);
        }
    }
}
```

`removeViewLocked`是通过`ViewRootImpl`来完成删除操作的。在`WindowManager`中提供了两种删除接口`removeView`和`removeViewImmediate`，它们分别表示异步删除和同步删除，其中`removeViewImmediate`使用起来需要特别注意，一般来说不需要使用此方法来删除`Window`以免发生意外的错误。这里主要说异步删除的情况，具体的删除操作由`ViewRootImpl`的`die`方法来完成。

```java
boolean die(boolean immediate) {
    // Make sure we do execute immediately if we are in the middle of a traversal or the damage
    // done by dispatchDetachedFromWindow will cause havoc on return.
    if (immediate && !mIsInTraversal) {
        doDie();
        return false;
    }

    if (!mIsDrawing) {
        destroyHardwareRenderer();
    } else {
        Log.e(mTag, "Attempting to destroy the window while drawing!\n" +
                "  window=" + this + ", title=" + mWindowAttributes.getTitle());
    }
    mHandler.sendEmptyMessage(MSG_DIE);
    return true;
}
```

在`die`方法内部只是做了简单的判断：如果是异步删除，那么就发送一个`MSG_DIE`的消息，`ViewRootImpl`中的`Handler`会处理此消息并调用`doDie`方法；如果是同步删除（立即删除），那么就不发送消息直接调用`doDie`方法，这就是两种删除方式的区别。在`doDie`内部会调用`dispatchDetachedFromWindow`方法， 真正删除`View`的逻辑在`dispatchDetachedFromWindow`方法的内部实现。

`dispatchDetachedFromWindow`方法主要做四件事：

1. 垃圾回收相关的工作，比如清除数据和消息、移除回调。
2. 通过`Session`的`remove`方法删除`Window`：`mWindowSession.remove(mWindow)`，这同样是一个IPC过程，最终会调用`WindowManagerService`的`removeWindow`方法。
3. 调用`View`的`dispatchDetachedFromWindow`，在内部会调用`View`的`onDetachedFromWindow`以及`onDetachedFromWindowInternal`。对于`onDetachedFromWindow`方法，当`View`从`Window`中移除时，这个方法就会被调用，可以在这个方法内部做一些资源回收的工作，比如终止动画、停止线程等。
4. (`doDie`方法内部)调用`WindowManagerGlobal`的`doRemoveView`方法刷新数据，包括`mRoots`、`mParams`以及`mDyingViews`，需要将当前`Window`所关联的这三类对象从列表删除。

### Window的更新过程

`Window`的更新过程也是看`WindowManagerGlobal`的`updateViewLayout`方法，如下所示：

```java
public void updateViewLayout(View view, ViewGroup.LayoutParams params) {
    if (view == null) {
        throw new IllegalArgumentException("view must not be null");
    }
    if (!(params instanceof WindowManager.LayoutParams)) {
        throw new IllegalArgumentException("Params must be WindowManager.LayoutParams");
    }

    final WindowManager.LayoutParams wparams = (WindowManager.LayoutParams)params;

    view.setLayoutParams(wparams);

    synchronized (mLock) {
        int index = findViewLocked(view, true);
        ViewRootImpl root = mRoots.get(index);
        mParams.remove(index);
        mParams.add(index, wparams);
        root.setLayoutParams(wparams, false);
    }
}
```

`updateViewLayout`的方法做的事情就比较简单了，首先它需要更新`View`的`LayoutParams`并替换掉老的`LayoutParams`，接着更新`ViewRootImpl`中的`LayoutParams`，使用`setLayoutParams`方法。

在`setLayoutParams`方法中，`ViewRootImpl`中通过`scheduleTraversals`方法来对`View`重新布局，包括测量、布局、重绘这三个过程。

除了`View`本身的重绘之外，`ViewRootImpl`还会通过`WindowSession`来更新`Window`的视图，这个过程最终是由`WindowManagerService`的`relayoutWindow`来具体实现的，它同样是一个IPC过程。

Window的添加、更新、删除都是IPC过程，其中`Session`是一个`Binder`。

| `WindowManager`方法  | `WindowManagerService`方法 |
| :------------------: | :------------------------: |
|     `addView()`      |       `addWindow()`        |
| `updateViewLayout()` |     `relayoutWindow()`     |
|    `removeView()`    |      `removeWindow()`      |

## 四、Window的创建过程

通过上面的分析可以看出，`View`是`Android`中的视图的呈现方式，但是`View`不能单独存在，它必须依附着在`Window`这个抽象得概念上面，因此有视图的地方就有`Window`。

Android中可以提供视图的地方有`Activity`、`Dialog`、`Toast`等，除此之外，还有一些依托`Window`而实现的视图，比如`PopUpWindow`、菜单，它们也是视图，有视图的地方就有`Window`，因此`Activity`、`Dialog`、`Toast`等视图都对应着一个`Window`。

本节将分析这些视图元素中的`Window`的创建过程，通过本节可以使读者进一步加深对`Window`的理解。

### (1). Activity的Window创建过程

要分析`Activity`中的`Window`的创建过程就必须了解`Activity`的启动过程，这里先只是大搞了解下。

`Activity`的启动过程很复杂，最终会由`ActivityThread`中的`performLaunchActivity()`来完成整个启动过程，在这个方法内部会通过类加载器创建`Activity`的实例对象，并调用其`attach`方法为其关联运行所依赖的一系列上下文环境变量。代码如下所示：

```java
java.lang.ClassLoader cl = appContext.getClassLoader();
activity = mInstrumentation.newActivity(
        cl, component.getClassName(), r.intent);
...
if (activity != null) {
    CharSequence title = r.activityInfo.loadLabel(appContext.getPackageManager());
    Configuration config = new Configuration(mCompatConfiguration);
    if (r.overrideConfig != null) {
        config.updateFrom(r.overrideConfig);
    }
    if (DEBUG_CONFIGURATION) Slog.v(TAG, "Launching activity "
            + r.activityInfo.name + " with config " + config);
    activity.attach(appContext, this, getInstrumentation(), r.token,
            r.ident, app, r.intent, r.activityInfo, title, r.parent,
            r.embeddedID, r.lastNonConfigurationInstances, config,
            r.referrer, r.voiceInteractor, window, r.configCallback,
            r.assistToken);
    ...
}
```

在`Activity`的`attach`方法里，系统会创建`Activity`所属的`Window`对象并为其设置回调接口，`Window`对象的创建是通过实例化一个`PhoneWindow`实例实现的。

由于`Activity`实现了`Window`的`Callback`接口，因此当`Window`接收到外界的状态改变时就会回调`Activity`的方法。

`Callback`接口中的方法很多，但是有几个却是我们都非常熟悉的，比如`onAttachedToWindow`、`onDetachedFromWindow`，`dispatchTouchEvent`等等，代码如下所示：

```java
mWindow = new PhoneWindow(this, window, activityConfigCallback);
mWindow.setWindowControllerCallback(mWindowControllerCallback);
mWindow.setCallback(this);
mWindow.setOnWindowDismissedCallback(this);
mWindow.getLayoutInflater().setPrivateFactory(this);
if (info.softInputMode != WindowManager.LayoutParams.SOFT_INPUT_STATE_UNSPECIFIED) {
    mWindow.setSoftInputMode(info.softInputMode);
}
if (info.uiOptions != 0) {
    mWindow.setUiOptions(info.uiOptions);
}
```

从上面的分析可以看出，`Activity`的`Window`是直接实例化一个`PhoneWindow`实例。到这里`Window`已经创建完成了，下面分析`Activity`的视图是怎么附属在`Window`上的。

由于`Activity`的视图由`setContentView`方法提供，我们只需要看`setContentView`方法的实现即可。

```java
public void setContentView(@LayoutRes int layoutResID) {
    getWindow().setContentView(layoutResID);
    initWindowDecorActionBar();
}
```

从`Activity`的`setContentView`的实现可以看出，`Activity`将具体的实现交给了`Window`处理，而`Window`的具体实现是`PhoneWindow`，所以只需要看`PhoneWindow`的相关逻辑即可。

`PhoneWindow`的`setContentView`方法大致遵循如下几个步骤：

#### 1. 如果没有`DecorView`，那么就创建它

`DecorView`是一个`FrameLayout`，这里简单说一下。`DecorView`是`Activity`中的顶级`View`，一般来说它的内部包含标题栏和内部栏，但是这个会随着主题的变换而发生改变。

不管怎么样，内容栏是一定要存在的，并且内容来自具体固定的`id`，那就是“`content`”，它的完整`id`是`android.R.id.content`。`DecorView`的创建过程由`installDecor`方法来完成，在方法内部会通过`generateDecor`方法来直接创建`DecorView`，这个时候`DecorView`还只是一个空白的`FrameLayout`：

```java
protected DecorView generateDecor(int featureId) {
    // System process doesn't have application context and in that case we need to directly use
    // the context we have. Otherwise we want the application context, so we don't cling to the
    // activity.
    Context context;
    if (mUseDecorContext) {
        Context applicationContext = getContext().getApplicationContext();
        if (applicationContext == null) {
            context = getContext();
        } else {
            context = new DecorContext(applicationContext, this);
            if (mTheme != -1) {
                context.setTheme(mTheme);
            }
        }
    } else {
        context = getContext();
    }
    return new DecorView(context, featureId, this, getAttributes());
}
```

为了初始化`DecorView`的结构，`PhoneWindow`还需要通过`generateLayout`方法来加载具体的布局文件到`DecorView`中，具体的布局文件和系统版本以及主题有关。使用(`installDecor`方法中)`generateLayout`方法得到`mContentParent`

#### 2. 将`View`添加到`DecorView`的`mContentParent`中

这个过程就比较简单了，由于在步骤1中已经创建并初始化了`DecorView`，因此这一步直接将`Activity`的视图添加到`DecorView`的`mContentParent`中即可：`mLayoutInflater.inflate(layoutResID, mContentParent)`。

到此为止，`Activity`的布局文件已经添加到`DecorView`里面了，由此可以理解`Activity`的`setContentView`这个方法的来历了。`Activity`的布局文件只是被添加到`DecorView`的`mContentParent`中，因此叫`setContentView`更加准确。

#### 3. 回调`Activity`的`onContentChanged`方法通知`Activity`视图已经发生改变

这个过程就更简单了，由于`Activity`实现`Window`的`CallBack`接口，这里表示`Activity`的布局文件已经被添加到`DecorView`的`mContentParent`中了，于是需要通知`Activity`，使其可以做相应的处理。

`Activity`的`onContentChanged`方法是个空实现，我们可以在子`Activity`中处理这个回调。这个过程的代码如下：

```java
final Callback cb = getCallback();
if (cb != null && !isDestroyed()) {
    cb.onContentChanged();
}
```

经过上面的三个步骤，到这里为止`DecorView`已经被创建并初始化完毕，`Activity`的布局文件也已经成功添加到`DecorView`的`mContentParent`中，但是这个时候`DecorView`还没有被`WindowManager`正式添加到`Window`中。

这里需要正确理解`Window`的概念，`Window`更多表示的是一种抽象得功能集合，虽然说早在`Activity`的`attach`方法中`Window`就已经被创建了，但是这个时候由于`DecorView`并没有被`WindowManager`识别，所以这个时候的`Window`无法提供具体功能，因为它还无法接受外接的输入信息。

在`ActivityThread`的`handleResumeActivity`方法中，首先会调用`Activity`的`onResume`方法，接着会调用`Activity`的`makeVisible()`，正是在`makeVisible`方法中，`DecorView`真正地完成了添加和显示这两个过程，到这里`Activity`的视图才能被用户看到，如下所示：

````java
void makeVisible() {
    if (!mWindowAdded) {
        ViewManager wm = getWindowManager();
        wm.addView(mDecor, getWindow().getAttributes());
        mWindowAdded = true;
    }
    mDecor.setVisibility(View.VISIBLE);
}
````

到这里，`Activity`中的`Window`的创建过程已经分析完了。

### (2). Dialog的Window创建过程

Dialog的Window的创建过程和Activity类似，有如下几个步骤。

#### 1. 创建Window

`Dialog`中`Window`的创建同样是直接实例化`PhoneWindow`实例，跟`Activity`的`Window`的创建是一致的。

```java
Dialog(@NonNull Context context, @StyleRes int themeResId, boolean createContextThemeWrapper) {
    mWindowManager = (WindowManager) context.getSystemService(Context.WINDOW_SERVICE);

    final Window w = new PhoneWindow(mContext);
    mWindow = w;
    w.setCallback(this);
    w.setOnWindowDismissedCallback(this);
    w.setOnWindowSwipeDismissedCallback(() -> {
        if (mCancelable) {
            cancel();
        }
    });
    w.setWindowManager(mWindowManager, null, null);
    w.setGravity(Gravity.CENTER);

    mListenersHandler = new ListenersHandler(this);
}
```

#### 2. 初始化DecorView并将Dialog的视图添加到DecorView中

这个过程也和`Activity`的类似，都是通过`Window`去添加指定的布局文件。

```java
public void setContentView(@LayoutRes int layoutResID) {
    mWindow.setContentView(layoutResID);
}
```

#### 3. 将DecorView添加到Window中并显示

在`Dialog`的`show`方法中，会通过`WindowManager`将`DecorView`添加到`Window`中，如下所示：

```java
mWindowManager.addView(mDecor, l);
mShowing = true;
```

从上面三个步骤可以发现，Dialog的Window创建和Activity的Window创建过程很类似，二者几乎没有什么区别。

当`Dialog`被关闭时，它会通过`WindowManager`来移除`DecorView`: `mWindowManager.removeViewImmediate(mDecor)`。

普通的`Dialog`有一个特殊之处，那就是必须采用`Activity`的`Context`，如果采用`Application`的`Content`，那么就会报错。

### (3). Toast的Window创建过程

`Toast`和`Dialog`不同，它的工作过程就稍显复杂。首先`Toast`也是基于`Window`来实现的，但是由于Toast具有定时取消这一功能，所以系统采用了`Handler`。

在`Toast`的内部有两类IPC过程，第一类是`Toast`访问`NotificationManagerService`，第二类是`NotificationManagerService`回调`Toast`里的`TN`接口。为了便于描述，下面将`NotificationManagerService`简称为`NMS`。

`Toast`属于系统`Window`，它内部的视图由两种方式指定，一种是系统默认的样式，另一种是通过`setView`方法来指定一个自定义`View`，不管如何，它们都对应`Toast`的一个`View`类型的内部成员`mNextView`。

`Toast`提供了`show`和`cancel`分别用于显示和隐藏`Toast`，它们的内部是一个IPC过程，`show`方法和`cancel`方法的实现如下：

```java
public void show() {
    if (Compatibility.isChangeEnabled(CHANGE_TEXT_TOASTS_IN_THE_SYSTEM)) {
        checkState(mNextView != null || mText != null, "You must either set a text or a view");
    } else {
        if (mNextView == null) {
            throw new RuntimeException("setView must have been called");
        }
    }

    INotificationManager service = getService();
    String pkg = mContext.getOpPackageName();
    TN tn = mTN;
    tn.mNextView = mNextView;
    final int displayId = mContext.getDisplayId();

    try {
        if (Compatibility.isChangeEnabled(CHANGE_TEXT_TOASTS_IN_THE_SYSTEM)) {
            if (mNextView != null) {
                // It's a custom toast
                service.enqueueToast(pkg, mToken, tn, mDuration, displayId);
            } else {
                // It's a text toast
                ITransientNotificationCallback callback =
                        new CallbackBinder(mCallbacks, mHandler);
                service.enqueueTextToast(pkg, mToken, mText, mDuration, displayId, callback);
            }
        } else {
            service.enqueueToast(pkg, mToken, tn, mDuration, displayId);
        }
    } catch (RemoteException e) {
        // Empty
    }
}

public void cancel() {
    if (Compatibility.isChangeEnabled(CHANGE_TEXT_TOASTS_IN_THE_SYSTEM)
            && mNextView == null) {
        try {
            getService().cancelToast(mContext.getOpPackageName(), mToken);
        } catch (RemoteException e) {
            // Empty
        }
    } else {
        mTN.cancel();
    }
}
```

从上面的代码可以看出，显示和隐藏`Toast`都需要通过`NMS`来实现，由于`NMS`运行在系统的进程中，所以只能通过远程调用的方式来显示和隐藏`Toast`。

需要注意的是`TN`这个类，它是一个`Binder`类，在`Toast`和`NMS`进行`IPC`的过程中，当`NMS`处理`Toast`的显示或隐藏请求时会跨进程回调`TN`中的方法，这个时候由于`TN`运行在`Binder`线程池中，所以需要通过`Handler`将其切换到当前线程中。这里的当前线程是指发送`Toast`请求所在的线程。

注意：由于这里使用了`Handler`，所以这意味着`Toast`无法在没有`Looper`的线程中弹出。

```java
INotificationManager service = getService();
String pkg = mContext.getOpPackageName();
TN tn = mTN;
tn.mNextView = mNextView;
final int displayId = mContext.getDisplayId();
try{
	service.enqueueToast(pkg, mToken, tn, mDuration, displayId);    
} catch (RemoteException e) {
    // Empty
}
```

`enqueueToast`首先将`Toast`请求封装为`ToastRecord`对象并将其添加到一个名为`mToastQueue`的队列中。`mToastQueue`其实是一个`ArrayList`。

对于非系统用户来说，`mToastQueue`中最多能同时存在50个`ToastRecord`，这样做是为了防止DOS(Denial of Service)。如果不这么做，试想一下，如果我们通过大量的循环去连续弹出`Toast`，这将会导致其他应用没有机会弹出`Toast`，那么对于其他应用的`Toast`请求，系统的行为就是拒绝服务，这就是拒绝服务攻击的含义，这种手段常用于网络攻击中。

```java
// Limit the number of toasts that any given package can enqueue.
// Prevents DOS attacks and deals with leaks.
int count = 0;
final int N = mToastQueue.size();
for (int i = 0; i < N; i++) {
    final ToastRecord r = mToastQueue.get(i);
    if (r.pkg.equals(pkg)) {
        count++;
        if (count >= MAX_PACKAGE_NOTIFICATIONS) {
            Slog.e(TAG, "Package has already posted " + count
                    + " toasts. Not showing more. Package=" + pkg);
            return;
        }
    }
}
```

正常情况下，一个应用不可能达到上限，当`ToastRecord`被添加到`mToastQueue`中后，`NMS`就会通过`showNextToastLocked`方法来显示当前的`Toast`。下面的代码很好理解，需要注意的是，`Toast`的显示是由`ToastRecord`的`callback`来完成的，`ToastRecord`是一个抽象类，实现类是`CustomToastRecord`，这个`callback`实际上就是`Toast`中的`TN`对象的远程`Binder`，通过`callback`来访问`TN`中的方法需要跨进程来完成的，最终被调用的`TN`中的方法会运行在发起`Toast`请求的应用的`Binder`线程池中。

```java
void showNextToastLocked() {
    ToastRecord record = mToastQueue.get(0);
    while (record != null) {
        if (record.show()) {
            scheduleDurationReachedLocked(record);
            return;
        }
        int index = mToastQueue.indexOf(record);
        if (index >= 0) {
            mToastQueue.remove(index);
        }
        record = (mToastQueue.size() > 0) ? mToastQueue.get(0) : null;
    }
}
```

`Toast`显示以后，`NMS`还会通过`scheduleTimeoutLocked`方法来发送一个延时消息，具体的延时取决于`Toast`的时长，如下所示：

```java
void scheduleTimeoutLocked(NotificationRecord record) {
    if (record.getNotification().getTimeoutAfter() > 0) {
        final PendingIntent pi = PendingIntent.getBroadcast(getContext(),
                REQUEST_CODE_TIMEOUT,
                new Intent(ACTION_NOTIFICATION_TIMEOUT)
                        .setData(new Uri.Builder().scheme(SCHEME_TIMEOUT)
                                .appendPath(record.getKey()).build())
                        .addFlags(Intent.FLAG_RECEIVER_FOREGROUND)
                        .putExtra(EXTRA_KEY, record.getKey()),
                PendingIntent.FLAG_UPDATE_CURRENT);
        mAlarmManager.setExactAndAllowWhileIdle(AlarmManager.ELAPSED_REALTIME_WAKEUP,
                SystemClock.elapsedRealtime() + record.getNotification().getTimeoutAfter(), pi);
    }
}
```

延迟相应的时间后，`NMS`会通过`cancelToastLocked`方法来隐藏`Toast`并将其从`mToastQueue`中移除，这个时候如果`mToastQueue`中还有其他`Toast`，那么`NMS`就继续显示其他`Toast`。

`Toast`的隐藏也是通过`ToastRecord`的`callback`来完成，这同样也是一次`IPC`过程，它的工作方式和`Toast`的显示过程是类似的，如下所示：

```java
public void hide() {
    try {
        callback.hide();
    } catch (RemoteException e) {
        Slog.w(TAG, "Object died trying to hide custom toast " + token + " in package "
                + pkg);

    }
}
```

通过上面的分析，大家知道`Toast`的显示和影响过程实际上是通过`Toast`中的`TN`这个类来实现的，它有两个方法`show`和`hide`，分别对应`Toast`的显示和隐藏。由于这两个方法是被`NMS`以跨进程的方式调用的，因此它们运行在`Binder`线程池中。为了将执行环境切换到`Toast`所在的线程，在它们的内部使用`Handler`，如下所示：

```java
/**
 * schedule handleShow into the right thread
 */
@Override
@UnsupportedAppUsage(maxTargetSdk = Build.VERSION_CODES.P, trackingBug = 115609023)
public void show(IBinder windowToken) {
    if (localLOGV) Log.v(TAG, "SHOW: " + this);
    mHandler.obtainMessage(SHOW, windowToken).sendToTarget();
}

/**
 * schedule handleHide into the right thread
 */
@Override
public void hide() {
    if (localLOGV) Log.v(TAG, "HIDE: " + this);
    mHandler.obtainMessage(HIDE).sendToTarget();
}
```

上述代码中，使用`Handler`发送消息，然后`Handler`处理消息，最终调用`handleShow`和`handleHide`方法来进行显示和隐藏。`TN`的`handleShow`中会将`Toast`的视图添加到`Window`中，`mWindowManager.addView(mView, mParams)`

而`TN`的`handleHIde`中会将`Toast`的视图从`Window`中移除，`mWindowManager.removeViewImmediate(mView)`。

到这里`Toast`的`Window`的创建过程已经分析完了，相信读者对`Toast`的工作过程有了一个更加全面的理解了。

## 总结

本章的意义在于让读者对`Window`有一个更加清晰的认识，同时能够深刻理解`Window`和`View`的依赖关系，这有助于理解其他更深层次的概念，比如`SurfaceFlinger`。

通过本章，我们知道，任何`View`都是附属在一个`Window`上面的。

