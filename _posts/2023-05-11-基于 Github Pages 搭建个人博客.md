---
layout: post
title: 基于 Github Pages 搭建个人博客 - Window10 平台
date: 2023-05-11 15:13:45 +0800
categories: [blog, github pages]
tags: [blog, github pages]
---
## 简介

GitHub Pages 是通过 GitHub 托管和发布的公共网页。 启动和运行的最快方法是使用 Jekyll 主题选择器加载预置主题。 然后，您可以修改 GitHub Pages 的内容和样式。

本指南将引导你在 username.github.io 创建用户站点。

## 创建网站

1. 创建Github个人博客Repo仓库

   ![](https://docs.github.com/assets/cb-31554/mw-1440/images/help/repository/repo-create.webp)
2. 输入 username.github.io 作为存储库名称。 将 username 替换为你的 GitHub 用户名。 例如，如果用户名为 octocat，则存储库名称应为 octocat.github.io。

   ![](https://docs.github.com/assets/cb-48482/mw-1440/images/help/pages/create-repository-name-pages.webp)
3. 在存储库名称下，单击 “设置”。

   ![](https://docs.github.com/assets/cb-28266/mw-1440/images/help/repository/repo-actions-settings.webp)
4. 在边栏的“代码和自动化”部分中，单击“ Pages”。
5. 在“生成和部署”的“源”下，选择“从分支进行部署”。
6. 在“生成和部署”的“分支”下，使用分支下拉菜单并选择发布源。

   ![](https://docs.github.com/assets/cb-47267/mw-1440/images/help/pages/publishing-source-drop-down.webp)

## 基于Ruby的本地编写和调试博客内容

1. 安装Ruby
2. 安装 jekyll 和 bundler gems

   ```cmd
   gem install jekyll bundler
   ```
3. 创建新的 Jekyll 网站（本地文件夹），并进入该文件夹

   ```cmd
   cd C:\Users\xxx\Workspace\GitHub
   jekyll new yourname.github.io
   cd yourname.github.io
   ```
4. 构建站点并使其在本地服务器上可用

   ```cmd
   bundle exec jekyll serve
   ```

   如果您使用的是 Ruby 版本 3.0.0 或更高版本，这一步可能会失败。可以通过添加到 webrick 依赖项来修复它： bundle add webrick

   ```cmd
   bundle add webrick exec jekyll serve
   
   // 指定本地端口号， 启动jekyll服务
   bundle exec jekyll serve -P 5555 --watch
   ```
5. 浏览器中打开

   http://localhost:5555

## _config.yml文件配置

### 通用参数

```java
// Site settings
title: UCM Blog                    // 你的博客网站标题
SEOTitle: 朝花夕拾 | UCM Blog	   // SEO 标题
description: "越努力越幸运"	   // 随便说点，描述一下

// SNS settings  
github_username: leid     // 你的github账号
jianshu_username: leid    // 你的简书ID。

// Build settings
paginate: 10              // 一页你准备放几篇文章
```

Jekyll官方网站还有很多的参数可以调，比如设置文章的链接形式...网址在这里：Jekyll - Official Site 中文版的在这里：Jekyll中文.

### 撰写博文

要发表的文章一般以 Markdown 的格式放在这里_posts/，你只要看看这篇模板里的文章你就立刻明白该如何设置。

yaml 头文件长这样:

```java
---
layout:     post
title:      定时器 你真的会使用吗？
subtitle:   iOS定时器详解
date:       2023-07-05
author:     BY
header-img: img/post-bg-web.jpg
catalog: 	 true
tags:
    - Android
    - Binder
---
```

### 侧边栏

设置是在 `_config.yml`文件里面的 `Sidebar settings`那块。

```java

// Sidebar settings
sidebar: true  // 添加侧边栏
sidebar-about-description: "简单的描述一下你自己"
sidebar-avatar: /img/avatar-by.jpg     // 你的大头贴，请使用绝对地址.注意：名字区分大小写！后缀名也是
```

参考链接：

> [GitHub Pages 快速入门](https://docs.github.com/zh/pages/quickstart)
>
> [Jekyll Quickstart](https://jekyllrb.com/docs/)
>
> [如何在Windows平台上基于github搭建个人博客平台](https://zhuanlan.zhihu.com/p/102346113)
>
> [利用 GitHub Pages 快速搭建个人博客](https://github.com/qiubaiying/qiubaiying.github.io)
