# Android9以上不允许明文通信

## 解决办法

1. 新建res/xml文件夹

2. 新建`network_config.xml`文件，名字可自定

3. 内容为

   ```xml
   <?xml version="1.0" encoding="utf-8"?>
   <network-security-config>
       <base-config cleartextTrafficPermitted="true" />
   </network-security-config>
   ```

4. 在AndroidManifest中添加即可

   ```xml
   android:networkSecurityConfig="@xml/network_config"
   ```

   