---
title: Git常用汇总
date: 2019-09-10 14:22:07
tags:
---
由于在开发过程中经常使用vscode自带的工具进行一些简单的sync、merge等操作，使得我很难熟记git的命令，故记录于此，免得以后到处google。

### 几个术语

* workingtree 工作区（当前本地文件夹）
* index / state 暂存区（add之后commit之前）
* HEAD 当前活跃分支的游标，也是workingtree、index的修改最终要提交到的位置
几个概念：
* git相对于其他VCS工具，主要的一个区别是，操作的是“修改”，而不是“文件”。这个很重要，对理解git的很多命令有帮助
* 修改的提交过程是：workingtree -> index -> HEAD （本地），提交到了index和HEAD的修改都可以通过命令回退，但workingtree的修改一旦撤回，则不可找回，将彻底回退到指定的状态
* 当前文件夹，特指当前执行命令所在的文件夹位置
* 版本库中的每一个提交节点，类似于一个列表，各个分支其实就是不同的指针，分别指向特定的节点，而HEAD就是指向当前分支的游标；切换分支实际上就是切换HEAD游标指向的分支指针

### 初始化repository

* `git init` # 用于将当前文件夹初始化为一个repo
* `git clone 「remote repo site」` # 克隆远程仓库于本地文件夹
本地的git单独分支操作
* `git add 「singlefile」` # 将workingtree的指定文件的修改提交至index
* `git add .` # 将workingtree的当前文件夹所有修改提交至index
* `git branch 「branch_name」` # 创建分支
* `git branch --set-upstream-to 「local_branch」 origin/「origin_branch」` # 关联本地分支与远程分支
* `git checkout 「branch_name」` # 切换到现有分支
* `git checkout -b 「branch_name」` # 切换到一个新的分支（创建并切换）
* `git checkout -- 「file_path」` # 撤销workingtree指定文件的修改（不可找回）
* `git config --list` # 查询当前repo的一些配置
* `git config --list --global` # 查询全局配置
* `git config user.name "yourname"` # 配置你的用户名
* `git config user.email "youremail"` # 配置你的邮箱
* `git diff` # 查看workingtree与index的区别
* `git diff --cached` # 查看index与HEAD的区别
* `git diff HEAD` # 查看workingtree与HEAD的区别
* `git diff HEAD -- 「file_path」` # 查看workingtree与HEAD指定文件的区别
* `git log` # 查看提交历史记录，每一条记录都对应一个版本号
* `git log --pretty=oneline` # 查看提交历史记录，显示于一行
* `git reset --mixed HEAD` # 回退index，使其对应HEAD最新版本，--mixed为默认值，可以省略为：git reset HEAD
* `git reset --mixed HEAD 「file_path」` # 回退index指定文件，使其对应HEAD最新版本
* `git reset --hard HEAD` # 回退workingtree，使其对应HEAD最新版本（慎用，工作区不可找回）
* `git reset --soft HEAD~` # 回退HEAD到前一个版本，workingtree和index都不会改变
* `git reset HEAD~「number」` # 回退到前第number个版本（--「mode」可用）
* `git reset 「version number」` # 回退到指定版本（--「mode」可用）
* `git reflog` # 查看当前用户的命令历史
* `git rm 「file_path」` # 从版本库中（index & HEAD）删除指定文件
* `git status` # 查看当前修改状态

### 远程仓库

* `git remote add origin 「remote git repository address」` # 将本地已有的仓库与指定远程仓库关联，yourname为你的远程仓库账户名，yourrepo为你的远程仓库名，origin为远程仓库命名，这是git默认，不用修改（一看就是远程仓库）
* `git push -u origin master` # 将本地库内容推送到远程库。第一次推送时，“-u”参数加上后，不但会推送内容，还会把本地的master分支与origin的master分支关联起来，之后就可以省略这个参数了
