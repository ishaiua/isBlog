---
layout: post
title: Git 常用资源
category: 资源
tags: Git 
description: Git 常用资源
---

- Windows 系统

### 安装 Git

[下载地址](https://git-for-windows.github.io/)


### 配置 Git

```
$ git config --global user.name "Your Name"
$ git config --global user.email "email@example.com"
```

`--global` 参数，表示你这台机器上所有的 Git 仓库都会使用这个全局配置。

 `git config` 命令可以查看所有可设置参数。

### 创建（初始化）仓库

在已有项目或空目录下输入：`git init`  即可。

### 管理项目

1. `git add .` 把文件添加到仓库。  
2. `git commit -m "" ` 把文件提交到仓库。
3. `git push` 提交到远程仓库。
4. `git status` 查看状态。
5. `git diff` 比较、查看修改内容。
6. `git log` 提交日记。
7. `git reset --hard HEAD^` 回退到上一版本。
8. `git reset ID` 回退到提交的 ID 版本。
9. `git reflog` 查看命令历史。
10. `git checkout -- ` 撤销修改。

### 远程仓库

`git remote add origin git@github.com:ishaiua/isBlog.git` 添加远程仓库。

`git push -u origin master` 初次提交。

`git push origin master ` 提交都远程。

### 远程克隆

 `git clone `

### 分支管理

1. 查看分支：`git branch`

2. 创建分支：`git branch <name>`

3. 切换分支：`git checkout <name>`

4. 创建+切换分支：`git checkout -b <name>`

5. 合并某分支到当前分支：`git merge <name>`

6. 删除分支：`git branch -d <name>`
7. 分支合并图 ：`git log --graph`

