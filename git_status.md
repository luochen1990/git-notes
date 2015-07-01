Git仓库的状态
=============

初用git的时候，最容易出现的问题就是，弄不清自己的仓库现在处于一个什么样的状态。

- *为什么我代码提交不了？*
- *为什么代码pull不下来？*
- *我怎么不能checkout到别的分支？*

弄不清状态的我们就像瞎子，不知道前面发生了什么，不知道应该往哪里走。

Git仓库的状态，是所有Git命令的上下文。所以弄清楚你的仓库现在所处的状态，这一点很重要，它的重要性，就像你走路要看地板一样。

我有个好习惯：

1. 做任何操作之前，确认一遍自己仓库当前的状态。
2. 完成任何操作之后，确认一遍自己仓库当前的状态。

一个Git仓库有哪些状态？
----------------------

### 头指针（HEAD）

头指针是很多操作的上下文，如：

- `git push`, `git pull`, `git fetch`
- `git merge ...`, `git rebase ...`
- `git log`, `git commit`

在一些以提交的引用（commit ref）为参数的命令中，可以直接用`HEAD`来引用头指针所指向的提交，用`HEAD^`来引用头指针所指向的提交的父提交 等等。

头指针有一些特殊状态，如“分离的头指针”，这是指当前头指针并未指向任何分支，而是直接指向提交，这种情况下最好不要执行commit命令，因为你并没有一个分支来跟踪这次提交。

头指针还有一些临时状态，如“master|merging”或者“master|rebasing”，这些表明你目前正在执行一个合并或者衍合操作。在这种状态下，你不应该再做任何其它和这次合并无关的操作（如checkout），而是应该尽快处理完这次合并，然后执行下一步（git status通常会告诉你下一步操作是什么）操作，直到这次合并完成，头指针回到正常状态。

查看：`git status`
操作：`git checkout ...`

### 工作目录（Working Tree）、暂存区（Index）

工作目录通常就是指你存放文件的文件夹。
暂存区中的内容是你的下一次commit将会提交的内容。

通常你在操作头指针的时候，工作目录中的内容会和头指针所指向的新提交保持一致。如果在checkout到别的提交之前，你的工作目录或暂存区中有已经编辑但尚未提交的内容，那么你应该先安置好这些内容再执行checkout（否则git会拒绝checkout命令）。

查看：`git status`
操作：`git add ...`, `git reset ...`, `git stash ...`

### 当前分支的上游分支（upstream）

当前分支的上游分支是这些操作的上下文：

- `git push`, `git pull`, `git fetch`

查看：`git status`
操作：`git push -u ...`, `git branch --set-upstream br origin/br`

### 提交历史

提交历史是最重要的状态。我们的所有git命令最终都是为了操作提交历史。

查看: `git log`, `git log --stat`, `git log --graph`, `git show ...`
操作：`git commit`, `git commit --amend`, `git rebase -i`, `git reset ...`

