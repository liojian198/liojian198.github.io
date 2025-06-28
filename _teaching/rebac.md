---
title: "rebac权限系统"
collection: teaching
type: "teaching"
excerpt: ''
permalink: /teaching/rebac
date: 2025-06-28
toc: true # 启用目录
toc_label: "aws 问题及解决方案" # 目录标题
toc_sticky: true # 目录是否固定在侧边 (可选)
---



# Jekyll 简介

Jekyll 是一个静态站点生成器，它将纯文本文件渲染成静态网站。它非常适合博客、项目网站和文档。

## 什么是静态站点生成器？

静态站点生成器（SSG）是一种工具，它从一组输入文件生成一个完整的静态 HTML 网站。它不依赖数据库和服务器端处理，而是生成可以直接由 Web 服务器提供服务的纯 HTML、CSS 和 JavaScript 文件。

### SSG 的优势

* **速度快：** 网站是预构建的 HTML 文件，所以访问速度非常快。
* **安全性高：** 没有动态服务器端代码，攻击面更小。
* **部署简单：** 更容易托管和部署。

### 流行的 SSG

1.  Jekyll
2.  Hugo
3.  Gatsby
4.  Next.js (也可以作为 SSG 使用)

## Jekyll 入门

要开始使用 Jekyll，你需要在你的系统上安装 Ruby。

### 安装步骤

1.  安装 Ruby。
2.  安装 Bundler：`gem install bundler`。
3.  在你的项目目录中安装 Jekyll：`bundle install`。

## 自定义你的主题

Minimal Mistakes 主题提供了广泛的自定义选项。

### 布局和包含文件

Jekyll 主题使用**布局**（`_layouts/`）和**包含文件**（`_includes/`）来构建网站结构。你可以通过将它们复制到你项目的相应目录中来覆盖它们。

### Sass 和 CSS

样式通常通过 Sass 进行管理（`_sass/`，`assets/css/`）。你可以修改现有的 Sass 文件或添加自己的自定义 CSS。

---

#### 核心 Front Matter 参数解释（针对 Minimal Mistakes 主题）：

* **`title`**：页面的标题。
* **`layout`**：指定 Jekyll 应该为这个页面使用哪个布局文件（来自 `_layouts/`）。`single` 是 Minimal Mistakes 中用于独立内容页面的常见布局。
* **`permalink`**：定义这个页面的 URL（例如，`你的网站地址/guides/jekyll-detailed-guide/`）。
* **`toc: true`**：**这是核心参数。** 将其设置为 `true` 会告诉 Minimal Mistakes 主题根据页面标题自动生成并显示目录。
* **`toc_label` (可选)**：允许你为目录设置一个自定义标题（例如，“本文目录”）。如果省略，主题将使用默认标题。
* **`toc_icon` (可选)**：允许你为 `toc_label` 旁边指定一个 Font Awesome 图标。确保你的主题正确链接了 Font Awesome。
* **`toc_sticky` (可选)**：如果设置为 `true`，当用户滚动页面时，目录会“粘”在屏幕侧边，保持可见。对于长文档来说，这是一个很棒的用户体验功能。

---

#### 步骤 2：确保你的 `_config.yml` 已设置（通常主题已完成）

Minimal Mistakes（以及大多数设计良好的主题）的 `_config.yml` 通常已经配置好如何处理 Markdown 并使用必要的 Liquid 和 JavaScript 来渲染目录。你通常不需要在 `_config.yml` 中为目录添加任何特定内容，除非你想自定义全局 Kramdown 设置（如 `toc_levels`）。

#### 3. 运行你的 Jekyll 网站

保存 `my-detailed-guide.md` 文件，然后启动 Jekyll 服务器：

``` bash
bundle exec jekyll serve