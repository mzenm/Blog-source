title: Hexo+github创建你自己的博客

author: 卫振家

categories: 

- 技术

tags: 

- github
- hexo
- blog

data: 2017-05-20 14:34:50

---

# HEXO+Github,搭建属于自己的博客

​	在搭建之前，大家首先要知道，在你的Github存入和运行的只有纯静态的html以及CSS，JS和资源程序。HEXO的最终仅仅是可以辅助你将对应的`MD(MARKDOWN)`文件处理成html文件，然后将这些生成的文件部署(Deploy)到Github上。

​	也就是说如果我们需要更新博客，并不是直接在GITHUB上处理静态页面，而是首先在在本地撰写markdown文件，在本地将markdown 文件放入指定的目录中(一般是_post)中，而后启动HEXO 的generate生成MarkDown对应的HTML文件。生成HTML文件后使用HEXO的server命令在搭建本地运行环境localhost:4000，查看预览效果。确认效果后，使用HEXO的deploy命令发布到Github。



[TOC]

## 创建仓库

​	首先你要做的是前往[github](https://github.com/join?source=header-home)创建账号,流程很简单。按照页面指示完成就可以了。

​	![创建GIT账号](http://oq3ecl7n4.bkt.clouddn.com/github_account_create.png)

​	创建账户完成后，你就可以前往GITHUB首页进行对应库的创建。

​	![创建你的库](http://oq3ecl7n4.bkt.clouddn.com/github_new_rep.png)

​	点击New repository 创建新库，注意Repository name 必须是 用户名`.github.io`。就是`$Owner.github.io`

​	![创建新库](http://oq3ecl7n4.bkt.clouddn.com/creta_io_rep.png)

​	创建完成后打开你库名对应的链接，我的就是**http://mzenm.github.io**，你替换成你自己的链接就好了。你可以看到网站已经有了，但是没有内容。现在就需要进行HEXO来大展伸手。但是在安装HEXO 之前还需要进行一些操作。



## 准备工作

1. 软件安装和图床

   必须要安装[git](https://git-scm.com/downloads).前往git下载网站，下载对应的Git版本，直接安装即可。

   必须安装[node](https://nodejs.org/en/) ，请前往node网站，下载稳定版本node安装即可。

   最好注册[七牛云存储](https://portal.qiniu.com/signin)，个人账号可以有免费的500M空间来存入一些图片文件或者其他。暂时我正在使用七牛云作为 图床。

2. 设置Github `SSH`支持

   ​	安装好了git之后，我们就可以在任意文件目录右键git bash 打开bash窗口，bash命令基本与linux下完全相同。首先我们要做的就是添加SSH支持，不用在想Github push或pull时需要额外的输出账号和密码。

   1. 创建全局变量用户名和用户邮箱

   ​	![](http://oq3ecl7n4.bkt.clouddn.com/set_github_user.png)

   ​	账号就是你的github owner ，邮箱就是你的github邮箱。

   2. 生成SSH密钥

      1. .查看是否已经有了ssh密钥：

         ```bash
         $ cd ~/.ssh
         ```

         如果没有密钥则不会有此文件夹，有则备份删除.

      2. 生成密钥：

         ```sh
         $ ssh-keygen -t rsa -C “your.github.account@mail.com”
         ```

         按3个回车，密码为空。

         ```shell
         //结果
         Your identification has been saved in ~/.ssh/id_rsa.
         Your public key has been saved in ~/.ssh/id_rsa.pub.
         The key fingerprint is:
         ………………
         ```

         ```sh
         #查看
         $ cd ~/.ssh && ls
         ```

         最后得到了两个文件：id_rsa和id_rsa.pub

      3. 检查ssh agent是否可用

         ```shell
         $ eval "$(ssh-agent -s)"
         ```

         显示

         ```shell
         Agent pid 13772
         ```

         可用

      4. 添加生成的 SSH key 到 ssh-agent。

         ```shell
         $ ssh-add ~/.ssh/id_rsa
         ```

      5. 在Github中添加公钥

         ![添加ssh](http://oq3ecl7n4.bkt.clouddn.com/add_github_ssh.png)

         ![](http://oq3ecl7n4.bkt.clouddn.com/add_new_key.png)

         点击 Add SSH key 添加成功

      6. 测试SSH可用

         ```shell
          $ ssh git@github.com
         ```

         这句话出现就表示成功了

         ```shell
         Hi mzenm! You've successfully authenticated, but GitHub does not provide shell access.
         ```

   3. 前往[hexo](https://hexo.io/)官方网站，准备安装HEXO

      ![hexo](http://oq3ecl7n4.bkt.clouddn.com/hexo_page.png)

      官方快速指引,如果以下任何命令出现问题，请重新按照官方标准指引操作

      ![](http://oq3ecl7n4.bkt.clouddn.com/hexo_start.png)

## 正式安装HEXO

1. 安装hexo

   ​	任意目录打开git bash运行

   ```shell
   $  npm install hexo-cli -g
   ```

2. 初始化博客目录

   ​	前往你想要使用的任意目录，使用

   ```shell
   $ hexo init 你想要的文件夹名称

   #以下是我的命令
   $ cd /e
   $ hexo init mzenm
   $ cd mzenm

   ```

   ​	创建一个标准的hexo目录，稍后的所有操作，都由此目录开始。然后使用

   ```shell
   $ hexo generate
   #或
   $ hexo g
   ```

   命令，该命令的作用是将你自己写好的markdown文件，生成为标准的html文件。

   ​	html生成成功后，使用

   ```shell
   $ hexo serve
   #或
   $ hexo s
   ```

   命令，启动一个本地的web服务器localhost:4000来查看这些生成的博客页面。

   ![默认页面](http://oq3ecl7n4.bkt.clouddn.com/hexo_my_page.png)

   这就是默认生成的html页面。默认主题是landscape，我们可以去官网选择自己喜欢的主题安装。

3. 更换主题

   ​	我们前往官网推荐的主题页面[theme](https://hexo.io/themes/),选择自己喜欢的主体，点击进入其github主页面。在github主页的README信息中一般都会告诉你如何安装一个新主题。在这里我选择[Anatole](https://github.com/Ben02/hexo-theme-Anatole)，[预览页面](http://anatole.munen.cc/)。

   ​	前往我们本地的bolg文件夹，就是刚才我们Init创建的那个mzenm目录，运行命令

   ```shell
   $ cd /e/mzenm
   $ git clone https://github.com/Ben02/hexo-theme-Anatole.git themes/anatole
   ```

   使用git clone对主题文件到/e/mzenm/themes/anatole，如果想要更新主题，直接到anatole目录下运行

   ```shell
   $ git pull
   ```

   即可。

   > 很多主题的配置需要其他插件的支持，这种情况下，请根据主题作者在Github中的描述自行安装。
   >
   > Anatole 主题配置 [Document](https://github.com/Ben02/hexo-theme-Anatole/wiki/Installation)

   ​	根据文档信息配置完主题后，我们现在就要更换主题了。更换主题很简单，打开/e/mzenm/_config.yml文件 编辑Extension单元中的theme项为你clone的问题的目标文件名称 即themes目录下的文件夹名称,anatole，然后使用

   ```shell
   $ hexo g
   ```

   重新生成。生成的页面可以使用

   ```shell
   $ hexo s
   ```

   本地预览。

4. 添加后台插件

   ​	前往Hexo官网[插件中心](https://hexo.io/plugins/)，选择适合的插件进行安装后，进行操作，这里我选择安装[hexo-admin](https://github.com/jaredly/hexo-admin)这样就可以在本地后台直接编辑blog了，相当于给网站增加了一个后台功能。

   ​	进入mzenm目录，打开git bash运行

   ```shell
   $ npm install --save hexo-admin
   $ hexo server -d
   ```

   程序将自动安装hexo-admin插件，deploy到本地网站并打开本地网站的后台页面,打开[管理后台]( http://localhost:4000/admin/).剩下你们懂得。

   > 如果未设置deploy部署的目标路径，deploy将自动部署到本地。稍后我们会学习如何部署一个hexo到我们的github博客上去。

   ​

## 让我们开始写博客吧

1. 如何开始写hexo博客呢？方式有多种多样:

   - hexo_admin自行编辑
   - hexo生成文件后使用markdown文件编辑器
   - 使用markdown编辑器后粘贴到hexo_admin后台

2. 使用hexo_admin编辑

   ![](http://oq3ecl7n4.bkt.clouddn.com/hexo_admin_edit.jpg)

   因为我们已经使用了hexo_admin插件，我们可以直接前往[管理后台]( http://localhost:4000/admin/)管理页面，当然前提是server必须是开启状态的。

3. 使用hexo生成文件

   ```shell
   $ hexo new "新博客名"
   # 成功后返回以下信息
   INFO  Created: E:\mzenm\source\_posts\如何建立一个hexo博客.md
   ```

   然后使用[typora](https://www.typora.io/)或其他软件来编辑改文件，美观好用，支持多平台，就是文件管理做的很一般。

4. markdown基本的书写规范

   [入门教程](http://www.jianshu.com/p/1e402922ee32/)

   [详细文档](http://www.appinn.com/markdown/)

   Tips 好事情是大部分markdown软件都支持右键插入

   >`#`标示标题，几个#号就是几个标题
   >
   >`- `或`* `标示无序列表
   >
   >`1. `标示有序列表
   >
   >```java
   >​```java
   >​```
   >```
   >
   >以上形式标示java代码块，其他代码块只需要将java换成php,shell之类的即可。
   >
   >`**ss**` 包裹标示加粗，`*`标示倾斜，``包裹标示主要提示。
   >
   >- [ ] 使用`- [ ]`标示，但必须放在头部 
   >
   >`[连接名称](link)`标示超链接
   >
   >`![图片名](图片链接)`标示图片
   >
   >`>`标示引用
   >
   >`---`标示的水平线
   >
   >`|_`先是组成表的一部分，请自行探索吧
   >
   >使用`<!-- more -->`可以提取前面的信息为提要
   >
   >使用`ctrl + /`在有些软件可以查看markdown的源码
   >
   >`$$ $$`双$$包裹的是数学公式。
   >
   >`--- --- `包裹的信息放在文档开始作为配置文件出现。

   ​

## 终于到发布了

​	经过这么长时间的努力，我们的Mark文件都写好了，本地的样式看上也是美美哒，应该没有问题了吧。那么现在我们要做的就是发布到线上去。

```shell
$ hexo deploy
#或
$ hexo d
```

使用这个命令就可以将hexo发布到github上，前提你配置好了你的发布信息。

​	打开你本地博客文件的根目录，打开_config.yml文件就可以看到一下代码

```yml

# Writing
new_post_name: :title.md # File name of new posts
default_layout: post
titlecase: false # Transform title into titlecase
external_link: true # Open external links in new tab
filename_case: 0
render_drafts: false
post_asset_folder: false
relative_link: false
future: true
highlight:
  enable: true
  line_number: true
  auto_detect: false
  tab_replace:

# Category & Tag
default_category: uncategorized
category_map:
tag_map:

# Date / Time format
## Hexo uses Moment.js to parse and display date
## You can customize the date format as defined in
## http://momentjs.com/docs/#/displaying/format/
date_format: YYYY-MM-DD
time_format: HH:mm:ss

# Pagination
## Set per_page to 0 to disable pagination
per_page: 10
pagination_dir: page

# Extensions
## Plugins: https://hexo.io/plugins/
## Themes: https://hexo.io/themes/
theme: anatole

# Archive
archive_generator:
  per_page: 0
  yearly: false
  monthly: false
  daily: false

# Deployment
## Docs: https://hexo.io/docs/deployment.html
deploy:
  type:
```

其中Deployment就是配置发布到github的，但是在此之前我们先要安装一个hexo插件来帮助我们发布

```shell
$ npm install hexo-deployer-git --save
```

从插件的命名我们就可以知道，这个插件本身就是用来发布到git的。

现在我们修改#Deployment

```yml
# Deployment
## Docs: https://hexo.io/docs/deployment.html
deploy:
  type: git
  repo:
      github: git@github.com:mzenm/mzenm.github.io.git,master
```

这种写法的好处是我们可以同时发布我们的网站到多个git库，比如github或者国内的git。当然这里还有其他人提供的解决方案

```yml
deploy:
     type: git
     repo: https://github.com/leopardpan/leopardpan.github.io.git
     branch: master
```

这样写结构更加清晰，但对多网站的发布支持不好。



保存yum文件。让我们开始发布。

如果我们刚刚更换过主题，最好清理原有的生成文件，然后在重新生成，预览后发布。

```shell
$ hexo clean
$ hexo generate
$ hexo serve
```

查看本地的blog样式和内容 [本地路径](http://localhost:4000),如果没有问题，就是用`crtl+c`命令关闭本地然后，

```shell
$ hexo deploy
```

发布到Github库。

当然 如果你确定真的没有问题，我们可以在保持现有server的状态下，重新打开一个git bash,使用

```shell
$ hexo g && hexo d
```

直接发布到网站。现在去你的网站看看吧，哈哈。







