---
title: nexus3.x 迁移 从 windows 转移到 linux
date: 2016-11-15 22:49:59
tags:
- 运维配置
- 杂记
categories:
- Maven
---

转载请注明来源 [赖赖的博客](http://laiyijie.me/2016/11/15/nexus3.x%20%E8%BF%81%E7%A7%BB/)
## 导语
> google 和百度可以解决绝大部分问题，但往往体现你能力的，却是解决不能搜索到的问题。

能用google和百度解决问题，那是效率最高也是学习成本最低的事情，然而世事难料，并不是所有问题都可以从google获得答案。我写的这个问题，google和百度都没有给我正确的解决方案，甚至连官方文档也没有给出方案，特此贡献给大家，希望可以帮助一小部分人！

<!-- more -->
## 背景

笔者初次搭建MAVEN私服，挑选了 nexus作为服务器。 而nexus3.0.2恰巧为最新版本，什么事情都喜欢新的笔者，挑选了最新版本，又很凑巧的选择了一键安装的版本！双击以后的感觉是，**我x原来世界这么简单！**

> 对不熟悉的应用，安装的越简单，维护起来吃的shi越多

时隔两个月，局域网内的nexus私服已经不能满足需求，需要转移到线上的linux服务器（为什么不直接上线啊 (╯‵□′)╯︵┻━┻）

然后悲剧就发生了

## 转移过程和坑点排除

有别于nexus2.x ，nexus 3.x 已经将数据和系统完全分离（其实2.x后面几个版本已经分离完全了）所以转移非常简单：

### 为新的linux准备全新的安装包和配置

1. 访问[nexus 官网](http://www.sonatype.org/nexus/)
2. 进入[下载页面](https://www.sonatype.com/download-oss-sonatype)并且选择 Nexus Repository Manager OSS 3.x 进行下载
3. 放到你喜欢的目录并且解压
4. 依照官网的[文档](http://books.sonatype.com/nexus-book/3.0/reference/index.html)的第二章进行安装配置

### 迁移数据到新的服务器上（重点）

* 首先找到旧服务器上的数据文件夹，如果原先的nexus运行在linux服务器上，请直接到 nexus3.x的目录下面找到 data 文件夹
 
然而笔者原来用的是windows啊(╯‵□′)╯︵┻━┻ ，当你打开nexus3.x下的data文件夹的时候，看到里面一片空白(╯‵□′)╯︵┻━┻


#### 第一坑 如何找到windows中对应的数据文件夹

> 1. 首先打开 nexus-3.x/.install4j/response.varfile
> 2. 找到 nexus.dataDir=D\:\\nexus-data （原来在这里）

* 备份旧的数据
* 这个时候理所当然的是用nexus-data 替换掉新服务器的data文件夹！然而，这样却不行！而且到处找到不到答案，笔者排查了两个小时的问题啊！
#### 第二坑 删除data文件夹下面的 cache文件夹

> 请一定要删除这个文件夹！否则无法跑起来！

* 重新启动新服务器上的nexus，一切大功告成，连原来的账号密码都在！

有可能是迁移过于简单，所以查不到这个问题，而nexus3.x的文档只有从2.x向3.x迁移的办法，却是我实实在在踩过的坑