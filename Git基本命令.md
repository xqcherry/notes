# Git基本命令



## 创建仓库

| **命令**  |      **说明**       |
| :-------: | :-----------------: |
| git init  | 初始化一个 Git 仓库 |
| git clone |  克隆一个 Git 仓库  |

```
git clone <repo> <directory>
```

> **参数说明：**
>
> **repo:** Git 仓库
>
> **directory:** 本地目录



## 提交与修改

|                           **命令**                           |               **说明**               |
| :----------------------------------------------------------: | :----------------------------------: |
|   [git status](https://www.runoob.com/git/git-status.html)   | 查看仓库当前的状态，显示有变更的文件 |
|      [git add](https://www.runoob.com/git/git-add.html)      |           添加文件到暂存区           |
|   [git commit](https://www.runoob.com/git/git-commit.html)   |         提交暂存区到本地仓库         |
|    [git reset](https://www.runoob.com/git/git-reset.html)    |               回退版本               |
|       [git rm](https://www.runoob.com/git/git-rm.html)       |     将文件从暂存区和工作区中删除     |
|       [git mv](https://www.runoob.com/git/git-mv.html)       |        移动或重命名工作区文件        |
| [git checkout](https://www.runoob.com/git/git-checkout.html) |               分支切换               |
| [git log](https://www.runoob.com/git/git-commit-history.html#git-log) |           查看历史提交记录           |



## Git 分支管理

切换分支

```
git checkout
git checkout -b <branchname> // 创建新分支并切换到该分支
```

查看分支

```
git branch
git branch -M <newname> // 分支重命名
git branch -r // 查看远程分支
git branch -a // 查看所有本地和远程分支
git branch -d <branchname> // 删除本地分支
git push origin --delete <branchname> // 删除远程分支
```

合并分支

```
git merge <branchname> // 将其他分支合并到【当前】分支
```

> 合并前先切换到主分支上再合并！

恢复与回退

```
git reset --soft <commit> // 只重置 HEAD 到指定的提交，暂存区和工作目录保持不变
git reset --hard <commit> // 重置 HEAD 到指定的提交，暂存区和工作目录都重置
git reset <commit> // 重置 HEAD 到指定的提交，暂存区重置，但工作目录保持不变
```

```
git revert <commit> // 撤销指定的提交
```

远程操作

|                         **命令**                         |      **说明**      |
| :------------------------------------------------------: | :----------------: |
| [git remote](https://www.runoob.com/git/git-remote.html) |    远程仓库操作    |
|  [git fetch](https://www.runoob.com/git/git-fetch.html)  |  从远程获取代码库  |
|   [git pull](https://www.runoob.com/git/git-pull.html)   | 下载远程代码并合并 |
|   [git push](https://www.runoob.com/git/git-push.html)   | 上传远程代码并合并 |

