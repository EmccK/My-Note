# 全局Context

## 新建MyApplication类

```kotlin
class MyApplication : Application() {

    companion object {
        lateinit var context: Context
    }

    override fun onCreate() {
        super.onCreate()
        context = applicationContext
    }
}
```

## 在Manifest中添加MyApplication类

```kotlin
android:name=".MyApplication"
```

## 使用

```kotlin
MyApplication.context
```

