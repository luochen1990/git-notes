Git进阶指导
===========

### 高级功能

- 权限管理（通常Git以项目为单位，控制每个开发者对每个项目的读、写、设置的权限）
- 代码审查（通过pull request）
- 依赖管理（git submodule, git subtree, repo）

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
git subtree add -P lib subtree-lib master --squash #将demo_lib的master分支合并到当前分支的`lib/`目录下，--squash选项避免引入demo_lib的提交历史（注意：lib后面不能加`/`）
```

之后可以这样更新`demo_repo`中的`lib/`目录，使得它和`demo_lib`项目同步

```shell
git fetch subtree-lib master
git subtree pull -P lib subtree-lib master --squash #（注意：lib后面不能加`/`）
```

如果你更改了`demo_repo/lib/`中的内容，可以将它上传到`demo_lib`项目

```shell
git subtree push -P lib subtree-lib master #（注意：lib后面不能加`/`）
```

### 将项目`demo_repo`的子目录`lib/`导出成独立的新项目`demo_lib`

```shell
cd demo_repo
git subtree split -P lib -b lib_branch #把`lib/`这个目录的内容导出为`lib_branch`分支，执行这个命令可能需要较长时间（取决于项目历史提交的数目）（注意：lib后面不能加`/`）
cd ..
mkdir demo_lib
cd demo_lib
git init
git pull ../demo_repo lib_branch #从`demo_repo`中拉取`lib_branch`分支
```

