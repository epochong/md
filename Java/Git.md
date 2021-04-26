---
title: Git-Study
date: 2019-10-03 12:26:43
tags: Git
---

### 常用命令总结

- 版本控制
- 分支管理
- 标签管理
- 自定义Git

<!--more-->

#### 版本控制

- `git status` 命令可以让我们时刻掌握仓库当前的状态
- `git diff <file>` 顾名思义就是查看 difference，显示的格式正是 Unix 通用的 diff 格式
- `git log` 命令显示从最近到最远的提交日志及git id
- `git log --pretty=oneline` 查看日志，简化到每行
- `git reflog` 用来记录你的每一次命令，可查看未来的git id

- `git checkout -- <file>` 意思就是，把 `<file>` 文件在工作区的修改全部撤销。其实是用版本库里的版本替换工作区的版本，无论工作区是修改还是删除，都可以一键还原
    -  `<file>` 自修改后还没有被放到暂存区，撤销修改就回到和版本库一模一样的状态
    - `<file>` 已经添加到暂存区后，又作了修改，撤销修改就回到添加到暂存区后的状态
    - 当已经`commit`的本地文件被删除时，此命令可以恢复到本地
- `git reset HEAD <file>` 可以把暂存区的修改撤销掉（`unstage`），重新放回工作区
- `git reset --hard HEAD^` 回退到一个版本，`^`的个数代表回退几次，`HEAD~10`回退10次
- `git reset -- hard <git id>` 回退到对应的`git id` 版本处
- `git rm <file>` 从库中删除file文件，同时在本地库中也删除

#### 分支管理

> 在 Git 里，这个分支叫主分支，即 `master` 分支。`HEAD` 严格来说不是指向提交，而是指向 `master`，`master` 才是指向提交的，所以，`HEAD` 指向的就是当前分支。
>
> 每次提交，`master` 分支都会向前移动一步，这样，随着你不断提交，`master` 分支的线也越来越长
>
> 当我们创建新的分支，例如 `dev` 时，Git 新建了一个指针叫 `dev`，指向 `master` 相同的提交，再把 `HEAD` 指向 `dev`，就表示当前分支在 `dev` 上

- `git checkout -b <branch>` 命令加上 `-b` 参数表示**创建并切换分支**，等同于两条语句

    - `git branch <branch>` 创建分支
    - `git checkout <branch>` 切换分支

- `git branch` 查看当前分支

- `git merge <branch>`  在`master`分支下执行，将 `<branch>` 分支工作成果**合并**到`master`

- ` git branch -d <branch>` **删除**`<branch>` 分支

- `git switch -c <branch>` 创建并切换到新的 `<branch>` 分支，新版本支持

- `git switch master` 直接切换到已有的 `master` 分支

- `git log --graph --pretty=oneline --abbrev-commit`  可以看到分支的合并情况

- `git merge --no-ff -m "merge with no-ff" <branch>` 强制禁用 `Fast forward` 模式，Git 就会在 merge 时生成一个新的 commit，这样，从分支历史上就可以看出分支信息

    > 加上 `--no-ff` 参数就可以用普通模式合并，合并后的历史有分支，能看出来曾经做过合并，而 `fast forward` 合并就看不出来曾经做过合并

- `git stash` 把当前工作现场 “储藏” 起来，等以后恢复现场后继续工作

- `git stash list` 查看存储的工作现场

- `git stash apply <stash name>` 在之前保存现场的分支执行，恢复现场，但是恢复后，stash 内容并不删除，你需要用 `git stash drop` 来删除

- `git stash pop`，恢复的同时把 stash 内容也删了

- `cherry-pick <master modified's commit git id>` 命令，让我们能复制一个特定的提交到当前分支

- `git branch -D <name>`  强行删除，丢弃一个没有被合并过的分支

- `git remote` 要查看远程库的信息

- `git remote -v` 显示更详细远程仓库的信息

- `git checkout -b <branch1> origin/<branch>` 创建远程 `origin` 的 `<branch>` 分支到本地`<branch1>`，在此分支下执行的`push` 操作是提交到远程的`<branch>`分支下

- `git branch --set-upstream-to=origin/<branch> <branch>` 从远程抓取分支，指定本地 `<branch>` 分支与远程 `origin/<branch>` 分支的链接

- `git rebase` 把分叉的提交历史 “整理” 成一条直线，看上去更直观。缺点是本地的分叉提交已经被修改过了

#### 标签管理

- `git tag <name>` 创建一个新标签
- `git tag` 查看所有标签
- `git tag <name> <commit id>` 指定`commit id` 打标签
- `git show <tagname>` 查看标签信息
- `git tag -a <tagname> -m "say something" <commit id>` 创建带有说明的标签，用 `-a` 指定标签名，`-m` 指定说明文字
- `git tag -d v0.1` 标签打错了，也可以删除
- `git push origin <tagname>`  推送某个标签到远程
- `git push origin --tags` 一次性推送全部尚未推送到远程的本地标签
- `git push origin :refs/tags/<tagname>` 可以删除一个远程标签，前提先在本地删除`tag`
- `git remote rm origin` 删除远程分支
- `git remote add <branch name> <github url>` 自定义远程存储库名称

#### 自定义Git

- `git config --global color.ui true` 让 Git 显示颜色，会让命令输出看起来更醒目

- `git add -f App.class` `-f` 强制添加被Git忽略的文件

- `git check-ignore -v <file name>` 检查`.gitignore` 写得问题，找出来写错的规则

    > 在 Git 工作区的根目录下创建一个特殊的`.gitignore` 文件，然后把要忽略的文件名填进去，Git 就会自动忽略这些文件，如下
    >
    > ```.gitignore
    > # Windows:
    > Thumbs.db
    > ehthumbs.db
    > Desktop.ini
    > 
    > # Python:
    > *.py[cod]
    > *.so
    > *.egg
    > *.egg-info
    > dist
    > build
    > 
    > # My configurations:
    > db.ini
    > deploy_key_rsa
    > ```

- `git config --global alias.st status` 告诉 Git，以后 `st` 就表示 `status`

- `git config --global alias.allow 'pull origin master —allow-unrelated-histories'`

### 分支管理

#### 简易查看分支合并情况

`git log --graph --pretty=oneline --abbrev-commit`

#### 详细查看分支合并情况

`git log --graph`

#### 合并分支的时候出现冲突（两种解决方法）

- 直接合并：`git merge 分支名`

  1. 本地修改冲突文件（可以通过工具方便快捷）
     - 保留自己的分支修改
     - 保留合并的分支修改
     - 两个修改都保留
  2. `git add 文件`
  3. `git commit -m ''`

  ![image-20200120115403053](/Users/wangchong/Library/Application Support/typora-user-images/image-20200120115403053.png)

- 新建分支合并： 分支名`（自动新建分支）

  1. 在要合入的那个分支hotfix上执行`git rebase master`
  2. 本地修改冲突文件（可以通过工具方便快捷）
  3. `git add .`
  4. `git rebase --continue`（add后不能commit，变基不是一次提交，而是改变提交的基础）
  5. `git log --oneline --graph --all`查看版本合并情况，可以看到变成串行而不是多分支
  6. `git checkout master`切换到master分支
  7. `git merge hotfix`合并分支 

  ![image-20200120142251509](/Users/wangchong/Library/Application Support/typora-user-images/image-20200120142251509.png)

#### 显示出所有有差异的文件列表

`git diff branch1 branch2 --stat`

#### 显示指定文件的详细差异

`git diff branch1 branch2 文件名(带路径)`   

#### 显示出所有有差异的文件的详细差异

`git diff branch1 branch2` 

### 放弃修改

#### 未add要放弃修改

##### git放弃修改（modified）

单个文件/文件夹：

```
git checkout -- filename
```


所有文件/文件夹：

```
git checkout .
```

##### 放弃增加文件（Untracked files）

单个文件/文件夹（命令/手动删除都行）：

```
rm filename / rm dir -rf
```


所有文件/文件夹：

```bash
git clean -xdf
```

#### add了但是未commit

##### 本地修改/新增了一堆文件，已经git add到暂存区，想放弃修改。

单个文件/文件夹：

```bash
git reset HEAD filename
```


所有文件/文件夹：

```bash
git reset HEAD .
```

#### add & commit 之后，想要撤销此次commit

```
git reset commit_id
```

这个id是你想要回到的那个节点，可以通过git log查看，可以只选前6位
// 撤销之后，你所做的已经commit的修改还在工作区！

```
git reset --hard commit_id
```

这个id是你想要回到的那个节点，可以通过git log查看，可以只选前6位
// 撤销之后，你所做的已经commit的修改将会清除，仍在工作区/暂存区的代码也将会清除！

### fast-forward

意思就是本次提交被远程仓库拒绝了，因为当前分支无法与远程仓库对应起来。远程仓库对应分支默认有个指针指向最新提交到仓库的 commit ，而所有的本地仓库的分支都可以看做是从这个 commit 分散开来的。也就是本地分支的最后一次 push 到仓库的 commit 一定与仓库对应分支的最新一次 commit 是相同的，否则就无法对接。也就是会出现上面的错误提示。如果是正常 push 到仓库，正确的完成 commit 更新，那么这次更新就是一个 `fast-forward` 更新,而如果不理会错误警告用本地更新强制覆盖仓库，就是一次 `no-fast-forward` 更新，很明显，**`no-fast-forward` 更新会导致记录丢失**。

那么这种问题是如何发生的呢？比如有两个人都是从仓库的 master 分支克隆到本地，然后分别开发。master 本身有一个指针 HEAD 指向最后一次 commit 记录 commit-0 。A 先完成一个功能，并 push 到仓库，这次 commit 记为 commit-A，这也就是一次 `fast-forward` 更新，此时仓库的 master 分支的 HEAD 指针就指向了 commit-A。接下来 B 也完成了一个功能，要向仓库 push commit-B，如果没有做额外操作，肯定会出现上面的错误。

知道错误是如何发生的，就可以避免了。既然仓库有了更新，那么就要先把仓库的更新拉取到本地。这里有两种方式可以拉取：一是直接使用 `git pull` 命令，该命令会在拉取的同时会直接与本地对应分支进行合并，如果确信仓库的更新与本地不会发生冲突，那么可以直接使用。但是很可能 A 与 B 都对同一些文件做出了修改，那么必然导致冲突。不过既然知道会冲突也只能老老实实解决冲突了，不管是 fetch 先解决冲突在合并还是 pull 先合并再解决冲突，这个过程少不了的，除非你确定仓库的更新是没用的可以直接抛弃，就可以执行 `git push -f` 强制覆盖到仓库，这会导致仓库中某些记录丢失。

我们借助于 githug 28 关来模拟看看：

```bash
    $ git push -u origin master
    To C:/Users/Alpha/AppData/Local/Temp/d20170731-15124-1ywoym1/.git
     ! [rejected]        master -> master (non-fast-forward)
    error: failed to push some refs to 'C:/Users/Alpha/AppData/Local/Temp/d20170731-15124-1ywoym1/.git'
    hint: Updates were rejected because the tip of your current branch is behind
    hint: its remote counterpart. Integrate the remote changes (e.g.
    hint: 'git pull ...') before pushing again.
    hint: See the 'Note about fast-forwards' in 'git push --help' for details.
    
    $ git log origin/master --oneline
    68ad000 Fourth commit
    
    $ git log --oneline
    b977ec3 Third commit
    6d15890 Second commit
    5266aa2 First commit
    
    $ git push -f -u origin master
    Counting objects: 7, done.
    Delta compression using up to 4 threads.
    Compressing objects: 100% (5/5), done.
    Writing objects: 100% (7/7), 546 bytes | 0 bytes/s, done.
    Total 7 (delta 2), reused 0 (delta 0)
    To C:/Users/Alpha/AppData/Local/Temp/d20170731-15124-1ywoym1/.git
     + 68ad000...b977ec3 master -> master (forced update)
    Branch master set up to track remote branch master from origin.
    
    $ git log origin/master --oneline
    b977ec3 Third commit
    6d15890 Second commit
    5266aa2 First commit
```

可以看到，强制覆盖 push 后，仓库的 `Fourth commit` 已经不见了。

但如果不想丢掉 commit-A 的同时又不想与 commit-A 合并，B 想继续接着本地仓库工作，可以使用 `git rebase origin/master` ,表示将本地所有 commit 排在仓库 的 commit 记录之后。然后向仓库的 push 就会被接受。同样借助于 githug 28 关，而且，这才是 28 关正确的过关方式：

详细过关过程如下：

```bash
    $ git push -u origin master
    To C:/Users/Alpha/AppData/Local/Temp/d20170731-14980-yb0fll/.git
     ! [rejected]        master -> master (non-fast-forward)
    error: failed to push some refs to 'C:/Users/Alpha/AppData/Local/Temp/d20170731-14980-yb0fll/.git'
    hint: Updates were rejected because the tip of your current branch is behind
    hint: its remote counterpart. Integrate the remote changes (e.g.
    hint: 'git pull ...') before pushing again.
    hint: See the 'Note about fast-forwards' in 'git push --help' for details.
    
    $ git log origin/master --oneline
    015383a Fourth commit
    
    $ git log --oneline
    38aa398 Third commit
    1c03f48 Second commit
    ad32a6d First commit
    
    $ git rebase origin/master
    First, rewinding head to replay your work on top of it...
    Applying: First commit
    Applying: Second commit
    Applying: Third commit
    
    $ git log --oneline
    fbf0528 Third commit
    03d1240 Second commit
    9828360 First commit
    015383a Fourth commit
   
    $ git push -u origin master
    Counting objects: 6, done.
    Delta compression using up to 4 threads.
    Compressing objects: 100% (6/6), done.
    Writing objects: 100% (6/6), 607 bytes | 0 bytes/s, done.
    Total 6 (delta 2), reused 0 (delta 0)
    To C:/Users/Alpha/AppData/Local/Temp/d20170731-14980-yb0fll/.git
       015383a..fbf0528  master -> master
    Branch master set up to track remote branch master from origin.
```

但其实还有一种比较常见的出现 `no-fast-forward` 这种错误的情境，是在你向一个只有你自己可访问的仓库 push 的时候发生的。当你已经将一次 commit-A push 到仓库后，然后因为某些原因又使用了 git commit --amend 修改了 commit-A ,这个时候 commit-A 就变成了 commit-B，而此时本地仓库就没有关系 commit-A 的记录了，这个时候再次向仓库 push ，很明显，commit-B 无法与仓库的 commit-A 进行对接，所以出现了 `no-fast-forward` 错误。这种情况下其实也很好解决，**如果你确定 commit-A 已经完全无用并且没有人将 commit-A 拉取到本地进行进一步开发**之后，你就使用 `git push -f` 来覆盖仓库记录。之后，你就会永远丢失 commit-A 记录了。

而对比发现，我之所以会遇到本文开头的错误，就是因为之前使用了 `git commit --amend` 命令修改了已经 push 到仓库的 commit 的注释导致的。因此，一旦已经 push 到仓库，想要做出修改，就只能通过一次新的 commit 来完成对某次已经 push 到仓库的 commit 记录的修改了，可以参考 githug 52 关 revert。

#### git add -A和 git add . 

git add . ：他会监控工作区的状态树，使用它会把工作时的**所有变化提交**到暂存区，包括文件内容修改(modified)以及新文件(new)，但不包括被删除的文件。

git add -u ：他仅监控**已经被add的文件**（即tracked file），他会将被修改的文件提交到暂存区。add -u 不会提交新文件（untracked file）。（git add --update的缩写）

git add -A ：是上面两个功能的合集（git add --all的缩写）