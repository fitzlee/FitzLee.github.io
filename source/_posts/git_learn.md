
title: Git Learn
date: 2014-01-06 22:07:43
comments: true
categories: document
#tags: [git]
keywords: git
description: 学习使用git来提高工作效率
tags:
    - git
    - learn
    
---


# [Git在线文档](http://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000)
```
$ git config --global user.name "Your Name"
$ git config --global user.email "email@example.com"
```

## New Project:
> new project with eclipse/android studio/vs2012/codeblocks/makefile/sublimetext
```
$ git init
$ git add .
$ git commit -m "new project,first commit"
onsite github add your repository....  (may you should add your ssh key!)
reference the note:
	echo "# Improve" >> README.md
	git add README.md
	git commit -m "first commit"
	git remote add origin https://github.com/iofcas/Improve.git              (为版本库添加名为origin的远程版本库)
	git push -u origin master                                  (把本地库的内容推送到远程，用git push命令，实际上是把当前分支master推送到远程。)
```


## Distributed Development:
```
git clone https://github.com/iofcas/learngit1.git (git:/ssh:)
```

<!--more-->


## Daily Build:
```
$ mv rm vim modify....that's ok
$ git add -A
$ git status   (git diff) 
$ git commit -m "your build" 
$ git push -u origin master
```


## Branch New Function:
```
git branch dev
git checkout dev      1,2<=>git checkout -b dev
mv rm vim modify....that's ok
git commit -m "branch new function ok"
git checkout master
```

### No conflict:
```
git merge dev  (Fast-forward)
git branch -d dev
```

### if conflict:(when master is ahead of dev_newfunction_master,no Fast-forward)
```
$ vim & modify the conlicted file
$ git add .
$ git commit -m "conflict ok"
$ git log --graph --pretty=oneline --abbrev-commit
$ git log --graph --pretty=oneline
```

```
	*   e2b770b473f4b01c95051f67df183b716c588554 process conflict
	|\
	| * 4cfb80d84f31611f53e643a408c62e99e25a1cbc branch add a file
	* | d954435c6637927da5bdf440ed80c41e3fa346d2 add aaa
	|/
	* 2f435ae3a03e6b5225e91fb4d704c1e2eb85d562 modify git learn
	* b60880ac8fcecf1a20a332f12222d5e6f316aef4 modify file.cpp added->add
	* 1ac939b4d2b67777d40ae5a5d3c8cecec51049d0 add branch hello function
	* 07e8db4d764b1440bf4fa3c275e1ec339f24f97b modify learngit & daily build model
	* b6408d0509fef560205a0a64fec27afc61285cdf del a file & new fileB
	* 1c408eafa0db0306a63fe9a44f8edced2e72b898 modify file->file.cpp;new file fileA
	* 6db1625bd6355cb7e53e8035dad50ff2ace6a193 file.c->file and so on
	* c7e0be6095dd168603a8661e308d9db1de660955 add a new file
	* b6ee53e718df3f135c16a94890b305204f7a6ad5 add readme
	* a61163eeef7dcac1a592252cf30fd1451e08fd04 leartgit1 first commit 
```

## --no-ff:
```
git merge --no-ff -m "merge with no-ff" dev
(合并分支时，加上--no-ff参数就可以用普通模式合并，合并后的历史有分支，能看出来曾经做过合并，而fast forward合并就看不出来曾经做过合并。)
```

## BUG:
(当你接到一个修复一个代号101的bug的任务时，很自然地，你想创建一个分支issue-101来修复它，但是，等等，当前正在dev上进行的工作还没有提交)
```
$ git stash
Process BUG Branch
$ git stash list
$ git stash pop (git stash apply + git stash drop)
```

## Del New function with no merge:
(接到上级命令，因经费不足，新功能必须取消)
```
$ git branch -D dev
```

## Dev Branch:
(创建远程new分支并推送本地dev分支到远程)
1 git push origin dev:new


## Version Tag:
(Git的标签虽然是版本库的快照，但其实它就是指向某个commit的指针)
```
$ git tag v1.0 (HEAD)
$ git tag v0.9 6224937 (git log --pretty=oneline --abbrev-commit)
$ git tag
$ git show v0.9
$ git tag -a v0.1 -m "version 0.1 released" 3628164
$ git tag -d v0.1
$ git push origin v1.0
$ git push origin --tags (push所有分支)
$ git push origin :refs/tags/v0.9 (删除远程分支)
```

## Open Source Join:
(一定要从自己的账号下clone仓库，这样你才能推送修改。如果从bootstrap的作者的仓库地址git@github.com:twbs/bootstrap.git克隆，因为没有权限，你将不能推送修改。)
```
$ fork  
$ git clone https://...my.git
$ vim...
$ pull request
```


## .gitignore:
(https://github.com/github/gitignore)
> 1 另存为.gitignore即可


## alias:
```
$ git config --global alias.st status   => git st
$ git config --global alias.unstage 'reset HEAD' => git unstage test.py  <=> git reset $ HEAD test.py
$ git config --global alias.last 'log -1'  => git last
$ git config --global alias.lg "log --color --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit"   => $ git lg
```

## config:
```
$ cat .gitconfig
[alias]
    co = checkout
    ci = commit
    br = branch
    st = status
[user]
    name = Your Name
    email = your@email.com
```

## Reset File:
```
Not staged for commit (not add .):
(git checkout -- 丢弃工作区的修改)
(git checkout其实是用版本库里的版本替换工作区的版本，无论工作区是修改还是删除，都可以“一键还原”)
$ git checkout -- readme.txt
```

## Changes to be committed(has added .):
(git reset既可以回退版本，也可以把暂存区回退到工作区的修改)
```
$ git reset HEAD readme.txt
```

## Reset:
```
$ git reset --hard HEAD^(HEAD当前版本，HEAD^^前两版本，HEAD~100前100版本)
$ git log --pretty=oneline
$ git reset --hard 3628164 
$ git reflog  (查看命令历史，以便确定要回到未来的哪个版本.)
```

## Fetch&Pull:
( ! [rejected]        master -> master (fetch first))
```
1 git fetch origin master  (master:master 中第一个master是origin/master)
git fetch origin master
git log -p master..origin/master
git merge origin/master

2 git fetch origin master:tmp
	git fetch origin master:tmp
	git diff tmp 
	git merge tmp

3 git pull:
	(相当于git fetch 和 git merge)
	git pull origin master

4 决策：
git fetch更安全一些
因为在merge前，我们可以查看更新情况，然后再决定是否合并

5 git fetch origin branch1:branch2
  (使用远程branch1分支在本地创建branch2,但不会切换到该分支)
	如果本地不存在branch2分支, 则会自动创建一个新的branch2分支, 
	如果本地存在branch2分支, 并且是`fast forward', 则自动合并两个分支, 否则, 会阻止以上操作.

6 git fetch origin master:tmp , git push origin master:dev 

	git fetch origin server:local  从server fetch 到local
	git push origin local:server   从local  push 到server
```

## Diff:
```
git diff origin/master (当前版本跟origin/master比较)
git diff dev (当前版本跟dev版本比较)
```


## Git commit:
一次良好的提交信息应该包含一个信息头，尽量用一行文字总结那些必要信息
然后空下一行，
接着写下关于这次提交的具体内容，有关修复bug的背景等等，尽量表达你为什么
要做这次提交而不是你正在作什么提交。这样被人才能很好的理解而不会弄不清楚
为什么需要这个东西。所有的提交内容保持在74个字符之内，这样才能被 git log
正常的显示，而不会超过屏幕。
下面是一些： 提交者信息，或者其他信息，一般都是 git 里面需要的代码编写者或者
测试者名字和邮箱，具体参考 git 。
(http://blog.csdn.net/jingxia2008/article/details/41290837  http://blog.jobbole.com/92713/)
```
	Summarize changes in around 50 characters or less
	 
	More detailed explanatory text, if necessary. Wrap it to about 72
	characters or so. In some contexts, the first line is treated as the
	subject of the commit and the rest of the text as the body. The
	blank line separating the summary from the body is critical (unless
	you omit the body entirely); various tools like `log`, `shortlog`
	and `rebase` can get confused if you run the two together.
	 
	Explain the problem that this commit is solving. Focus on why you
	are making this change as opposed to how (the code explains that).
	Are there side effects or other unintuitive consequenses of this
	change? Here's the place to explain them.
	 
	Further paragraphs come after blank lines.
	 
	 - Bullet points are okay, too
	 
	 - Typically a hyphen or asterisk is used for the bullet, preceded
	   by a single space, with blank lines in between, but conventions
	   vary here
	 
	If you use an issue tracker, put references to them at the bottom,
	like this:
	 
	Resolves: #123
	See also: #456, #789
	Reported-by: whoever-reported-it
	Signed-off-by: Your Name <youremail@yourhost.com>
```


## modified: xxx(modified content, untracked content)
大概意思是xxx目录没有被跟踪。那自然push上去的时候是空的了
解决办法：后来发现这主要是xxx目录下有一个.git 目录，可能是被人给你这个目录的时候里面有了.git目录。删除.git目录。重新git add .就可
```
$ git add . 前最好删除所有字母下的.git 一个工程最好只有一个.git
$ find . -name .git | xargs rm -fr
$ git add .
```

* 如果在master做了修改，没有commit，就新建分支，分支commit，将导致master恢复版本，之前的修改自动放到branch新分支里
