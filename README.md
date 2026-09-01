# TonyDeng's Blog

[![Deploy Hexo](https://github.com/tonydeng/tonydeng.github.io/actions/workflows/hexo-deploy.yml/badge.svg)](https://github.com/tonydeng/tonydeng.github.io/actions/workflows/hexo-deploy.yml)

基于 [Hexo](https://hexo.io) 的个人技术博客，主题为 [NexT](https://theme-next.js.org)（Mist scheme，通过 git subtree 管理 fork 版本）。源码在 `develop` 分支，GitHub Actions 构建后自动部署到 `master` 分支并发布至 GitHub Pages。

## 环境要求

- Node.js ≥ 20（推荐 20 LTS，与 CI 保持一致）
- npm ≥ 10
- 可选：Git for Windows

## 快速开始（Windows）

```bash
# 1. 克隆并切换到开发分支
git clone git@github.com:tonydeng/tonydeng.github.io.git
cd tonydeng.github.io
git checkout develop

# 2. 安装依赖（依赖已锁定在 package-lock.json，建议用 npm ci 可复现安装）
npm ci

# 3. 本地预览
npx hexo server
# 浏览器打开 http://localhost:4000

# 4. 生成静态站点
npx hexo clean && npx hexo generate
```

## 写一篇新文章

```bash
npx hexo new "文章标题"
# 编辑 source/_posts/YYYY-MM-DD-文章标题.md 后重新生成预览
```

## 部署

推送到 `develop` 分支即触发 GitHub Actions 自动构建并部署（无需本地部署命令）。

如需手动部署（备用方案）：

```bash
npx hexo clean && npx hexo generate
npx hexo deploy
```

## 主题维护（git subtree）

主题使用 git subtree 管理 fork（`tonydeng/hexo-theme-next`）：

```bash
# 首次添加
git remote add -f theme-next git@github.com:tonydeng/hexo-theme-next.git
git subtree add -P themes/next theme-next master

# 拉取上游更新
git subtree pull -P themes/next theme-next master
```

> 注意：当前主题做了定制（字体、CDN、统计等），拉取上游更新前请先确认定制修改无冲突。

## 目录结构

```
source/_posts/   # 博客文章（Markdown）
source/images/   # 文章配图（已做压缩优化）
themes/next/     # NexT 主题（git subtree）
_archive/        # 归档文件（不参与站点发布，如 EA 工程源文件）
_config.yml      # Hexo 主配置
```
