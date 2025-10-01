+++
title = 'Hugo Blog Tutorial'
date = 2025-10-01T14:23:51+08:00
draft = false
+++


## Context
本文记录了我如何从零开始，用 **Hugo + GitHub Pages + PaperMod** 搭建个人博客。  
如果你也想要一个完全免费的、自动部署的技术博客，可以跟着这篇文章一步一步来。  


## Steps
1. 安装Hugo

```bash
# 在 Linux/WSL 下安装 Hugo（extended版）：
sudo apt update
sudo apt install hugo -y

# 检查版本：
hugo version
```

2. 创建博客项目

```bash
# 进入你的工作目录：
hugo new site myblog
cd myblog
```

3. 安装主题PaperMod
```bash
# 把 PaperMod 作为子模块添加：
git init
git submodule add https://github.com/adityatelange/hugo-PaperMod.git themes/PaperMod

# 修改配置文件 hugo.toml，添加主题：
theme = "PaperMod"
```

4. 写一篇文章
```bash
# 创建新文章：
hugo new posts/hello-world.md

# 编辑 content/posts/hello-world.md：
+++
title = "Hello World"
date = 2025-09-29T21:00:00+08:00
draft = false
tags = ["hello", "test"]
categories = ["notes"]
+++

这是我的第一篇博客文章！  
Hugo + PaperMod + GitHub Pages，完美组合 🎉
```

5. 本地预览
```bash
# 运行本地服务器：
hugo server -D

# 然后在浏览器打开：
http://localhost:1313/
```

6. 部署到GitHub Pages
```bash
# 在 GitHub 创建仓库，比如'myblog'

# 添加远程仓库
git remote add origin git@github.com:你的用户名/myblog.git
git branch -M master
git push -u origin master

# 添加 GitHub Actions 工作流'.github/workflows/gh-pages.yml'：
name: Deploy Hugo site to Pages

on:
  push:
    branches:
      - master

permissions:
  contents: write

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4
        with:
          submodules: true
          fetch-depth: 0

      - name: Setup Hugo
        uses: peaceiris/actions-hugo@v3
        with:
          hugo-version: "latest"
          extended: true

      - name: Build
        run: hugo --minify --baseURL "https://你的用户名.github.io/myblog/"

      - name: Deploy
        uses: peaceiris/actions-gh-pages@v3
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: ./public

# 推送代码，等待 Actions 跑完
```

7. 配置GitHub Pages  
到仓库'Settings'→'Pages'，设置'Source'为'gh-pages'分支。  
几分钟后，你就能通过'https://你的用户名.github.io/myblog/'访问你的网站啦。 🎉

8. 基础优化  
修改'hugo.toml'，配置导航栏、社交链接、favicon。  
新建'content/about/index.md'，写“关于我”页面。  
给文章加分类、标签，方便归档。

## 总结
Hugo + PaperMod + GitHub Pages 可以让你快速拥有一个完全免费的、可自定义的个人博客。
接下来你只需要用 Markdown 写文章，每次 git push，网站就会自动更新。
