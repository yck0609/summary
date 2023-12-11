# git常用命令

## 下载安装

从git官网下载安装包，安装完毕后就可以使用命令行的 git 工具，在开始菜单里找到"Git"->"Git Bash"，会弹出 Git 命令窗口，你可以在该窗口进行 Git 相关命令行的操作。

```git
具体可参考https://www.runoob.com/git/git-install-setup.html
```

## 全局配置环境

配置个人用户名和电子邮箱

```git
git config –globle user.name “runoob”
git config –globle user.email text@runoob.com
```

配置完毕后，可以通过$ git config –list命令查看所有的配置信息。
也可直接查询某个环境变量的信息。

```git
git config user.name
git config user.email
```

## 查看工作区状态

```git
git status
```

- 状态一：修改了没有添加到缓存区（红色），此时可以通过git diff 查看修改了的内容，“-”号是修改前，“+”号是修改后，第一个加号后修改的前一行。第二个加号是修改的内容。
- 状态二：修改了添加到缓存区（绿色）
- 状态三：On branch master nothing to commit, work tree clean 表明无修改内容

## 添加文件到git仓库

分两步：
把修改的修改添加到版本库里的暂存区，可以单独添加某个文件，可多次使用

```git
git add <file>
```

把暂存区的所有内容提交到当前分支，提交的说明一定要写（字符串加双引号）

```git
git commit -m <message>
```

如果 commit 的注释写错了，想要修改注释

```git
git commit --amend
```

## 本地同步更新远程分支

```git
git pull
```

如果项目是多人合作的，那么就需要在拉去别人更新的代码合并到本地。Git会自动合并本地代码。

## 把缓存中的代码推送到远程分支

```git
git push
```

## 撤销修改

- 场景一：修改了文件但是未被add

```git
git checkout -- <file>
```

- 场景二：修改了工作区内容，还添加到了暂存区时，想丢弃修改，分两步

```git
git reset HEAD <file> 就回到了场景一
git checkout -- <file>
```

- 场景三：修改文件已被commit,但是没有推送到远程库，想要撤销本次提交，只能切换版本

```git
git reset --hard HEAD^
```

## 从远程分支拉取项目

```git
git clone SSH/HTTPS地址 -b <分支名>
```

## 分支管理

当前分支作业时

```git
1)查看分支：git branch
2)创建分支：git branch <name>
3)切换分支：git checkout <name>
4)创建+切换分支：git checkout -b <name>
5)合并某分支到当前分支：git merge <name>
6)删除分支git branch -d <name>
```

临时切换分支作业时

```git
1)暂存分支工作状态： git stash
2)查看分支存储的工作状态： git stash list
3)恢复分支工作状态： git stash apply
4)删除分支存储的工作状态：git stash drop
5)恢复并删除分支存储工作状态：git stash pop
```

切换远程分支
：当前分支branch1工作，现在需要在分支branch2上工作，则需要切换

```git
git fetch origin branch2(分支名)
git checkout branch2
```
