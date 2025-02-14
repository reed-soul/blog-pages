# 博客开发指南

本指南将帮助你了解如何在这个博客项目中添加新文章、进行本地开发和调试，以及部署流程。

## 环境要求

- [Hugo](https://gohugo.io/installation/) (extended version)
- Git
- 代码编辑器 (VS Code, Sublime Text 等)

## 本地开发

1. 克隆仓库
```bash
git clone <your-repo-url>
cd blog-pages
```

2. 本地运行
```bash
hugo server -D
```
这将启动一个本地服务器，通常在 http://localhost:1313 。`-D` 参数表示同时显示草稿文章。

## 添加新文章

1. 创建新文章
```bash
hugo new content/posts/your-post-name.md
```

2. 文章格式
所有文章都应该放在 `content/posts` 目录下，使用 Markdown 格式。每篇文章的头部需要包含以下 front matter：

```yaml
---
title: "文章标题"
date: 2024-03-21
draft: false
tags: ["标签1", "标签2"]
categories: ["分类"]
---
```

3. 添加图片
- 将图片文件放在 `static/images` 目录下
- 在文章中使用相对路径引用：`![图片描述](/images/your-image.jpg)`

## 开发调试

1. 实时预览
- 运行 `hugo server -D` 后，任何文件的修改都会自动重新构建
- 浏览器会自动刷新显示最新内容

2. 调试技巧
- 检查 Hugo 控制台输出的错误信息
- 使用浏览器开发者工具查看页面问题
- 验证 front matter 格式是否正确

## 部署流程

本博客使用 Vercel 进行自动部署：

1. 提交代码到 GitHub
```bash
git add .
git commit -m "你的提交信息"
git push origin main
```

2. Vercel 自动部署
- Vercel 会自动检测到代码更新
- 自动构建并部署新版本
- 可以在 Vercel 仪表板查看部署状态和日志

## 注意事项

1. 文章编写
- 使用 Markdown 语法
- 代码块要指定语言
- 图片建议压缩后再使用
- 中文文件名使用英文命名更好

2. 主题相关
- 主题配置在 `hugo.toml` 中
- 自定义样式可以在 `assets` 目录下添加

3. 部署相关
- 确保 `hugo.toml` 中的 `baseURL` 配置正确
- 提交前先在本地预览确认无误
- 留意 Vercel 的构建日志

## 常见问题

1. 本地预览正常但部署后显示异常
- 检查 `baseURL` 配置
- 检查文件路径大小写
- 查看 Vercel 构建日志

2. 图片显示问题
- 确认图片路径正确
- 检查图片是否已提交到仓库
- 验证图片格式是否支持

如有其他问题，请提交 Issue 或联系维护者。 