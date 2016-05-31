---
title: 大型网站技术架构读书笔记--概述
date: 2016-04-27 13:47:38
categories: website_architecture
tags: [架构,笔记]
---

### 大型网站软件系统特点
>**高并发，大流量：** 高并发，大流量访问。如Google日均PV35亿
**高可用：** 系统需要24小时不间断服务
**海量数据：** 需要存储管理海量数据，使用大量的服务器
**用户分布广泛** 为全球用户提供服务，用户分布范围广，各地区情况差别较大
**安全环境恶劣** 由于互联网开放性,网站容易被黑客攻击
**需求变更快，快速迭代** 互联网产品为适应市场，需求变更非常快
**渐进式发展：** 渐进式发展，从小网站慢慢迭代发展起来的

<!-- more -->

### 大型网站演化发展历程
#### 初始阶段
使用类似于LAMP结构的网站，所有资源、数据库、程序都部署在同一服务器上
![](http://static.tmaczhao.cn/images/large_website_notes/large_website_archi_1.1.png)

#### 应用与数据服务分离
文件、程序、数据库按照需求分离成多台服务器，按各自特点(储存，性能，缓存等)进行分别部署
![](http://static.tmaczhao.cn/images/large_website_notes/large_website_archi_1.2.png)

#### **缓存**改善网站性能
缓存高访问数据，缓存主要有**本地缓存**和**远程分布式缓存**(集群，可扩展服务器)
![](http://static.tmaczhao.cn/images/large_website_notes/large_website_archi_1.3.png)

#### 应用服务器集群
解决高并发海量数据带来的性能不足问题，使用**负载均衡服务器**合理调度集群
![](http://static.tmaczhao.cn/images/large_website_notes/large_website_archi_1.4.png)

#### 数据库读写分离
主从热备功能,两台数据库服务器配置主从关系，写数据访问主数据库并主从复制同步从数据库
![](http://static.tmaczhao.cn/images/large_website_notes/large_website_archi_1.5.png)

#### CDN和反向代理
CDN和反向代理的原理都是缓存，CDN访问CDN提供商的机房，反向代理部署在网站的中心机房，如果请求的资源存在缓存，直接返回
![](http://static.tmaczhao.cn/images/large_website_notes/large_website_archi_1.6.png)

#### 分布式文件系统和分布式数据库系统
数据库拆分一般情况下业务分库，将不同业务的数据分别部署，只有单表规模非常大的时候会使用分布式数据库
![](http://static.tmaczhao.cn/images/large_website_notes/large_website_archi_1.7.png)

#### 使用NoSQL和搜索引擎
网站业务和数据非常复杂时采用一些非关系型数据库技术如NoSQL或非数据库查询技术搜索引擎进行查询
![](http://static.tmaczhao.cn/images/large_website_notes/large_website_archi_1.8.png)

#### 业务拆分
拆分网站业务，拆分产品线，分而治之，独立部署和维护
![](http://static.tmaczhao.cn/images/large_website_notes/large_website_archi_1.9.png)

#### 分布式服务
提取不同应用系统的共同业务独立部署，由这些可以复用的业务连接数据库，提供业务服务，应用系统提供管理界面，通过分布式服务完成业务操作
![](http://static.tmaczhao.cn/images/large_website_notes/large_website_archi_1.10.png)


### 网站架构价值观
>**在网站结构的搭建中应该实事求是，不能盲目追求大型网站结构，一味追随大公司的解决档案，网站的架构技术应该随着网站的业务发展而驱动应对，并且不能为了技术而技术，网站的技术是为了业务而存在的，技术也解决不了所有的业务问题，所以将业务逻辑和技术架构合理的结合才能构建出更加合理的网站。**















