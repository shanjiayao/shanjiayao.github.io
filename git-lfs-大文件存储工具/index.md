# git lfs 大文件存储工具



## 使用场景

在Git仓库中，对于非文本文件，如各种多媒体文件，软件制品文件，二进制文件等等，这些文件往往体积比较大，使用Git直接管理会导致仓库的体积迅速膨胀，进而导致Git的许多操作变慢，同时也影响仓库上传到远程端。

![](https://pictures-1309138036.cos.ap-nanjing.myqcloud.com/img/143214_bc84b95c_551147.webp)

Git LFS相当于Git的一种插件式增强工具，简单讲，它是在Git仓库使用这些文件的 `指针`代替 `实际文件`，而把实际文件存储在远程端LFS服务器，同时在本地仓库中实时追踪这些文件的变动。

## 原理

根据 `Git LFS` 官方帮助文档描述：

Git LFS是基于Git的 [.gitattributs](https://gitee.com/link?target=http%3A%2F%2Fgit-scm.com%2Fbook%2Fzh%2Fv2%2F%25E8%2587%25AA%25E5%25AE%259A%25E4%25B9%2589-Git-Git-%25E5%25B1%259E%25E6%2580%25A7) 配置文件的特性，用 `smudge`过滤器基于 `指针文件`寻找大文件内容， 用 `clean`过滤器在对大文件改动时，创建指针文件的新版本。同时还用 `pre-push`钩子将大文件上传到Git LFS服务器， 即在 `git-push`时， 如果提交中包含被LFS跟踪的大文件，`pre-push`钩子会检测到，并执行上传Git LFS服务器的动作。

因此，如果一个仓库中包含LFS内容，但是在推送时不想推送这类文件，只要加上 `--no-verify`选项就行，即：

```bash
$ git push --no-verify
```


`--no-verify`选项告诉 `git push`完全跳过 `pre-push`钩子。

前面提到被LFS管理的文件，本地仓库中保存的内容实际上是指针文件，其格式类似于下面这样：

```bash
$ git show HEAD:2.svg
version https://git-lfs.github.com/spec/v1
oid sha256:158213f90f8b27012034c6f58db63e1861b12aa122d98910de311bf1cb1e50a0
size 14651
(END)
```

`version`表示LFS的版本，`oid`表示文件对象的唯一hash值，`size`表示文件的大小


## 安装git lfs

1. 先添加源到apt中
```bash
$ curl -s https://packagecloud.io/install/repositories/github/git-lfs/script.deb.sh | sudo bash
```

2. 终端会提示脚本运行的输出
```bash
Detected operating system as Ubuntu/bionic.
Checking for curl...
Detected curl...
Checking for gpg...
Detected gpg...
Running apt-get update... done.
Installing apt-transport-https... done.
Installing /etc/apt/sources.list.d/github_git-lfs.list...done.
Importing packagecloud gpg key... gpg: WARNING: unsafe ownership on homedir '/home/echo/.gnupg'
done.
Running apt-get update... done.

The repository is setup! You can now install packages.
```

3. 然后使用apt工具安装即可

```bash
sudo apt install git-lfs
```

## 使用流程

### Examples

在终端直接敲 `git lfs`，就会显示一些说明文档，如下

```bash
Examples
--------
To get started with Git LFS, the following commands can be used.
 1. Setup Git LFS on your system. You only have to do this once per
    repository per machine:
        git lfs install
        
 2. Choose the type of files you want to track, for examples all ISO
    images, with git lfs track:
        git lfs track "*.iso"
        
 3. The above stores this information in gitattributes(5) files, so
    that file needs to be added to the repository:
        git add .gitattributes

 4. Commit, push and work with the files normally:
        git add file.iso
        git commit -m "Add disk image"
        git push
```

总结来说，应该就是三板斧，init->track->add

需要注意的是，track的时候应该加**双引号**。

### 添加大文件并上传

#### 初始化 git 仓库
```bash
$ git init
Initialized empty Git repository in xxx
```

#### 初始化git lfs

```bash
$ git lfs install
Updated Git hooks.
Git LFS initialized.

Tips:
这个命令会自动改变Git配置文件 `.gitconfig`，而且是全局性质的，会自动在配置文件中增加如下配置：

[filter "lfs"]  
clean = git-lfs clean -- %f  
smudge = git-lfs smudge -- %f  
process = git-lfs filter-process  
required = true
```

#### 添加要跟踪的大文件类型

```bash
$ git lfs track "*.svg"
Tracking "*.svg"
$ git lfs track "*.png"
Tracking "*.png"

Tips:

这个命令会更改仓库中的 `.gitattributes`配置文件(如果之前不存在这个文件，则会自动新建):  
查看如下：  
$ cat .gitattributes  
*.svg filter=lfs diff=lfs merge=lfs -text  
*.png filter=lfs diff=lfs merge=lfs -text
```

#### 保存配置并commit

```bash
$ git add .gitattributes
$ git commit -m "add .gitattributes"
```

#### 上传到remote
上传的时候，直接使用 git push，就可以将lfs文件自动上传，而git实际跟踪的是文件的哈希值

```bash
$ git push -u origin main
Uploading LFS objects: 100% (10/10), 10 MB | 2.2 MB/s, done.                    
Counting objects: 34, done.
Delta compression using up to 12 threads.
Compressing objects: 100% (34/34), done.
Writing objects: 100% (34/34), 37.70 KiB | 2.22 MiB/s, done.
```


### 查看git lfs跟踪的文件列表

如下，前面的表示文件的哈希值oid，后面的表示实际的大文件名称路径。

```bash
$ git lfs ls-files
71890506b2 * model/big_dataset_384.png
fdcbf5e956 * model/big_dataset_384.svg
```

### 取消跟踪文件

```bash
git lfs untrack "xxx"  # 取消跟踪某个文件
git lfs uninstall  # 取消lfs的全局配置
```


### 使用LFS管理历史大文件

如果一个仓库中原来已经提交了一些大文件，此时即使运行 `git lfs track`也不会有效的。

为了将仓库中现存的大文件应用到LFS，需要用 `git lfs migrate`导入到LFS中：

```bash
$ git lfs migrate import --include-ref=master --include="biger.zip"
migrate: override changes in your working copy?  All uncommitted changes will be lost! [y/N] y
migrate: changes in your working copy will be overridden ...
migrate: Sorting commits: ..., done.
migrate: Rewriting commits: 100% (11/11), done.
  master        f9be3c554e9010ea5e0e23a6a0c6e53dca6c23b0 -> 53d5e655fe7cfd985f75384b92ac5414ad2ff394
migrate: Updating refs: ..., done.
migrate: checkout: ..., done.
```

> --include-ref 选项指定导入的分支
> 
> 如果向应用到所有分支，则使用--everything选项
> 
> --include 选项指定要导入的文件。可以使用通配符，批量导入。

上述操作会改写提交历史，如果不想改写历史，则使用 `--no-rewrite`选项，并提供新的commit信息：

```bash
$ git lfs migrate import --no-rewrite -m "lfs import"
```

将本地历史提交中的文件纳入到LFS管理后，如果重改了历史，再次推送代码时，需要使用强制推送。

这里选择改变提交历史，所以还需要使用 `--force`强制推送：

```bash
$ git push origin master --force
Enter passphrase for key '/home/git/.ssh/id_ed25519':
Locking support detected on remote "origin". Consider enabling it with:
  $ git config lfs.https://gitee.com/hightest/lfs-demo.git/info/lfs.locksverify true
Uploading LFS objects: 100% (8/8), 419 MB | 0 B/s, done.
Enumerating objects: 38, done.
Counting objects: 100% (38/38), done.
Delta compression using up to 6 threads
Compressing objects: 100% (37/37), done.
Writing objects: 100% (38/38), 136.26 MiB | 943.00 KiB/s, done.
Total 38 (delta 12), reused 10 (delta 0), pack-reused 0
remote: Powered by GITEE.COM [GNK-6.1]
To gitee.com:hightest/lfs-demo.git
 + cefd169...53d5e65 master -> master (forced update)
```

至此，已经将历史提交中的大文件迁移到远程LFS服务器，本地Git仓库，只保留这个大文件的指针文件，所以推送也不会再触发额度限制。推送成功之后，远程仓库就与本地保存一致，如场景一中的图示，它只管理这个大文件的指针文件。


## 参考

https://gitee.com/help/articles/4235#article-header3
