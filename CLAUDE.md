# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 项目概述

这是一个基于 Hugo 静态网站生成器的博客项目,使用 hugo-blog-awesome 主题,通过 Vercel 自动部署。

### 技术栈
- **Hugo**: v0.123.6 (extended version)
- **Go**: v1.23.4
- **主题**: hugo-blog-awesome v1.19.0 (作为 Hugo Module 引入)
- **部署**: Vercel
- **语言**: 当前配置为英文 (en-us),但内容主要为中文

## 开发命令

### 本地开发
```bash
# 启动开发服务器 (包含草稿)
hugo server -D
# 访问 http://localhost:1313

# 生产构建
hugo --gc --minify
```

### 创建新文章
```bash
hugo new content/posts/your-post-name.md
```

## 项目架构

### 目录结构
```
content/                 # 所有内容文件
  ├── _index.md         # 首页
  ├── about.md          # 关于页面
  └── posts/            # 博客文章目录

assets/                  # 资源文件 (会被 Hugo 处理)
  ├── avatar.jpg        # 作者头像
  ├── css/extended/     # 自定义 CSS 扩展
  ├── icons/            # 网站图标和 PWA manifest
  └── images/           # 图片资源

hugo.toml               # 主配置文件
vercel.json             # Vercel 部署配置
hugo-blog-awesome.work  # Hugo 工作空间配置 (主题开发)
```

### 主题管理
项目使用 Hugo Modules 管理主题:
- 主题在 `go.mod` 中声明为依赖
- 通过 `hugo-blog-awesome.work` 工作空间配置,支持本地主题开发
- 生产环境使用 GitHub 上的主题模块

### 内容组织
- **文章位置**: `content/posts/`
- **Front Matter 格式**:
  ```yaml
  ---
  title: "文章标题"
  date: 2025-02-14T18:02:31+08:00
  draft: false
  description: "文章描述"
  isStarred: false
  ---
  ```
- **自动生成页面**:
  - `/posts/` - 所有文章列表
  - `/categories/` - 分类索引
  - `/tags/` - 标签索引

## 关键配置

### Hugo 配置 (hugo.toml)
- **baseURL**: https://blog-pages-mu.vercel.app/
- **语言代码**: en-us
- **默认主题**: dark
- **Google Analytics**: G-5S9636T9CQ
- **菜单**: Home (/), Posts (/posts/), About (/about/)
- **作者信息**: Sidharth R (Reed Soul)

### 内容设置
- **代码高亮**: 已启用 (使用 class 方式)
- **目录**: 从 H2 到 H4,默认折叠
- **Go to Top 按钮**: 已启用
- **RSS feed**: 使用摘要模式 (summary)
- **Markdown 渲染**: unsafe=true (支持 HTML)

### Vercel 部署配置
- **HUGO_VERSION**: 0.123.6
- **构建命令**: 自动安装 Go 1.23.4 并执行 `hugo --gc --minify`
- **Clean URLs**: 已启用

## 内容编写规范

### 文章 front matter
所有文章必须包含 front matter,必需字段:
- `title`: 文章标题
- `date`: 发布日期 (ISO 8601 格式)
- `draft`: 是否为草稿 (true/false)

可选字段:
- `description`: 文章描述
- `isStarred`: 是否标记为特色文章
- `tags`: 标签数组
- `categories`: 分类数组

### 图片引用
- 将图片放在 `assets/images/` 或 `static/images/` 目录
- 使用相对路径引用: `![描述](/images/文件名.jpg)`
- 建议压缩图片以优化加载性能

### 代码块
Markdown 代码块应指定语言:
``` ```javascript
console.log('Hello World')
`` ```
```

## 部署流程

项目使用 Vercel 自动部署:
1. 推送代码到 GitHub main 分支
2. Vercel 自动检测并触发构建
3. 使用自定义 Go 1.23.4 + Hugo 0.123.6 构建
4. 自动部署到 https://blog-pages-mu.vercel.app/

## 常见问题排查

### 本地预览正常但部署后异常
- 检查 `hugo.toml` 中的 `baseURL` 配置
- 检查文件路径大小写 (Linux 区分大小写)
- 查看 Vercel 构建日志

### 主题更新
- 更新 `go.mod` 中的主题版本
- 运行 `hugo mod tidy` 更新依赖
- 主题通过 Hugo Modules 自动拉取,无需手动克隆

### 自定义样式
- 在 `assets/css/extended/` 添加自定义 CSS
- 主题会自动加载该目录下的 CSS 文件

## 重要注意事项

1. **文件命名**: 中文文件名在部署时可能产生问题,建议使用英文或拼音命名
2. **日期格式**: 使用 ISO 8601 格式 (如: 2025-02-14T18:02:31+08:00)
3. **草稿预览**: 使用 `-D` 参数启动开发服务器可预览草稿文章
4. **构建缓存**: Vercel 会缓存构建,首次部署可能较慢
5. **Google Analytics**: 已配置 GA4,无需手动添加跟踪代码
