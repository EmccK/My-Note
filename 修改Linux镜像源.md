# 修改Linux镜像源

1. 备份自带镜像源

   ` cp /etc/apt/sources.list /etc/apt/sources.list_backup `

2. 编辑源配置文件/etc/apt/sources.list

   ` vim /etc/apt/sources.list `

3. 将原有的源替换为阿里云或清华大学的源

   - 阿里源

     ``` 
     deb http://mirrors.aliyun.com/ubuntu/ trusty main restricted universe multiverse
     deb http://mirrors.aliyun.com/ubuntu/ trusty-security main restricted universe multiverse
     deb http://mirrors.aliyun.com/ubuntu/ trusty-updates main restricted universe multiverse
     deb http://mirrors.aliyun.com/ubuntu/ trusty-proposed main restricted universe multiverse
     deb http://mirrors.aliyun.com/ubuntu/ trusty-backports main restricted universe multiverse
     deb-src http://mirrors.aliyun.com/ubuntu/ trusty main restricted universe multiverse
     deb-src http://mirrors.aliyun.com/ubuntu/ trusty-security main restricted universe multiverse
     deb-src http://mirrors.aliyun.com/ubuntu/ trusty-updates main restricted universe multiverse
     deb-src http://mirrors.aliyun.com/ubuntu/ trusty-proposed main restricted universe multiverse
     deb-src http://mirrors.aliyun.com/ubuntu/ trusty-backports main restricted universe multiverse
     ```

   - 清华大学源

     ``` 
     deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial main restricted
     deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial-updates main restricted
     deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial universe
     deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial-updates universe
     deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial multiverse
     deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial-updates multiverse
     deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial-backports main restricted universe multiverse
     deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial-security main restricted
     deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial-security universe
     deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial-security multiverse
     ```

4. 更新镜像源

   ` sudo apt-get update ` 