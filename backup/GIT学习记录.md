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
工作区：任何一个在本地仓库根文件夹下的文件都在工作区，新建的文件状态为未跟踪，新修改的状态为未暂存，状态可以通过git status指令查看，修改过多可以添加-s参数来进行修改。
```
简写意义
??(未跟踪)
M(已修改)
A(已添加)
D(已删除)
R(重命名)
U(已更新)
```

暂存区：需要通过git add [文件名] 指令从工作区添加，暂存区的内容需要

git commit -m [注释]指令才能进入本地仓库

注：git commit -a -m [注释]指令可以直接完成添加暂存与提交两个操作
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

## 5.查看工作区、暂存区、本地仓库之间的差异 ##
```
git diif  默认比较工作区与暂存区的区别
git diff HEAD 比较工作区与本地仓库的区别
git diff --cached 比较暂存区与本地仓库之间的区别
git diff [ID1] [ID2] [filename]比较特定文件版本差别
```
## 6. 删除文件 ##
### 方式一 ###
```
1. 删除本地文件
2. git add.
3. git commit `注释`
```
### 方式二 ###
```
1. git rm [filenames]
2. git commit `注释`
```
注：git rm --cached [filename]只删除暂存区文件
## 7.gitignore文件 ##
在文件中写入需要忽略的文件名即可 Blob语法模式匹配

## 8.生成ssh密钥 ##
```
cd ~
cd .ssh
ssh-keygen -t rsa -b 4096
```
随后将公钥上传到github上
若生成的本地密钥文件名不为默认文件名，需要进行如下操作：
```
tail -5 config
#github
Host github.com
HostName github.com
PreferredAuthentications publickey
IdentityFile ~/.ssh/test 
```
## 9.下载远程仓库 ##
git clone [ssh链接]
### 先有本地再有远程该怎么办？ ###
```
git remote add origin [link]        origin表示远程仓库的别名
git remote -v                              可以查看远程仓库的别名和地址
git brach -M main                      设置本地分支为main
git push -u origin main:main      将本地仓库的main分支推从给远程仓库的main分支
```
## 10.同步远程仓库与本地仓库 ##
```
git push <remote> <brach> 推送到远程仓库
git pull <remote> 从远程仓库获取
```
注意，执行git pull时，git回自动执行一次合并，如果发生冲突，则需要解决冲突
执行git fetch，只会获取远程仓库中的文件，不会自动合并，需要手动进行
## 11.分支 ##
```
git brach查看当前仓库所有分支[*为当前分支]
git brach [分支名] 创建新分支
git switch [分支名] 切换分支
git checkout [分支名] 切换分支
git checkout filename 放弃指定文件工作区的修改
git checkout -f 放弃所有工作区和暂存区的修改

git merge [分支名] 将分支合并到当前分支

分支被合并后，分支不会自动消失，需要使用
git brach -d [分支名]命令来删除已合并分支
可以用
git brach -D [分支名]命令来删除未合并分支

git tag [标签名]为当前分支打版本号
git tag [标签名]  [提交编号] 为某次commit打版本号
```
发生冲突，可以手动修改，也可以直接使用git merge abort来终止合并。

cherry-pick：挑选一些commit，将其合成为一条新的分支

多人协作时可以使用fork，参考下图：
![image](https://github.com/user-attachments/assets/0bbdf14b-0b70-4f28-b3c0-769b035ed64f)

也可以是分别自建分支，最后pull request一起合并。（团队合作倾向于本方式）