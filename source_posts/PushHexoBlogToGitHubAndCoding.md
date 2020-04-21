---
title: 如何将Hexo Blog同时发布到GitHub跟Coding上
date: 2016-04-25 01:22:44
tags:
- Hexo
- GitHub
- Coding
---
# 前言

上一篇文章已经讲过怎样使用Hexo建立博客以及怎样将Hexo博客发布到GitHub上.如果对Hexo建立博客还不熟悉请先看看之前的那篇文章[我的Hexo博客建站日志](http://pvphero.github.io/2016/04/24/我的Hexo博客建站日志/).大家可能跟我一样,建站的时候很轻松,但是等往github上提交的时候会等待很长时间,毕竟GitHub是国外的东西,很多地方会被墙掉,在我们现在不翻墙的情况下如何能将自己的博客迅速的托管到免费平台上呢?于是我想到了我比较喜欢保存自己项目的[Coding.net](https://coding.net/).一款国内的,强大的代码托管,项目演示平台.平台找到了,但又如何能不费劲一次就同时部署到[Coding.net](https://coding.net/)跟[GitHub](http://github.com)上呢?于是带着这两个问题,我查了很多资料,又经过一番折腾,终于实现了,一次部署同时发布.
<!-- more -->
![这里写图片描述](http://img.blog.csdn.net/20160425013150955)

# 在Coding上创建一个项目
## 准备工作
首先打开个账户的[个人设置](https://coding.net/user/account/setting/basic)中找到Global Key(个性后缀),然后新建一个Coding项目,项目名字跟Global Key相同.(大家也可以不这么建,可以直接建立项目,但是最后生成的页面会很长,显得不美观)
![这里写图片描述](http://img.blog.csdn.net/20160424234852399)
![这里写图片描述](http://img.blog.csdn.net/20160425013553308)

Tips:
- 最好创建跟Global Key相同的项目这样访问起来直接就是http://yourGlobalKey.coding.me. 比如说我的Coding的博客[CodingBlog](http://shenzhenwei.coding.me/),否则的话后面得加上项目名.
- 这里创建的是公有项目,为什么要创建公有项目,是因为如果项目弄成私有的,那么你的项目的pages页面就看不到里面的js效果了,就是只有文字的那种,主题什么的都白设置了.
- 如果项目已经设置成私有项目了并且还想看到效果,那只能用coding的演示功能了.只是coding功能是需要花费码币的,24小时0.01码币.
- 如果项目设置成公有的项目了,然后也部署成功了,在手机上打开的时候建议使用腾讯内核外的浏览器,否则的话可能会被当成恶意网站屏蔽掉.在pc上任何浏览器打开都是没问题的.
## 配置CodingGit的SSH
如果是第一次使用CodingGit提交的话,建议先配置SSH公匙.Coding生成公匙的方法可以查看[配置CodingSSH公钥](https://coding.net/help/doc/git/ssh-key.html).如果陌生可以按以下步骤来:
1. 打开个人中心的[SSH公匙](https://coding.net/user/account/setting/keys)
2. 如果之前配置过GitHub的公匙的话直接打开,.ssh文件夹里面的_rsa.pub
![这里写图片描述](http://img.blog.csdn.net/20160425001714644)
比如我的是pvphero_rsa.pub,然后将里面的内容全部复制,填写到ssh_rsa公匙处,公匙的名称可以随便起,然后点击'添加',再接着输入密码就可以完成添加了
![这里写图片描述](http://img.blog.csdn.net/20160425002144536)
3. 添加后测试一下
``` bash
ssh -T git@git.coding.net
```
如果出现下面的提示则表示公匙添加成功了:
``` 
Hello shenzhenwei You've connected to Coding.net by SSH successfully!
```
# 配置_config.yml的部署

准备工作都做好了,现在开始配置_config.yml,大家经过前面的文章[我的Hexo博客建站日志](http://pvphero.github.io/2016/04/24/我的Hexo博客建站日志/)相信对发布到GitHub上并不陌生,发布到GitHub上是在_config.yml文件中的deploy加上了GitHub的项目地址,以及发布的分支.那么要想同时发布到Coding上肯定是需要在配置文件中加上Coding的项目地址的,但是应该怎么加?格式又是如何呢?,根据[Hexo官方文档](https://hexo.io/zh-cn/docs/deployment.htmll)只需要将deploy的格式更改成下面的就可以了
``` 
deploy:
  type: git
  repo:
    github: <repository url>,[branch]
    coding: <repository url>,[branch]
```
比如我的是这样的:
```
deploy:
  type: git
  repo:
    github: git@github.com:pvphero/pvphero.github.io.git,master
    coding: git@git.coding.net:shenzhenwei/shenzhenwei.git,master 
```
# 部署Hexo博客
## 部署到GitHub跟Coding
- 前面的工作都做好了以后,生成静态网页
```
 $ hexo g
```
- 本地查看效果
```
$ hexo s
```
- 部署到git
```
$ sudo hexo d 
```
之后我们可以看到Coding跟GitHub中项目有我们提交上来的代码
![这里写图片描述](http://img.blog.csdn.net/20160425013918474)
![这里写图片描述](http://img.blog.csdn.net/20160425013942380)
并且GitHub上已经可以看到发布的内容[Github Blog](http://pvphero.github.io/)
![这里写图片描述](http://img.blog.csdn.net/20160425014004224)

## 设置Coding项目中的配置

**在Coding上部署博客有两种方式,前面提到过,在做下说明.Coding上部署博客总共有两种:**
1. 通过coding pages的方式进行博客的部署.coding为每个项目都推出了pages,不管是公有的还是私有的都有pages功能.我也比较推荐这种方式去搭建Hexo Coding博客.有很多好处,比如说免费,比如说可以绑定域名等等吧.
2.  通过Coding的演示功能进行Hexo Coding博客的部署.这种方式是收费的,每天最少0.01码币,大家可以体验体验.但不推荐.
**如果采用Pages方式的话就必须要在source/新建一个空白文件,名字必须是Staticfile**
```
 cd source
 touch Staticfile  #名字必须是Staticfile
```
因为用过coding演示功能的小伙伴都可能会知道,如果演示的时候没有Staticfile,coding的检测会提示检测不到,询问你是否强制开启.具体的原因的话,可能是coding是用静态的方式部署的,检测到这个的时候就知道你的项目是以静态方式发布的.
### 开启coding项目的pages功能
在刚刚建的项目中开启pages功能,这里的部署分支选择master,因为你在_config.yml中设置的分支是master,然后点击立即开启.
![这里写图片描述](http://img.blog.csdn.net/20160425014038443)
![这里写图片描述](http://img.blog.csdn.net/20160425014106287)
这时候如果点击链接出现404的话,并且本地测试是没有问题的,github上打开的链接也是没有问题的话,那么久需要耐心的多等几分钟了,这个coding.net部署的稍微慢点.coding的博客部署就ok了.这样就可以提交一次同时部署了~~

### 演示方式部署
关于演示方式部署,我就不费口舌了,因为演示方式部署肯定支持静态网页的,不管你是私有项目还是公有项目都是可以看到的.如果实在想去进行网站部署,建议大家参看[ 嘟嘟MD ](http://www.jianshu.com/p/7ad9d3cd4d6e)

**希望这篇文章对大家有所帮助~~我也是看了[ 嘟嘟MD ](http://www.jianshu.com/p/7ad9d3cd4d6e)的博客,然后跟着一步一步实现的~希望大家有问题多交流~**

[本文参与 Coding 征文计划](https://coding.net/wow/stories/)

