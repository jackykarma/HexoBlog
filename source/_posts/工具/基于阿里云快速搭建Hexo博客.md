---
title: 基于阿里云搭建Hexo博客
date: 2022-03-26 12:58:57
tags: 工具
categories: 工具
---

参考资料：

1. [Hexo博客框架官方](https://hexo.io/)
2. [基于阿里云搭建Hugo博客](https://zhuanlan.zhihu.com/p/115219597)
3. [阿里云搭建Hexo博客方法](http://blog.zhangkexuan.cn/2020/10/25/hexo-aliyun/)

本文主要参考：[阿里云搭建Hexo博客方法](http://blog.zhangkexuan.cn/2020/10/25/hexo-aliyun/)

其中deploy部署的部分改为：

```js
# Deployment
## Docs: https://hexo.io/docs/one-command-deployment
deploy:
  type: git
  repo: root@120.25.162.10:/home/git/MyBlog.git
  branch: master
```

hexo d命令报错 ERROR Deployer not found: git [解决方法](https://blog.csdn.net/qq_21808961/article/details/84476504) 如下：

```js
npm install hexo-deployer-git --save
```

配置域名解析：域名与ip地址的对应关系。——注意：**域名需要备案**，否则无法使用。

[https://dns.console.aliyun.com/#/dns/domainList](https://dns.console.aliyun.com/#/dns/domainList) 解析设置。
[https://dns.console.aliyun.com/#/dns/setting/trackon.top](https://dns.console.aliyun.com/#/dns/setting/trackon.top) 添加记录。

nginx无法访问的问题：需要去阿里云配置http、https的访问安全组。默认只开放22端口。 配置安全组的方法如下：[阿里云安全组配置](https://blog.csdn.net/keepfriend/article/details/110563717)

![](%E6%88%AA%E5%B1%8F2022-01-29%2016.27.29.png)

![](%E6%88%AA%E5%B1%8F2022-01-29%2016.28.19.png)

## 二、无需密码自动部署

自动化执行部署命令的脚本：

```js
➜  Hexo cat autodeploy.sh 
#!/usr/bin/expect -f

set password 你的密码 
set timeout -1

spawn hexo g -d
expect "*assword:*"
send "$password\r"
interact
```

## 三、主题配置与定制

Hexo支持许多主题插件，其中我使用的是matery，主要是好看、配置起来比较省心还有中文注释。

[matery](https://blinkfox.github.io/2018/09/28/qian-duan/hexo-bo-ke-zhu-ti-zhi-hexo-theme-matery-de-jie-shao/)

![](%E6%88%AA%E5%B1%8F2022-01-30%2009.20.38.png)

![](%E6%88%AA%E5%B1%8F2022-01-30%2009.26.02.png)

缺点：**不支持多层级分类**。实际上我希望Framework是Android下的子分类，可以分层级查看。

![](%E6%88%AA%E5%B1%8F2022-01-30%2009.27.29.png)

搜索配置：[search](https://www.cnblogs.com/coderma/p/13453876.html) 
其他配置：[自定义](https://www.cnblogs.com/mfrank/p/12830097.html)

## 三、文章管理

[定制](https://www.cnblogs.com/mfrank/p/12830097.html)
[github官方文档](https://github.com/blinkfox/hexo-theme-matery/blob/develop/README_CN.md)

### 3.1. 文章创建

```js
hexo new post -post $1
```

可以传入目录将文章按照目录进行管理，方便后续查找。但是这与categories文章所属分类无任何关系。

```js
➜  Hexo (master) ✔ hexo new post -p /Android原生开发/平台基础部分/应用核心/深入理解Activity   
INFO  Validating config
INFO  Created: ~/softarclee/hexoblog/Hexo/source/_posts/Android原生开发/平台基础部分/应用核心/深入理解Activity.md
```

### 3.2. 图片资源管理

[https://yanyinhong.github.io/2017/05/02/How-to-insert-image-in-hexo-post/](https://yanyinhong.github.io/2017/05/02/How-to-insert-image-in-hexo-post/)

修改hexo配置：`post_asset_folder: true`；当创建文章的时候就会创建与文章名称相同的文件夹，用于存放资源。

### 3.3. Ulysess文章快速发布到Hexo方法

![](%E6%88%AA%E5%B1%8F2022-03-26%2012.28.02.png)![](%E6%88%AA%E5%B1%8F2022-03-26%2012.28.25.png)![](%E6%88%AA%E5%B1%8F2022-03-26%2012.28.48.png)

然后：

1. 利用hexo命令创建文章（会生成存放资源的同名文件夹）
2. 将图片资源等拷贝到文件夹
3. 将index.md文件内容拷贝到hexo生成的md文件，并作简单修改：
	![](%E6%88%AA%E5%B1%8F2022-03-26%2012.31.57.png)

	- 删除一级标题，因为hexo是用title配置的；
	- 为文章添加tags和分类categories：可以有多个分类和多个tag；

## 四、多层级分类实现

目前没有发现怎么进行多层级的分类。父-子-子…分类，按照文件目录结构的方式进行父子分类。

![](%E6%88%AA%E5%B1%8F2022-01-30%2012.57.40.png) 










