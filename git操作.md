# git 操作 
## 用户信息
- 设置信息 


		// 设置全局信息
		$ git config --global user.name "Your Name"
		$ git config --global user.email "email@example.com"
			
		// 查看全局信息
		git config -l

- 又需要登录公司的账号，又想在电脑上使用自己的账号。

		解决思路，只针对某个项目配置自己的信息

		首先,取消全局配置
		git config --global --unset user.name  #取消全局设置
		git config --global --unset user.email #取消全局设置
		git config -l #查看当前目录的git config

		再分别去不同的项目目录中，设置这个目录中项目对应的账号
		git config user.name "newname"
		git config user.email "newemail"
		
		我们需要删除以前的默认名的密钥，生成新的密钥
		rm ~/.ssh/id_rsa.pub
		rm ~/.ssh/id_rsa
		ssh-keygen -t rsa -C "your-email-address" -f "rsa_name"

		设置 ssh config ，使ssh 知道什么域名由什么密钥去处理
		#Default Git
		Host defaultgit
		HostName IP Address #域名也可以
		User think
		IdentityFile ~/.ssh/rsa_name

		执行ssh-agent bash让ssh识别新的私钥。
		ssh-add ~/.ssh/rsa_name

## 创建版本库

	$ mkdir learngit
	$ cd learngit
	$ pwd
	# 初始化仓库
	git init
	#  查看隐藏目录
	ls -ah 

	# 添加
	git add readme.txt
	# 提交
	git commit -m 'write a readme file'
	# 查看状态
	git status
	# 查看做了哪些修改
	git diff
	
		# 查看工作区与暂存区的不同
		$ git diff readme.txt  
		diff --git a/readme.txt b/readme.txt
		index 46d49bf..9247db6 100644
		--- a/readme.txt
		+++ b/readme.txt
		@@ -1,2 +1,2 @@
		-Git is a version control system.
		+Git is a distributed version control system.
		 Git is free software.

	一些问题
	
	1. windows使用git时出现：warning: LF will be replaced by CRLF
		
	$ rm -rf .git  // 删除.git  
	$ git config --global core.autocrlf false  //禁用自动转换   


## 版本回退
- git log

		看提交日志
		如果输出太多可用 git log --pretty=oneline

		HEAD 表示当前版本  HEAD^ 上个版本 , HEAD^^ 上上个版本 , HEAD~100 前100个版本
		# 回退到上个版本
		git reset --hard HEAD^

		如果1，2，3版本，你从3回到2，再次 git log 是看不到3 的
		要再回到3版本
		git reset --hard version_code(3的版本号,没必要写全，写几个就行了)

		git reflog 可以看到所有的操作记录

		5f48472 (HEAD -> master) HEAD@{0}: reset: moving to HEAD^
		5ecaa6e HEAD@{1}: reset: moving to 5ecaa
		5f48472 (HEAD -> master) HEAD@{2}: reset: moving to HEAD^
		5ecaa6e HEAD@{3}: commit: modified a word
		5f48472 (HEAD -> master) HEAD@{4}: commit (initial): write a readme file
		
		前面是对应的版本号，这样你就能去到你想去的版本了


## 关于撤销修改
	
		聊撤销之前，先看git 的版本库
		git 分为工作区  暂存区 
		你对文件的任何操作都是在工作区的，使用 git add 将文件从工作区推到暂存区， git commit 将暂存区的文件提交出去

		撤销修改包括四种 在工作区修改了，想撤销；提交到了暂存区想撤销;已经commit了想撤销；已经push了想撤销
		1. 在工作区做了修改，想撤销这些修改.这里又分为两种情况，如果你的暂存区有东西，则变成你暂存区的样子；如果没有就变成你版本库里的样子。
			# 撤销某个文件
			 git checkout -- readme.txt

		2. 如果已经add 到了暂存区，但是还没有提交。下面的操作，能把暂存区的修改回退到工作区，就变成了1的情况，同时暂存区就变干净了

			git reset HEAD readme.txt

		3. 如果已经commit 了，那么只能使用版本回退到上个版本

			git reset --hard HEAD^
		
		4. 如果已经push 出去了，等死吧

## 关于删除文件

	在git 眼里一切操作都是修改，删除操作，也是修改。本地删除文件之后，要将修改 add,commit 这样版本库才知道你删除了东西.所以删除时要用 git rm ... 然后再提交,如果删错了。就看上面撤销的操作吧，一样的。
	

## 关于添加远程仓库

	三种情况，本地有项目要放到远程；远程有项目要拉到本地；两边都没有
	1. 本地有项目要放到远程，比如我本地有项目，想放到github 上

		# 先在远程创建一个仓库
		# 在本地文件夹下，关联到远程仓库
		git remote add origin https://github.com/guawazi/learngit.git
		# 将项目推到远程,第一次带 u 后面就可以不用带了
		git push -u origin master

	2. 远程有项目克隆到本地
		# 直接克隆下来就能用
		git clone git@github.com:michaelliao/gitskills.git

	3. 两边都没有，随便你咋搞

## 关于分支管理

	git 的分支管理是很轻便的，因为它只是将指针指向某个分支，每次移动指针。
	
	任务来了，新建一个分支在Dev，在这个分支上进行开发
	# 创建 dev 分支，并切换到dev
	git checkout -b dev
	# 相当于下面两句
	git branch dev
	git checkout dev
	# 查看所有分支
	git branch
	
	每天在这个分支上进行开发提交
	git commit....
	
	开发完了，将代码合并到master
	git checkout master
	git merge dev
	
	合并完成，删除dev 分支
	git branch -d dev
	
	下面命令可以看到合并的过程
 	git log --graph --pretty=oneline --abbrev-commit

	
	在使用 Fast forward 合并模式下，dev 做了修改，提交，master再merge 后，是直接将 master的指针指向 dev 所以速度很快。但是这样会丢掉分支信息，并且看不出来是合并操作，这时可以用下面的方法，强制禁用 fast forward

	git merge --no-ff -m "merge with no-ff" dev
	

## 关于 bug分支

	如果你在 work分支愉快的干活，突然，出了一个bug.需要你马上解决，但是你又不能 commit 代码，因为功能没完成。这时你需要
	#  保存现场
	git stash 
	然后 git status 你会发现你的工作区干净了，然后切到主分支，新建一个bug 分支来处理bug
	git checkout master 
	git checkout -b bug_1
	# 处理完bug add commit
	git add ..
	git commit -m 'xxx'
	# 切到主分支，合并
	git checkout master
	git merge -no-ff -m '合并bug分支' bug_1
	# 完事，又该回去work分支工作了
	git checkout work
	# 很干净
	git status
	# 隐藏列表
	git stash list
	# 恢复空间
	git stash apply stash@{0}
	#　删除以前的保存
	git stash drop
	#　以上两步可以一步
	git stash pop
	
	又可以愉快的工作了

## 关于Feature分支

	#又来新需求了，新建一个 new_feature分支
	git checkout -b new_feature
	# 然后各种开发，完成commit
	# 回到主分支，要合并了
	git checkout master
	git merge --no-ff -m '合并新功能' new_feature
	# 上面操作还没进行，突然不要这个功能了
	# 删除分支
	git branch -d new_feature
	# git 会阻止你，因为该分支还没有合并
	# 这时你需要强行删除
	git branch -D new_feature

## 多人协作
	# 查看远程库信息
	git remote 
	# 详细信息
	git remote -v

	# 推送分支
	git push origin master 将master 分支推送到远端
	git push origin dev 将dev 分支推送到远端
	
	# 从远程库克隆之后默认是 master 分支
	# 本地创建dev 与远端关联 ,如果远端没有，则会报错，先在远端建立该分支
	git checkout -b dev origin/dev
	# 提交完了推送到远端
	git push origin dev
	# 如果推送出错则有冲突，先 pull 解决冲突
	git pull
	# 如果pull 也有问题，说明你的dev 没有与远端建立关系,先建立关系，再pull
	git branch --set-upstream dev origin/dev

	如果你在本地的work分支上开发,远程的master 修改了，而你不得不更新，这时你切到master pull 之后，master 是最新代码，但是切到work分支，代码不是最新的，建议merge 主分支，拿到最新代码继续开发

## 忽略文件

[github上的忽略文件](https://github.com/github/gitignore)

	新建一个 .gitignore文件，例如Android上写上
		
	*.iml
	.gradle
	/local.properties
	/.idea/workspace.xml
	/.idea/libraries
	.DS_Store
	/build
	/captures
	.externalNativeBuild
	

	如果在操作时出现下面这种情况，则说明你要添加的文件被忽略了

	$ git add test.iml
	The following paths are ignored by one of your .gitignore files:
	test.iml
	Use -f if you really want to add them.

	你还可以通过以下命令来查看是哪条 .ignore 写错了
	
	$ git check-ignore -v test.iml
	.gitignore:1:*.iml      test.iml

	如果你确实想加入他，那么
	git add -f test.iml
	