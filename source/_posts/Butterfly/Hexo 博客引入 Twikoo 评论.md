---
title: Hexo 博客引入 Twikoo 评论
date: 2025-08-27 
tags:
  - Hexo
  - Butterfly
categories:
  - Butterfly
description: >-
  这篇文章介绍了如何在Hexo博客中引入Twikoo评论系统。作者最初使用的是Livere评论，但后来发现Twikoo更受欢迎且界面更好，因此决定进行替换。
  文章详细描述了部署Twikoo的过程，包括申请MongoDB Atlas账号、通过Vercel部署Twikoo云函数以及绑定自定义域名的步骤。
  同时，还讲解了如何在Butterfly主题中配置Twikoo，并提供了美化评论模块的方法，如添加表情包放大效果的JavaScript代码。整体内容涵盖了从环境搭建到前端集成的完整流程。
summary: >-
  这里是爱谦AI，这篇文章介绍了如何在Hexo博客中引入Twikoo评论系统。作者最初使用的是Livere评论，但后来发现Twikoo更受欢迎且界面更好，因此决定进行替换。文章详细描述了部署Twikoo的过程，包括申请MongoDB
  Atlas账号、通过Vercel部署Twikoo云函数以及绑定自定义域名的步骤。同时，还讲解了如何在Butterfly主题中配置Twikoo，并提供了美化评论模块的方法，如添加表情包放大效果的JavaScript代码。整体内容涵盖了从环境搭建到前端集成的完整流程。
---

# 前言

> [Butterfly 官方文档的评论模块](https://butterfly.js.org/posts/4aa8abbe/#%E8%A9%95%E8%AB%96)

一开始我使用的是 `livere (来必力)` 的评论，但是后来浏览了众多博客之后，发现使用 `Twikoo` 评论的比较多，所以我就萌生了修改评论的方法。而且 `Twikoo` 评论确实比较好看。

`Twikoo` 是无后端的网站评论系统，基于腾讯云开发，可以查看其 [官方文档](https://twikoo.js.org/quick-start.html#%E7%8E%AF%E5%A2%83%E5%88%9D%E5%A7%8B%E5%8C%96)。

# 部署 Twikoo 评论

我们可以使用免费的 `Vercel` 进行部署 `Twikoo` 评论，否则就要使用服务器去部署了。

官方文档：[Vercel 部署 Twikoo](https://twikoo.js.org/backend.html#vercel-%E9%83%A8%E7%BD%B2)

{% link https://twikoo.js.org/backend.html#vercel-%E9%83%A8%E7%BD%B2, Vercel 部署 Twikoo, https://twikoo.js.org/twikoo-logo-home.png %}

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

<img src = "https://picgo-blog-1335849645.cos.ap-guangzhou.myqcloud.com/images/20250827221615202.png" style="zoom:67%;"  alt="20250827221615202.png"/>

部署成功后会显示 "云函数运行正常 ..."，然后复制 Domains 下的一个链接，作为 Butterfly 集成 Twikoo 的 `envId` 。

![image-20250827222114724](https://picgo-blog-1335849645.cos.ap-guangzhou.myqcloud.com/images/20250827222114826.png)

## Vercel 绑定域名 (可选)

由于 `vercel.app` 域名已经被 DNS 污染，那么国内网络应该都是无法进行访问的，所以我们需要绑定自己的域名来转发 Vercel DNS Server 地址。如果你不想这样做，那么访问评论就只能通过 "魔法" 了。

1、找到 Vercel 中部署的 Twikoo 项目，点击 `Settings` 选项卡，跳转页面后点击左侧的 `Domains` ，输入你自己定义的域名。

<img src="https://picgo-blog-1335849645.cos.ap-guangzhou.myqcloud.com/images/20250828140254540.png" alt="image-20250828140254351" style="zoom:50%;" />

2、输入自己的域名即可，这里建议单独买一个域名，避免后续造成 CNAME 指向网址，A 指向 IP 造成的矛盾。

<img src="https://picgo-blog-1335849645.cos.ap-guangzhou.myqcloud.com/images/20250828151448063.png" alt="image-20250828151447852" style="zoom:50%;" />

3、接下来会出现配置问题，根据指引去添加域名解析即可。有一个报错就添加一个，有两个就添加两个，这里我是两个。

<img src="https://picgo-blog-1335849645.cos.ap-guangzhou.myqcloud.com/images/20250828151634194.png" alt="image-20250828151634053" style="zoom:50%;" />

4、根据提示进行解析即可，没什么特别的，我是用的是腾讯云买的域名。

<img src="https://picgo-blog-1335849645.cos.ap-guangzhou.myqcloud.com/images/20250828151848632.png" alt="image-20250828151848501" style="zoom:50%;" />

5、回到 Vercel 界面，`Refresh` 刷新，看看域名是否解析成功 (需要等待一段时间)。

<img src="https://picgo-blog-1335849645.cos.ap-guangzhou.myqcloud.com/images/20250828152328231.png" alt="image-20250828152328106" style="zoom:50%;" />

6、此时如果在浏览器中输入 `https://aiqian.online` 显示如下的内容就代表域名解析成功。

```json
{
    "code": 100,
    "message": "Twikoo 云函数运行正常，请参考 https://twikoo.js.org/frontend.html 完成前端的配置",
    "version": "1.6.44"
}
```

7、注意注意注意，第五步的图片是错误的，我们需要打开 `Edit` 使其禁止重定向，不然会出现跨域问题！！！

<img src="https://picgo-blog-1335849645.cos.ap-guangzhou.myqcloud.com/images/20250828155349744.png" alt="image-20250828155349586" style="zoom:67%;" />

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

> 如果绑定了域名，可以将 `envId` 填写为 `https://aiqian.online` 。

随后使用 `hexo clean && hexo generate && hexo server` 命令即可本地预览是否配置 Twikoo 评论成功。

<img src="https://picgo-blog-1335849645.cos.ap-guangzhou.myqcloud.com/images/20250828093308711.png" alt="image-20250828093308530" style="zoom:67%;" />

然后使用命令 `hexo clean && hexo generate && hexo deploy` 即可托管博客到 Github。

# Twikoo 评论模块美化

## 表情包放大效果

1、粘贴以下内容到自定义的 js 文件即可。

```javascript
// 如果当前页有评论就执行函数
if (document.getElementById('post-comment')) owoBig();

// 表情放大
function owoBig() {
    let flag = 1, // 设置节流阀
        owo_time = '', // 设置计时器
        m = 3; // 设置放大倍数
    // 创建盒子
    let div = document.createElement('div'),
        body = document.querySelector('body');
    // 设置ID
    div.id = 'owo-big';
    // 插入盒子
    body.appendChild(div)

    // 构造observer
    let observer = new MutationObserver(mutations => {

        for (let i = 0; i < mutations.length; i++) {
            let dom = mutations[i].addedNodes,
                owo_body = '';
            if (dom.length == 2 && dom[1].className == 'OwO-body') owo_body = dom[1];
                // 如果需要在评论内容中启用此功能请解除下面的注释
            // else if (dom.length == 1 && dom[0].className == 'tk-comment') owo_body = dom[0];
            else continue;

            // 禁用右键
            if (document.body.clientWidth <= 768) owo_body.addEventListener('contextmenu', e => e.preventDefault());
            // 鼠标移入
            owo_body.onmouseover = (e) => {
                // 检查父元素的 className 是否包含 'OwO-packages'
                if (e.target.parentElement.parentElement.parentElement && e.target.parentElement.parentElement.parentElement.className.includes('OwO-packages')) return;
                if (flag && e.target.tagName == 'IMG') {
                    flag = 0;
                    // 移入300毫秒后显示盒子
                    owo_time = setTimeout(() => {
                        let height = e.target.clientHeight * m, // 盒子高
                            width = e.target.clientWidth * m, // 盒子宽
                            left = (e.x - e.offsetX) - (width - e.target.clientWidth) / 2, // 盒子与屏幕左边距离
                            top = e.y - e.offsetY; // 盒子与屏幕顶部距离
						 // 右边缘检测, 防止超出屏幕
                        if ((left + width) > body.clientWidth) left -= ((left + width) - body.clientWidth + 10);
                        if (left < 0) left = 10; // 左边缘检测, 防止超出屏幕
                        // 设置盒子样式
                        div.style.cssText = `display:flex; height:${height}px; width:${width}px; left:${left}px; top:${top}px;`;
                        // 在盒子中插入图片
                        div.innerHTML = `<img src="${e.target.src}">`
                    }, 300);
                }
            };
            // 鼠标移出隐藏盒子
            owo_body.onmouseout = () => {
                div.style.display = 'none', flag = 1, clearTimeout(owo_time);
            }
        }
    })
    observer.observe(document.getElementById('post-comment'), {subtree: true, childList: true}) // 监听的元素和配置项
}
```

2、将以下内容粘贴到自定义的 CSS 文件即可。

```css
#owo-big {
    position: fixed;
    align-items: center;
    background-color: rgb(255, 255, 255);
    border: 1px #aaa solid;
    border-radius: 10px;
    z-index: 9999;
    display: none;
    transform: translate(0, -105%);
    overflow: hidden;
    animation: owoIn 0.3s cubic-bezier(0.42, 0, 0.3, 1.11);
}

[data-theme=dark] #owo-big {
    background-color: #4a4a4a
}

#owo-big img {
    width: 100%;
}

@keyframes owoIn {
    0% {
        transform: translate(0, -95%);
        opacity: 0;
    }
    100% {
        transform: translate(0, -105%);
        opacity: 1;
    }
}
```

3、效果展示

<img src="https://picgo-blog-1335849645.cos.ap-guangzhou.myqcloud.com/images/20250828101805440.png" alt="image-20250828101805202" style="zoom:67%;" />

## 自定义表情包

1、点击评论区右下角的 `齿轮` 图标 (如上图)，设置一个密码，进入管理界面。

<img src="https://picgo-blog-1335849645.cos.ap-guangzhou.myqcloud.com/images/20250828101956335.png" alt="image-20250828101956208" style="zoom:67%;" />

2、进入 Twikoo 管理面板后，依次找到：配置管理、插件。

<img src="https://picgo-blog-1335849645.cos.ap-guangzhou.myqcloud.com/images/20250828102052453.png" alt="image-20250828102052341" style="zoom:67%;" />

3、最后找到 `EMOTION_CDN` ，填入 `https://cdn.cbd.int/daliyuer-static@latest/bq/twikoo.json` ，使用大佬制作好的表情包。

<img src="https://picgo-blog-1335849645.cos.ap-guangzhou.myqcloud.com/images/20250828102232618.png" alt="image-20250828102232494" style="zoom:50%;" />

4、最后划到页面最下方，点击保存即可。