# Iconify使用教程

## 1. 简介

Iconify为使用字体代替图标的一个库

Iconify为您提供了大量可供选择的矢量图标，以及在Android应用中添加和自定义它们的直观方式。

[github地址](https://github.com/JoanZapata/android-iconify)

## 2. 导入依赖

```
dependencies {
    compile 'com.joanzapata.iconify:android-iconify-fontawesome:2.2.2' // (v4.5)
    compile 'com.joanzapata.iconify:android-iconify-entypo:2.2.2' // (v3,2015)
    compile 'com.joanzapata.iconify:android-iconify-typicons:2.2.2' // (v2.0.7)
    compile 'com.joanzapata.iconify:android-iconify-material:2.2.2' // (v2.0.0)
    compile 'com.joanzapata.iconify:android-iconify-material-community:2.2.2' // (v1.4.57)
    compile 'com.joanzapata.iconify:android-iconify-meteocons:2.2.2' // (latest)
    compile 'com.joanzapata.iconify:android-iconify-weathericons:2.2.2' // (v2.0)
    compile 'com.joanzapata.iconify:android-iconify-simplelineicons:2.2.2' // (v1.0.0)
    compile 'com.joanzapata.iconify:android-iconify-ionicons:2.2.2' // (v2.0.1)
}
```

## 3. 导入外部矢量图标

例如阿里巴巴矢量图标库

![](http://blog.img.wangdankai.cn/1566133654493.png)

将下载下来的ttf文件放到src/main/assets文件夹内

## 4. 自定义FontModule

- 创建`FontModule`类，使它实现`IconFontDescriptor`接口

  ```java
  public class FontModule implements IconFontDescriptor {
  
      @Override
      public String ttfFileName() {
          //此处返回添加的ttf文件的文件名
          return "iconfont.ttf";
      }
  
      @Override
      public Icon[] characters() {
          //此处返回自定义Icons枚举类的values
          return LotteryIcons.values();
      }
  }
  ```

  

- 创建`FontIcons`枚举类，使它实现`Icon`接口

  ![](http://blog.img.wangdankai.cn/1566134920072.png)

```java
public enum FontIcons implements Icon {
    //添加icon的标识,这里的'\ue65c'为上方扫描'&#xe65c;'变过来的
    icon_scan('\ue65c'),
    icon_ali_pay('\ue717');

    private char character;

    FontIcons(char character) {
        this.character = character;
    }

    //这里是看的官方文档中的
    @Override
    public String key() {
        return name().replace('_', '-');
    }

    @Override
    public char character() {
        return character;
    }
}
```

## 5. 在APP的onCreate方法中初始化

```java
public class MyApplication extends Application {

    @Override
    public void onCreate() {
        super.onCreate();
        Iconify
            .with(new FontAwesomeModule())
            .with(new EntypoModule())
            .with(new TypiconsModule())
            .with(new MaterialModule())
            .with(new MaterialCommunityModule())
            .with(new MeteoconsModule())
            .with(new WeathericonsModule())
            .with(new SimpleLineIconsModule())
            .with(new IoniconsModule())
            .with(new FontModule());			//自定义的FontModule
    }
}
```



## 6. 在布局中使用

```xml
<com.joanzapata.iconify.widget.IconTextView
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:gravity="center"
        android:text="{icon-ali-pay}"				//特别注意，这里必须是中划线
        android:textColor="@android:color/holo_orange_dark"
        android:textSize="50sp" />
```

## 7. 显示结果

![](http://blog.img.wangdankai.cn/1566136246824.png)