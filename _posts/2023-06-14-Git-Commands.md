---
layout:     post
title:      Git指令整理
subtitle:   常用的 Git 指令
date:       2023-06-14
author:     BuBu
header-img: img/purple_background.jpg
catalog: true
tags:
    - Git
---

>随便整理的一些自用的Git指令


# GitHub创建仓库提示代码

	echo "# 项目名" >> README.md
	git init
	git add README.md
	git commit -m "first commit"
	git remote add origin git@github.com:long/项目名.git
	git push -u origin master

若仓库存在直接push

	git remote add origin git@github.com:long/test.git
	git push -u origin master


# 常用操作

#### 创建仓库（初始化）
	在当前指定目录下创建
	git init
	
	新建一个仓库目录
	git init [project-name]
	
	克隆一个远程项目
	git clone [url]
	
#### 添加文件到缓存区

	添加所有变化的文件
 	git add .

	添加名称指定文件
	git add text.txt

#### 配置

	设置提交代码时的用户信息
	git config [--global] user.name "[name]"
	git config [--global] user.email "[email address]"
	
	
#### 提交
	提交暂存区到仓库区
	git commit -m "msg"
	
	# 提交暂存区的指定文件到仓库区
	$ git commit [file1] [file2] ... -m [message]
	
	# 提交工作区自上次commit之后的变化，直接到仓库区
	$ git commit -a
	
	# 提交时显示所有diff信息
	$ git commit -v
	
	# 使用一次新的commit，替代上一次提交
	# 如果代码没有任何新变化，则用来改写上一次commit的提交信息
	$ git commit --amend -m [message]
	
	# 重做上一次commit，并包括指定文件的新变化
	$ git commit --amend [file1] [file2] ...
	
#### 远程同步

	# 下载远程仓库的所有变动
	$ git fetch [remote]
	
	# 显示所有远程仓库
	$ git remote -v
	
	# 显示某个远程仓库的信息
	$ git remote show [remote]
	
	# 增加一个新的远程仓库，并命名
	$ git remote add [shortname] [url]
	
	# 取回远程仓库的变化，并与本地分支合并
	$ git pull [remote] [branch]
	
	# 上传本地指定分支到远程仓库
	$ git push [remote] [branch]
	
	# 强行推送当前分支到远程仓库，即使有冲突
	$ git push [remote] --force
	
	# 推送所有分支到远程仓库
	$ git push [remote] --all


	
#### 分支

	# 列出所有本地分支
	$ git branch
	
	# 列出所有远程分支
	$ git branch -r
	
	# 列出所有本地分支和远程分支
	$ git branch -a
	
	# 新建一个分支，但依然停留在当前分支
	$ git branch [branch-name]
	
	# 新建一个分支，并切换到该分支
	$ git checkout -b [branch]
	
	# 新建一个分支，指向指定commit
	$ git branch [branch] [commit]
	
	# 新建一个分支，与指定的远程分支建立追踪关系
	$ git branch --track [branch] [remote-branch]
	
	# 切换到指定分支，并更新工作区
	$ git checkout [branch-name]
	
	# 切换到上一个分支
	$ git checkout -
	
	# 建立追踪关系，在现有分支与指定的远程分支之间
	$ git branch --set-upstream [branch] [remote-branch]
	
	# 合并指定分支到当前分支
	$ git merge [branch]
	
	# 选择一个commit，合并进当前分支
	$ git cherry-pick [commit]
	
	# 删除分支
	$ git branch -d [branch-name]
	
	# 删除远程分支
	$ git push origin --delete [branch-name]
	$ git branch -dr [remote/branch]
	
#### 标签Tags

	添加标签 在当前commit
	git tag -a v1.0 -m 'xxx' 
	
	添加标签 在指定commit
	git tag v1.0 [commit]
	
	查看
	git tag
	
	删除
	git tag -d V1.0
	
	删除远程tag
	git push origin :refs/tags/[tagName]
	
	推送
	git push origin --tags
	
	拉取
	git fetch origin tag V1.0

	新建一个分支，指向某个tag
	git checkout -b [branch] [tag]

#### 查看信息

	# 显示有变更的文件
	$ git status
	
	# 显示当前分支的版本历史
	$ git log
	
	# 显示commit历史，以及每次commit发生变更的文件
	$ git log --stat
	
	# 搜索提交历史，根据关键词
	$ git log -S [keyword]
	
	# 显示某个commit之后的所有变动，每个commit占据一行
	$ git log [tag] HEAD --pretty=format:%s
	
	# 显示某个commit之后的所有变动，其"提交说明"必须符合搜索条件
	$ git log [tag] HEAD --grep feature
	
	# 显示某个文件的版本历史，包括文件改名
	$ git log --follow [file]
	$ git whatchanged [file]
	
	# 显示指定文件相关的每一次diff
	$ git log -p [file]
	
	# 显示过去5次提交
	$ git log -5 --pretty --oneline
	
	# 显示所有提交过的用户，按提交次数排序
	$ git shortlog -sn
	
	# 显示指定文件是什么人在什么时间修改过
	$ git blame [file]
	
	# 显示暂存区和工作区的差异
	$ git diff
	
	# 显示暂存区和上一个commit的差异
	$ git diff --cached [file]
	
	# 显示工作区与当前分支最新commit之间的差异
	$ git diff HEAD
	
	# 显示两次提交之间的差异
	$ git diff [first-branch]...[second-branch]
	
	# 显示今天你写了多少行代码
	$ git diff --shortstat "@{0 day ago}"
	
	# 显示某次提交的元数据和内容变化
	$ git show [commit]
	
	# 显示某次提交发生变化的文件
	$ git show --name-only [commit]
	
	# 显示某次提交时，某个文件的内容
	$ git show [commit]:[filename]
	
	# 显示当前分支的最近几次提交
	$ git reflog
	
#### 撤销
	
	# 恢复暂存区的指定文件到工作区
	$ git checkout [file]
	
	# 恢复某个commit的指定文件到暂存区和工作区
	$ git checkout [commit] [file]
	
	# 恢复暂存区的所有文件到工作区
	$ git checkout .
	
	# 重置暂存区的指定文件，与上一次commit保持一致，但工作区不变
	$ git reset [file]
	
	# 重置暂存区与工作区，与上一次commit保持一致
	$ git reset --hard
	
	# 重置当前分支的指针为指定commit，同时重置暂存区，但工作区不变
	$ git reset [commit]
	
	# 重置当前分支的HEAD为指定commit，同时重置暂存区和工作区，与指定commit一致
	$ git reset --hard [commit]
	
	# 重置当前HEAD为指定commit，但保持暂存区和工作区不变
	$ git reset --keep [commit]
	
	# 新建一个commit，用来撤销指定commit
	# 后者的所有变化都将被前者抵消，并且应用到当前分支
	$ git revert [commit]
	
	# 暂时将未提交的变化移除，稍后再移入
	$ git stash
	$ git stash pop
	
#### 代码回退 

默认参数 `-soft`,所有commit的修改都会退回到git缓冲区
参数`--hard`，所有commit的修改直接丢弃

	$ git reset --hard HEAD^ 		回退到上个版本
	$ git reset --hard commit_id	退到/进到 指定commit_id
推送到远程	

	$ git push origin HEAD --force

如果遇到报错，使用参数 `-f`,强行推送到远程	

	$ git push -f  


`git push -f` 会直接覆盖掉回退的代码，更建议使用`git revert`	

	$ git revert e7c8599d29b61579ef31789309b4e691d6d3a83f
	$ git push origin main
	

#### 可以吃的后悔药->版本穿梭

当你回滚之后，又后悔了，想恢复到新的版本怎么办？

用`git reflog`打印你记录你的每一次操作记录

	$ git reflog
	
	输出：
	c7edbfe HEAD@{0}: reset: moving to c7edbfefab1bdbef6cb60d2a7bb97aa80f022687
	470e9c2 HEAD@{1}: reset: moving to 470e9c2
	b45959e HEAD@{2}: revert: Revert "add img"
	470e9c2 HEAD@{3}: reset: moving to 470e9c2
	2c26183 HEAD@{4}: reset: moving to 2c26183
	0f67bb7 HEAD@{5}: revert: Revert "add img"
	
找到你操作的id如：`b45959e`，就可以回退到这个版本
	
	$ git reset --hard b45959e
	
#### 其他

	# 生成一个可供发布的压缩包
	$ git archives