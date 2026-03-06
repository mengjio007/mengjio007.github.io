# mengjio007.github.io

个人博客，使用 [Hugo](https://gohugo.io/) + [PaperMod](https://github.com/adityatelange/hugo-PaperMod) 主题搭建。

## 本地开发

1. 安装 Hugo（需 Extended 版本）: https://gohugo.io/installation/
2. 克隆仓库并安装依赖：
   ```bash
   git clone https://github.com/mengjio007/mengjio007.github.io.git
   cd mengjio007.github.io
   hugo mod get
   ```
3. 启动本地预览：
   ```bash
   hugo server -D
   ```
4. 访问 http://localhost:1313

## 写作

- 新建文章：`hugo new posts/文章标题.md`
- 文章存放在 `content/posts/` 目录

## 部署

推送到 `main` 分支后，GitHub Actions 会自动构建并部署到 GitHub Pages。

首次使用需在仓库 **Settings → Pages** 中：
- Source 选择 **GitHub Actions**
