### git clone　　
cd 文件目录

git clone 已有仓库

### git status
查看当前本地分支的状态

### git add
git add 添加需上传的文件

### git commit
git commit -m "上传文件描述"

git commit --amend //修改提交信息

### git log
git log        //查看提交日志

git log 文件    //用于查看指定的目录、文件的日志

git log -p     //查看提交所带来的改动

git log -p 文件 //查看指定文件的改动

git log --graph //以图表形式查看分支

### git diff
git diff  //可以查看工作树，暂存区，最新提交之间的差别

git diff HEAD //查看工作树与最新提交的差别

### git branch
显示分支一览表，同时确认当前所在的分支

git branch 新建分支名

git branch -b 分支名 //创建分支，并且切换到该分支

### git checkout
git checkout 切换的分支名

git checkout -b 分支名 //创建分支，并且切换到该分支

### git push
本地分支提交远端

### git merge
git merge --no--ff 分支 // 加--no--ff 参数可以在历史记录中明确地记录本次分支的合并

### git reset
git reset //回溯历史版本

git reset --hard 推进历史的哈希值 //回溯到指定状态，只要提供目标时间点的哈希值

### git reflog
查看仓库的操作日志，找到要推历史的哈希值

### Git Pull Failed	 
在拉取远程代码时，出现这样的情况，Git Pull Failed:You have not concluded your merge.Exiting because of unfinished merge。出现这种情况的原因如系统提示，可能在pull代码之前merge合并失败。 
在解决这个问题之前，先看看需要了解的知识。

### git fetch 命令

用于从另一个存储库下载对象和引用。远程跟踪已更新分支（git术语叫commit），需要将这些更新取回本地，这时就要用到git fetch命令。 

语法：git fetch <远程主机名>。例如：git fetch orgin master，表示取回origin主机的master分支。更新所有分支，命令可以简写为git fetch。

### git pull 命令

用于取回远程主机某个分支的更新，再与本地的指定分支合并。这时你可能已经真正明白为什么会出现拉取失败的原因了，原因就在于拉取之后的代码合并失败造成的。 

语法：git pull <远程主机名><远程分支名>:<本地分支名>。例如：git pull origin next:master，表示取回origin主机的next分支，与本地的master分支合并。如果远程分支（next）要与当前分支合并，则冒号后面的部分可以省略。

### git reset命令

语法：git reset [- -hard|soft|mixed|merge|keep][<commit id>或HEAD]，将当前的分支重新设置到指定的commit id或者HEAD，其中HEAD是默认路径。其中hard、soft、mixed、merge、keep是设置的模式。通常使用 --hard，表示自commit id以来，工作目录中的任何改变都被丢弃，并把HEAD指向commit id。

### 解决方法
方法一：舍弃本地代码，远程版本覆盖本地版本。使用这种方法之前，可以先将本地修改的代码备份一下，避免重敲代码。具体命令如下：
```
	$:git fetch --all
	$:git reset --hard origin/master
	$:git pull
```

方法二：保留本地代码，中止合并–>重新合并–>重新拉取
```
	$:git merge --abort
	$:git reset --merge
	$:git pull
```
这种做法需要处理代码冲突，因此以上两种做法，根据你的需要，选择合适的解决办法。
