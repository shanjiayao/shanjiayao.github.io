# git学习记录


重温git，并整理各个命令的用法。

<!--more-->

## 1. Git的历史

Linux，1991年， 创建开源Linux

2008年，开源GitHub

两周时间，用C代码编写而成

分布式的框架，与SVN等不同

## 2. Git的四个区

Git可以分成四个部分，工作区、暂存区、本地仓库、远程仓库

### 工作区

其实就是git仓库所在的工作目录，表示的是当前这个目录下的改动状态，如果有新的改动，就可以提交到暂存区

### 暂存区(stage)

暂存区的功能是介于工作区的实时改动和版本仓库之间的媒介，类似于cache缓存一样。

使用`git add <file>`，把修改的文件添加到暂存区

使用`git status`，可以查看git以及暂存区的状态


### 本地仓库

本地仓库存储着git的各个分支以及版本

使用git commit命令，可以将暂存区中的信息提交到本地仓库

> 注意：_git管理的是修改_，只有经过git add之后，存储在暂存区的文件才可以经过git commit进行提交

![](https://pictures-1309138036.cos.ap-nanjing.myqcloud.com/img/20220622231214.png)


### 远程仓库

远程仓库的提交于下载是需要联网的，远程仓库实现了多终端分布式协同管理。

-   要关联一个远程库，使用命令`git remote add origin git@server-name:path/repo-name.git`；
-   关联一个远程库时必须给远程库指定一个名字，`origin`是默认习惯命名；
-   关联后，使用命令`git push -u origin master`第一次推送master分支的所有内容；
-   此后，每次本地提交后，只要有必要，就可以使用命令`git push origin master`推送最新修改；

> Git支持SSH协议，所以，GitHub只要知道了你的公钥，就可以确认只有你自己才能推送。默认的git://使用ssh

## 3. Git的分支命令之`git branch`

Git分支的主要命令是`git branch` ，分支的基础操作包括创建分支、删除分支、合并分支等。

```jsx
git branch <branch_name> <commit_id>   # 在对应位置创建分支
git checkout -b <branch_name> <commit_id>   # 在对应位置创建分支并切换到新分支
git branch -f <branch_name> <commit_id>   # 强制将某分支移动到某次commit_id的位置
git branch -f main HEAD~3   # 强制将main分支指向当前HEAD的3个之前的提交版本
git branch -d dev  # 删除dev分支
git branch -u <origin/branch1> <branch2>   # 使branch分支跟踪远程分支的状态
git checkoput -b <branch2> <origin/branch1>   # 同上
git branch   # 查看分支
git checkout <name>   # 切换分支
git switch <name>   # 切换分支 同上
```

## 4. Git的分支合并之merge与rebase

![](https://pictures-1309138036.cos.ap-nanjing.myqcloud.com/img/20220622225409.png)


对于常见的开发环境而言，无论是单人项目还是多人协作，都会涉及到多个分支的创建与分支之间的合并。`git merge`与`git rebase`是git中两个常用的分支合并命令，二者各有优劣，示意图如上所示。

### 4.1 git merge

**🔔操作流程**

merge合并两个分支时会产生一个特殊的提交记录，它有两个父节点。其指向两个父节点并继承二者的更改内容，最终新的commit_id对应的改动同时包含了二者。可以发现merge并不会改动已有的分支。

**🔔命令用法**

```jsx
git merge <name>   # 将某分支合并到当前分支，注意当前的位置，可以是分支名、commit_id，tag等
```

**🔔缺点**

虽然merge 不会改变已有的提交历史，但是当commit操作不规范，或者项目过于庞大时，过多的分支会导致项目难以管理。

### 4.2 git rebase

**🔔操作流程**

rebase，顾名思义，是将对分支设定一个新的base，以实现分支之间的合并，具体是通过设置开始和结束位置，将另外一个分支已有的feature依据commit_id打包成不同的patch，存储到`.git/rebase`下面。然后，将feature的修改依次应用到main分支上，包括分支指向以及分支内容。rebase 使你的提交树变得很干净, 所有的提交都在一条线上。

**🔔命令用法**

```jsx
git rebase  [startpoint]  [endpoint]   # 这里的start和end可以是分支名或commit_id等
git rebase -i [startpoint]  [endpoint]   # 交互式，在vim中修改各个分支的顺序、状态等
git rebase --continue   # 用于解决rebase的冲突之后，继续进行rebase
git rebase --abort   # 终止rebase
```

**🔔缺点**

rebase 修改了提交树的历史，比如, 提交 C1 可以被 rebase 到 C3 之后。这看起来 C1 中的工作是在 C3 之后进行的，但实际上是在 C3 之前。

## 5. Git分支操作之cherry-pick

某些时候我们可能需要创建一个新的分支，新的分支中包含了已有各个feature分支中的若干改动，这时可以使用`cherry-pick`操作。

### 5.1 **操作流程**

cherry-pick 可以将提交树上任何地方的提交记录取过来追加到 HEAD 上（只要不是 HEAD 上游的提交就没问题），具体就是按照选择的顺序依次放置在新的分支上。

![](https://pictures-1309138036.cos.ap-nanjing.myqcloud.com/img/CPT2206221651-932x525.gif)


```jsx
git cherry-pick [point1] [point2] [point3] [point4] ...
```

## 6. Git中的HEAD以及相关操作

### 6.1 **什么是HEAD？**

HEAD 是一个对当前检出记录的符号引用 —— 也就是指向你正在其基础上进行工作的提交记录。

HEAD 总是指向当前分支上最近一次提交记录。大多数修改提交树的 Git 命令都是从改变 HEAD 的指向开始的。HEAD 通常情况下是指向分支名的（如 bugFix）。

### 6.2 **分离HEAD**

使用 `git checkout <commit_id>` 可以将HEAD与分支名分离开，使其指向具体某一次的提交历史。

### 6.3 **在提交历史中移动HEAD**

此外，同样基于`git checkout`命令，可以将HEAD移动到提交历史中的任意地方，并且HEAD也可以用作一个相对引用位置在其他命令中使用，如下：

-   `HEAD^`表示`HEAD`的前一次版本，也就是`HEAD`的父节点，同理，`HEAD^^`表示第二个父节点。
    
-   `HEAD~n`表示`HEAD`之前的第n次，也就是让`HEAD`指向其第n个父节点。
    
-   针对`HEAD`具有多个一级父节点的情况，`HEAD^n`表示移动到第1级父节点中，第n个节点的位置。Git 默认选择合并提交的“第一个”父提交，在操作符 `^`  后跟一个数字可以改变这一默认行为
    
-   他们还支持链式操作 `git checkout HEAD~^2~2`
    
![](https://pictures-1309138036.cos.ap-nanjing.myqcloud.com/img/20220622225734.png)


**以上操作同样适用于分支名，**比如，`branch_name^`表示某分支的默认的一级父节点，其余同上。

## 7. Git中撤销变更

[Resetting, Checking Out & Reverting | Atlassian Git Tutorial](https://www.atlassian.com/git/tutorials/resetting-checking-out-and-reverting)

在 Git 里撤销变更的方法很多。和提交一样，**撤销变更**由底层部分（变更来源？）和上层部分（如何撤销）组成。

### 7.1 对号入座

首先，考虑到Git的四个区，针对变更来源一般会有两部分：

1.  工作区，文件修改，未进行`add/commit`时，想要丢弃，使用`git checkout -- file` 命令可以将其还原到未修改的状态
    
    <aside> 💡 `git checkout`其实是用版本库里的版本替换工作区的版本，无论工作区是**修改**还是**删除**，都可以“一键还原”。 `git checkout -- file`命令中的`--`很重要，没有`--`，就变成了“切换到另一个分支”的命令。 `git checkout .` 撤销所有未提交但已经发生的修改，不包括新增文件
    
    </aside>
    
2.  暂存区/本地仓库，已经使用了`git add`或者是`git commit`操作添加到了暂存区，可以使用`git reset`或者是`git revert` ，此外，如果是不同版本之间的切换，可以使用`git checkout`
    
    ```javascript
    ############### git reset #########################
    git reset HEAD   # 回退到指定版本，具体的HEAD的值可以通过git log查看
    git reset —soft HEAD   # --soft 用于回退到某个版本
    git reset —hard HEAD   # --hard 表示撤销工作区中所有未提交的修改内容，将暂存区与工作区都回到HEAD对应的版本，并删除之前的所有信息提交
    
    ############### git revert ############################
    git revert HEAD   # 与HEAD版本中相反修改的操作，并形成一次新的提交，不会修改历史，适用于公共仓库的修改
    ```
    

### 7.2 三种方式的对比

`git checkout` 与 `git reset` 通常被用于本地的撤销操作，他们可能会修改已有的提交历史，以至于在公共项目中导致冲突。其中`git checkout`针对不同提交之间，只是做了分离`HEAD`的操作，不会修改分支，而`git reset`则是会改变提交历史，进而修改分支。

`git revert`则是通过构建新的反向修改提交来撤销变更，这种虽然多了一次提交记录，但是却很好地保留了历史信息，适用于大型公共维护的项目。

![](https://pictures-1309138036.cos.ap-nanjing.myqcloud.com/img/20220622225803.png)


## 8. Git Tag

### 8.1 Tag的作用以及使用方式

分支名指向的版本很容易被人为移动，并且当有新的提交时，它也会移动。

因此，分支很容易被改变，大部分分支还只是临时的，并且还一直在变。那么`git tag`就是为了给特定版本打上一生的烙印。

比如软件发布新的大版本，或者是修正一些重要的 Bug 或是增加了某些新特性时，可以通过`git tag`，相比于创建一个新的分支更加合理。

Tag并不会随着新的提交而移动。你也不能切换到某个标签上面进行修改提交，它就像是提交树上的一个锚点，标识了某个特定的位置。

```jsx
git tag <tag_name> <commit_id>   # 给对应的位置打标签，可以是版本号，或分支名所指向的位置
```

### 8.2 使用describe查看tag的信息

`git describe` 的语法是：`git describe <ref>`

`<ref>` 可以是任何能被 Git 识别成提交记录的引用，如果你没有指定的话，Git 会以你目前所检出的位置（`HEAD`）。

它输出的结果是这样的：`<tag>_<numCommits>_g<hash>`

`tag` 表示的是离 `ref` 最近的标签， `numCommits` 是表示这个 `ref` 与 `tag` 相差有多少个提交记录， `hash` 表示的是你所给定的 `ref` 所表示的提交记录哈希值的前几位。

当 `ref` 提交记录上有某个标签时，则只输出标签名称

## 9. 远程仓库管理

### 9.1 git clone将远程克隆到本地

git clone 之后，本地仓库多了一个名为 `o/main` 的分支, 这种类型的分支就叫**远程**分支。由于远程分支的特性导致其拥有一些特殊属性。

-   远程分支反映了远程仓库(在你上次和它通信时)的**状态**。
-   `checkout`到远程分支时自动进入分离 HEAD 状态。Git 这么做是出于不能直接在这些分支上进行操作的原因, 你必须在别的地方完成你的工作, （更新了远程分支之后）再用远程分享你的工作成果。即，如果切换到远程分支，即使做了改动，commit之后也不会更新，只有远程仓库更新之后它才会更新

```jsx
git clone <address>   # 下载指定地址的git 仓库
git clone --recurisive <address>   # 递归地下载，适用于仓库中包含submodule的情况
```

### 9.2. git fetch将远程仓库同步到本地

**git fetch 做了些什么**

`git fetch` 完成了仅有的但是很重要的两步:

-   从远程仓库下载本地仓库中缺失的提交记录
-   更新远程分支指针(如 `o/main`)

`git fetch` 实际上将本地仓库中的远程分支更新成了远程仓库相应分支最新的状态。

如果你还记得上一节课程中我们说过的，远程分支反映了远程仓库在你**最后一次与它通信时**的状态，`git fetch` 就是你与远程仓库通信的方式了！希望我说的够明白了，你已经了解 `git fetch` 与远程分支之间的关系了吧。

`git fetch` 通常通过互联网（使用 `http://` 或 `git://` 协议) 与远程仓库通信。

**git fetch 不会做的事**

`git fetch` 并不会改变你本地仓库的状态。它不会更新你的 `main` 分支，也不会修改你磁盘上的文件。

理解这一点很重要，因为许多开发人员误以为执行了 `git fetch` 以后，他们本地仓库就与远程仓库同步了。它可能已经将进行这一操作所需的所有数据都下载了下来，但是**并没有**修改你本地的文件。我们在后面的课程中将会讲解能完成该操作的命令 :D

所以, 你可以将 `git fetch` 的理解为单纯的下载操作

```jsx
git fetch origin main   # 拉取远程origin中main分支的改动
git fetch origin main:bug   # 拉取远程origin中main分支到本地的bug分支中
git fetch origin :bug   # 创建新的分支bug在本地
```

### 9.3 git pull在拉取的同时整合版本

既然我们已经知道了如何用 `git fetch` 获取远程的数据, 现在我们学习如何将这些变化更新到我们的工作当中。

其实有很多方法的 —— 当远程分支中有新的提交时，你可以像合并本地分支那样来合并远程分支。也就是说就是你可以执行以下命令:

-   `git cherry-pick o/main`
-   `git rebase o/main`
-   `git merge o/main`
-   等等

实际上，由于先抓取更新再合并到本地分支这个流程很常用，因此 Git 提供了一个专门的命令来完成这两个操作。它就是我们要讲的 `git pull`。

`git pull`  就是 fetch 和 merge 的简写，类似的 `git pull --rebase`  就是 fetch 和 rebase 的简写！

```jsx
git pull origin main   # 拉取远程origin中main分支的改动，并merge
git pull origin main:bug   # 拉取远程origin中main分支到本地的bug分支中，并merge
git pull origin :bug   # 创建新的分支bug在本地
```

### 9.4 git push

`git push` 负责将**你的**变更上传到指定的远程仓库，并在远程仓库上合并你的新提交记录。一旦 `git push` 完成, 你的朋友们就可以从这个远程仓库下载你分享的成果了！

你可以将 `git push` 想象成发布你成果的命令。

> _注意 —— `git push` 不带任何参数时的行为与 Git 的一个名为 `push.default` 的配置有关。它的默认值取决于你正使用的 Git 的版本，但是在教程中我们使用的是 `upstream`。 这没什么太大的影响，但是在你的项目中进行推送之前，最好检查一下这个配置。_

我们可以为 push 指定参数，语法是：

```jsx
git push <remote> <place>   # 推送远程origin中place分支的改动
git push origin <source>:<destination>   # 将source本地分支推送到远程名为destination的分支，如果没有，会自动创建
git push origin :<destination>   # 删除远程分支destination
```

`<place>` 参数是什么意思呢？我们稍后会深入其中的细节, 先看看例子, 这个命令是:

`git push origin main`

把这个命令翻译过来就是：

_切到本地仓库中的“main”分支，获取所有的提交，再到远程仓库“origin”中找到“main”分支，将远程仓库中没有的提交记录都添加上去，搞定之后告诉我。_

我们通过“place”参数来告诉 Git 提交记录来自于 main, 要推送到远程仓库中的 main。它实际就是要同步的两个仓库的位置。

需要注意的是，因为我们通过指定参数告诉了 Git 所有它需要的信息, 所以它就忽略了我们所检出的分支的属性！

要同时为源和目的地指定 `<place>` 的话，只需要用冒号 `:` 将二者连起来就可以了：

`git push origin <source>:<destination>`

这个参数实际的值是个 refspec，“refspec” 是一个自造的词，意思是 Git 能识别的位置（比如分支 `foo` 或者 `HEAD~1`）

如果你要推送到的目的分支不存在会怎么样呢？没问题！Git 会在远程仓库中根据你提供的名称帮你创建这个分支！`git push origin main:new_branch_name`

## 10. 让Git自动记住Token

![](https://pictures-1309138036.cos.ap-nanjing.myqcloud.com/img/20220622225835.png)


## 11. Git Commit规范

```jsx
<type>(<scope>): <subject>
```

-   type
    
    -   用于说明 `commit` 的类别，只允许使用下面7个标识。
        
        ```
        feat：新功能（feature）
        fix：修补bug
        docs：文档（documentation）
        style： 格式（不影响代码运行的变动）
        refactor：重构（即不是新增功能，也不是修改bug的代码变动）
        test：增加测试
        chore：构建过程或辅助工具的变动
        ```
        
-   scope
    
    -   用于说明 `commit` 影响的范围，比如数据层、控制层、视图层等等，视项目不同而不同。
-   subject
    
    -   是 `commit` 目的的简短描述，不超过50个字符。
        
        ```
        1.以动词开头，使用第一人称现在时，比如change，而不是changed或changes
        2.第一个字母小写
        3.结尾不加句号（.）
        ```
        

## 12. Git指令总结

```bash
git init
git status
git add <file>
git commit - m 'xxx'
git log  # 显示从最近到最远的提交日志
git reflog  # 记录你的每一次命令,相比于log，记录每次动作

git branch <branch_name> <commit_id>   # 在对应位置创建分支
git checkout -b <branch_name> <commit_id>   # 在对应位置创建分支并切换到新分支
git branch -f <branch_name> <commit_id>   # 强制将某分支移动到某次commit_id的位置
git branch -f main HEAD~3   # 强制将main分支指向当前HEAD的3个之前的提交版本
git branch -d dev  # 删除dev分支
git branch -u <origin/branch1> <branch2>   # 使branch分支跟踪远程分支的状态
git checkoput -b <branch2> <origin/branch1>   # 同上
git branch   # 查看分支
git checkout <name>   # 切换分支
git switch <name>   # 切换分支 同上

git checkout <branch name>   # 切换到某个分支
git checkout <commit id>   # 分离HEAD 到某次历史版本
git checkout <tag name>   # 分离HEAD到某次Tag中
git checkout -b <branch name> <commit id>   # 创建并切换到指定分支，基于某次历史版本
git checkout -- <file>   # 撤销某个文件的修改，对照本地版本库

git merge bug #合并bug分支到当前分支

git rebase  [startpoint]  [endpoint]   # 这里的start和end可以是分支名或commit_id等
git rebase -i [startpoint]  [endpoint]   # 交互式，在vim中修改各个分支的顺序、状态等
git rebase --continue   # 用于解决rebase的冲突之后，继续进行rebase
git rebase --abort   # 终止rebase

git cherry-pick ...

git reset --hard HEAD^  # 回退到上一个版本
git reset --hard HEAD^^  # 回退到上上个版本
git reset --hard HEAD~100  # 回退到100个之前的版本
git revert HEAD   # 回退到上一个版本，但是会在当前基础上改回去，新建一个commit

git remote add origin git@github.com:michaelliao/learngit.git
git remote -v  # 查看远程库信息
git remote rm origin  # 删除远程origin

git clone --recurisive <address>   # 递归地下载，适用于仓库中包含submodule的情况

git pull origin main   # 拉取远程origin中main分支的改动，并merge
git pull origin main:bug   # 拉取远程origin中main分支到本地的bug分支中，并merge
git pull origin :bug   # 创建新的分支bug在本地

git push <remote> <place>   # 推送远程origin中place分支的改动
git push origin <source>:<destination>   # 将source本地分支推送到远程名为destination的分支，如果没有，会自动创建
git push origin :<destination>   # 删除远程分支destination
```

-   `HEAD`指向的版本就是当前版本，因此，Git允许我们在版本的历史之间穿梭，使用命令`git reset --hard commit_id`。 此外，可以使用`cat .git/HEAD` 查看`HEAD`的指向
-   穿梭前，用`git log`可以查看提交历史，以便确定要回退到哪个版本。
-   要重返未来，用`git reflog`查看命令历史，以便确定要回到未来的哪个版本

## 学习资料

[Commit message 和 Change log 编写指南](http://www.ruanyifeng.com/blog/2016/01/commit_message_change_log.html)
[Learn Git Branching](https://learngitbranching.js.org/?NODEMO=&locale=zh_CN)
[git flow备忘清单](http://danielkummer.github.io/git-flow-cheatsheet/index.zh_CN.html)
[gitignore templates](https://github.com/github/gitignore)
