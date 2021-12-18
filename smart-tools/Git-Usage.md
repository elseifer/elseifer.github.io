# Git

工作区：放置代码等文件的目录；  
暂存区：英文为 stage 或 index，一般存放在 .git 目录下的 index 文件，所以有时我们把暂存区也称作索引；  
版本库：工作区有一个隐藏目录 .git，这个不算工作区，而是 Git 的版本库。  

![GIT 概念](./images/git-workspace.png)

## 1.添加文件
把文件加入到暂存区  
`git add <file>`

## 2.删除文件
将文件从暂存区和工作区中删除
`git rm <file>`

文件从暂存区域移除，但保留在当前工作目录中
`git rm --cached <file>`

## 3.撤销修改
放弃单个文件修改
`git checkout -- <file>`

放弃所有文件修改  
`git checkout .`

把在工作空间但不在暂存区的文件撤销更改  
`git restore <file>`

把暂存区的文件从暂存区撤出，但不会更改文件  
`git restore --staged <file>`

拉去最新一次提交到暂存区
`git reset HEAD`

已经commit了从head回退1步
`git reset --hard HEAD^`  
`git reset --hard HEAD^1` 

## 4.重命名文件

`git mv <oldFile> <newFile>`

或者
```shell
mv <oldFile> <newFile>
git add <newFile>
git rm <oldFile>
```

## 5.提交文件
git commit 命令将暂存区内容添加到本地仓库中

提交修改并备注为 fix typo  
`git commmit -m 'fix typo'`

修改最近一次提交的提交信息
`git commit --amend -m 'amend commit'`


## 6. 切换分支

`git checkout <branch>`

`git switch <branch>`

`git switch` 在 git 2.23 上提供，会代替 `git checkout` 对分支的管理


# REF
[https://git-scm.com/docs/git-switch/2.23.0](https://git-scm.com/docs/git-switch/2.23.0)