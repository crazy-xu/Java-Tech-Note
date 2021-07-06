1. 在github上创建一个新仓库。

2. 通过cmd 进入到命令行

3. 初始化文件夹

   ```sh
   git init
   ```

4. 关联本地已有仓库

   ```shell
   git remote add origin https://github.com/crazy-xu/design-patterns.git
   ```

5. 添加文件
   ```shell
   # 不但可以跟单一文件，还可以跟通配符，更可以跟目录。一个点就把当前目录下所有未追踪的文件全部add了 
   git add .
   ```
6. 提交文件到仓库
   ```shell
   git commit -m "first commit" 
   ```
7. 将本地库的内容推送到远程库
   ```shell
   git push -u origin master/main
   ```





* 更新代码 git pull
* 切换分支 git checkout 分支
* 克隆代码到本地：git clone https://github.com/crazy-xu/design-patterns.git



另外的

1. 在github上创建一个新仓库。
2. 通过cmd进入到命令行。
3. 克隆代码到本地：git clone xxxx
4. **切记修改提交用户名和邮箱，不要用公司的！！！**
5. 将代码复制到目录里，然后通过idea提交，屏蔽一些没有用的文件夹。
