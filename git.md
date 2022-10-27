# 一、git操作指令

**1、查看当前状态、修改**

git remote -v 查看远程仓库地址

git status 查看详细状态

git status -s 查看简略状态

git diff 查看具体每一行改动

git config credential.helper store 保存用户密码，windows客户端保存密码见文末

**2、让git忽略掉文件权限检查：**

old mode 100755

new mode 100644

git config --add core.filemode false

**3、Git 本地的撤销修改和删除操作**

**撤销修改**

git checkout - bin/postproc_test.cfg 注意中间一定要有-

**删除操作**

直接rm掉本地文件然后git commit即可

**4、创建显示分支切换分支**

git branch test 创建test分支

git branch 显示当前所在分支吗

git branch -v 显示当前分支详细信息

git branch -a 显示本地所有分支

git push origin 2.3.1803.2:2.3.1803.2 将本地创建的分支branch同名push到远程

git pull origin post_1805:post_1805 将远程同名branch更新到本地branch

git checkout master 切换到master分支

git checkout -b caigou_v1.0 origin/release/caigou_v1.0

[git checkout -b 本地分支名 origin/远程分支名]该命令可以将远程git仓库里的指定分支拉取到本地，这样就在本地新建了一个dev分支，并和指定的远程分支release/caigou_v1.0关联了起来。

git push origin –delete 分支名 删除远程分支

git branch –d 分支名 删除本地合并了的分支

git branch –D 分支名 删除本地未合并的分支

git remote prune origin 清理本地显示但是远程并不存在的分支

**5、查看历史记录 查看远程git地址**

git log 查看全部提交记录

git log --oneline 查看记录，一次提交只显示一行记录

git remote -v 查看git地址，查看url

**6、创建查看删除tag**

git tag -a 2.3.1802.0 -m "tag info" 创建tag

git push --tags push到仓库

git tag | git tag -l 显示当前所有tag

git tag -d 2.3.1802.0 删除本地tag

git push origin --delete tag <tag名> 删除远程tag

git clone xxx 先clone整个仓库，然后

git checkout tag_name check到某个tag

**7、保存和恢复工作进度**

git stash

git stash list

git stash pop [--index] [<stash>]

如果不使用参数，会恢复最新保存的工作进度，并将恢复的工作进度从存储的工作进度列表中清除。

如果提供<stash>参数（来自git stash list显示的列表），则从该<stash>中恢复。恢复完毕也将从进度列表中删除<stash>。

选项--index除了恢复工作区的文件外，还尝试恢复暂存区。这也就是为什么在本章一开始恢复进度的时候显示的状态和保存进度前的略有不同。

git stash apply [--index] [<stash>] 除了不删除恢复的进度之外，其余和git stash pop 命令一样。

git stash drop [<stash>] 删除一个存储的进度。默认删除最新的进度。

git stash clear 删除所有存储的进度。

git stash branch <branchname> <stash> 基于进度创建分支。

**8、添加子模块submodule**

git submodule update --init -- recursive

**更新子模块时，需要在引擎(Quark)的子模块(lattice)文件夹pull，才能将最新的子模块内容拉下来；**

**然后再在整个引擎文件夹commit，commi**t

**9、[git在本地修改并删除一个文件后怎样从服务端拉取？](https://segmentfault.com/q/1010000004227370)**

单个文件

git checkout a.php

当前目录

git checkout .

linux上下载下来的branch，修改后如果想提交，需要先在linux本地创建一个新的分支，然后将该新分支push到远程的老branch。

**10、git log显示历史提交记录 [本地commit但没有push也这样查看]**

git log

git log --oneline

此命令：

- 每行显示一个 commit
- 显示 commit 的 SHA 的前 7 个字符
- 显示 commit 的消息

git log --stat

运行该命令并查看显示详细结果。**配合git show [SHA前几个数字]查看某一次提交的详细内容**

**11、git撤销commit，但未git push的文件**

在git push的时候，有时候我们会想办法撤销git commit的内容

a.查看找到之前提交的git commit的id

git log

b. 找到想要撤销的id【**注意不是刚commit的这个id，通常应该是下一个**】

git reset –hard id 完成撤销,同时将代码恢复到前一commit_id 对应的版本

git reset id 完成Commit命令的撤销，但是不对代码修改进行撤销，可以直接通过git commit 重新提交对本地代码的修改

**git撤销add，但未commit的文件**

git status   先看一下add中的文件，依次可以看到add但未commit，修改但未add的，不在git管辖的文件

git reset HEAD    如果后面什么都不跟的话 就是上一次add 里面的全部撤销了

git reset HEAD XXX/XXX/XXX.cpp     就是对某个文件进行撤销了


# 二、git windows客户端tortoisegit保存密码

【注意】要在带 【.git 文件夹】的目录单击右键

[1](pic/1.png)

[2](pic/2.png)

[3](pic/3.png)

[credential] helper = store


# 三、SVN命令：

1、、查看仓库或分支信息

svn info

2、查看最近几次提交         svn log -l 10

3、提交指定文件

svn commit  ./include/dtcp_ip ./lib/dtcp_ip/   -m "[add] dtcp related file and so"