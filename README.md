# Hugo Blog

这是一个基于 Hugo 的静态博客项目，主题使用 hugo-blog-awesome，并通过 Vercel 自动部署。

## 技术栈

- Hugo (extended) 0.123.6
- Go 1.23.4
- 主题：hugo-blog-awesome（Hugo Module）
- 部署：Vercel

## 本地开发

```bash
hugo server -D
```

## 生产构建

```bash
hugo --gc --minify
```

## 内容管理

- 文章目录：`content/posts/`
- 页面：`content/_index.md`、`content/about.md`
- 资源：`assets/`

## 新建文章

```bash
hugo new content/posts/your-post-name.md
```
