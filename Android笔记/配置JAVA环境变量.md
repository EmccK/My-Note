# 配置JAVA环境变量

## 1. 下载java安装包

现在下载java安装包必须登录才行，所以还得注册一个账号

[**下载地址**](https://www.oracle.com/technetwork/java/javase/downloads/index.html)

## 2. 安装

下载好只需要一直点下一步即可安装成功

## 3. 配置环境变量

右键单击 **`此电脑->属性->高级系统设置->环境变量`**

![](http://blog.img.wangdankai.cn/Snipaste_2019-09-26_20-52-02.png)

![](http://blog.img.wangdankai.cn/Snipaste_2019-09-26_20-52-20.png)

1.  **新建->变量名：`JAVA_HOME` 变量值：`C:\Program Files\Java\jdk1.8.0_181` (即JDK的安装路径)**

   ![](http://blog.img.wangdankai.cn/Snipaste_2019-09-26_20-57-59.png)

2. **在 `系统变量` 栏里找到`Path`,然后双击**

   ![](http://blog.img.wangdankai.cn/Snipaste_2019-09-26_20.png)

3. **然后单击`新建`，填入`%JAVA_HOME%\bin`**
   **再`新建`，填入`%JAVA_HOME%\jre\bin`**

   **然后点击确定保存**

   ![](http://blog.img.wangdankai.cn/Snipaste_2019-09-26_21-02-06.png)

4. **新建->变量名：`CLASSPATH` 变量值：`.;%JAVA_HOME%\lib;%JAVA_HOME%\lib\dt.jar;%JAVA_HOME%\lib\tools.jar**`

   ![](http://blog.img.wangdankai.cn/Snipaste_2019-09-26_21-03-57.png)

## 4. 通过控制台命令验证配置是否成功

**win+R**打开运行，输入`cmd`，打开控制台
在控制台分别输入**`java`、`javac`、`java -version**`命令，出现如下所示即为配置成功

`java`命令：

![](http://blog.img.wangdankai.cn/Snipaste_2019-09-26_21-08-03.png)

`javac`命令：

![](http://blog.img.wangdankai.cn/Snipaste_2019-09-26_21-08-29.png)

`java -version`命令：

![](http://blog.img.wangdankai.cn/Snipaste_2019-09-26_21-10-54.png)