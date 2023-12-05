# git学习

## 基础命令

### 安装

ubuntu 安装：

```
sudo apt-get install git
```

设置：

```
git config --global user.name "NAME"
git config --global user.email "email@email.com"
```

### 创建版本库

初始化

```
git init
```

文件基础操作

```
git add target.file
git commit -m "the commit message"
```

## 版本管理

### 查看状态

查看当前状态（可以告诉有文件被修改过）

```
git status
```

对比文件（可以查看修改内容）

```
git diff target.file
```

### 版本回退

查看提交日志

```
git log
# 只显示一行
git log --pretty=oneline
```

版本回退

```
# 回退到上一个版本
git reset --hard HEAD^
# 回退n个版本
git reset --hard HEAD~n
# 回退指定版本，xxx为版本号
git reset --hard xxxx
```

如果回退之后找不到新版本的 commit id

```
# 查看命令日志
git reflog
```

### 工作区和暂存区

**工作区**：电脑能看到的目录

**暂存区**：在`.git`中

`git add`实际上就是把文件修改添加到暂存区。

`git commit`实际上就是把暂存区的所有内容提交到当前分支。

### 管理修改

git 跟踪并管理的是修改，而非文件。

提交后，查看工作区和版本库里面最新版本的区别

```
git diff HEAD -- target.file
```

每次修改如果不用 `git add` 到暂存区，就不会加入到 `commit` 中。

### 撤销修改

**丢弃工作区的修改**

```
git checkout -- target.file
```

如果文件自修改后还没有被放到暂存区，撤销修改就回到和版本库一模一样的状态。

如果文件已经添加到暂存区后又做了修改，则回到添加到暂存区后的状态。

命令中的 `--` 很重要，如果没有就变成了切换到另一个分支的命令。

**修改添加到了暂存区，还没有提交**

```
git reset HEAD target.file
```

把暂存区的修改回退到工作区。

**如果提交到了版本库**

回退上一版本

```
git reset --hard HEAD^
```

### 删除文件

**从版本库中删除文件**

```
git rm target.file
git commit -m "mesage"
```

**将误删文件恢复到最新版本**

```
git checkout -- target.file
```

**注意：**从来没有被添加到版本库就被删除的文件是无法恢复的。

## 远程仓库

### 创建 SSH Key

在用户主目录下看有没有`.ssh` 目录，如果有，查看是否有 `id_rsa` 和 `id_rsa.pub` ，如果没有

```
ssh-kengen -t rsa -C "email@email.com"
```

`id_rsa`为私钥，`id_rsa.pub`为公钥。

### 添加远程库

**本地仓库关联远程库**

```
git remote add origin git@github.com:xxxx/xxxx.git
```

添加后，远程库的名字就是 `origin`。

**推送本地库内容至远程库**

```
git push -u origin master
```

把当前分支`master`推送到远程。

### 删除远程库

```
git remote rm <name>
```

使用前建议先用 `git remote -v` 查看远程库信息

```
git remote -v
```

**注意：**这里的删除其实是解除了本地和远程的绑定关系，并不是物理上删除了远程库。

### 克隆远程库

```
git clone git@github.com:xxxx/xxxx.git
```

使用 ssh 协议速度最快。

## 分支管理

### 创建与合并分支

`HEAD` 指向的就是当前分支。

创建分支

```
git checkout -b new_branch
# git checkout 命令加上 -b 参数表示创建并切换，相当于以下两条命令
git branch new_branch
git checkout new_branch
```

查看当前分支

```
git branch
```

合并指定分支到当前分支

```
git merge new_branch
```

`Fast-forward` “快进模式”，直接把 `master` 的指针指向 `dev` 当前的提交。

删除分支

```
git branch -d new_branch
```

也可以使用 switch

创建并切换到新的分支

```
git switch -c new_branch
```

切换已有分支

```
git switch new_branch
```

### 解决冲突

查看冲突文件

```
git status
```

处理冲突文件后再`git add`后提交

查看分支合并情况

```
git log --graph --pretty=oneline --abbrev-commit
```

### 分支管理策略

git 合并分支时使用 `Fast forward` 模式，但这种模式下，删除分支后，会丢掉分支信息。

如果强制禁用 `Fast forward` 模式，git 就会在 merge 时生成一个新的 commit。

```
git merge --no-ff -m "message" new_branch
```

### bug 分支

现场储存

```
git stash
```

查看现场

```
git stash list
```

恢复现场

```
# 恢复后，stash内容不删除
git stash apply
# 删除
git stash drop
# 恢复并删除
git stash pop
```

可以多次 stash，恢复的时候先用 `git stash list` 查看，然后恢复指定的 stash

```
git stash apply stash@{0}
```

复制特定提交到当前分支

```
git cherry-pick xxxx
```

### Feature分支

强行删除分支

```
git branch -D new_branch
```

### 多人协作

查看远程库信息

```
git remote
```

显示更详细的信息

```
git remote -v
```

推送分支

```
git push origin branch
```

创建远程分支到本地

```
git checkout -b branch origin/branch
```

设置本地分支和远程分支链接

```
git branch --set-upstream-to=origin/branch branch
```

### rebase

将分支的提交历史整理成一条直线

```
git rebase
```

## 标签管理

### 创建标签

创建标签

```
git tag xxx
```

查看所有标签

```
git tag
```

默认标签是打在最新提交的commit上的，也可以对历史提交的 commmit id 打标签。

```
git tag xxx commmit_id
```

**注意：**标签不是按时间顺序列出，而是按字母排序。

查看标签信息

```
git show tag_name
```

创建带有说明的标签

```
git tag -a xxx -m "message" commit_id
```

### 操作标签

删除标签

```
git tag -d xxx
```

推送标签到远程

```
git push origin xxx
```

一次性推送全部尚未推送到远程的本地标签

```
git push origin --tags
```

如果标签已经推送到远程，要删除

```
# 先从本地删除
git tag -d xxx
# 从远程删除
git push origin :refs/tags/xxx
```

## 自定义git

git 显示颜色

```
git config --global color.ui true
```

### 忽略特殊文件

使用 `.gitignore`

添加被 `.gitignore` 忽略的文件

```
git add -f xxx
```

可以使用 `git check-ignore` 检查规则

```
git check-ignore -v xxxx
```

可以使用`!` 来添加例外规则。

可以通过[https://gitignore.itranswarp.com](https://gitignore.itranswarp.com/)在线生成`.gitignore`文件。

### 设置别名

设置别名

```
git config --global alias.st status 
```

`--global`参数是全局参数。

每个仓库的 git 配置文件都放在 `.git/config` 文件中，当前用户的 git 配置文件放在用户主目录下的一个隐藏文件 `.gitconfig` 中。