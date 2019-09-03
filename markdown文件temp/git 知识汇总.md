### `git 知识汇总`

1. git 切换分支时会把未add或未commit的内容带过去， 这一点值得注意。

  为什么呢？

   因为未add的内容不属于任何一个分支， 未commit的内容也不属于任何一个分支。 也就是说，**对于所有分支而言， 工作区和暂存区是公共的**。

   要想在分支间切换， 又不想又上述影响， 怎么办呢？ git stash搞起。要注意，在当前分支git stash的内容， 在其他分支也可以git stash pop出来，为什么？ 因为：工作区和暂存区是公共的。

2. [IntelliJ IDEA 提交代码到 GitHub](https://blog.csdn.net/jeffleo/article/details/56017644)

