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

