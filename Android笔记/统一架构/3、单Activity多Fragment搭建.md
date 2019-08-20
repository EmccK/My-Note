# Fragmentation简单使用

## 1. 简介

为`单Activity ＋ 多Fragment`,`多模块Activity + 多Fragment`架构而生，简化开发，轻松解决动画、嵌套、事务相关等问题。

**[github地址](https://github.com/YoKeyword/Fragmentation)**

## 2. 导入依赖

这里简单使用，只要导入fragmentation-core包就行了

```java
// appcompat-v7包是必须的
implementation 'me.yokeyword:fragmentationx:1.0.1'

// 如果不想继承SupportActivity/Fragment，自己定制Support，可仅依赖:
implementation 'me.yokeyword:fragmentationx-core:1.0.1'

// 如果想使用SwipeBack 滑动边缘退出Fragment/Activity功能，完整的添加规则如下：
implementation 'me.yokeyword:fragmentationx:1.0.1'
// swipeback基于fragmentation, 如果是自定制SupportActivity/Fragment，则参照SwipeBackActivity/Fragment实现即可
implementation 'me.yokeyword:fragmentationx-swipeback:1.0.1'

// Activity作用域的EventBus，更安全，可有效避免after onSavenInstanceState()异常
implementation 'me.yokeyword:eventbus-activity-scope:1.1.0'
// Your EventBus's version
implementation 'org.greenrobot:eventbus:{version}'
```

## 3. 新建单Activity

复制github项目中的`MySupportActivity.java`

重写`onCreate`方法

添加`setRootDelegate`抽象方法

创建一个`ProxyActivity.class`

```java
/**
 * 这个为设置Activity的第一个Fragment
 */
public abstract BaseDelegate setRootDelegate();

@Override
protected void onCreate(@Nullable Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    DELEGATE.onCreate(savedInstanceState);
    initContainer(savedInstanceState);
}

/**
 * 在Activity中初始化容纳Fragment的容器
 */
private void initContainer(@Nullable Bundle savedInstanceState) {
    final FrameLayout container = new FrameLayout(this);
    //这里需要在values文件夹中新建ids.xml文件，保存id
    container.setId(R.id.delegate_container);
    setContentView(container);
    if (savedInstanceState == null) {
        DELEGATE.loadRootFragment(R.id.delegate_container, setRootDelegate());
    }
}
```

`ids.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <item name="delegate_container" type="id" />
</resources>
```

## 4. 新建基础Fragment

复制github项目中的`MySupportFragment.java`

重写`onCreateView`方法

添加`setLayout()`和`onBindView()`抽象方法

创建一个`BaseFragment.class`

```java
//使用ButterKnife
private Unbinder mUnbinder = null;

/**
 * 设置Fragment的Layout
 */
public abstract Object setLayout();

/**
 * 绑定View的数据
 */
public abstract void onBindView(@Nullable Bundle savedInstanceState, View view);

@Nullable
@Override
public View onCreateView(@NonNull LayoutInflater inflater, @Nullable ViewGroup container, @Nullable Bundle savedInstanceState) {
    View rootView = null;
    if (setLayout() instanceof Integer) {
        rootView = inflater.inflate((Integer) setLayout(), container, false);
    } else if (setLayout() instanceof View) {
        rootView = (View) setLayout();
    } else {
        throw new ClassCastException("type of setLayout() must be int or View!");
    }
    if(rootView != null){
        mUnbinder = ButterKnife.bind(this,rootView);
        onBindView(savedInstanceState, rootView);
    }
    return rootView;
}

@Override
public void onDestroyView() {
    super.onDestroyView();
    if (mUnbinder != null) {
        mUnbinder.unbind();
    }
}
```

## 5. 使用单Activity

创建一个`MainActivity`，使它继承自自己创建的Activity，如上面的`ProxyActivity`

重写`setRootDelegate()`方法，设置根Fragment

如果需要隐藏Toolbar或者状态栏，只需要重写onCreate方法，在里面设置Toolbar和状态栏

```java
public class MainActivity extends ProxyActivity {

    @Override
    public BaseDelegate setRootDelegate() {
        return new MainDelegate();
    }

    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        //隐藏Toolbar
        final ActionBar actionBar = getSupportActionBar();
        if (actionBar != null) {
            actionBar.hide();
        }
    }
}
```

## 6. 使用多Fragment

创建一个Fragment，使它继承自自己创建的`BaseDelegate`

重写`setLayout()`方法，绑定Fragment的Layout

重写`onBindView()`方法，绑定用户数据

**`MainDelegate`**

```java
public class MainDelegate extends BaseDelegate {

    @Override
    public Object setLayout() {
        return R.layout.delegate_main;
    }

    @Override
    public void onBindView(@Nullable Bundle savedInstanceState, View view) {
        Button button = view.findViewById(R.id.button);
        button.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                //直接跳转到TestDelegate
                start(new TestDelegate());
            }
        });
    }
}
```

**`TestDelegate`**

```java
public class TestDelegate extends BaseDelegate {
    @Override
    public Object setLayout() {
        return R.layout.delegate_test;
    }

    @Override
    public void onBindView(@Nullable Bundle savedInstanceState, View view) {

    }
}
```

## 7. 显示结果

点击测试进入TestFragment界面

<img src="E:\MyDocument\Document\My-Note\Images\Screenshot_1566205933.png" style="zoom:100" />

<img src="E:\MyDocument\Document\My-Note\Images\Screenshot_1566205943.png" style="zoom:100" />

