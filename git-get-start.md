Git 分布式版本控制系统，可分为三个区域：工作区、暂存区、归档区。

工作区(working tree) $\xRightarrow{add}$ 暂存区(stage, index) $\xRightarrow{commit}$ 归档区 $\xRightarrow{push}$ 远程仓库

add 将工作区的文件提交到暂存区，commit 是将暂存区的数次提交存入本地归档区，此时已经完成了版本控制，但通常还将其通过 `push` 上传到远程仓库，如 github。另外文件夹的 .git 不属于工作区，其是版本库（暂存区和归档区）。

查看 git 官方帮助文档: `git help <command>`，如: `git help restore`，会通过浏览器打开 git 安装目录的 html 文档。官方帮助文档是英文的，这是网上翻译的中文文档：[Git 教程](https://www.yiibai.com/git)。

> git version: 2.26。由于 checkout 命令功能繁多易混杂，从 2.22 版本后官方将其分裂推出了 restore 和 switch 两个命令，下面笔记也不在使用 checkout。

## 1. Git 基本流程

1. `git init`: 初始化工作区，创建 master 分支;
2. `git add .`: 将此目录下所有文件均提交到暂存区;
3. `git commit -m message`: 添加到暂存区，并附带本次提交的描述
4. `git remote add origin https://github.com/xxx/xxx.git`: 添加远程仓库，名称为 origin，远程地址为 xxx;
5. `git push -u origin master`: 将本地归档区内容上传至上面指定的 origin 的远程仓库，*-u* 参数用来指定默认远程主机，详细见下

后续工作区文件有更改或者删除时，可以直接将这些变化提交到归档区: `git commit -am "message"`。

## 2. 状态查看

- `git status`: 查看工作区状态，有无更改、新增等。加参数 `-s` 获得简短信息

查看差异(可加文件名只查看某一文件):

- `git diff`: 查看工作区与暂存区差别;
- `git diff --cached`: 查看暂存区与归档区差别;
- `git diff HEAD`: 查看工作区与归档区差别;
- `git diff branch1 branch2`: 查看两个分支差别;

查看提交日志: `git log`，通过参数看简洁日志: `git log --pretty=oneline`，其支持定制颜色、长度等。另外网上查找的比较美观的日志:

```git
git log --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit
```

设置 alias 快捷，此后调用 `git lg`:

```git
git config --global alias.lg "log --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit"
```

## 3. 版本回退

版本回退之前需要归档的版本号(*commit id*)，git 中版本号是一串 SHA1 计算的消息摘要，通过  查看，或者在 git 中还有一个 HEAD 指针，指向当前版本，上版本为 *HEAD^*，上上版本为 *HEAD^^*，前10个版本为 *HEAD~10*，下面指令均可使用 HEAD 指代版本号。

涉及工作区回退要注意，如果工作区内容有更改，但未 add 或者 commit，则一旦被回退这些更改就会被丢弃，无法找回。

### 3.1 正常回退

版本回退 `git reset [opt] 版本号`

- 版本号：*commit id* 前几位，只要可以唯一定位
- 参数有以下几种:
  - `--soft`: 归档区进行版本回退
  - `--mixed`: 归档区和暂存区均进行版本回退，默认参数
  - `--hard`: 归档区、暂存区、工作区均进行版本回退
- 特殊用法: `git reset HEAD file`：删除暂存区 file 的改动。

如果需要仅仅只回退某文件而不产生新的提交，使用 `git restore`，详细见下面的文件删除与恢复。

其会将文件回退到最近一次的暂存区或者归档区的状态，注意工作区此文件的修改会被丢弃，故其也可以用作错删文件在工作区的恢复。

### 3.2 回退某版本并保留其后的版本

如果需要删除版本中某一次提交，且保留其之后的提交：`git revert 版本号`，相当于删除那一次提交后创建一次新的提交，注意其会将归档区、暂存区、工作区均进行版本回退，且工作区的修改会被丢弃。使用此命令后 vim 提示编辑此次的描述。

### 3.3 还原被回退掉的版本

如果需要还原某次被回退掉的版本，可以通过 `git reflog` 查看所有提交记录，获取版本号。

## 4. 工作区文件删除与恢复

文件恢复命令是 restore，功能有:

- `git restore file`: 将暂存区的 file 恢复到工作区，可以由通配符指定多个文件，如 `.`, `'*.py'`;
- `git restore --source=HEAD file`: 将归档区的 file 恢复到工作区，暂存区不变化，source 可以指定为前几个版本，如前两个版本 *HEAD^^*;
- `git restore --source=HEAD --staged --worktree file`: 将归档区的 file 恢复到暂存区与工作区，如果只想恢复到暂存区，去掉 worktree即可，但实际没有用不到。

文件删除是 rm，会删除工作区以及暂存区的文件。

## 5. 修改描述

修改上一次提交的描述: `git commit --amend -m message`。

修改历史一次或者多次描述: `git rebase -i commit_id`，修改 commit_id 之后的提交描述（不包括 commit_id）。使用此命令后会进入 vim，在需要修改描述的版本前将 *pick* 修改为 *reword*，保存退出后，会依次按照版本增序进入编辑器，修改描述保存即可。

## 6. 分支

分支的作用（引用[廖雪峰对 git 分支的介绍](https://www.liaoxuefeng.com/wiki/896043488029600/896954848507552)）：在目前的版本基础上开发新功能，但开发中如果一点点提交新版本，那当前版本的代码不能再工作，如果选择新功能开发完成再一次提交，又会存在丢失进度的风险。这时分支的重要性就体现了，可以创建一个自己的分支，在自己的分支上进行开发，直到完成后再合并到原分支上。

### 6.1 创建、切换和删除

- 查看分支: `git branch -v` (星号标识当前分支)
- 创建分支: `git branch branchName`
- 切换分支: `git switch branchName`
- 创建并切换分支: `git switch -c branchName`
- 删除分支: `git branch -d branchName`

### 6.2 分支合并

将 dev 合并到 master 分支，指令依次为:

```git
git switch master
git merge dev
```

合并冲突：如果分支1 修改了某个文件，而分支2 也修改了此文件，则这两分支合并时就会发生冲突，并会在冲突的文件中以 *<<<<<<<*, *=======*, *>>>>>>>* 标识出冲突的内容以及分支。尽量不要直接在 master 上合并，等到确保其他分支合并成功后再合并到 master 分支。

解决冲突：修改冲突文件内容以解决冲突，再提交，再合并。

## 7. 远程库

远程仓库的添加与删除:

- `git remote` 查看远程仓库信息，加参数 *-v* 显示更详细信息;
- `git remote add origin <remoteUrl>`: 添加远程仓库，并别名为 origin;
- `git remote remove origin`: 删除 origin 远程仓库

设置拉取和推送的默认远程仓库和分支，设置后在当前分支下拉取和推送就不需要再指定了，功能与下面的 *-u* 参数相同:

```git
git branch --set-upstream-to=origin/dev dev  # local dev <-----> origin dev
```

本地推送到远程，`git push` 常用参数有以下:

- `git push <remote> <localBranch>:<remoteBranch>`: 本地的 localBranch 分支推送到远程的 remoteBranch 分支;
- `git push -u origin dev`: 将本地 dev 分支推送到远程的 dev，*-u* 参数指定推送的默认分支和默认远程，进而后续在默认分支时可以直接使用 `git push`，而无须其他参数。;
- `git push -all origin`: 将本地所有分支均推送到远程

本地拉取远程仓库：`git pull`，其他参数同上，其相当于先 `git fetch` 再 `git merge`，如:

- `git pull origin master` 就相当于 `git fetch origin` 再 `git merge origin/master`，如果未指定本地分支，意味着合并到当前分支。

在多人协作时，远程库可能比本地版本新，故本地在推送前需要先 pull，再 push，如果发生冲突，解决完冲突后再次操作即可。

总结下一般流程:

```git
# 1. set upstream
git branch --set-upstream-to=origin/dev dev

# 2. pull
git switch dev
git pull

# 3. push
git push
```
