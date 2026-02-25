# 极简主义博客系统 (Naval-Style)

这是一个基于 React + Tailwind CSS 的极简主义博客系统，灵感来源于 [nav.al](https://nav.al)。

## 项目特点

*   **极简设计**：黑白灰配色，重点突出内容。
*   **Markdown 驱动**：文章内容通过 `.md` 文件管理，无需复杂的后端数据库。
*   **暗黑模式**：完美支持系统自动切换和手动切换。
*   **自定义链接**：支持自定义文章 URL (Slug)，有利于 SEO。
*   **全站搜索**：归档页面支持实时过滤搜索。

## 📂 目录结构

*   `public/posts/`：**核心内容目录**。所有的文章 `.md` 文件和索引配置都在这里。
    *   `index.json`：文章清单（Manifest）。网站根据这个文件来决定显示哪些文章。
    *   `*.md`：具体的文章内容文件。
*   `pages/`：页面组件 (Home, Archive, Post)。
*   `components/`：公共组件 (Layout)。
*   `services/`：数据读取服务。

---

## ✍️ 如何管理文章

### 1. 新增文章

1.  **创建文件**：在 `public/posts` 文件夹内新建一个 Markdown 文件，例如 `my-new-post.md`。
2.  **编写内容**：在文件中使用标准的 Markdown 语法写作。
3.  **注册文章**：打开 `public/posts/index.json`，在数组中添加一个新的对象：

```json
{
  "id": "unique-id-001",           // 唯一ID
  "slug": "my-custom-url",         // 自定义链接，例如 domain.com/my-custom-url
  "title": "我的新文章标题",
  "date": "2024-03-21",            // 显示日期
  "category": "思考",              // 分类
  "summary": "这是文章的简短摘要，会显示在首页和列表中。",
  "source": "my-new-post.md"       // 对应的文件名
}
```

### 2. 删除文章

1.  打开 `public/posts/index.json`。
2.  找到你想删除的文章对应的 `{...}` 块。
3.  将其删除并保存。
4.  (可选) 你可以物理删除 `public/posts/` 文件夹下对应的 `.md` 文件，但只要它不在 `index.json` 中，网站就不会显示它。

### 3. 修改文章链接 (URL)

只需修改 `public/posts/index.json` 中对应文章的 `"slug"` 字段即可。不需要重命名 Markdown 文件。

---

## 🛠 配置与修改

### 修改 Logo 和 页脚

打开 `components/Layout.tsx` 文件：
*   **Logo**: 搜索 `Link to="/"` 附近的代码。
*   **页脚引言**: 搜索文件底部的文字（例如 "财富是你..."）进行修改。
*   **版权信息**: 搜索 `©` 符号进行修改。

### 修改颜色

项目主要使用 Tailwind CSS 的 `neutral` 色板。
*   **强调色**: 鼠标悬停时的红色代码为 `#d93d3d`。你可以在 `pages/Home.tsx`, `pages/Archive.tsx` 和 `components/Layout.tsx` 中搜索这个颜色代码进行替换。

## 📦 部署

本项目是纯静态前端项目（SPA）。
运行构建命令后（通常是 `npm run build`），将生成的 `dist` 或 `build` 文件夹内容上传到任何静态托管服务（如 Vercel, Netlify, GitHub Pages, 或你的 VPS Nginx 目录）即可。

**重要提示：** 
由于这是单页应用（SPA），如果你使用 Nginx，必须在配置中添加 fallback 规则，否则刷新二级页面会报错 404：

```nginx
server {
    listen 80;
    server_name localhost;
    root /usr/share/nginx/html;
    index index.html;

    location /posts/ {
        try_files $uri =404;
    }
    location /assets/ {
        try_files $uri =404;
    }
    location / {
        try_files $uri $uri/ /index.html;
    }
}
```

---

## 2026-02-26 踩坑记录：HashRouter → BrowserRouter 迁移

### 背景

将 URL 从 `https://y.dog/#/slug` 改为干净的 `https://y.dog/slug`。

### 改动点

1. **App.tsx**：`HashRouter` → `BrowserRouter`（一行改动）
2. **Nginx**：新增 `try_files $uri $uri/ /index.html` SPA fallback 配置，通过 Docker volume 挂载
3. **docker-compose.yml**：增加一行 nginx conf 挂载

### 踩坑：react 与 react-dom 版本不匹配（blog-qianduan 仓库）

**现象**：部署后页面白屏，控制台报错：
```
Uncaught TypeError: Cannot read properties of undefined (reading 'S')
```

**原因**：`react` 锁定为 `18.2.0`，但 `react-dom` 写的是 `^19.2.1`，实际安装了 19.2.4。React 18 与 React DOM 19 内部 API 不兼容，打包后运行时崩溃。

**教训**：
- 不要用 `--legacy-peer-deps` 绕过版本冲突，应先排查根因
- `react` 和 `react-dom` 的大版本必须一致
- 部署后应在浏览器中实际验证，`curl` 返回 200 不代表页面能正常渲染