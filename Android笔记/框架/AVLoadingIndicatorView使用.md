# AVLoadingIndicatorView使用

## 1. 简介

AVLoadingIndicatorView是一个好用的加载动画的集合，里面有各种加载动画

[github地址](https://github.com/81813780/AVLoadingIndicatorView)

## 2. 导入

```java
implementation 'com.wang.avi:library:2.1.3'
```

## 3. 在布局中添加

```xml
<com.wang.avi.AVLoadingIndicatorView
        android:id="@+id/avi_loader"
        style="@style/AVLoadingIndicatorView"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        app:indicatorColor="@android:color/holo_orange_dark"
        app:indicatorName="BallPulseIndicator" />
```

## 4. 在Activity中管理

- 开始动画

  ```java
  avi_loader.show();
  ```

- 停止动画

  ````java
  avi_loader.hide();
  ````

  # AVLoadingIndicatorView封装

  ## 1. `LoaderCreater`类
  
  ```java
  public final class LoaderCreator {
  
      private static final WeakHashMap<String, Indicator> LOADING_MAP = new WeakHashMap<>();
  
      static AVLoadingIndicatorView create(String type, Context context) {
  
          final AVLoadingIndicatorView avLoadingIndicatorView = new AVLoadingIndicatorView(context);
          if (LOADING_MAP.get(type) == null) {
              final Indicator indicator = getIndicator(type);
              LOADING_MAP.put(type, indicator);
          }
          avLoadingIndicatorView.setIndicator(LOADING_MAP.get(type));
          return avLoadingIndicatorView;
      }
  
      private static Indicator getIndicator(String name) {
          if (name == null || name.isEmpty()) {
              return null;
          }
          final StringBuilder drawableClassName = new StringBuilder();
          if (!name.contains(".")) {
              final String defaultPackageName = AVLoadingIndicatorView.class.getPackage().getName();
              drawableClassName.append(defaultPackageName)
                      .append(".indicators")
                      .append(".");
          }
          drawableClassName.append(name);
          try {
              final Class<?> drawableClass = Class.forName(drawableClassName.toString());
              return (Indicator) drawableClass.newInstance();
          } catch (Exception e) {
              e.printStackTrace();
              return null;
          }
      }
  }
  ```
  
  ## 2. `MyLoader`类
  
  ```java
  public class MyLoader {
  
      // Loader根据屏幕的大小缩小为原来的1/8
      private static final int LOADER_SIZE_SCALE = 8;
      // 根据屏幕的上下有一个偏移量，为10
      private static final int LOADER_OFFSET_SCALE = 10;
  
      private static final ArrayList<AppCompatDialog> LOADERS = new ArrayList<>();
  
      // 默认的Loader
      private static final String DEFAULT_LOADER = LoaderStyle.BallClipRotatePulseIndicator.name();
  
      public static void showLoading(Context context, String type) {
          //设置Dialog的Style，在style.xml添加dialog的Style
          final AppCompatDialog dialog = new AppCompatDialog(context, R.style.dialog);
  
          final AVLoadingIndicatorView avLoadingIndicatorView = LoaderCreator.create(type, context);
          dialog.setContentView(avLoadingIndicatorView);
  
          int deviceWidth = getScreenWidth(context);
          int deviceHeight = getScreenHeight(context);
  
          final Window dialogWindow = dialog.getWindow();
  
          if (dialogWindow != null) {
              WindowManager.LayoutParams lp = dialogWindow.getAttributes();
              lp.width = deviceWidth / LOADER_SIZE_SCALE;
              lp.height = deviceHeight / LOADER_SIZE_SCALE;
              lp.height = lp.height + deviceHeight / LOADER_OFFSET_SCALE;
              lp.gravity = Gravity.CENTER;
          }
          //不可手动取消
          dialog.setCancelable(false);
          LOADERS.add(dialog);
          //默认显示dialog
          dialog.show();
      }
  
      /**
       * 重载，加载默认Loader
       */
      public static void showLoading(Context context) {
          showLoading(context, DEFAULT_LOADER);
      }
  
      /**
       * 停止Dialog
       */
      public static void stopLoading() {
          for (AppCompatDialog dialog : LOADERS) {
              if (dialog != null) {
                  if (dialog.isShowing()) {
                      dialog.cancel();
                  }
              }
          }
      }
  
      /**
       * 获取屏幕宽度
       */
      private static int getScreenWidth(Context context) {
          final Resources resources = context.getResources();
          final DisplayMetrics dm = resources.getDisplayMetrics();
          return dm.widthPixels;
      }
  
      /**
       * 获取屏幕高度
       */
      private static int getScreenHeight(Context context) {
          final Resources resources = context.getResources();
          final DisplayMetrics dm = resources.getDisplayMetrics();
          return dm.heightPixels;
      }
  }
  ```
  
  ## 3. 在`style.xml`添加Dialog的Style
  
  ```xml
  <resources>
  
      <style name="dialog" parent="Theme.AppCompat.Dialog">
          <!--去掉边框-->
          <item name="android:windowFrame">@null</item>
          <!--悬浮-->
          <item name="android:windowIsFloating">true</item>
          <!--半透明效果-->
          <item name="android:windowIsTranslucent">false</item>
          <!--不需要标题-->
          <item name="android:windowNoTitle">true</item>
          <!--背景透明-->
          <item name="android:windowBackground">@android:color/transparent</item>
          <!--允许模糊-->
          <item name="android:backgroundDimEnabled">true</item>
          <!--全屏幕-->
          <item name="android:windowFullscreen">true</item>
      </style>
  
  </resources>
  ```
  
  ## 4. 使用Dialog
  
  ```java
  public class MainActivity extends AppCompatActivity {
  
      @Override
      protected void onCreate(Bundle savedInstanceState) {
          super.onCreate(savedInstanceState);
          setContentView(R.layout.activity_main);
          //进入就显示Dialog
          MyLoader.showLoading(this);
          //显示两秒后隐藏dialog
          new Handler().postDelayed(new Runnable() {
              @Override
              public void run() {
                  MyLoader.stopLoading();
              }
          }, 2000);
      }
  }
  ```