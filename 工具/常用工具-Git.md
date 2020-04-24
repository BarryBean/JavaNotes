# 1. 版本控制VCS
答：版本控制是用来记录文件内容变化，便于将来查阅特定版本修订情况的系统。

## 1.1 必要性
答：在团队协作时，能将文件/项目回溯到某一版本，还能查看是谁修改了代码，从而找到锅在谁那。

## 1.2 分类
### 1.2.1 本地版本控制
- 复制整个项目目录，手动增加版本号；
- 本地数据库记录更新。

### 1.2.2 集中式版本控制
- 一个集中管理的服务器，所有成员通过访问此服务器获得最新文件或者提交更新。SVN。
- 单点宕机风险，必须联网。

### 1.2.3 分布式版本控制
- 分布式管理，每次clone都是完整的备份；
- 不需联网，每个Client都是一个版本库，最后需要推送修改；
- 分支管理。

# 2. Git
Git的图形化工具用smartGit，可以很方便解决冲突。

## 2.1 介绍
### 2.1.1 特点
答：Git对于数据是记录快照的方式。每次commit或stage，git会对全部文件制作一个快照并保存对快照的索引。文件没有修改，就只保存指向原来文件的索引。

### 2.1.2 状态
- 已提交commit：数据已经保存在本地数据库；
- 已修改modified：修改了文件，但还没提交入库；
- 已暂存staged：对已修改文件的当前版本做标记，能在之后提交时用。

### 2.1.3 工作区域
- 版本库：.git文件夹
- 工作目录：自己电脑上能看到的目录；
- 暂存区

### 2.1.4 工作流程
- 在工作目录修改文件；
- git add 把文件修改添加到暂存区；
- git commit把暂存区的所有内容提交到当前分支。


## 2.2 常用命令
### 2.2.1 初始化/拉取仓库
```java
//初始化仓库
git init
//拉取远程库
git clone git@github.com:BarryBean/Sword2Offer.git
```
### 2.2.2 更新提交
```java
//检查当前文件状态
git status
//所有文件加入暂存区
git add */.
//提交更新
git commit -m "日志提交信息"
//移除文件
git rm filename
```
### 2.2.3 推送远程仓库
```java
//本地库和远程库连接
git remote add origin git@github.com:BarryBean/Sword2Offer.git
//推送远程库
git push origin master/任意分支
```

### 2.2.4 撤销操作
```java
//撤销提交
git commit --amend
//取消暂存文件
git reset filename
//撤销文件修改
git checkout -- filename
```

### 2.2.5 分支
```java
//创建分支
git branch dev
//切换分支
git checkout feature-bys
//创建并切换
git checkout -b feature-bys
//合并分支，将分叉的分支合并一条
git rebase develop
//合并分支，手动解决冲突，--no-ff能保存本地分支的状态
git merge --no-ff test
```



