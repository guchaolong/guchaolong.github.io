---
layout:     post
title:   Git总结
subtitle:   
header-img: img/post-bg-re-vs-ng2.jpg
catalog: true
category: 工具
tags:
       - Git
---

>stay hungry. stay foolish 

# 前言
Linux大神Linus 1991年创建了Linux，2002年以前，由世界各地的志愿者提供code给Linus,由他手工方式合并,2002年，bitmover公司授权免费使用bitkeeper版本控制系统，不需要再手动。但是Linux社区人才辈出，有人试图破解bitkeeper被发现，警告多次没用，
不能再免费使用，于是Linus花了10天时间写了Git

# Git相关
![Image text](https://raw.githubusercontent.com/guchaolong/guchaolong.github.io/master/_posts_img/git.jpg)

  
<Strong>工作区</Strong>:整个文件目录为工作区  

<Strong>版本库</Strong>:隐藏的.git目录是版本库，Git的版本库中存了很多东西，其中最重要的就是称为stage（或者称为index）的暂存区，还有Git自动创建的master，以及指向master的指针HEAD
  
git add是把修改提到暂存区，commit是将修改提交到分支

### 安装、配置
     git config --global user.name "Your Name"
     git config --global user.email "email@example.com"            --global 表示应用到所有的仓库

### SSH配置
```
    #进入～/.ssh文件夹，如果没有，则mkdir创建文件夹
    cd ~/.ssh
    
    #创建密钥，然后问价下会生成两个东西
    ssh-keygen -t rsa -C guclno1@qq.com
    
    #打开id_rsa.pub文件，复制里面的内容，贴到github里   
```

### 初始化
     mkdir learngit                创建一个目录
     cd learngit                   
     pwd  （用于显示当前目录）
     git init                      把当前所在目录变成git管理的仓库


​    
### 提交    
     git add readme.txt            把readme.txt文件添加到仓库
     git add file1.txt file2.txt
     git commit -m"wrote a file"   把文件提交到仓库 -m后是提交说明

### 状态、差异  
     git status                    查看当前仓库的状态 是否有文件被修改过
     git diff readme.txt           查看具体修改了什么内容

### 日志、回退 
     git log                       查看提交记录,每次提交会有一个版本号 如1d830190b46dc928bef9e52bc9ac09c0cfc5a7e6
     git reflog                    显示所有的git命令记录 以及commit id
     
     git reset --hard HEAD^        版本回退  HEAD是当前版本 HEAD^是上一个版本，HEAD^^是上上个  HEAD~100是往上100个
     git reset --hard 1d830        把最新的1d830回退之后,重新回到1d830（版本号没必要写全，前几位就可以了，Git会自动去找）
     
     对于已经push到远程分支的版本，1 先回退本地版本 2 紧接着强制推送到远程分支
     git push -f origin master      强制推送到远程master分支

### 撤销修改（工作区）
     git checkout -- file1.txt     撤销本地add前所做的修改，add到暂存区的不会被撤回（撤回命令中 一定要有--，不然就变成切换分支命令了）

### 撤销修改（暂存区）
     git reset HEAD file1.txt      撤销已经add到暂存区的修改->放到工作区，然后可以用$ git checkout -- file1.txt撤回工作区的修改

### 删除文件
     rm file3.txt                  工作区删除file3,然后git status查看：工作区和版本库就不一致了，这时候有一下两个选择
     git rm file3.txt
     git commit -m"delete file3"   1.把版本库的file3也删除
     git checkout -- file3.txt     2.删错了，然后恢复

### 远程仓库
     git remote add origin https://github.com/guchaolong/learngit.git    本地仓库关联远程仓库
     git push -u origin master                第一次推送master分支的所有内容
     git push origin master                   推送最新修改

>首先在github上创建远程仓库，git支持两种协议
 https://github.com/guchaolong/gitskills.git     速度慢，
 git@github.com:guchaolong/gitskills.git         使用SSH

     git clone git@github.com:guchaolong/gitskills.git     克隆远程仓库到本地

 >分支：有一个功能，需要两周，第一周完成50%，如果这时候代码提上去，不完整代码导致别人无法工作，不提又影响每天的进度
 ，因此，可以创建属于自己的分支，别人看不到，在自己的分支上干活，做完之后在一次性和并到原来的分支

 >在版本库中，主分支是master ,指向最新的提交，而HEAD指向master

    git branch dev          新建分支dev
    git checkout dev        切换到dev分支
    git checkout -b dev2    新建并切换到dev2，-b参数
    git branch              查看当前分支
    
    git merge dev           合并指定分支（dev)到当前分支
    git branch -d dev2      删除dev2分支
    
    git branch dev         创建本地分支
    git push origin dev:dev发布dev分支 这样远程仓库也有一个dev分支了


​    
### 冲突
    合并出现冲突，手动解决冲突之后再提交
    git log --graph --pretty=oneline --abbrev-commit        查看分支合并图
    git merge --no-ff -m "merge with no-ff" dev             Fast forward模式下，删除分支后，会丢掉分支信息
                                                            --no-ff参数就可以用普通模式合并，合并后的历史有
        
                                                            分支，能看出来曾经做过合并，而fast forward合并就看不出来曾经做过合并
### bug分支
    git stash               当前工作现场“储藏”起来，等以后恢复现场后继续工作
    git stash list          查看之前的工作现场
    git stash pop           恢复的同时把stash内容也删了

### Feature分支
    git checkout -b  xxx    当前分支假设为dev,新建xxx分支，用于开发新功能
    git checkout dev
    git branch -d xxx       若还没合并到dev就删除xxx,会失败，若要强行删除 用-D
    git branch -D xxx       强行删除xxx

### 多人协作
    克隆远程仓库时，git自动把本地的master和远程的master分支对应起来，并且远程仓库的默认名称是origin
    git remote          查看远程库的信息
    git remote -v       更详细的信息
    
    git pull origin master  抓取远程的新提交
    git push origin master  推送到远程分支

### 使用github
    Fork别人的项目，如人气极高的bootstrap项目https://github.com/twbs/bootstrap
    Fork之后就会在自己的账号下克隆一个bootstrap项目，有读写权限
    如果要让别人接受自己的推送 就pull request

### 自定义Git配置
    git config --global color.ui true   Git显示颜色，会让命令输出看起来更醒目
    git config --global alias.st status 配置别名 status变为st


​    
>以下是别人整理的


# GitHub创建仓库提示代码

	echo "# 项目名" >> README.md
	git init
	git add README.md
	git commit -m "first commit"
	git remote add origin git@github.com:qiubaiying/项目名.git
	git push -u origin master

若仓库存在直接push

	git remote add origin git@github.com:qiubaiying/test.git
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


​	
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


​	
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

#### 其他

	# 生成一个可供发布的压缩包
	$ git archives
	
