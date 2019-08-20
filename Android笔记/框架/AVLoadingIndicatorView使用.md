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

  