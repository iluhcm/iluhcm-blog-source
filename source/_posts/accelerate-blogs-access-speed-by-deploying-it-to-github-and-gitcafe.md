title: 将博客同时部署至Github和GitCafe加速访问站点
tags: [Blog,Github,GitCafe,Hexo]
date: 2015-09-15 15:52:14
categories: Course
---

# 起因

在github上建立了个人博客，发现用电脑访问已经缓存过的博客速度挺快，但用别的机子访问就惨不忍睹，迟迟加载不出来。其中缘由大家都懂的，github被特殊照顾也不是一天两天的事了。后来在Godaddy买了个人域名以后，用了DNSPod解析，做了CDN加速，效果还是寥寥。于是乎考虑看能不能在国内也建立一个博客镜像，局域网访问速度总比外网快吧！

其实这个想法也是偶然的。今天无聊刷着[V2EX](https://v2ex.com)的时候看到一篇文章[/t/220564](https://www.v2ex.com/t/220564)，大致就是调查有多少个人在用GitCafe。看到有用户评论说可以用来放博客。这是一家国内的创业公司，速度应该可以保证，所以我就萌生了想法，寻求解决方案。经过一番摸索，最终成功地实现了**国内的IP地址走GitCafe服务器，国外的IP地址走Github服务器**。原理也很简单，根据访问的IP进行DNS重定向就可以了，这个服务当然是由DNSPod来提供(免费的)。

# 生成博客镜像

本文重点不是怎么建立博客。我相信介绍博客搭建的博文随便一搜就有很多了，我的博客是用Hexo来搭建的，可以参考[这篇文章](http://www.jianshu.com/p/05289a4bc8b2)搭建起来。

## 配置GitCafe

假设你已经建好了博客并且可以通过`**.github.io`来访问了(\*\*是你的用户名)，那么下一步就是去GitCafe注册一个账号，配置SSH公共密钥(如果你愿意每次都输入密码的话则不用)，然后新建一个项目，项目名字就是你当前用户名(github新建个人主页是`**.github.io`)，参考[官方教程](https://gitcafe.com/GitCafe/Help/wiki/Pages-%E7%9B%B8%E5%85%B3%E5%B8%AE%E5%8A%A9#wiki)。

按照官方文档建立好项目以后，默认分支为`gitcafe-pages`分支，这样就可以通过`**.gitcafe.io`来访问你的博客了。

## 绑定自定义域名

GitCafe可以非常方便地绑定个人域名，在个人项目设置的`Pages服务`中，可以添加自定义域名（前提是你拥有这个域名的所有权），具体地址位于`https://gitcafe.com/**/**/pages_service`，根据官方文档有几个注意事项：

- 请正确填写你期望自定义的域名，填写时请不要填写`http://`这样的协议，当然如果你填写了，会智能的擦除这些字符。
- 最后在你的域名管理界面添加一个CNAME记录，将它指向GitCafe Pages服务器的domain：gitcafe.io。
- 目前 GitCafe Pages 已不支持 A 记录绑定自定义域名。

# 同步博客

这个时候我们就建立好了GitCafe博客镜像。那么问题来了，写博客一般是在本地写的，怎么方便地同步到Github和GitCafe上呢？又或者我还有第三个需要同步的地方(比如自己在VPS上建立了Gitlab)，怎么做同步呢？好在[Hexo](https://hexo.io/docs/deployment.html)给我们提供了非常简单的配置方案。修改Hexo目录下的站点配置文件`_config.yml`

```
# Deployment
## Docs: http://hexo.io/docs/deployment.html
deploy:
- type: git
  repository: git@github.com:**/**.github.io.git
  branch: master
- type: git
  repository: git@gitcafe.com:**/**.git
  branch: gitcafe-pages
```

把配置文件中的`**`改成你的用户名，这样，用Hexo生成文章并部署的时候，就会同时发布到Github和GitCafe上，达到同步博客的目的。

# 加速访问站点

如果需要根据IP来判断应该访问哪个站点，那么最好的解决方案就是修改DNS解析结果，根据IP记录解析到不同的站点。我使用的是国内的DNSPod，支持`CNAME`记录。下面是我的配置记录。

![DNSPod](http://7xl6ic.com1.z0.glb.clouddn.com/blog_dnspod.png)

**主要思想**就是将线路类型分为国内和国外，国内线路访问GitCafe，国外线路访问Github。添加了国内的记录以后，可以通过`ping`看看访问结果。由于目前在公司，不允许使用基于`PPP`协议的VPN，因此暂时没办法模拟国外访问站点的情况。

![Ping](http://7xl6ic.com1.z0.glb.clouddn.com/blog_ping.png)

实际测试了一下，访问速度确实有了提升，特别是使用3G网络的移动端，基本在1s左右就可以打开。

# 参考文章

<https://ruby-china.org/topics/18084>  
<http://www.v2ex.com/t/106866>  
<http://blog.devtang.com/blog/2014/06/02/use-gitcafe-to-host-blog/>  
<https://hexo.io/docs/deployment.html>