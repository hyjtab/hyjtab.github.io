## 1. git配置 ##
设置用户名与邮箱信息
```
git config --global user.name
git config --global user.email
```
设置命令行别名：
```
touch ~\.bashrc
```
随后在文件中写入以下命令别名
```
alias git-log='git log --pretty=oneline --all --graph --abbrev-commit'
alias ll='ls -al'
```
## 2. git本地仓库初始化 ##
很简单
```
git init
```
建立一个.git文件夹。
## 3.跟踪、工作区与暂存区 ##
工作区：任何一个在本地仓库根文件夹下的文件都在工作区，新建的文件状态为未跟踪，新修改的状态为未暂存

暂存区：需要通过git add [文件名] 指令从工作区添加，暂存区的内容需要git commit -m [注释]指令才能进入本地仓库
```
git ls-files                  此命令可以查看暂存区的所有文件
```
通配符：git add .这个命令就可以认为是将当前目录下所有文件都加入暂存区

可以通过git log命令来获取当前分支的版本信息，方便转换和回退，该命令有几个选项：
```
--all                           显示所有分支
--pretty=oneline       将提交信息显示为一行
--abbrev-commit      使得输出的信息更加简短 
--graph                     以图的方式进行显示
```
## 4.版本回退 ##
git reset --[hard | mix | soft] commiyID

soft只回退仓库状态（版本HEAD指针），不清空暂存区和工作区（只是撤回了一次commit）

hard工作区与暂存区一起撤回（可以使用git reflog挽回）

mix撤回仓库与暂存区内容

HEAD~ HEAD^表示前一个版本，后面可加数字，当前版本作为0，越早越大。

##5.查看工作区、暂存区、本地仓库之间的差异 ##
git diif  默认比较工作区与暂存区的区别

git diff HEAD 比较工作区与本地仓库的区别

git diff --cached 比较暂存区与本地仓库之间的区别

git diff [ID1] [ID2] [filename]比较特定文件版本差别
## 6. 删除文件 ##
### 方式一 ###
1. 删除本地文件
2. git add.
3. git commit `注释`
### 方式二 ###
1. git rm [filenames]
2. git commit `注释`