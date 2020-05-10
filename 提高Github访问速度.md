**由于Github的服务器在美国，所以在中国直接访问Github速度会有点慢，下面通过修改hosts文件来提高访问速度**

1. 打开网站**[http://github.global.ssl.fastly.net.ipaddress.com/](http://github.global.ssl.fastly.net.ipaddress.com/)**
2. 记下ip地址，如199.232.xx.xxx
3. <img src="http://blog.img.wangdankai.cn/20200411173226.png" style="zoom:67%;" />
4. 打开网站**[https://github.com.ipaddress.com/](https://github.com.ipaddress.com/)**
5. 记下ip地址，如140.82.xx.xxx
6. <img src="http://blog.img.wangdankai.cn/20200411172518.png" style="zoom:67%;" />
7. 编辑hosts文件

**Linux和Mac系统下在终端输入：**

```bash
sudo vim /etc/hosts
```

在文件中添加下面两行

```bash
199.232.xx.xxx github.global.ssl.fastly.net
140.82.xx.xxx github.com
```

**Windows：**

使用notepad++管理员模式打开目录**C:\Windows\System32\drivers\etc****下的hosts文件

添加下面两行代码

```bash
199.232.xx.xxx github.global.ssl.fastly.net
140.82.xx.xxx github.com
```

8. 在终端或者CMD下，输入以下命令并执行

```bash
ipconfig /flushdns
```

