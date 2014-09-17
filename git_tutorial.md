Git入门指导
===========

版本管理工具应有的功能
----------------------

### 基本功能

- 查看或编辑项目（单个文件或整个项目）的某个指定历史版本
- 查看项目（单个文件或整个项目）某两个历史版本的差异
- 查看项目（单个文件或整个项目）的变更记录
- 合并项目（单个文件或整个项目）的两个或多个分支版本

### 高级功能

- 权限管理（通常Git以项目为单位，控制每个开发者对每个项目的读、写、设置的权限）
- 代码审核（通过pull request）
- 依赖管理（git submodule, git subtree, repo）

Git中的一些基本概念
-------------------

### Git目录（Git Dir），工作目录（Working Tree），暂存区（Index）

Git目录是储存Git赖以工作的数据的目录，在一个普通Git项目目录（例如`demo_repo/`）中Git目录是`demo_repo/.git/`，而在一个裸Git项目（通过`git init --bare`得到）目录（通常命名为`demo_repo.git/`）中则是项目目录`demo_repo.git/`本身。通常情况下我们不需要进入该目录去查看或编辑其中的文件，而是使用`git`命令来完成大部分事情。

工作目录，准确地讲应该叫工作区，它基本上跟你项目原始的目录内容是一样的，只是底下多了一个`demo_repo/.git/`目录，这里的内容是供你查看和编辑的，它是项目的某个版本的内容的副本。你编辑工作目录下的文件并不会推进项目的演进历史，只有提交（commit）它们才会。通常你不用太过担心丢失或者弄坏了工作目录中的文件，因为你至多只会丢失尚未提交的内容，通过重置工作区命令（`git reset --hard`）你就能轻松将工作目录重置到你最后一次提交的版本。

暂存区其实简单来说，就是用来暂存（命令是`git add {filename1} ...`）你即将要提交的文件的。为什么需要这么个东西呢，因为一方面，一个提交是不一定要提交你当前修改过的所有文件的，比如你修改了a,b,c三个文件，但是a,b是修复bug1的，而c是修复bug2的，那么逻辑上，你应该创建两个提交，每个提交都修复了一个bug，但是工作的时候我们很可能是改完三个文件之后再决定提交的事情，这种情况下，暂存区就能使工作过程更加自由。

正确的姿势是这样的（在修改完a,b,c三个文件后）：

```shell
git add a b
git commit -m 'bug1 fixed.'
git add c
git commit -m 'bug2 fixed.'
```

### 块（blob），树（tree），提交（commit）

块，树和提交是Git中的三种基本对象，他们都根据其内容的哈希值（sha1）命名。其中块是跟文件系统中的文件对应的，树是跟文件系统中的目录对应的，而提交是在用户执行Git的提交命令（`git commit`）时根据提交的父提交（们）的sha1值和提交的元数据（包括提交时间，作者信息，提交注释等）创建的。

### 分支（Branch），标签（Tag），头指针（HEAD）

分支用来跟踪用户的开发过程，标签用来标记某个特定的版本，而头指针是用来记录**用户当前所在分支**的，当你执行`git checkout some_branch`命令的时候，其实就是在改变头指针所指向的分支（当然还有工作区的内容）。头指针是Git里一个非常重要的概念，当你的头指针指向某个分支的时候，一方面意味着你的所有操作都是针对这个分支的，它将成为多数常用Git命令的隐含参数，另一方面意味着你对当前工作区的文件所作的修改都是基于该分支的，当你创建提交的时候，该分支也会随之指向新的提交。

分支，标签和头指针都是引用。其中分支和标签是对提交（commit对象）的引用，而头指针通常情况下是对分支的引用（是commit对象的引用的引用），某些时候也可以直接指向commit对象（指`git checkout some_commit_id_or_some_tag`的情况），这时的头指针叫做“分离的头指针”。

分支和标签的区别在于，当你在某个分支下创建提交后，分支会自动更新指向新的提交，而标签是稳定的，不会改变它所指向的提交。由于标签不会改变它的指向，所以标签更像是某个commit id的别名。

每个引用都会对应Git目录下的一个文件，头指针对应的文件是`.git/HEAD`，而其它引用对应的文件都在`.git/refs/`目录下。另外由于标签除了引用信息外，还会附加注释等其它信息，所以标签还会对应一个`tag`类型的Git对象（也就是说，Git中有四种基本对象）。

### 分支的合并（merge）及衍合（rebase）

#### 分支的合并

要理解什么叫合并分支，首先要了解为什么版本历史会产生分支，这个过程其实很容易理解，假设开发者A和开发者B在从中心库R检出了相同的版本v0后，各自分别修复了一个bug并分别作了一次提交，记作v1、v2，那么从版本演化过程的逻辑来看，v1和v2是没有先后顺序的，这个时候如果没有开发者的参与，版本管理系统是无法决定v1和v2合并之后应该是什么样子的，这就是为什么需要开发者参与到分支的合并过程中来。

虽然分支合并的过程逻辑上是需要开发者的参与和确认的，但是Git会试图通过一些假设和猜测尽量地帮助你完成这个过程，那么具体是怎样的逻辑呢？

我们假设现在是从A分支合并B分支：

1. 如果A分支所指向的提交是B分支所指向的提交的祖先提交（指父提交 或 父提交的父提交 或。。。） ——还记得么，提交对象中会记录其父提交的commid id(s) ——通俗地讲就是“B比A快”，那么可以说明，B分支上的工作是建立在A分支的基础之上的，这层关系至少是经过某个开发者确认（commit）过的，因此无需你的确认（commit）Git就能认为将A分支演进至B分支处是安全的。这种情况称为快进（fast-forward）。当然如果A比B快则什么也不会发生。于是这条规则就可以简单地描述为“哪个分支快用哪个”。
2. 当两个分支分不出先后的时候，。。。。。。你可以把规则2当作是对每个文件（而不是整个提交）分别应用规则1，也就是“对于每个文件f，f在哪个分支中快，就用那个分支中的f”。这种情况会创建一个新的提交对象，并且新提交的父提交是被合并的两个分支（对，没人说过父提交只能有一个是吧）。
3. 当存在某些文件也分不出先后的时候，解决冲突的工作就不得不交给开发者了，这时Git会提示“合并发生冲突”，然后你需要编辑冲突的文件以解决冲突，再重新提交。

注：Git中的`git merge {branch}`命令，是指将branch分支的内容合并到HEAD所指的分支中，也就是说只有你所在的分支（也就是HEAD所指的分支）的版本历史会得到演进，而被合并的branch分支是不会被修改的。这就是说，merge操作对于被merge的两个分支来说，并不是对称的，所以在这篇文章中，你需要注意我的措辞：对于`git checkout A && git merge B`的情况，我会说“从A分支合并B分支”或“将B分支合并进A分支”，而不会说“合并A分支和B分支”，后者的说法让人不明所以，我曾在很多关于Git的文章中都看到过，这困扰了我很久。

#### 分支的衍合（rebase，关于中文“衍合”的译法我不知道是否恰当，只是由于没有想到合适的词，所以沿用了前人的译法）

关于衍合，有个说法是：衍合跟合并得到的结果（考虑`git merge B`和`git rebase B`）是一样的，只是产生的提交序列不一样。这个说法是正确的，但是它不能告诉我们衍合的意义是什么，什么时候应该用它，以及用它有什么好处，有什么坏处。

我希望关于衍合，大家可以从“rebase”这个词顾名思义，“base”是“基础，根据”的意思，“re-”表示“重新”，所以rebase就是重新设置基础（重设父提交）的意思。当从A衍合B的时候，我们希望做的其实是，以B为基础，把A中的工作往B上添加，怎么加呢，其实还是“谁快用谁”无法比较就交给开发者的策略，这就是为什么会有“衍合跟合并得到的结果一样，只是产生的提交序列不一样”这个说法了。

所以rebase主要有两个使用场景：
- `git pull --rebase`：我们总希望我们当前的工作是基于上游项目的最新版本的，普通的pull会使用merge而使提交历史产生分支，而`git pull --rebase`可以帮我们保证这一点（虽然我们实际上还是做了merge的工作）。
- `git rebase -i`：这条命令用来修饰你尚未推送到远端的提交序列，比如把多个提交合并成一个，更改某些提交的注释，调换某些提交的顺序，或者把某些提交分解成多个。由于这条命令需要你执行一系列相关操作来完成它，所以叫作“交互式rebase”。

一些常用Git命令列举
-------------------

### 基本命令

- `git status`：查看当前工作区和暂存区（相对于当前分支）的状态
- `git show {object_id}`: 查看某个对象，可以是块，树，提交，或标签对象
- `git checkout -f {commit}`：检出项目的某个指定历史版本到工作区以便查看或编辑
- `git diff`：查看工作区与“HEAD+暂存区”的差别，即查看尚未被暂存的工作
- `git diff --cached`：查看暂存区与HEAD的差别，即查看已被暂存但尚未被提交的工作
- `git diff {commit_1} [{commit_2}]`：查看项目某两个历史版本的差异，方向为从commit_1到commit_2，commit_2默认为HEAD
- `git log --stat`：查看当前版本的提交记录，--stat参数表示列出变更的文件
- `git merge {branch_a} ...`：将branch_a合并进当前分支，若branch_a所指向的提交恰好是当前分支所指向的提交的祖先提交，则当前分支将快进到branch_a所指向的提交（不会创建新提交，这种情况称为fast-forward），否则会创建一个新提交，新提交将有多个（正在合并的所有提交）父提交。该命令支持同时合并多个分支，等价于依次合并其中的每一个。
- `git config {key} {value}`：设置本项目的Git配置，配置文件在`.git/config`文件中可以看到，关于配置可以在[这里](http://git-scm.com/book/zh/%E8%87%AA%E5%AE%9A%E4%B9%89-Git-%E9%85%8D%E7%BD%AE-Git)看到。

### 远端操作

- `git remote add {remote} {url}`：添加远端仓库（相当于给远端仓库的URL取一个别名）
- `git fetch {remote}`：从远端仓库获取数据并更新本地的远端分支副本，但不会将它合并进相应的本地分支
- `git fetch --all`：fetch所有远端仓库（而非某一个）
- `git pull [{remote} {branch}]`：从远端获取数据并更新本地的远端分支副本，再从跟踪它的本地分支合并该远端分支副本，相当于fetch后再merge
- `git pull --rebase [{remote} {branch}]`：从远端获取数据并更新本地的远端分支副本，再从跟踪它的本地分支rebase该远端分支副本，相当于fetch后再rebase
- `git push [{remote} {branch}]`：推送本地分支到远端，远端相应分支会因此更新，如果不能快进（fast-forward）则会拒绝本次推送
- `git push --tags [{remote} {branch}]`：推送本地分支到远端，将本地的标签一并推送
- `git push -f {remote} {branch}`：强制推送本地分支到远端，远端相应分支会被强制更新
- `git push -u {remote} {branch}`：推送本地分支到远端，并将该远端分支设为该本地分支的上游分支

### 编辑提交历史

- `git commit --amend`：修改最近一次提交的注释
- `git rebase -i`：修改尚未推送到远端的提交序列

### 子模块

- `git submodule add {url} {dir_path}`：添加子模块
- `git submodule update --init [--recursive]`：更新子模块，若需要则在更新前初始化
- `git submodule update [--recursive]`：检出子模块的对应版本，通常在检出带子模块的项目的历史版本的时候需要同时执行该命令
- `git submodule update --remote [--recursive]`：拉取并检出子模块的远端最新版本
- `git submodule deinit {dir_path} && rm -rf {dir_path}`：删除子模块
- `git submodule foreach [--recursive] {command}`：遍历子模块执行{command}命令

### 子树

- `git subtree add [--squash] -P {dir_path} {remote} {branch}`：添加子树
- `git subtree pull [--squash] -P {dir_path} {remote} {branch}`：从远端拉取子树
- `git subtree push [--squash] -P {dir_path} {remote} {branch}`：推送子树到远端
- `git subtree merge [--squash] -P {dir_path} {commit}`：合并子树
- `git subtree split -P {dir_path} -b {new_branch_name}`：子树拆分


Git场景分析
-----------

### 建立不带工作区的中心仓库

```shell
mkdir demo_repo.git
cd demo_repo.git
git init --bare #初始化Git目录，--bare选项表示不需要工作目录
```

### 使仓库`demo_repo`可以被当作远端访问

在服务端（电脑A，设其IP为192.168.1.123）设置相应选项并开启服务：

```shell
cd demo_repo
git config daemon.receivepack true
git config receive.denyCurrentBranch ignore
git config receive.denyNonFastForwards false
nohup git daemon --verbose --export-all --base-path=.. --reuseaddr --enable=receive-pack &
```

从客户端（电脑B）访问：

```shell
git clone git://192.168.1.123/demo_repo
```

### 将项目`demo_lib`引入`demo_repo`成为其中的子目录`lib/`

```shell
git remote add -f subtree-lib git://192.168.1.123/demo_lib #添加一个叫`subtree-lib`的远端仓库，方便之后访问，`-f`选项使得add之后立即fetch一次
git subtree add -P lib/ subtree-lib master --squash #将demo_lib的master分支合并到当前分支的`lib/`目录下，--squash选项避免引入demo_lib的提交历史
```

之后可以这样更新`demo_repo`中的`lib/`目录，使得它和`demo_lib`项目同步

```shell
git fetch subtree-lib master
git subtree pull -P lib/ subtree-lib master --squash
```

如果你更改了`demo_repo/lib/`中的内容，可以将它上传到`demo_lib`项目

```shell
git subtree push -P lib/ subtree-lib master
```

### 将项目`demo_repo`的子目录`lib/`导出成独立的新项目`demo_lib`

```shell
cd demo_repo
git subtree split -P lib/ -b lib_branch #把`lib/`这个目录的内容导出为`lib_branch`分支，执行这个命令可能需要较长时间（取决于项目历史提交的数目）
cd ..
mkdir demo_lib
cd demo_lib
git init
git pull ../demo_repo lib_branch #从`demo_repo`中拉取`lib_branch`分支
```

