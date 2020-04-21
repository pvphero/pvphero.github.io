---
title: 我的Hexo博客建站日志
date: 2016-04-24 03:01:23
tags:
- Hexo
- Mac
- Git
- Blog
categories:
- Hexo
---




# 前言
我是一枚安卓猿,js css以及基于Nodejs的Hexo对我来说完全是个新领域.由于之前查资料查到看到一个人的博客很不错[MOxFIVE](http://moxfive.xyz/),于是就决定模仿人家的博客进行搭建自己的Hexo博客.程序员就是爱折腾.于是查了很多资料,最终实现了自己的Hexo博客.下面我就把一个完全不懂nodejs以及之前根本没听说过Hexo的人建站的历程给大家分享下.

我是在Mac环境下搭建的,Win环境跟Mac的差不多,如果大家是win环境,我建议大家参看[粉丝日志](http://blog.fens.me/hexo-blog-github/)

<!-- more -->
# Hexo 介绍
[Hexo](https://hexo.io) 是一个简单的,轻量级,基于Nodejs的一个静态的博客框架.我们可以很方便的使用Hexo搭建个人博客.

对一个外行来说,在一开始使用Hexo的时候心理难免有很多疑问.那我来说,我刚接触Hexo的时候我一直在想:Hexo到底是什么?他是如何把网站部署到github或者conding上的?Hexo怎么进行博客内容的编辑啊?于是经过几番折腾,我对Hexo有了初步的了解.希望我的理解对大家刚刚搭建有所帮助吧.
- 首先,Hexo是一个基于Nodejs的框架,我们通过Hexo里面的命令在Vim上进行hexo初始化,hexo生成index.html静态页面,然后再通过命令发布到github或者conding
- Hexo里面集成有git的插件,所以你发布到你的 yourname.github.io的时候只需要把_config.yml中的配置配置好.然后使用
``` bash
$ hexo g
$ hexo d
```
就可以将通过Hexo编译好的文件推送到git项目中

* hexo里面的内容怎么编辑?hexo里面的文件是以md结尾的.是markdown语法.所以大家编辑的时候可以先看看[markdown语法](http://www.cnblogs.com/hnrainll/p/3514637.html).我这边是使用的mac的Mou一款免费的markdown软件.大家也可以使用phpStorm,里面在plugins中可以下载markdown的插件.建议大家下载个PHPStorm IDE,因为将来涉及到图片更换,主题更换,目录结构的查看等等,都会比较直观.

# 配置环境

1. 安装Node (必须) 作用是用来生成静态页面.到[Node.js](https://nodejs.org/en/)官网上下载相应平台的最新版本,一路安装即可,没有难度...此处略过!但是一定要装啊.

1. 安装Git (必须) 作用是把本地hexo内容提交到git上去,安装xcode就自带Git了,就不多说了~

1. 申请[Github](https://github.com/)(想同时发布到coding上的,可以再申请个[coding账号](https://coding.net/),后面会讲到)

1. 配置github SSH keys(可以不配置,不配置的话就的每次提交的时候手动输入账号密码了,如果配置了就不需要了,就会很方便)[GitHub-Help-SSH配置](https://help.github.com/articles/adding-a-new-ssh-key-to-your-github-account/)

# Hexo的安装
系统环境

**Mac OS**

**Hexo安装,要用全局安装,加参数-g**
     
    $ sudo npm install-g hexo  
 
  
*查看Hexo的版本*

    $ hexo version
    hexo-cli: 1.0.1
    os: Darwin 14.5.0 darwin x64
    http_parser: 2.5.2
    node: 4.4.3
    v8: 4.5.103.35
    uv: 1.8.0
    zlib: 1.2.8
    ares: 1.10.1-DEV
    icu: 56.1
    modules: 46
    openssl: 1.0.2g 
    
或者
    
    $ hexo v
    hexo-cli: 1.0.1
    os: Darwin 14.5.0 darwin x64
    http_parser: 2.5.2
    node: 4.4.3
    v8: 4.5.103.35
    uv: 1.8.0
    zlib: 1.2.8
    ares: 1.10.1-DEV
    icu: 56.1
    modules: 46
    openssl: 1.0.2g 
这个时候可能会出现

    { [Error: Cannot find module './build/Release/DTraceProviderBindings'] code: 'MODULE_NOT_FOUND' }

解决方法
    
    $ npm install hexo --no-optional
参考[Hexo常见问题解决方案](http://www.baidu.com/link?url=Zhut7pph6xKKmF5ADMlGTEDpN5NmgV8vahlW1EqcN3uR7KVFojiykBccQSqcw7CwAzJX0x2bSLvKie5SDyYVMUhl29NcV6LGDLWo7CyUE3816hITneb5IAWZUfqgPVDrA69MqdDs__GS1R1VIk-xi2vqkaeJ27OIvxHtSF-eXZ3&wd=&eqid=ec1ccb24001e820300000004571b8f14)

**安装好以后我们就可以使用Hexo创建项目了**

    $ hexo init nodejs-blog

然后我们打开Finder发现,刚刚的路径下多了一个文件夹,并且文件夹有Hexo相应的初始化模块.

**初始化完成后进入刚刚创建的文件夹,并启动服务器,看看效果**

*进入刚刚创建的文件夹*

    $ cd nodejs-blog/
    $ ls -a
    .   .gitignore	_config.yml   package.json  source
    ..  .npmignore	node_modules  scaffolds     themes
    
*启动hexo服务器查看效果*

    $ hexo s
    INFO  Start processing
    INFO  Hexo is running at http://0.0.0.0:4000/. Press Ctrl+C to stop.
 然后通过浏览器访问 http://0.0.0.0:4000/查看效果
 
 ![img](http://img.blog.csdn.net/20160424121528685)
 
# Hexo 目录结构及用法

**4.1目录结构**

![img](/source/img/project_jiegou.png "jiegou")

1. scaffolds 脚手架，也就是一个工具模板

1. source 存放博客正文内容 比如说我们新建一个page 那么改page就会出现在这个文件中,我们如果想对原先的博客进行修改的话,就打开这个文件夹下相应的文件进行编辑.如果是想删除的话,那就直接删除该文件夹下相应的文件

1. themes 存放皮肤的目录 用户可以根据自己的喜好更换_config.yml中的themes

1. _config.yml全局的配置文件

1. db.json 静态常量

**4.2全局变量_config.yml的配置**

* 站点信息: 定义标题，作者，语言
* URL: URL访问路径
* 文件目录: 正文的存储目录
* 写博客配置：文章标题，文章类型，外部链接等
* 目录和标签：默认分类，分类图，标签图
* 归档设置：归档的类型
* 服务器设置：IP，访问端口，日志输出
* 时间和日期格式： 时间显示格式，日期显示格式
* 分页设置：每页显示数量
* 评论：外挂的Disqus评论系统
* 插件和皮肤：换皮肤，安装插件
* Markdown语言：markdown的标准
* CSS的stylus格式：是否允许压缩
* 部署配置：github发布

编辑_config.yml文件
 
    $ vim _config.yml 
    
    
**4.3创建新的文章**

    $ hexo new 我的第一篇Hexo博客
    
启动服务器
    
![img](http://img.blog.csdn.net/20160424121706439)

**4.4文章语法**

[Markdown语法](http://www.cnblogs.com/hnrainll/p/3514637.html)

[怎样引导新手使用 Markdown？](https://www.zhihu.com/question/20409634)

    实例:
    title: 新的开始
    date: 2014-05-07 18:44:12
    permalink: abc
    tags:
    - 开始
    - 我
    - 日记
    categories:
    - 日志
    - 第一天
    ---
    
    这是**新的开始**，我用hexo创建了第一篇文章。
    
    通过下面的命令，就可以创建新文章
    
    ```{bash}
    $ hexo new 我的第一篇Hexo博客
        
    ```
    感觉非常好。
# 发布到Github上
**5.1生成静态的index.thml文件**

    $ hexo g
    
**5.2创建githb pages**

在Github上创建一个项目 username.github.io  比如我的用户名是pvphero 那么我创建的项目的名字就是pvphero.github.io [pvphero's GitHub](https://github.com/pvphero/pvphero.github.io)

参考[GitHubPages](https://pages.github.com/)

然后可能有人觉得访问github过慢,有什么好的解决方法么?

github访问慢的原因是因为CDN: github.global.ssl.fastly.net,被墙了
![这里写图片描述](http://img.blog.csdn.net/20160424121935833)

解决方法

* 我这边的的解决方法是更改github的hosts[快速更改MacHosts](http://pan.baidu.com/s/1c1VePTe)
![这里写图片描述](http://img.blog.csdn.net/20160424122438761)
* 然后访问[IPAddress.com](http://www.ipaddress.com/)根据域名找到本来的IP
![这里写图片描述](http://img.blog.csdn.net/20160424122631064)
* 更改Mac的hosts
![这里写图片描述](http://img.blog.csdn.net/20160424122726718)
**5.3发布到github上**

编辑全局配置文件_config.yml，找到deploy的部分，设置github的项目地址

    deploy:
    type: git
    repo:
      github:git@github.com:pvpheropvphero.github.io.git,master
      
接下来使用hexo命令部署

    $ hexo d
 如果出现
 
 ```bash
 { [Error: Cannot find module './build/Release/DTraceProviderBindings'] code: 'MODULE_NOT_FOUND' }
{ [Error: Cannot find module './build/default/DTraceProviderBindings'] code: 'MODULE_NOT_FOUND' }
{ [Error: Cannot find module './build/Debug/DTraceProviderBindings'] code: 'MODULE_NOT_FOUND' }
```

这样的错误

输入命令行

    $ npm install hexo --no-optional
    
[Hexo常见问题解决方案](http://wp.huangshiyang.com/hexo%E5%B8%B8%E8%A7%81%E9%97%AE%E9%A2%98%E8%A7%A3%E5%86%B3%E6%96%B9%E6%A1%88)

这样自己的博客就部署到github上了.可以通过  username.github.io查看最后的效果


# 更换主题

**6.1找一个皮肤**

比如我用的皮肤是[spfk](https://github.com/luuman/hexo-theme-spfk),下载下来后直接复制到/nodejs-blog/themes/里面

**6.2_config.yml中设置皮肤**

编辑文件_config.yml，找到theme一行，改成 theme: pacman  

整个Hexo搭建个人博客就结束了.当然大家还可以个性化设置自己的博客主题,以及加载更多的Hexo插件.下一篇将会讲如何将博客同时部署到github上跟coding上.敬请期待~


[CodingBlog](http://shenzhenwei.coding.me/)

[GitHubBlog](http://pvphero.github.io/)


