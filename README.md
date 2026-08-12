# 我的博客

一个极简风格的 Jekyll 博客，通过 GitHub Pages 免费发布到 `https://<用户名>.github.io`。
外观风格参考 citrusice.github.io：深色纹理页头 + 首页文章列表 + Markdown 写作。

## 目录结构

```
.
├── _config.yml          # 站点配置（标题/作者/域名 都在这改）
├── _layouts/            # 页面模板
├── _includes/           # 页头/页尾等可复用片段
├── _posts/              # ★ 所有文章放这里（Markdown）
├── assets/css/          # 样式，想改外观编辑 style.css
├── images/              # 文章图片放这里
├── index.html           # 首页（自动列出所有文章）
├── about.md             # About 页面
├── 404.html             # 404 页面
└── Gemfile              # 仅本地预览用
```

## 如何发新文章

在 `_posts/` 里新建一个 Markdown 文件，**文件名必须是**：

```
YYYY-MM-DD-文章英文名.md
```

例如 `2026-09-01-my-notes.md`。文件开头写上 front matter：

```markdown
---
title: "文章标题"
categories:
  - 分类名
---

这里是正文，支持所有 Markdown 语法。
首页会自动显示这一段作为摘要（第一个空行之前的内容）。
```

- 文章链接会是 `/posts/文章英文名/`
- 保存文件，`git push` 到 GitHub，1~2 分钟后博客自动更新

## 部署（第一次）

### 1. 改配置
编辑 `_config.yml`，把里面的 `YOUR-USERNAME`、`My Blog`、`Your Name` 换成你自己的。

### 2. 在 GitHub 建仓库
登录 GitHub → 右上角 **+** → **New repository**：
- Repository name **必须**填：`<你的用户名>.github.io`（这样才会发布到根地址）
- 选 **Public**（免费账户的 Pages 需要公开仓库）
- **不要**勾选 Add README

### 3. 推送代码（在本目录打开 PowerShell）

```powershell
git init
git add .
git commit -m "init: blog"
git branch -M main
git remote add origin https://github.com/<你的用户名>/<你的用户名>.github.io.git
git push -u origin main
```

（第一次 push 会弹窗让你登录 GitHub 授权）

### 4. 打开 GitHub Pages
仓库页面 → **Settings** → 左侧 **Pages**：
- Build and deployment → Source 选 **Deploy from a branch**
- Branch 选 `main`，文件夹选 `/ (root)`，点 **Save**

等 1~2 分钟，访问 `https://<你的用户名>.github.io` 就能看到博客了。
之后每次写新文章只需 `git add . ; git commit -m "new post" ; git push`。

## 本地预览（可选）

不装任何东西也能用——直接 push 到 GitHub 看效果。
如果想本地实时预览，两种方式：

**Docker（推荐，最省事）**：

```powershell
docker run --rm -it -p 4000:4000 -v ${PWD}:/srv/jekyll jekyll/jekyll jekyll serve
```

然后浏览器打开 http://localhost:4000

**安装 Ruby**：Windows 上用 [RubyInstaller](https://rubyinstaller.org/) 装 Ruby (含 MSYS2 devkit)，然后：

```powershell
bundle install
bundle exec jekyll serve
```

## 自定义

| 想改什么 | 改哪里 |
|---|---|
| 标题/副标题/作者 | `_config.yml` |
| 导航栏链接 | `_includes/header.html` |
| 颜色/字体/布局 | `assets/css/style.css` |
| About 页内容 | `about.md` |
| 访问量统计 | 去 goatcounter.com 注册，解开 `_includes/head.html` 里的注释 |
| 评论系统 | 推荐 [giscus](https://giscus.app/zh-CN)（基于 GitHub Discussions），把生成的 script 贴进 `_layouts/post.html` |

## 绑定自己的域名（可选）

1. Settings → Pages → Custom domain 填入 `blog.example.com`
2. 到你的 DNS 服务商处添加 CNAME 记录：`blog.example.com` → `<你的用户名>.github.io`
3. 勾选 **Enforce HTTPS**（等 DNS 生效后）

## 常见问题

**Q: push 了但网站没更新？**
A: 检查 Settings → Pages 状态；GitHub Pages 构建需要 1~2 分钟；仓库名必须是 `<用户名>.github.io`。

**Q: 图片怎么放？**
A: 扔到 `images/` 目录，文章里用 `![说明](/images/xxx.png)` 引用。大图建议用图床。

**Q: RSS 订阅？**
A: 自动生成在 `/feed.xml`，首页右上角的 RSS 图标就是。
