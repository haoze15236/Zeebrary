# 前言

参考：

[Git官网](https://git-scm.com/docs)
[廖雪峰的官方网站](https://www.liaoxuefeng.com/wiki/896043488029600/896067008724000)

# 基本概念
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200517000518601.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzIzMTQ2MzY3,size_16,color_FFFFFF,t_70)
## 工作区(working directory)
就是本地文件夹，点开文件看到的内容都是工作区的内容。
## 暂存区(staging area)
在对文件做了操作之后，把这些操作先暂时的保存下来，保存这些操作的区域叫做暂存区(staging area)，也叫索引(index)。而后一次性提交(commit)暂存区内的所有的修改，就会形成版本库中分支的一个节点。
## 版本库(repository)
一个文件每次修改被提交之后，就形成一个版本，保存所有文件所有版本的区域。
### 分支(branch)
对版本库内所有文件的所有版本的一个归类整理，每次暂存区commit一次，就会生成暂存区内修改的文件的一批版本，这一批版本归类成一个节点，一个一个节点自然就连成了一条线，也就是一个分支。
# git操作
## 工作区
工作区的修改方式
- 最直接的就是在本地电脑上打开文件进行修改。
- git 从版本库中覆盖工作区内容，从而修改工作区文件。

## 暂存区
```shell
#添加修改的文件到暂存区(staging area)
git add filename
git add -A  添加所有变化
git add -u  添加被修改(modified)和被删除(deleted)文件，不包括新文件(new)
git add .   添加新文件(new)和被修改(modified)文件，不包括被删除(deleted)文件
#删除文件并更新stage
git rm filename
#查看暂存区当前状态  有提示如何回滚操作
git status

#保存工作区和暂存区的内容,使用HEAD指向的内容覆盖工作区和暂存区
git stash
```

## 版本库(repository)
```shell
# 创建本地仓库
git init
```

### 分支

```shell
#创建新分支dev
git branch dev
#查看分支
git branch
#删除dev分支
git branch -d dev
#获取branch命令的帮助
git branch -h

#创建并切换至分支dev
git checkout -b dev
#切换至分支dev
git checkout dev
```
#### 分支管理
```shell
#讲当前分支的HEAD指针指向要branchname分支的HEAD处
git merge branchname
#不使用fast forward 模式合并
#自动增加一次commit,同步合并branchname分支内容，对于当前分支来说，增加了一个节点
git merge --no-ff -m "merge with no-ff" branchname
##合并一个版本的修改
git merge --squash <commit_id>
# 合并出现没有关联的错误时加后缀--allow-unrelated-histories
#将分支branchname的commit_id在当前分支提交生成一个commit
git cherry-pick <branchname> <commit_id>

#重建分支.
git rebase 
#--onto 从branch1与branch2分支分叉处截断,再从branche2,branch3分支处截断,丢弃branch2,连接branch1 和branch3
git rebase --onto <branch1> <branch2> <branch3>
```
[merge --no-ff cherry-pick操作详细图解](https://editor.csdn.net/md/?articleId=103032475)

#### 分支节点
```shell
#将stage的所有修改提交到当前(head指向的)分支,形成一个分支的节点
git commit -m "message"
#获取commit命令的帮助
git commit -h
#给HEAD指向的提交添加tag,可以替代commit_id来使用
git tag <tagname>
```
### 查看日志
```shell
#单行查看目前所在版本及之前日志
git log --pretty=oneline
#图形化形式查看提交分支线
git log --graph --pretty=oneline --abbrev-commit
#查看所有的版本变更记录 理论上可获取到所有版本的commit_id
git reflog 
#查看标签
git tag
```
### 解决冲突
当合并分支出现冲突时，git会停留在当前合并的过程，等待处理冲突
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200517213451985.png)
此时手动修改冲突文件，add,commit之后，整个合并分支过程才会结束。
### 回滚
```shell
# 使用版本库中当前分支的当前节点的文件内容覆盖工作区的文件
git checkout -- filename

# 使用版本库中当前分支的文件内容覆盖暂存区的内容,不修改工作区的内容
git reset HEAD filename
# 使用版本库中当前分支的当前节点的文件内容覆盖工作区和暂存区的内容
git reset --hard
#num是往回回退多少个节点(版本)
git reset --hard HEAD~num  
#可根据commit_id跳跃到任意节点(版本)
git reset --hard <commit_id> 
# 使用版本库中指定的commit_id(默认当前分支的当前节点)的文件内容覆盖暂存区的内容,不覆盖工作区
git reset --mixed <commit_id>
#移动HEAD指针至commit_id指向的版本,不修改工作区和暂存区
git reset --soft <commit_id>
#移动HEAD指针至commit_id指向的版本,覆盖工作区和暂存区,如果此时工作区和暂存区内容不一致，则保留工作区的内容
git reset --merge <commit_id>
```
# 远程仓库
```shell
#克隆远程仓库 克隆url指向的远程仓库
git clone url
#添加远程仓库连接,使本地库关联远程库,理论上一个本地库可以关联多个远程库
git remote add <remote_name> url
#从远程仓库下载指定分支到本地指定分支,若不指定本地分支，则保存在
git fetch origin <remote branch name>:<local branch name>
#直接用远程仓库的代码覆盖本地代码
git pull 
#强制推送本地分支到远程remote branch name(可用于回滚远程分支)
git push -f origin <remote branch name>
# 建立本地分支与远程分支的映射
git branch --set-upstream-to origin/branch_name local_branch_name
```

# 配置

## 本地git创建多个密钥
[https://www.cnblogs.com/SUNSHINEC/p/8617029.html](https://www.cnblogs.com/SUNSHINEC/p/8617029.html)

## 忽略文件
- 配置ignore文件
- 除了配置.ingore文件之外还可以配置如下参数
```shell
#默认文件是不会被改变的,若需要修改文件并提交需要先恢复此参数
git update-index --assume-unchanged <files>
#恢复
git update-index --no-assume-unchanged <files>
```

