Git常用命令列举
===============

### 基本命令

- `git status`：查看当前工作区和暂存区（相对于当前分支）的状态
- `git show {object_id}`: 查看某个对象，可以是块，树，提交，或标签对象
- `git checkout -f {commit}`：检出项目的某个指定历史版本到工作区以便查看或编辑
- `git checkout -b {new_branch}`：从当前分支创建新分支new_branch并检出到新分支
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

