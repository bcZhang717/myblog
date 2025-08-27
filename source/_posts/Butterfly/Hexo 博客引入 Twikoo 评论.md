---
title: Hexo 博客引入 Twikoo 评论
date: 2025-08-27
description: 
tags:
  - Hexo
  - Butterfly
categories:
  - Butterfly
---
# 前言

> [Butterfly 官方文档的评论模块](https://butterfly.js.org/posts/4aa8abbe/#%E8%A9%95%E8%AB%96)

一开始我使用的是 `livere (来必力)` 的评论，但是后来浏览了众多博客之后，发现使用 `Twikoo` 评论的比较多，所以我就萌生了修改评论的方法。而且 `Twikoo` 评论确实比较好看。

`Twikoo` 是无后端的网站评论系统，基于腾讯云开发，可以查看其 [官方文档](https://twikoo.js.org/quick-start.html#%E7%8E%AF%E5%A2%83%E5%88%9D%E5%A7%8B%E5%8C%96)。

# 部署 Twikoo 评论

我们可以使用免费的 `Vercel` 进行部署 `Twikoo` 评论，否则就要使用服务器去部署了。

官方文档：[Vercel 部署 Twikoo](https://twikoo.js.org/backend.html#vercel-%E9%83%A8%E7%BD%B2)

下面来详细的介绍部署过程。

## 申请 MongoDB Atlas 账号

MongoDB Atlas 是 MongoDB Inc 提供的 MongoDB 数据库托管服务。免费账户可以永久使用 500 MiB 的数据库，足够存储 Twikoo 评论使用。

首先进入 [MongoDB Atlas 注册界面](https://www.mongodb.com/cloud/atlas/register) 进行用户注册，这里我选择使用我的 Google 账号进行登录。

然后就可以进入一个介绍页面了，你可以选择跳过，也可以按照自己的需求填写。

<img src="https://picgo-blog-1335849645.cos.ap-guangzhou.myqcloud.com/images/20250827214224054.png" alt="image-20250827214223954" style="zoom:50%;" />

接下来我们选择免费的 MongoDB，这里我选择 HK，然后点击 `Create Deployment` 即可。

<img src="https://picgo-blog-1335849645.cos.ap-guangzhou.myqcloud.com/images/20250827214552122.png" alt="image-20250827214552014" style="zoom:50%;" />

然后我们需要创建数据库用户，并指定密码

<img src="https://picgo-blog-1335849645.cos.ap-guangzhou.myqcloud.com/images/20250827214851215.png" alt="image-20250827214851113" style="zoom:50%;" />

创建完用户以后，就可以 close 了。接下来在 Network Access 页面点击 Add IP Address 添加网络白名单。因为 Vercel 的出口地址不固定，所以我们这里 Access List Entry 输入 `0.0.0.0/0`（允许所有 IP 地址的连接）。

<img src="https://picgo-blog-1335849645.cos.ap-guangzhou.myqcloud.com/images/20250827215202580.png" alt="image-20250827215202431" style="zoom:50%;" />

接下来我们切换到 Clusters 标签，选择 Connect。

![image-20250827215354398](https://picgo-blog-1335849645.cos.ap-guangzhou.myqcloud.com/images/20250827215354496.png)

接下来选择连接方式为 `Driver` ，并复制连接信息。

![image-20250827215619900](https://picgo-blog-1335849645.cos.ap-guangzhou.myqcloud.com/images/20250827215620020.png)

```bash
mongodb+srv://bczhang:<db_password>@myblog.yd61ctz.mongodb.net/?retryWrites=true&w=majority&appName=myblog

# 修改为

mongodb+srv://bczhang:你刚刚设置的数据库密码@myblog.yd61ctz.mongodb.net/?retryWrites=true&w=majority&appName=myblog
```

## Vercel 部署 Twikoo

直接点击链接进行部署：[New Project-Vercel](https://vercel.com/new/clone?repository-url=https%3A%2F%2Fgithub.com%2Ftwikoojs%2Ftwikoo%2Ftree%2Fmain%2Fsrc%2Fserver%2Fvercel-min) ，需要进行登录，我这里选择 Github 登录。

<img src="https://picgo-blog-1335849645.cos.ap-guangzhou.myqcloud.com/images/20250827220505017.png" alt="image-20250827220504933" style="zoom:67%;" />

接下来等待 Vercel 部署即可，时间可能会比较久。

部署完成后按照下图步骤依次执行：

<img src="https://picgo-blog-1335849645.cos.ap-guangzhou.myqcloud.com/images/20250827221132455.png" alt="image-20250827221132319" style="zoom:67%;" />

接下来进入 `Deployment Protection` ，设置 `Vercel Authentication` 为 Disabled，并 Save 保存。

![image-20250827221354578](https://picgo-blog-1335849645.cos.ap-guangzhou.myqcloud.com/images/20250827221354683.png)

接下来按照下图进行：

<img src="https://picgo-blog-1335849645.cos.ap-guangzhou.myqcloud.com/images/20250827221520761.png" alt="image-20250827221520622" style="zoom:67%;" />

弹出的窗口中继续点击 `Redeploy` 进行重新部署，等待完成即可。

<img src = "https://picgo-blog-1335849645.cos.ap-guangzhou.myqcloud.com/images/20250827221615202.png" style="zoom:67%;" />

部署成功后会显示 "云函数运行正常 ..."，然后复制 Domains 下的一个链接，作为 Butterfly 集成 Twikoo 的 `envId` 。

![image-20250827222114724](https://picgo-blog-1335849645.cos.ap-guangzhou.myqcloud.com/images/20250827222114826.png)

# Twikoo 接入 Butterfly

回到博客中 Butterfly 主题的配置文件 `_config.butterfly.yml` ，修改如下配置：

```yaml
comments:
  use: Twikoo # 修改评论模块
  text: false
  lazyload: false
  count: false
  card_post_count: false

# Twikoo
twikoo:
  envId: https://xxxxxxxxxxx.vercel.app # 粘贴复制的内容，前面需要加 https://
  region:
  visitor: true # 是否展示文章阅读数
  option:
```

随后使用 `hexo clean && hexo generate && hexo server` 命令即可本地预览是否配置 Twikoo 评论成功。

<img src="https://picgo-blog-1335849645.cos.ap-guangzhou.myqcloud.com/images/20250827223341705.png" alt="image-20250827223341517" style="zoom:67%;" />

然后使用命令 `hexo clean && hexo generate && hexo deploy` 即可托管博客到 Github。

至此，大功告成！

















