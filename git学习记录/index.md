# git学习记录


----

<!-- more -->

### git常用命令

- 全局配置
  
  ```python
  # 配置用户名
  git config --global user.name "your_user_name"
  # 配置用户邮箱
  git config --global user.email "your_email_address"
  # 查看当前git状态
  git status
  # 查看git日志
  git log
  ```

- 版本控制
  
  ```python
  #　回退到上一个版本
  git reset --hard HEAD^ 
  #　回退到上上个版本
  git reset --hard HEAD^^
  #　查看相对日志，这个是相对于clone仓库之后的，就是说如果你从网上克隆一个仓库，那么reflog应该是空的
  git reflog
  # 查看所有日志，也包括克隆仓库的以前代码版本信息
  git log
  #　回退到任意版本
  git reset --hard 版本号
  ```

- 分支操作
  
  ```python
  # 查看分支
  git branch
  # 创建分支
  git branch name
  # 跳转分支
  git checkout name
  # 创建并跳转分支
  git checkout –b name
  # 合并某分支到当前分支
  git merge name
  # 删除分支
  git branch –d name
  # 重命名本地分支
  git branch -m oldName newName
  # 重命名本地分支后，如果想删除远程分支，那么
  git push --delete origin oldName
  ```
  
  [更多](https://blog.csdn.net/weixin_42152081/article/details/80558282)

### 创建git仓库的几种方式

- 通过本地空文件创建，使用以下命令
  
  ```
  git init
  ```

- 通过克隆远程仓库到本地，使用以下命令
  
  ```
  git clone your_repos_address
  ```

- 通过pull远程仓库到本地，使用以下命令
  
  ```
  git init
  git remote add origin your_repos_address
  git pull origin your_branch_name
  ```

### 提交本地仓库到远程仓库的方式

```
git add -A
git commit -m "your_commit_message"
git remote add origin your_repos_address
git push origin your_branch_name
```

- 如果push报错，可能需要先pull，在进行push
- 提交时注意要提交的分支以及本地当前所在的分支是否一致，若不一致，需要切换到对应分支

### 删除远程分支的方式

```
# 首先切换到要删除的分支，若本地没有对应分支，则创建
git checkout your_branch_name
git branch -r -d origin/branch-name
git push origin :branch-name
```

### 自动保存账户密码

有两种方式设置

- 第一种是设置当前git仓库的配置文件，这样只对当前仓库有效
  
  ```
  git config -e
  ```

- 第二种是设置全局git配置文件，对所有仓库均有效
  
  ```
  git config --global -e
  ```

无论哪一种，在打开的配置文件中，添加上以下命令

```
[credential]
    helper = store
```

保存后，重启终端，输入一次密码后就会自动保存，以后就不需要再输入密码啦～

### 常见错误记录

1. `RPC failed`
   
   如果在 `git commit` 时出现以下错误，可以尝试重新清空 git 全局配置
   `error: RPC failed; curl 56 GnuTLS recv error (-12): A TLS fatal alert has been received.`
   
   使用 `git config --global -e` 来查看全局配置文件
   使用 `git config -e` 来查看当前仓库的配置文件

### 2021.8.13之后不可以使用password执行git操作

大概意思是，原有的`git pull`或者`git push` 操作时，使用的`用户名+密码`的方式，改成了`用户名+token`，所以需要我们做一些改动

1. 生成token
   
   1. **Settings** => **Developer Settings** 
   
   2. **Personal Access Token** 
   
   3. **Generate New Token** 
   
   4. **Fillup the form**
   
   5. **Generate token**
   
   6. **Copy the generated Token**, it will be something like `ghp_sFhFsSHhTzMDreGRLjmks4Tzuzgthdvfsrta`

2. 配置git 自动缓存
   
   注意，这里如果使用 `cache` 的话，过段时间会自动忘记的，默认是15分钟，所以为了方便，可以存储密码到本地，将`cache`改为`store`，这个看自己选择了
   
   “store” 模式可以接受一个 `--file <path>` 参数，可以自定义存放密码的文件路径（默认是 `~/.git-credentials` ）
   
   “cache” 模式有 `--timeout <seconds>` 参数，可以设置后台进程的存活时间（默认是 “900”，也就是 15 分钟）
   
   ```bash
   $ git config --global user.name ""
   $ git config --global user.email ""
   $ git config -l
   $ git config --global credential.helper cache  # 设置自动缓存
   ```

3. 再次上传，第一次输入token

4. 后续git就会自己记住的
