---
title:  Jekyll+Chirpy+Github pages 搭建博客 
date:   2023-07-05 23:00:00 +0800
categories: [blog, jekyll]
tags: [blog, jekyll]
author: ponng
---

## 初始化Jekyll博客项目和Chirpy主题

### 1. 安装Jekyll

可以参照[Jekyll官网](https://jekyllrb.com/docs/)和[中文翻译文档](http://jekyllcn.com/docs/home/)进行搭建。

### 2. 新建Jekyll项目

使用命令行工具，运行新建项目命令:

```bash
   $ jekyll new ucan.github.io
```

运行成功后，会在当前文件夹中生成名字为`ucan.github.io`的文件夹，该文件夹即为项目的文件夹。

### 3. 使用Chirpy主题

1. 安装并使用Chirpy

   1. 参照[Chirpy文档](https://github.com/cotes2020/jekyll-theme-chirpy/blob/master/docs/README.zh-CN.md)，使用`RubyGems`的方式安装主题

   2. 打开项目中的`_config.yml`文件，并将其中默认的`minima`主题改为`jekyll-theme-chirpy`

   3. 执行安装程序

      ```shell
      $ bundle
      ```

   4. 打开Chirpy主题所在文件夹，并将其中的内容复制到项目目录下。

      ```shell
      $ cd "$(bundle info --path jekyll-theme-chirpy)"
      ```

   5. 

2. 运行程序并预览

   ```shell
   $ bundle exec jekyll serve -P 6666 --watch
   ```

3. 







参考：

> [Jekyll+Chirpy+自建服务器域名+Nginx博客搭建全纪录](http://blog.stoneway.cn/posts/Jekyll+Chirpy+%E8%87%AA%E5%BB%BA%E6%9C%8D%E5%8A%A1%E5%99%A8%E5%9F%9F%E5%90%8D+Nginx%E5%8D%9A%E5%AE%A2%E6%90%AD%E5%BB%BA%E5%85%A8%E7%BA%AA%E5%BD%95/)