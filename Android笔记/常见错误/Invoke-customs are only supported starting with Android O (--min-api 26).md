# Invoke-customs are only supported starting with Android O (--min-api 26)  

在gradle.build中添加以下内容

```java
    compileOptions {
        sourceCompatibility JavaVersion.VERSION_1_8
        targetCompatibility JavaVersion.VERSION_1_8
    }
```

