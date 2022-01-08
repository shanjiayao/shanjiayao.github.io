# Gitbook使用笔记


---

<!-- more -->

## Gitbook简介

`gitbook`是`github`与`markdown`的结合体，我们可以通过`markdown`笔记记录书籍内容，然后发布到`gitbook`上，使用操作和流程与`github`类似，详细了解见[官网](https://www.gitbook.com/)以及[Git](https://github.com/GitbookIO/gitbook)

## Gitbook安装及使用

### 1. 安装

```
npm install -g gitbook-cli
gitbook init    # then system will download gitbook automatically
```

[详见](https://github.com/GitbookIO/gitbook/blob/master/docs/setup.md)

### 2. 创建书籍

- **初始化一个`Gitbook`**
  
  `cd`到要创建书籍的目录下，比如 `~/books`，然后运行
  
  ```shell
  $ gitbook init
  ```
  
  终端会输出以下命令，并创建两个文件
  
  ```shell
  warn no summary file in this book 
  info: create README.md 
  info: create SUMMARY.md 
  info: initialization is finished
  ```

- **编写`README.md`和`SUMMARY.md`**
  
  这时文件夹下的目录为
  
  ```shell
  $ ls
  README.md SUMMARY.md
  ```
  
  `README.md`是对创建书籍的介绍，这里面的内容会在整个`gitbook`的简介中出现
  
  `SUMMARY.md`是整个书籍的目录结构，这个文件的编写需要按照对应的格式，以标题的形式，每一个标题对应一个`.md`文件的路径，类似这样
  
  ```markdown
  # Summary
  
  * [Introduction](README.md)
  
  -----
  * [Detection](detection/README.md)
      * [PointPillars](detection/pointpillars.md)
  * [Tracking](tracking/README.md)
      * [SC3D](tracking/sc3d.md)
  * [Segmentation](seg/README.md)
      * [SOLO](seg/solo.md)
  * [PointCloud](pointcloud/README.md)
      * [PV-CNN](pointcloud/pvcnn.md)
  ```
  
  我们可以按照自己对书籍的设想，编写对应的目录大纲
  
  当做完更改之后，再次执行`git init`，就会根据`SUMMARY.md`中编写的目录自动生成对应文件

- 编译
  
  仍然在`~/books`目录下，执行
  
  ```shell
  $ gitbook build
  ----------------------------------------------------
  info: 7 plugins are installed 
  info: 6 explicitly listed 
  info: loading plugin "highlight"... OK 
  info: loading plugin "search"... OK 
  info: loading plugin "lunr"... OK 
  info: loading plugin "sharing"... OK 
  info: loading plugin "fontsettings"... OK 
  info: loading plugin "theme-default"... OK 
  info: found 9 pages 
  info: found 0 asset files 
  info: >> generation finished with success in 0.5s !
  ```
  
  此时目录下会生成对应的书籍编译文件`_book`
  
  ```shell
  $ tree . -L 2
  -------------------
  .
  ├── _book
  │   ├── detection
  │   ├── gitbook
  │   ├── index.html
  │   ├── pointcloud
  │   ├── search_index.json
  │   ├── seg
  │   └── tracking
  ├── detection
  │   ├── pointpillars.md
  │   └── README.md
  ├── pointcloud
  │   ├── pvcnn.md
  │   └── README.md
  ├── README.md
  ├── seg
  │   ├── README.md
  │   └── solo.md
  ├── SUMMARY.md
  └── tracking
      ├── README.md
      └── sc3d.md
  
  10 directories, 12 files
  ```
  
  编译得到的`_book`目录就是对已有的书籍内容做了转换，变成了`html`网页文件，如果你想将`gitbook`链接到个人博客，只需要上传这个目录下的文件即可，具体操作见下文

- 本地运行
  
  仍然在`~/books`目录下，执行
  
  ```shell
  $ gitbook serve    # 这个命令会自动执行 gitbook build命令
  ---------------------------------
  Live reload server started on port: 35729
  Press CTRL+C to quit ...
  
  info: 7 plugins are installed 
  info: loading plugin "livereload"... OK 
  info: loading plugin "highlight"... OK 
  info: loading plugin "search"... OK 
  info: loading plugin "lunr"... OK 
  info: loading plugin "sharing"... OK 
  info: loading plugin "fontsettings"... OK 
  info: loading plugin "theme-default"... OK 
  info: found 9 pages 
  info: found 0 asset files 
  info: >> generation finished with success in 0.6s ! 
  
  Starting server ...
  Serving book on http://localhost:4000
  ```
  
  这时打开浏览器，输入`http://localhost:4000`，即可查看本地版的`gitbook`
  
  ![show](how-to-use-gitbooks/3a20712b8d287db8641417a818b67b670f864e77.png)

OK，到这里关于`gitbook`的简单使用就介绍完了，下面记录下如何在个人主页上使用`gitbook`

## 将GitBook链接到个人博客

简单记录下

个人使用的是 `hexo` 框架以及 `fluid` 主题搭建的个人博客

使用`gitbook`的初衷就是，我想在个人博客上链接一个地址，这个网页记录一些比如力扣刷题笔记等自己学习的内容，所以就简单了解了一下`gitbook`

OK，那现在我默认已经拥有了自己的博客，并且已经按照上述流程初步体验了`gitbook`的使用，并且拥有了自己的第一个`gitbook`，如果你想把他链接到个人博客上，应该分以下三步：

1. 将`gitbook`原始内容上传至`github`远程仓库
   
   这里指的是自己编写的markdown笔记等
   
   在`github`新建一个仓库，这个仓库的名字会是你博客跳转到`gitbook`网页的后缀
   
   在`gitbook`本地，初始化仓库，远程仓库设定为刚刚建立的仓库地址
   
   新建`.gitignore`文件，里面的内容设置为
   
   ```git
   *~
   _book
   ```
   
   上传当前文件到远程仓库的`main`分支

2. 编译`gitbook`，将生成的`_book`中的内容上传到新的分支，叫做`gh-pages`
   
   ```
   gitbook build
   mkdir book_build
   cp -r _book/* book_build
   cd book_build
   git init
   git remote add origin xxx
   git checkout -b gh-pages
   git add .
   git commit -m "first pub"
   git push origin gh-pages
   ```
   
   这样就将编译得到的`_book`下面的静态网页文件上传到了远程仓库的`gh-pages`分支下
   
   登录`github`，找到仓库->`setting`->`options`->`Github pages`，设置`gh-pages`为要展示的分支，保存，Over！

3. 设置个人博客中，对应的链接为 `your-blog-address/xxx/`，其中`xxx`既是远程仓库的名字，也是你的`gitbook`的名字

结果展示 [leetcode· GitBook](https://shmilywh.github.io/leetcode/)
