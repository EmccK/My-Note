## 安装V2rayU

订阅链接：https://doubledou.win/link/u5agDinK3eTEKLbQ?client=v2

安装`Notion`

## 1. 让dock更顺滑

```shell
defaults write com.apple.Dock autohide-delay -float 0 && killall Dock
```

恢复原来的样子

```shell
defaults delete com.apple.dock; killall Dock
```

## 2. 安装HomeBrew

```shell
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install.sh)"
```

## 3. 显示系统信息

```shell
brew install archey
brew install neofetch
```

然后执行`archey/neofetch`命令

## 4. 配置环境变量

```shell
vim ~/.zshrc
# 在文件的最后追加
export PATH="$PATH:环境的地址"
```

## 5. 设置默认Python3版本

先执行`which python3`，获得Python3的路径

例如`/usr/local/bin/python3`

然后`vim ~/.zshrc`

在最后插入

```shell
alias python="/usr/local/bin/python3"
```

然后执行

`source ~/.zshrc`

## 6. 配置Git

1. 首选安装git`brew install git`

2. 设置用户名和邮箱

    `git config --global user.name "wangkai9799"`
    `git config --global user.email 979904973@qq.com`

3. 检查配置信息

    `git config --list`

4. 签发秘钥

    `cd ~/.ssh`

    `ssh-keygen`

    然后将秘钥添加都GitHub中，复制`~/.ssh`文件夹中的`id_rsa.pub`中的内容

5. 项目管理

    1. 初始化，在要管理的项目目录下执行`git init`命令，这时会自动生成`.git`文件夹，这个就是本地的版本库。

## 7. 安装文件已损坏

`sudo xattr -d com.apple.quarantine /Applications/xxxx.app`

