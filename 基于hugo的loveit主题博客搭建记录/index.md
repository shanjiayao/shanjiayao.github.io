# 基于hugo的Loveit主题博客搭建记录



![tp.web.random_picture](https://images.unsplash.com/photo-1513875528452-39400945934d?crop=entropy&cs=tinysrgb&fit=crop&fm=jpg&h=1080&ixid=MnwxfDB8MXxyYW5kb218MHx8bGFuZHNjYXBlLHdhdGVyfHx8fHx8MTY0MjEyNTk2MQ&ixlib=rb-1.2.1&q=80&utm_campaign=api-credit&utm_medium=referral&utm_source=unsplash_source&w=1920)


出于对新鲜实物的好奇，我曾经探索过很多版本的主题样式，比如fluid以及butterfly，他们都是基于hexo框架的主题，其功能大多异常丰富，让初始者观之眼花缭乱。但使用过一段时间后，还是觉得自己需求的是一款简洁为主的主题，因此有了这款loveit主题的尝试，特此记录如下。

<!-- more -->

## 明确自己的需求喜好

既然想换主题，那么明确自己需求的主题样式很重要，所以我找了大量的博客以及网页推荐，主要渠道来源于以下几个方面：

- 知乎等平台的总结推荐
- github直接搜索主题字样
- 寻找一些关注度较高的博客网站， 查看他们的友链；或者是在诸如butterfly这种主题主页会有一些博客的展示

那么看过了大多数的主题之后，我越发确定自己想要的就是偏简洁风的样式。在这期间，我还尝试过使用Notion搭配一些软件或者是网站直接创建博客，但是相关的生态不是很多，仅有的几种方案列举如下：

- [nobelium](https://nobelium-xi.vercel.app/)  界面较为简洁, 使用vercel托管github的项目，可以与notion联动, 自动更新同步notion中的内容
- [Potion](https://potion.so/)  看起来不错，但是使用就需要付费
- [Super](https://super.so/)  使用较为简单, 可以自己定义很多css代码块, 以定义header navbar等，但是稍微高级一点的功能就需要付费

看起来唯一可行的就是nobelium这个项目，我也在少数派上发现了对应的文章教程，大概尝试了之后发现还是有很多bug需要改进的，所以最终不得不放弃了使用Notion这个想法~

放两个我觉得还算不错的博客主题: 

- https://geekplux.com/posts
- https://hugoloveit.com/posts/

所以初步选定loveit作为要配置的主题样式~


## 配置过程记录

### 下载hugo

因为loveit是基于hugo的，所以需要先下载，这里考虑到一些拓展功能，建议下载extend版本。

![](https://pictures-1309138036.cos.ap-nanjing.myqcloud.com/img/20220114110742.png)

```bash
sudo dpkg -i xxx.deb
```

### 使用hugo创建自己的博客文件夹

安装之后，在终端敲`hugo` 就可以有以下的输出了

```bash
> hugo
Error: Unable to locate config file or config directory. Perhaps you need to create a new site.
       Run `hugo help new` for details.

Total in 4 ms

```

接下来就是按照loveit的[主题教程](https://hugoloveit.com/theme-documentation-basics/)操作，使用以下命令新建博客文件夹

```bash
hugo new site my_website
cd my_website
```

### 安装主题

```bash
git clone https://github.com/dillonzq/LoveIt.git themes/LoveIt
```

关于主题的配置，自行参考教程即可。

### 创建博客

这部分在my_website目录下的content下面，可以自行创建，或使用hugo命令在终端创建。

```bash
hugo new posts/first_post.md
```

此外，我们还可以创建自己的专属内容，比如定义一个收藏页面，那就在content目录下创建一个collections的文件夹，再在其中创建一个index.md文件，这部分可以通过自定义manu bar的方式，将这个index对应的网页链接到导航栏。

我的改动如下：

改动config.toml，添加以下内容：

```toml
  [[menu.main]]
    identifier = "collections"
    pre = ""
    post = ""
    name = "收集"
    url = "/collections/"
    title = ""
    weight = 5
```

对应地，需要在content目录下创建collections文件夹，这样我们就完成了导航栏到自定义页面的链接。

综上，具体的config配置我就不记录太多了，各凭喜好~
