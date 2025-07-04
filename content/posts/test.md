---
# 示例：Hugo 博客文章
# 文件路径：content/posts/my-first-post.md
# 访问地址：https://yourdomain.com/posts/my-first-post/

title: "探索 Hugo 的静态网站魅力"
date: 2025-07-04T11:14:00+08:00
draft: false  # 设为 true 表示草稿，hugo 默认不会发布

# 文章元数据
author: "张三"
description: "探索 Hugo 静态网站生成器的主要功能和使用技巧"
tags: ["hugo", "静态网站", "markdown"]
categories: ["技术教程"]
series: ["Hugo 入门指南"]  # 系列文章

# 特色图片（会在首页和社交分享中显示）
featured_image: "/images/hugo-logo.png"

# 相关配置
toc: true         # 启用目录
math: true         # 启用数学公式支持
code_copy: true    # 启用代码复制按钮

# 预定义模板（可选）
type: "posts"      # 使用主题中的 posts 模板
layout: "single"   # 使用 single 布局
---

## 为什么选择 Hugo？

[Hugo](https://gohugo.io) 是目前最快的静态网站生成器，使用 Go 语言构建。它的主要优势包括：

- ⚡ 极快的构建速度（每页约 1 毫秒）
- 🚀 内嵌实时重载开发服务器
- 📚 强大的内容管理功能
- 🎨 丰富的主题生态系统

## 基本安装步骤

```bash
# 安装 Hugo（macOS 示例）
brew install hugo

# 创建新站点
hugo new site my-blog

# 添加主题（示例）
cd my-blog
git init
git submodule add https://github.com/theNewDynamic/gohugo-theme-ananke themes/ananke
echo theme = \"ananke\" >> config.toml

# 创建新文章
hugo new posts/my-first-post.md

# 启动本地服务器
hugo server -D
