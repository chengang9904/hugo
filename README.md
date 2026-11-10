# codgem 个人博客

一个使用 Hugo 构建的多语言个人博客，支持中文、日文等多种语言，通过 GitHub Actions 自动部署。

- 🌍 **多语言支持**: 中文(简体/繁体)、日文、阿拉伯文等
- 🎨 **精美设计**: 采用 Stack 主题的卡片式设计
- 📱 **响应式布局**: 完美适配各种设备
- 🚀 **自动部署**: GitHub Actions 自动构建和发布
- 🧮 **数学公式**: 支持 LaTeX/KaTeX 数学排版
- 📖 **丰富功能**: 目录、阅读时间、代码高亮等

## 快速开始

### 前置要求
- Hugo extended 版本（0.87.0+）
- Git（用于管理 submodule）

### 本地开发

```bash
# 克隆仓库（包含 submodule）
git clone --recurse-submodules https://github.com/tech-ox/tech-ox.github.io.git
cd tech-ox

# 启动本地开发服务器（包括草稿）
hugo server -D

# 打开浏览器访问
# http://localhost:1313
```

### 创建新文章

```bash
# 创建新文章
hugo new content/post/<文章名>/index.md

# 或创建特定语言的文章
hugo new content/post/<文章名>/index.zh-cn.md

# 编辑文章
# 在 content/post/<文章名>/index.md 中编辑内容
# 将图片放在同目录下，使用相对路径引用：![](image.jpg)
```

### 生成生产版本

```bash
# 生成不包含草稿的生产版本
hugo

# 或包含草稿的开发版本
hugo -D

# 输出文件将保存到 ./public/ 目录
```

## 项目结构

```
.
├── archetypes/           # Hugo 文章模板
├── assets/               # 静态资源（被 Hugo 处理）
├── content/              # 博客内容
│   ├── post/             # 文章
│   ├── page/             # 页面（关于、归档等）
│   └── categories/       # 分类
├── data/                 # 数据文件
├── i18n/                 # 国际化文本
├── layouts/              # 自定义布局（覆盖主题）
├── static/               # 静态资源（直接复制）
├── themes/               # Hugo 主题
│   └── stack/            # Stack 主题（git submodule）
├── hugo.yaml             # Hugo 配置文件
└── public/               # 生成的网站（忽略版本控制）
```

## 配置说明

### 主配置文件 (hugo.yaml)

关键配置项：

- **baseurl**: 网站基础 URL
- **DefaultContentLanguage**: 默认语言（zh-cn）
- **pagerSize**: 每页文章数（10 篇）
- **params.article.math**: 启用数学排版（true）
- **params.article.toc**: 启用目录（true）
- **params.article.readingTime**: 显示阅读时间（true）

### 文章前置元数据（Front Matter）

使用 TOML 格式：

```markdown
+++
date = '2026-01-05T15:30:00+08:00'
title = '文章标题'
draft = false
tags = ['标签1', '标签2']
categories = ['分类名']
image = 'cover.jpg'  # 封面图片
+++

文章内容从这里开始...
```

## 开发指南

### 主题定制

Stack 主题作为 git submodule 管理，**不要直接修改** `themes/stack/` 目录。

自定义主题：
1. 在 `layouts/` 目录中创建同名文件覆盖主题模板
2. 在 `assets/` 目录中添加自定义样式或脚本
3. 在 `static/` 目录中放置静态文件

### Submodule 管理

```bash
# 首次克隆时获取 submodule
git clone --recurse-submodules <repo-url>

# 初始化已存在的 submodule
git submodule update --init --recursive

# 更新主题到最新版本
git submodule update --remote themes/stack

# 切换到特定主题版本
cd themes/stack
git checkout <tag-or-commit>
cd ../..
git add themes/stack
git commit -m "Update theme to <version>"
```

### 构建和测试

```bash
# 本地开发（包含草稿）
hugo server -D

# 生产构建
hugo

# 检查生成的文件
ls -la public/
cat public/CNAME  # 验证 CNAME 配置
```

## 部署

该项目使用 GitHub Actions 自动部署。

### 部署流程

1. 推送代码到 `main` 分支
2. GitHub Actions 自动触发工作流
3. 使用 Hugo extended 构建网站
4. 自动部署到 `chengang9904/chengang9904.github.io` 仓库

### 配置要求

在仓库 Settings 中配置：
- `secrets.token`: GitHub 个人访问令牌（用于部署到外部仓库）

工作流文件位置：`.github/workflows/hugo_deploy.yaml`

## 多语言设置

### 添加新语言

1. 在 `hugo.yaml` 中配置语言：
   ```yaml
   languages:
     ja:
       languageName: 日本語
       weight: 1
   ```

2. 在 `i18n/` 目录中添加翻译文件

3. 使用语言后缀创建内容文件：
   ```
   content/post/my-post/
   ├── index.md          # 中文
   ├── index.zh-cn.md    # 简体中文（同上）
   └── index.ja.md       # 日文
   ```

## 常见问题

### 怎样隐藏草稿文章？

在文章前置元数据中设置 `draft = true`，本地开发需要使用 `-D` 标志才能看到。

**注意**: GitHub Actions 使用 `hugo build` 命令（不加 `-D`），所以默认草稿文章不会部署到生产环境。

### 怎样添加数学公式？

启用后（默认已启用），可以使用 KaTeX 语法：

```markdown
行内公式：$a^2 + b^2 = c^2$

独立公式：
$$
\int_0^{\infty} e^{-x^2} dx = \frac{\sqrt{\pi}}{2}
$$
```

### 怎样自定义菜单？

在文章的前置元数据中添加 `menu` 字段，或修改 `hugo.yaml` 中的 `menu` 配置。

### 构建失败怎么办？

1. 检查 Hugo 版本：`hugo version`（需要 extended 版本）
2. 检查 submodule 是否正确初始化：`git submodule status`
3. 清除缓存重建：`rm -rf resources/` 然后 `hugo`

## 相关资源

- [Hugo 官方文档](https://gohugo.io/documentation/)
- [Stack 主题文档](https://stack.jimmycai.com/)
- [codgem 博客](https://codgem.com/)

## AI 开发指南

本项目包含 `.github/copilot-instructions.md`，为 AI 编程助手提供详细的项目上下文、架构信息和开发流程指引。

## 许可证

本博客内容和代码根据各自的许可证发布。

---

**快速链接**:
- 博客: https://codgem.com
- GitHub: https://github.com/chengang9904
- 源代码仓库: https://github.com/tech-ox/tech-ox
- 部署仓库: https://github.com/chengang9904/chengang9904.github.io
