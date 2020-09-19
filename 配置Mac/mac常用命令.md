1. 让dock更顺滑

   ```shell
   defaults write com.apple.Dock autohide-delay -float 0 && killall Dock
   ```

   恢复原来的样子

   ```shell
   defaults delete com.apple.dock; killall Dock
   ```

2. 安装HomeBrew

   
   
3. 显示系统信息

   ```shell
   brew install archey
   ```

   然后执行`archey`命令

4. 配置环境变量

    ```shell
    vim ~/.zshrc
    # 在文件的最后追加
    export PATH="$PATH:环境的地址"
    ```

5. 

