+++
date = '2026-01-05T13:30:31+08:00'
title = 'Hugo + GitHub Actions + Cloudflare 搭建个人博客'
categories = ['折腾']
draft = true
tags = ['hugo', '部署', 'GitHub Actions', 'Cloudflare']
+++


## 前言

本文记录了我使用 Hugo 搭建个人博客，并通过 GitHub Actions 自动部署到 Cloudflare Pages 的全过程。
官方文档参考：
- [Hugo 官方文档](https://gohugo.io/documentation/)


## 安装Hugo


需要安装 `hugo-extended` 版本，我是用命令行安装的
```bash
choco install hugo-extended -confirm
```

安装完成后可以通过 `hugo version` 查看版本信息，确认是否安装成功。

## 创建博客站点
使用命令行进入你想创建博客的目录，执行以下命令创建一个新的 Hugo 站点：

```bash 
hugo new site my-blog
```

这将创建一个名为 `my-blog` 的新目录，里面包含 Hugo 站点的基本结构。

## 选择并安装主题
选择一个你喜欢的 Hugo 主题，可以从 [Hugo Themes](https://themes.gohugo.io/) 网站上浏览和选择主题。
1. 方法1：例如使用 `git` 克隆主题仓库到 `themes` 目录：

```bash
git clone https://github.com/CaiJimmy/hugo-theme-stack.git themes/stack
```

2. 方法2：官方推荐方法，我不推荐，因为在 `git clone` 的时候回多一步骤

```bash
git submodule add https://github.com/CaiJimmy/hugo-theme-stack.git themes/stack
```

3. 配置主题：

我直接在主题里的 `exampleSite/hugo.yaml` 里复制配置到我的站点根目录下的 `hugo.yaml` 文件中，然后根据需要修改站点信息。

也可以直接拷贝我的 `hugo.yaml` 配置文件，然后进行修改

```toml

