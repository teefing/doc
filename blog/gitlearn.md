---
title: gitlearn
date: 2017-10-18 12:06:36
tags:
  - git
category: 专业知识
---

### 初始准备工作

mkdir learngit 创建 learngit 文件夹  
cd learngit 进入 learngit 文件夹  
pwd 显示当前目录

### 基本操作

git init git 仓库初始化，把目录变成 git 可以管理的仓库

git add readme.txt 将当前目录下的 readme.txt 加入到 git 仓库的暂存区  
git commit -m "wrote a readme file" 将暂存区的 readme.txt 提交到分支上

git status 查看状态

git diff HEAD -- readme.txt 查看版本库和工作区里面最新版本的区别

.gitignore 用于 git 忽略某些文件和文件夹的配置文件

git log 显示从最近到最远的提交日志

git log --pretty=oneline git log 的美观版本，过滤掉许多信息，更简洁

### 版本回退

git reset --hard HEAD^ 回退到上一个版本  
git reset --hard "HEAD^" 在 windows 中需要加入双引号  
git reset --hard HEAD"^" 在 windows 中的另一种写法  
git reset --hard "HEAD^^" 回退到上上个版本  
git reset --hard HEAD~1 回退到上一个版本，将数字 1 改为 n 就是回退到前 n 个版本
git reset --hard commit 的版本号 回退到制定版本，参数为该版本的版本号

git reflog 显示每一次命令，版本回退的版本号也可以从中查看

### 文件恢复与删除

git checkout -- readme.txt 让 readme.txt 回到最近一次 git commit 或者 git add 时的状态，也就是把工作区中的文件恢复成版本库中的文件

git rm test.txt 将本地的和版本库中的 test.txt 文件都删除，想要恢复就得先将操作过的其他文件转移，再进行版本回退，之后再将其他文件转移回来  
如果只是本地的文件被删除了，可以 git checkout -- file 恢复  
git commit -m "remove test.txt" 再更新一下分支

### github 仓库使用

ssh-keygen -t rsa -C "youremail@example.com" 创建 SSH Key，创建后在用户主目录下的.ssh 目录中会有 id_rsa 和 id_rsa.pub 两个文件，其中 id_rsa.pub 是公钥，id_rsa 是私钥

git remote add origin git@server-name:path/repo-name.git 将本地 git 和 GitHub 上的仓库关联

git push -u origin master 将本地 git 提交到 GitHub 仓库的 master 分支，-u 在第一次推送 master 分支的所有内容时要使用

git push origin master 不是第一次就用这个

git clone 用于从远程库克隆

### git 分支操作

git checkout -b dev 创建并切换到 dev 分支，效果等同于下面两个命令

git checkout -b dev origin/dev 创建远程 origin 的 dev 分支到本地

git branch dev 创建 dev 分支
git checkout dev 切换到 dev 分支

git branch 查看有哪些分支和当前所处的分支

#### git 分支合并
git checkout master 切换到 master 分支
git merge dev 当当前分支在 master 下时，（master 分支未改动，dev 分支已经改动），将 master 分支和 dev 分支合并  
git branch -d dev 之后删除 dev 分支

创建一个分支，之后分别在 master 和分支上修改 readme.txt，之后合并 master 和分支会产生冲突，
此时需要把冲突后的文件手动解决冲突，修改文件后 add commit，冲突就解决了,之后就可以删除分支了

git log --graph --pretty=oneline --abbrev-commit 比较图像化地显示分支历史

### git 工作区备份操作

git stash 将当前的工作区备份存档

git stash list 显示当前分支的工作区存档

git stash apply 恢复最近一次的工作区存档  
git stash apply stash@{0} 若经过多次存档，恢复指定的工作区存档

git stash drop 删除最近一次的工作区存档

git stash pop 恢复并删除最近一次的工作区存档，等同于 apply+drop

git branch -D dev 强行删除 dev 分支

###git 远程库操作
git remote 查看远程库信息

git remote -v 查看详细的远程库信息

git remote rm origin 把本地与 origin 远程库的关联取消

### git 自定义

git config --global color.ui true 美化

git add -f App.class 将被.ginignore 忽略的文件强制添加到 Git

git check-ignore -v App.class 检查定位导致 App.class 添加到 Git 失败的规则

.gitignore. windows 上这样就能创建.gitignore 文件

git config --global alias.st status 为 git 命令取别名  
git config --global alias.last 'log -1' 也可以用引号把一句命令引起来，为它取个别名

git branch --set-upstream branch-name origin/branch-name 创建本地分支与远程分支的链接关系

### git 标签操作

git tag 查看所有标签

git tag v1.0 为最新提交的 commit 打上标签

git tag v1.0 commit 的版本号 为指定的 commit 打上标签

git show v1.0 查看标签信息

git tag -a v1.0 -m "version 1.0 released" 3435334 创建带有说明的标签

git tag -s v1.0 -m "version 1.0 released" 3435334 gpg 配置成功前提下用私钥签名一个标签

git tag -d v1.0 删除标签

git push origin v1.0 推送某个标签到远程

git push origin --tags 一次性推送所有尚未推送到远程的本地标签

git tag -d v1.0 如果标签已经推送到远程，要删除该标签先删除本地标签  
git push origin :refs/tags/v1.0 再把远程标签删除

参考于https://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000

