# Mac上出现Read-only file system

## 对于Big Sur

- 首先进入Recovery模式（重启的时候按住Command + R），然后关闭SIP，两个都要关掉

```
csrutil disable
csrutil authenticated-root disable
```

- 这里的/dev/disk1s5是磁盘工具中系统盘的设备去掉后面的s1，然后后面是挂载的路径，我这里放在了下载目录下的mac文件夹

<img src="http://blog.img.wangdankai.cn/image-20210105212418217.png" alt="image-20210105212418217" style="zoom:50%;" />

~~~
sudo mount -o nobrowse -t apfs /dev/disk1s5 /Users/dankai/Downloads/Mac
~~~

- 挂载之后就可以在Mac文件夹中修改系统文件了

- 然后恢复系统快照

~~~
sudo bless --folder /Users/dankai/Downloads/Mac/System/Library/CoreServices --bootefi --create-snapshot
~~~

- 然后重启

## 对于Catalina

- 重启进入恢复模式，在恢复模式命令行输入`csrutil disable`，再重启进入系统。
- 在系统的命令行输入 `sudu mount -uw /`。