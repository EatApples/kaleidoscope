### Git Pull Failed	 
在拉取远程代码时，出现这样的情况，Git Pull Failed:You have not concluded your merge.Exiting because of unfinished merge。出现这种情况的原因如系统提示，可能在pull代码之前merge合并失败。 
在解决这个问题之前，先看看需要了解的知识。

git fetch命令

	用于从另一个存储库下载对象和引用。远程跟踪已更新分支（git术语叫commit），需要将这些更新取回本地，这时就要用到git fetch命令。 
	语法：git fetch <远程主机名>。例如：git fetch orgin master，表示取回origin主机的master分支。更新所有分支，命令可以简写为git fetch。

git pull命令

	用于取回远程主机某个分支的更新，再与本地的指定分支合并。这时你可能已经真正明白为什么会出现拉取失败的原因了，原因就在于拉取之后的代码合并失败造成的。 
	语法：git pull <远程主机名><远程分支名>:<本地分支名>。例如：git pull origin next:master，表示取回origin主机的next分支，与本地的master分支合并。如果远程分支（next）要与当前分支合并，则冒号后面的部分可以省略。

git reset命令

	语法：git reset [- -hard|soft|mixed|merge|keep][<commit id>或HEAD]，将当前的分支重新设置到指定的commit id或者HEAD，其中HEAD是默认路径。其中hard、soft、mixed、merge、keep是设置的模式。通常使用- -hard，表示自commit id以来，工作目录中的任何改变都被丢弃，并把HEAD指向commit id。

	解决方法

	方法一：舍弃本地代码，远程版本覆盖本地版本

	使用这种方法之前，可以先将本地修改的代码备份一下，避免重敲代码。具体命令如下：

	$:git fetch --all
	$:git reset --hard origin/master
	$:git pull
	1
	2
	3
	方法二：保留本地代码，中止合并–>重新合并–>重新拉取

	$:git merge --abort
	$:git reset --merge
	$:git pull
	1
	2
	3
	这种做法需要处理代码冲突，因此以上两种做法，根据你的需要，选择合适的解决办法。
