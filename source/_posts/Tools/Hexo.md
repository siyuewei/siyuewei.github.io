---
title: Hexo
date: 2025-09-04 00:35:50
tags:
  - Hexo
  - 博客
  - 静态网站
  - 教程
---

# Hexo 静态博客框架使用指南

Hexo 是一个快速、简洁且高效的博客框架，基于 Node.js 构建。它可以将 Markdown 文件渲染成静态网页，支持多种主题和插件。

## 常用命令

### 基础命令
```bash
# 创建新文章
hexo new "文章标题"

# 创建新页面
hexo new page "页面名称"

# 创建草稿
hexo new draft "草稿标题"

# 生成静态文件
hexo generate
# 或简写
hexo g

# 启动本地服务器
hexo server
# 或简写
hexo s

# 部署到服务器
hexo deploy
# 或简写
hexo d

# 清理缓存和生成的文件
hexo clean

# 一键生成并部署
hexo g -d
```

### 草稿相关命令
```bash
# 发布草稿
hexo publish "草稿文件名"

# 预览草稿
hexo server --draft
```

### 主题管理
```bash
# 安装主题
git clone <theme-repository-url> themes/<theme-name>

# 更新主题
cd themes/<theme-name>
git pull
```

## 配置文件说明

### _config.yml 主要配置项
```yaml
# 网站基本信息
title: 网站标题
subtitle: 网站副标题
description: 网站描述
keywords: 关键词
author: 作者名
language: zh-CN

# URL 设置
url: https://your-domain.com
permalink: :year/:month/:day/:title/

# 目录设置
source_dir: source
public_dir: public
skip_render: []

# 写作设置
new_post_name: :title.md
default_layout: post
auto_spacing: false
titlecase: false

# 分类和标签
default_category: uncategorized
category_map:
tag_map:

# 日期格式
date_format: YYYY-MM-DD
time_format: HH:mm:ss

# 分页设置
per_page: 10
pagination_dir: page

# 主题设置
theme: next

# 部署设置
deploy:
  type: git
  repo: <repository-url>
  branch: main
```

## 文章写作

### Front-matter 格式
```yaml
---
title: 文章标题
date: 2025-09-04 00:35:50
updated: 2025-09-04 00:35:50
tags:
  - 标签1
  - 标签2
categories:
  - 分类1
  - 分类2
comments: true
toc: true
toc_number: true
toc_style_simple: false
mathjax: true
---

文章内容...
```

### 常用标签语法
```markdown
# 图片
![图片描述](图片链接)

# 链接
[链接文字](链接地址)

# 代码块
```javascript
console.log('Hello World');
```

<!-- # 引用
> 引用内容

# 表格
| 列1 | 列2 | 列3 |
|-----|-----|-----|
| 内容1 | 内容2 | 内容3 | -->


## 主题配置

### Next 主题常用配置
```yaml
# 主题设置
theme: next

# 菜单设置
menu:
  home: / || home
  archives: /archives/ || archive
  categories: /categories/ || th
  tags: /tags/ || tags
  about: /about/ || about

# 侧边栏设置
sidebar:
  position: left
  display: always

# 社交链接
social:
  github: https://github.com/yourname
  twitter: https://twitter.com/yourname
  email: your-email@example.com

# 评论系统
comments:
  active: disqus
  shortname: your-disqus-shortname
```

## 部署配置

### GitHub Pages 部署
```yaml
deploy:
  type: git
  repo: https://github.com/username/username.github.io.git
  branch: main
  message: "Site updated: {{ now('YYYY-MM-DD HH:mm:ss') }}"
```

### 其他部署方式
```yaml
# 部署到服务器
deploy:
  type: rsync
  host: your-server.com
  user: username
  root: /var/www/
  path: blog/
  delete: true

# 部署到腾讯云 COS
deploy:
  type: cos
  bucket: your-bucket-name
  region: ap-beijing
  secretId: your-secret-id
  secretKey: your-secret-key
```

## 插件推荐

### 常用插件
```bash
# 搜索插件
npm install hexo-generator-search --save

# 站点地图
npm install hexo-generator-sitemap --save

# RSS 订阅
npm install hexo-generator-feed --save

# 文章阅读时间统计
npm install hexo-reading-time --save

# 图片懒加载
npm install hexo-lazyload-image --save

# 代码高亮
npm install hexo-prism-plugin --save
```

## 性能优化

### 图片优化
```bash
# 安装图片压缩插件
npm install hexo-image-min --save

# 配置图片懒加载
lazyload:
  enable: true
  onlypost: false
  loadingImg: /images/loading.gif
```

### 代码压缩
```bash
# 安装压缩插件
npm install hexo-html-minifier --save
npm install hexo-clean-css --save
npm install hexo-uglify --save
```


---

<!-- *本文档会持续更新，如有问题欢迎交流讨论。* -->
