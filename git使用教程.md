# git使用教程

	## 新电脑第一次配置

1. 准备工作，设置用户名和邮箱

   ``` 
   git config --global user.name "wangkai9799"
   git config --global user.email 979904973@qq.com
   ```

2. 检查配置信息

   ` git config --list`

3. 签发秘钥

   ```
   cd ~/.ssh
   ssh-keygen
   ```

4. 将秘钥添加到github中去，复制`~/.ssh`文件夹中的` id_rsa.pub`中的内容

## 项目管理

1. 在要管理的项目目录下执行`git init`命令，这时会自动生成.git文件夹，这个就是本地的版本库

2. 执行`git add . `命令，将文件夹内的所有文件添加到本地的版本库中

3. ` git commit -m "这里填写提交的信息"`

4. 在github中创建项目，然后将项目地址ssh地址复制下来

   ` git remote add origin <项目ssh地址>`

5. 提交代码到github仓库

   `git  push -u origin master `

## 项目维护

每一次提交文件到github都要执行上面的 2,3,5步