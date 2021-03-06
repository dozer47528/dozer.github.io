---
title: 利用 DNSPod 解决 Godaddy 域名在国内无法解析的问题
author: Dozer
layout: post
permalink: /2012/03/using-dnspod-solve-the-problem-of-godaddy-domain-can-not-resolving-in-china.html
categories:
  - 互联网
tags:
---

### 我们是共产主义的接班人

“我们是共产主义的接班人，继承革命先烈的光荣传统…”

我的域名非常荣幸地被部分服务商墙了。

当我在学校里用 ChinaUnicom 上网的时候，我的网站一直是无法打开的，一直以为是偶然的问题，因为我的虚拟主机供应商不太稳定。

后来也有网友给我反映相同的问题，这时候我才意识到问题的严重性。

还好中国电信的可以上，所以没有影响到大部分网友的访问。

<!--more-->

### 出了什么问题

搜索后发现，其实有大量的网友遇到了这个问题。<a href="http://www.google.com/search?q=godaddy+域名+无法解析" target="_blank"><strong>传送门</strong></a>

可见这不是偶然的问题了，本来以为只有主机IP会被墙，所以在国外买了域名，主机就老老实实放国内了，还备了案。

其实这个问题的根本原因就是因为 Godaddy 的 NameServer 经常会被墙，所以导致域名无法解析。

[<img class="alignnone size-medium wp-image-689" title="ping" alt="ping" src="/uploads/2012/03/ping-300x193.png" width="300" height="193" />][1]

Godaddy 的解析速度在国外应该是快的惊人的，它的 NameServer 都是 NS[数字].DOMAINCONTROL.COM 形式。

从上图可以看到，NS69.DOMAINCONTROL.COM 这个 NameServer 已经被墙了，而上面那个 NS63.DOMAINCONTROL.COM 可以上。

其实，这里的屏蔽也不是针对我网站的，我可是一等良民啊，但是 Godaddy 树大招风，上面有大量不和谐的网站和域名，所以就悲剧了…

&nbsp;

### 如何解决

虽然手动换个 Godaddy 的 NameServer 就可以了，但是谁知道它什么时候又会被屏蔽呢？

所以干脆把 NameServer 迁移到国内吧，毕竟我这也没什么不和谐的东西，不怕审查，而且也备案过了。

&nbsp;

### DNSPod

网上搜索后知道，<a href="https://www.dnspod.cn/" target="_blank"><strong>DNSPod</strong></a> 是国内最牛的域名解析服务商，而且有个人永久免费套餐，提供的服务不比 Godaddy 差。

*（注意，切换 NameServer 并不是转移域名，域名还是在 Godaddy 那续费，管理，只不过是不使用 Godaddy 的域名解析服务了，域名还是放在国外好！）*

具体怎么迁移我就不介绍了，<a href="https://www.dnspod.cn/Support" target="_blank"><strong>官网上的介绍</strong></a>非常详细，一步步弄就行了。

&nbsp;

### 使用感受

别的不多说，就简单介绍下 DNSPod 的界面、功能和优势吧！

#### 界面：

整体界面非常清爽，记录类型全都支持，免费帐户也没有任何限制，比 Godaddy 的更好用哦！

不像新网的域名管理界面，我不多说了，用过的都知道…

[<img class="alignnone size-full wp-image-690" title="a" alt="a" src="/uploads/2012/03/a.png" width="790" height="463" />][2]

&nbsp;

#### 附加功能：

除此之外，DPNPod 还有很多 Godaddy 都没有的附加功能。

1.  宕机监控，免费邮件+短信通知，3min每次，比花生壳的监控好用多了
2.  域名解析日志
3.  无限域名无限记录、并支持 URL 转发
4.  自动搜索引擎优化，和国内几大搜索引擎合作了
5.  支持动态域名（类似花生壳那样的）
6.  支持手机版

另外免费版和收费版的区别只在于速度和稳定性，免费版对于我们这样的个人用户完全够用了。

 [1]: /uploads/2012/03/ping.png
 [2]: /uploads/2012/03/a.png
