# 在Linux中配置usb连接

1. 连接USB之后，使用 lsusb命令查看usb端口，拔插一次看哪个没了，哪个就是

2. ID 04e8:689e Google

3. 在/etc/udev/rules.d目录下创建一个规则文件，使用root权限创建，在文件中添加内容

   ` SUBSYSTEMS=="usb", ATTRS{idVendor=="04e8", ATTRS{idProduct}=="689e", MODE="0666", OWNER="huangbiao"} `

   这里的idVendor是第一个，idProduct是第二个，OWNER是当前用户

4. 重启USB服务

   ` sudo /etc/init.d/udev restart `

   