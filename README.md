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
├── index.html           # 首页（自动列出所有文章，置顶文章在最前）
├── tags.html            # 标签归档页（自动按标签分组文章）
├── about.md             # About 页面
├── 404.html             # 404 页面
└── Gemfile              # 仅本地预览用
```

## 写文章规范

### 1. 文件命名（必须遵守）

所有文章放在 `_posts/` 目录，文件名格式**必须**是：

```
YYYY-MM-DD-文章英文名.md
```

例如 `2026-09-01-my-notes.md`。规则：

- 日期前缀就是文章的发布日期，必须真实（**不能写成未来的日期**，否则文章不会被构建，会直接"消失"）
- 英文名部分：小写英文 + 数字，单词之间用 `-` 连接，不要空格、中文、下划线或特殊字符
- 英文名会成为文章链接：`/posts/文章英文名/`
- 扩展名统一用 `.md`

### 2. Front matter（文件开头，必须写）

每篇文章开头必须有 front matter（两行 `---` 之间的部分），否则不会作为文章渲染：

```markdown
---
title: "文章标题"
tags:
  - 标签1
  - 标签2
pin: true                       # 可选：置顶
date: 2026-09-01 14:30:00 +0800 # 可选：覆盖文件名里的日期
---

这里是正文……
```

| 字段 | 必填 | 说明 |
|---|---|---|
| `title` | ✅ | 文章标题，中文或英文都行，含特殊字符时用引号包起来 |
| `tags` | 建议 | 标签列表，每行一个 `- 标签名`；首页和文章页显示，`/tags/` 页自动按标签分组 |
| `pin` | 可选 | 写 `pin: true` 即置顶，固定在首页最上方并带"置顶"标记；多篇置顶时按日期倒序；删除这行即取消 |
| `date` | 可选 | 格式 `YYYY-MM-DD HH:MM:SS +0800`，优先级高于文件名日期 |

**日期规则**：

- 不写 `date` 时，日期 = 文件名前缀，时间为当天 00:00（时区按 `_config.yml` 的 `Asia/Shanghai`）
- **同一天发多篇文章**时建议写 `date` 精确到时分，否则排序可能不符合写作顺序

### 3. 摘要规则

首页自动取文章**第一个空行之前的内容**作为摘要（截取前 200 字符）。因此：

- 第一段写纯文字简介，别一上来就放代码块、图片或表格（首页排版会乱）
- 摘要过长会被截断，第一段控制在两三句话内即可

### 4. 正文写作

正文支持完整的 Markdown（GFM）语法，常用的几个注意点：

- **代码高亮**：```` ``` ```` 后写语言名，如 ```` ```c ````、```` ```python ````、```` ```bash ````
- **图片**：图片文件放仓库根目录的 `images/`，文件名用英文，引用写 `![说明](/images/xxx.png)`；大图（> 1MB）建议用图床
- **链接**：站内互链直接写路径，如 `[相关文章](/posts/my-notes/)`；外部链接正常写 URL
- **引用**：`> 内容`
- **表格/删除线/任务列表**：均支持
- 中文和英文/数字之间建议加一个空格，排版更好看

### 5. 标签命名约定

- 标签统一风格，避免同义标签（如别同时出现 `tech` 和 `技术`）
- 建议 1~2 个词，小写英文或中文均可，别太长
- `/tags/` 归档页由模板自动生成，不需要手动维护

### 6. 发布流程

```powershell
git add .
git commit -m "post: 文章标题"
git push
```

push 后 1~2 分钟博客自动更新。commit message 建议 `post: xxx`（新文章）/ `fix: xxx`（修改）格式，方便回看。

### 7. 检查清单

发文章前过一遍：

- [ ] 文件名格式正确：`YYYY-MM-DD-英文名.md`，日期不是未来
- [ ] front matter 完整：`title` 有了，`tags` 写了
- [ ] 第一段是纯文字摘要
- [ ] 本地预览渲染正常（可选），或直接 push 后检查线上页面

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
