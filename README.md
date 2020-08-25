# Git使用指南 #


## 基本概念理解 ##

*理解git的版本管理：使用git的每次提交，Git都会自动把它们串成一条时间线，这条时间线就是一个分支。如果没有新建分支，那么只有一条时间线，即只有一个分支，在Git里，这个分支叫主分支，即master分支。有一个HEAD指针指向当前分支（只有一个分支的情况下会指向master，而master是指向最新提交）。每个版本都会有自己的版本信息，如特有的版本号、版本名等。*

*分支：在同一时间不同版本的存储库。默认情况下，版本库中有一个名为master的分支，也是默认的正在工作的分支。*

*HEAD是什么：一个特别指针，在git中它是一个指向你正在工作的本地分支的指针（可以把HEAD当作当前分支的别名），有了它，git就知道当前是在哪个分支上工作。如下图，假设HEAD正指向分支A：*

![HEAD](https://s1.ax1x.com/2020/08/25/d6tetP.png)

*什么是commitID：一个SHA1 hash值，从暂存区提交(commit)代码到本地分支时会产生，相当于一次提交的标识，可用git log查看，使用时可用前7位代替完整的commitID。*

## git常规开发流程 ##
![gitWorkFlow](https://s1.ax1x.com/2020/08/25/d6tK1S.png)
*实际开发过程中，会基于master分支开很多的分支，乃至基于某分支新开分支；*

*有时push到远程仓库的代码有误或者其他原因需要进行回滚，具体操作后面会详述*

## 分支 ##
- 新建分支
	- git branch feature1	
- 切换分支
	- git checkout feature1	
- 新建并切换到新的分支
	- git checkout -b feature1(实则为前两条命令的合并)	
- 查看分支
	- git branch -a  （所有分支）
	- git branch    (本地分支)  
- 合并分支

	- git merge feature1	
- 删除分支
	- git branch -d feature1  


## 将远程主机的最新内容更新到本地 ##
![pullOrfetch](https://s1.ax1x.com/2020/08/25/d6tup8.jpg)
>可以简单的概括为：
**git fetch**是将远程主机的最新内容拉到本地，用户在检查了以后决定是否使用**git merge** 合并到本机分支中。
而**git pull** 则是将远程主机的最新内容拉下来后直接合并，即：**git pull = git fetch + git merge**，这样可能会产生冲突，需要手动解决。
下面我们来详细了解一下git fetch 和git pull 的用法：

- git fetch(推荐使用)
	- git fetch [远程主机名]	
		>将某个远程主机（例如origin）的更新全部拉取到本地
	- git fetch [远程主机名] [分支名]
		>拉取某分支的更新到本地
	
	示例：
	>
		git fetch origin master  //拉取origin主机的master分支的最新内容。此时会返回一个FETCH_HEAD,它指的是master分支在服务器上的最新状态
		git log -p FETCH_HEAD  //查看刚拉取的更新信息，通过信息来判断是否产生冲突
		git merge FETCH_HEAD  //如无冲突则执行此命令将拉取的内容合并到当前分支

- git pull
	- git pull [远程主机名] [远程分支名]:[本地分支名]
		>将远程主机的某个分支的更新取回，并与本地指定的分支合并(如果远程分支是与当前本地分支合并，可省略冒号及其后面部分)
	
	示例：
	>
		git pull origin master  //将远程主机的master分支的更新取回，并与本地master分支合并。

## 将本地分支推送到远程分支 ##

> git push的一般形式为git push <远程主机名> <本地分支名> <远程分支名>;
> 例如 git push origin master :refs/for/master ，即是将本地的master分支推送到远程主机origin上的对应master分支， origin 是远程主机名，第一个master是本地分支名，第二个master是远程分支名

- git push origin master 
	
	如果远程分支被省略，如上则表示将本地分支推送到与之存在追踪关系的远程分支（通常两者同名），如果该远程分支不存在，则会被新建
	
- git push origin :refs/for/master 

	如果省略本地分支名，则表示删除指定的远程分支，因为这等同于推送一个空的本地分支到远程分支，等同于 git push origin --delete master

- git push origin

	如果当前分支与远程分支存在追踪关系，则本地分支和远程分支都可以省略，将当前分支推送到origin主机的对应分支 

- git push

	如果当前分支只有一个远程分支，那么主机名都可以省略，形如 git push，可以使用git branch -r ，查看远程的分支名


*注意：refs/for 的意义在于我们提交代码到服务器之后是需要经过code review 之后才能进行merge的，而refs/heads则不需要*

## 打标签（Tag） ##
>通常在发布软件的时候打一个tag，tag会记录版本的commit号(如v1.0,v2.0等)，方便后期回溯

>过程：
>
	git add *
	git commit -m "v0.3"
	git tag v0.3
	git push
	git push origin v0.3

- 列出标签
	- git tag (列出已有标签)
	- git tag -l "v1.5.*" （列出符合通配符规则的标签）

- 创建标签
	>Git 支持两种标签：**轻量标签（lightweight）**与**附注标签（annotated）**。

	>轻量标签很像一个不会改变的分支——它只是某个特定提交的引用,本质上是将提交校验和存储到一个文件中——没有保存任何其他信息

	>而附注标签是存储在 Git 数据库中的一个完整对象， 它们是可以被校验的，其中包含打标签者的名字、电子邮件地址、日期时间， 此外还有一个标签信息，并且可以使用 GNU Privacy Guard （GPG）签名并验证。 通常会建议创建附注标签，这样你可以拥有以上所有信息。但是如果你只是想用一个临时的标签， 或者因为某些原因不想要保存这些信息，那么也可以用轻量标签。

	- git tag -a [tagName] -m "my version is tagName" (附注标签)
	- git tag [tagName] （轻量标签）
	- 追加标签
		>对过去的提交打标签，利用git log查看提交历史

		>
			$ git log --pretty=oneline
			0d52aaab4479697da7686c15f77a3d64d9165190 one more thing
			……
			9fceb02d0ae598e95dc970b74767f19372d61af8 updated rakefile
			……
			964f16d36dfccde844893cac5b347e7b3d44abbc commit the todo		

		>假设在 v1.2 时你忘记给项目打标签，也就是在 “updated rakefile” 提交。 你可以在之后补上标签。 要在那个提交上打标签，你需要在命令的末尾指定提交的校验和（或部分校验和）
		
		- git tag -a v1.2 9fceb02 或 git tag -a v1.2 9fceb02d0ae598e95dc970b74767f19372d61af8

- 查看tag详细信息
	>对于附注标签：显示打标签者的信息、打标签的日期时间、附注信息，然后显示具体的提交信息
	
	>对于轻量标签：只会显示出提交信息

	- git show [tagName]

- 将Tag同步到远程服务器
	>默认情况下，git push 命令并不会传送标签到远程仓库服务器上。 在创建完标签后你必须显式地推送标签到共享服务器上
	- git push origin [tagName] （推送单个分支tag）
	- git push origin --tags (推送本地所有tag)

- 切换到某个标签
	> 查看某个标签所指向的文件版本，可以直接切换到某个tag去
	
	>这个时候不位于任何分支，处于游离状态(detached HEAD),这个状态有些不好的副作用：
	>
		此状态下，如果你做了某些更改然后提交它们，标签不会发生变化， 但你的新提交将不属于任何分支，并且将无法访问，除非通过确切的commitID才能访问。 
		因此，如果你需要进行更改，比如你要修复旧版本中的错误，可以考虑基于这个tag创建一个分支
	- git checkout [tagName]
	- git checkout -b [newTagName] [tagName] （基于原Tag新建分支）

- 删除标签
	- git tag -d [tagName] （本地标签删除）
	- git push origin :refs/tags/[tagName] 或 git push origin --delete [tagName] (远端标签删除)


## 代码回滚 ##

1. 相关命令介绍
	
	1) git reset 将HEAD指针指到指定提交(commitID)，主要有以下三个操作选项：
	- git reset --soft [commitID]
		>将HEAD引用指向给定提交。索引（暂存区）和工作区的内容是不变的
	- git reset --mixed(默认) [commitID]
		>HEAD引用指向给定提交，并且索引（暂存区）内容也跟着改变，工作区内容不变。这个命令会将索引（暂存区）变成你刚刚暂存该提交全部变化时的状态
	- git reset --hard [commitID]
		>HEAD引用指向给定提交，索引（暂存区）内容和工作区内容都会变给定提交时的状态。也就是在给定提交后所修改的内容都会丢失(新文件会被删除，不在工作区中的文件恢复，未清除回收站的前提)
		
	*以表格列出各参数是否产生影响：*

	选项|HEAD|暂存区|工作区
	:--:|:--:|:--:
	--soft|是|否|否
	--mixed|是|是|否
	--hard|是|是|是
	2) git revert 放弃指定提交的修改，但是会生成一次新的提交，需要填写提交注释，以前的历史记录都在；

	*注意：reset与revert的区别*

	|reset|revert
	:--:|:--:|:--:
	作用|版本回退到某一版本/提交|撤销某一个版本/提交
	影响|对本地的版本产生影响，push后不对远程分支产生影响(如果回退的版本小于远程分支版本，通常会提示先将本地版本内容更新至与远程版本一致)|对本地的版本产生影响并新增一条记录，push后会对远程分支产生影响
	示例|1->2->3->4->5,“git reset [版本3的commitID]”，这条命令会把版本3的修改和记录都撤销掉，变为1->2->3|1->2->3->4->5,“git revert [版本3的commitID]”，这条命令会把版本3的修改撤销掉，但是记录不会删除，并且新增一条revert记录，变为1->2->4->5

	3) git status  查看文件、文件夹在工作区、暂存区的状态，有以下三种状态：

	> **Changes to be committed**:表示已经从工作区add到暂存区的file（文件或文件夹），可以通过 git reset filename命令将该file从暂存区移出

	>**Changes not staged for commit**:表示工作区，暂存区都存在的file（文件或文件夹）。

	>**Untracked files**:表示只在工作区有的file（文件或文件夹），也就是在暂时区没有该file
	
	4) git log  查看提交历史
	- git log
		>无参数，git log 会按提交时间列出所有的更新，最近的更新排在最上面
	- git log -p
		>代码审查-展开内容差异
		
		>例：展开显示每次提交的内容差异, 用 -2 则仅显示最近的两次更新
		>**git log -p -2**
	- git log --stat
		>仅显示简要的增改行数统计
	- git log --pretty
		>指定使用完全不同于默认格式的方式展示提交历史,--pretty=oneline/short/full/fuller/…… 此处四种简单模式

		>还有定制格式，此处不详述，可自行查看[https://www.git-scm.com/docs/git-log](https://www.git-scm.com/docs/git-log "git-log")


2. 使用场景
	- 场景一：同时对多个文件执行了git add操作，但本次只想提交其中一部分文件
	>
		git add .  //将工作区的所有文件添加到暂存区
		git status //查看文件状态
		git reset [fileName]  //将指定文件名的文件从暂存区撤出

	- 场景二：刚把不想要的代码，commit到本地仓库中了，但是还没有做push操作
	>
		git add ff.txt
		git commit -m "Add ff"
		//撤销上一次commit
		git reset --soft [HEAD^/上一次commitID] //ff.txt还保留在暂存区
		git reset [HEAD^/上一次commitID] //ff.txt不在暂存区，只有工作区还有
		git reset --hard [HEAD^/上一次commitID] //暂存区、工作区都没有保留ff.txt了

	- 场景三：刚push的代码有问题，需要进行这次提交的回滚 【*对远程仓库做回滚操作是有风险的，需提前做好备份和通知其他团队成员*】
	>
		(以master分支为例)
		方法一：使用revert
		git revert HEAD
		git push origin master
		方法二：使用reset
		git reset --hard HEAD^
		git push origin master -f	
