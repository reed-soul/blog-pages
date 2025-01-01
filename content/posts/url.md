+++
date = '2025-01-01T23:21:16+08:00'
draft = true
title = '你不知道的url'
+++
## 你不知道的 URL

URL（Uniform Resource Locator，统一资源定位符）是互联网中用于标识资源位置和访问方式的字符串。你可能每天都在使用 URL，但你真的了解它的全部细节吗？本文将深入探讨 URL 的各个方面，包括其组成部分、编码规则以及如何通过 JavaScript 操纵 URL。

### 基本概念

#### 什么是 URL？

一个典型的 URL 长这样：

```
http://github.com/path?query=search#/hashPath?hashQuery=hashSearch
```

我们可以将其分解为以下几个部分：

- **协议**：`http`, `https`, `ftp`, `file`, `mailto`, `telnet`, `ssh` 等。
- **主机名**：域名，如 `github.com`。
- **端口号**：如 `80`, `8080`, `8081` 等。
- **路径**：`/path`。
- **查询参数**：`query=search`。
- **哈希值**：`#hashPath`。
- **哈希查询参数**：`hashQuery=hashSearch`。

#### URL 编码

URL 编码用于将特殊字符转换为可以在 URL 中安全传输的格式。以下是常见的编码规则：

| 操作               | 结果                     |
|--------------------|--------------------------|
| 将字符转换为十六进制 | 例如: `A` -> `%41`       |
| 将特殊字符转换为 `%xx` | 例如: 空格 -> `%20`       |
| 将空格转换为 `+`    | 例如: `"Hello World"` -> `"Hello+World"` |
| 将中文转换为 `%xx`  | 例如: `中文` -> `%E4%B8%AD` |

#### 编码与解码方法

JavaScript 提供了以下方法来编码和解码 URL：

- `encodeURIComponent` 和 `decodeURIComponent`：用于编码和解码 URL 中的查询参数和哈希值。
- `encodeURI` 和 `decodeURI`：用于编码和解码整个 URL。

**区别：**

- `encodeURIComponent` 和 `decodeURIComponent` 会编码或解码 URL 中的所有字符，包括 `#`、`/` 等。
- `encodeURI` 和 `decodeURI` 会保留 URL 中的 `#`、`/` 等特殊字符。

**使用示例：**

```javascript
const url = 'http://github.com/path?query=search#/hashPath?hashQuery=hashSearch';

// 编码整个 URL
const encodedUrl = encodeURI(url);
console.log(encodedUrl);
// 输出: http://github.com/path?query=search#/hashPath?hashQuery=hashSearch

// 编码 URL 中的查询参数和哈希值
const encodedParams = encodeURIComponent(url);
console.log(encodedParams);
// 输出: http%3A%2F%2Fgithub.com%2Fpath%3Fquery%3Dsearch%23%2FhashPath%3FhashQuery%3DhashSearch
```

### 操纵 URL 的 JavaScript API

#### `window.location` 对象

`window.location` 对象提供了许多属性和方法，用于获取和操纵当前页面的 URL。

**属性：**

- `href`：当前完整的 URL。
- `protocol`：协议部分（例如：`http:`）。
- `host`：主机名和端口号（例如：`github.com:80`）。
- `hostname`：主机名（例如：`github.com`）。
- `port`：端口号（例如：`80`）。
- `pathname`：路径部分（例如：`/path`）。
- `search`：查询参数部分（例如：`?query=search`）。
- `hash`：哈希值部分（例如：`#hashPath?hashQuery=hashSearch`）。

**方法：**

- `assign(url)`：加载新的文档（相当于 `window.location.href = url`）。
- `replace(url)`：用新的文档替换当前文档（不会在浏览器历史记录中留下记录）。
- `reload()`：重新加载当前页面。

**常见误区：**

- `location.search` 获取的是路径（`pathname`）之后的查询参数，而不是哈希值（`hash`）之后的参数。

```javascript
// 获取查询参数
const query = location.search;
console.log(query);
// 输出: ?query=search

// 获取哈希值
const hash = location.hash;
console.log(hash);
// 输出: #hashPath?hashQuery=hashSearch
```

### 从 URL 中获取查询参数（包括路径和哈希部分）

以下是一个解析 URL 中所有查询参数（包括路径和哈希部分）的函数：

```typescript
function parseUrlWithAllQueryParams(url: string): { query: Record<string, string> } {
  try {
    const urlObject = new URL(url);
    const query: Record<string, string> = {};

    // 解析搜索参数
    urlObject.searchParams.forEach((value, key) => {
      query[key] = value;
    });

    // 解析路径中的参数
    const pathParts = urlObject.pathname.split('/');
    const lastPathPart = pathParts[pathParts.length - 1];
    if (lastPathPart.startsWith('?')) {
      const params = new URLSearchParams(lastPathPart.substring(1));
      params.forEach((value, key) => {
        query[key] = value;
      });
    }

    // 解析哈希参数
    if (urlObject.hash) {
      const hashParams = new URLSearchParams(urlObject.hash.substring(1));
      hashParams.forEach((value, key) => {
        query[key] = value;
      });
    }

    return {
      query: query,
    };
  } catch (e) {
    // 处理无效 URL
    console.error("无效的 URL:", url);
    return {
      query: {},
    };
  }
}

// 使用示例
const url = 'http://github.com/path?query=search#/hashPath?hashQuery=hashSearch';
const { query } = parseUrlWithAllQueryParams(url);
console.log(query);
// 输出: { query: 'search', hashQuery: 'hashSearch' }
```

这个函数使用了 `URL` 和 `URLSearchParams` API，能够方便地解析 URL 中的所有查询参数，包括路径和哈希部分的参数。

希望这篇文章能帮助你更深入地理解 URL 的工作原理，并在实际开发中更好地利用这些知识。
