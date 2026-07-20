# WOLF AI Research Papers Blog

一个基于 **Hugo** 构建的个人论文研读博客，当前主题集中在：

- 软件工程（Software Engineering）
- 电子表格（Spreadsheet）相关研究论文

站点用于沉淀论文阅读笔记、研究脉络与方法总结，并通过静态站点方式部署到 Cloudflare Pages。

---

## 1. 项目目的

本项目的核心目标：

1. **结构化记录论文研读过程**  
   将论文背景、方法、实验、启发与个人观点统一到标准化文章模板中，便于持续积累。

2. **形成可长期维护的研究博客**  
   使用 Hugo 的内容与模板机制，做到写作、构建、发布分离，降低维护成本。

3. **可复用的技术写作工作流**  
   包括：新建文章模板、Markdown 写作规范、本地预览、构建产物生成、Cloudflare Pages 持续部署。

---

## 2. 项目结构（基于仓库实际目录）

```text
.
├─ archetypes/         # Hugo 新文章模板（hugo new 时使用）
├─ content/            # 文章正文（Markdown）
├─ layouts/            # 自定义页面布局模板（覆盖/扩展主题）
├─ static/             # 静态资源（原样复制到站点根目录）
├─ themes/             # Hugo 主题（含子模块配置）
├─ resources/          # Hugo 资源缓存/管线产物（本地生成）
├─ public/             # Hugo 编译输出目录（最终静态站点文件）
├─ hugo.toml           # Hugo 主配置文件
├─ .gitmodules         # Git 子模块配置（通常用于主题）
└─ .gitignore          # Git 忽略规则
```

> 说明：`public/` 是构建结果目录，Cloudflare Pages 最终应部署这里的内容（或让 CF 在构建流程中自动生成）。

---

## 3. 环境与相关设置

### 3.1 依赖要求

- Hugo（建议使用 **extended** 版本，便于 SCSS/资源管线能力）
- Git（若主题使用子模块，需要支持 submodule）
- 可选：Node.js（若你后续引入前端构建链）

### 3.2 克隆仓库与子模块

如果主题在 `themes/` 下通过子模块管理，首次拉取建议：

```bash
git clone --recurse-submodules https://github.com/dangerwolf/WOLF-AI-Research-Papers-Blog.git
cd WOLF-AI-Research-Papers-Blog
```

若已 clone 但未初始化子模块：

```bash
git submodule update --init --recursive
```

### 3.3 Hugo 配置文件

项目主配置为：

- `hugo.toml`

通常在这里维护：

- `baseURL`
- `languageCode`
- `title`
- 主题配置（theme）
- 菜单、参数、分页、markup 渲染相关项

---

## 4. 撰写文章的细节（推荐工作流）

### 4.1 内容组织建议

建议在 `content/` 下按研究主题分类，例如：

```text
content/
└─ software-engineering/
   └─ spreadsheet/
      ├─ paper-a.md
      └─ paper-b.md
```

这样便于后续生成专题列表页与导航。

### 4.2 Front Matter 建议字段

每篇文章建议包含（可按你实际模板增减）：

- `title`: 文章标题
- `date`: 创建时间（ISO 格式）
- `lastmod`: 最后更新时间
- `draft`: 是否草稿
- `tags`: 标签数组（如方法、任务类型）
- `categories`: 分类数组
- `summary`: 摘要（用于列表页）
- `authors` / `paper` / `venue`（可选，自定义论文元信息）

示例：

```yaml
---
title: "论文标题：XXXX"
date: 2026-07-20T10:00:00+08:00
lastmod: 2026-07-20T10:00:00+08:00
draft: true
tags: ["software-engineering", "spreadsheet", "program-synthesis"]
categories: ["paper-reading"]
summary: "一句话总结论文核心贡献。"
---
```

### 4.3 论文研读正文建议结构

建议固定一个写作骨架，提高输出一致性：

1. 研究问题（Problem）
2. 背景与动机（Background / Motivation）
3. 核心方法（Method）
4. 实验与结果（Experiments）
5. 局限性（Limitations）
6. 我的理解与启发（Takeaways）
7. 参考链接（论文、代码、数据集）

---

## 5. 如何新建文章

在 Hugo 项目根目录执行：

```bash
hugo new content/software-engineering/spreadsheet/my-paper-note.md
```

或（旧写法也常见）：

```bash
hugo new software-engineering/spreadsheet/my-paper-note.md
```

生成后会自动套用 `archetypes/` 中对应模板（若存在）。

然后编辑该 Markdown 文件，补全 Front Matter 并开始写作。

---

## 6. 本地预览与 Hugo 编译

### 6.1 本地开发预览

```bash
hugo server -D
```

说明：

- `-D` 会包含 `draft: true` 的草稿文章
- 本地访问地址通常是 `http://localhost:1313`

### 6.2 生产构建

```bash
hugo --minify
```

构建结果输出到：

- `public/`

如果要清理历史产物后再构建，可先删除 `public/` 再执行构建。

---

## 7. 部署到 Cloudflare Pages（技术细节）

你可以采用 **Git 集成自动部署**（推荐）。

### 7.1 在 Cloudflare Pages 创建项目

1. 登录 Cloudflare Dashboard
2. 进入 **Pages** → **Create a project**
3. 选择并连接 GitHub 仓库：`dangerwolf/WOLF-AI-Research-Papers-Blog`

### 7.2 构建配置（Hugo 项目）

在 Pages 构建设置中配置：

- **Framework preset**: Hugo（如可选）
- **Build command**:  
  `hugo --minify`
- **Build output directory**:  
  `public`

### 7.3 环境变量（建议）

可按需设置（示例）：

- `HUGO_VERSION`：固定 Hugo 版本，避免云端与本地版本漂移
- `HUGO_ENV=production`
- `HUGO_ENABLEGITINFO=true`（如你模板中使用 Git 信息）

> 关键建议：固定 `HUGO_VERSION`，确保 Cloudflare 构建结果与本地一致。

### 7.4 分支与发布策略

- 生产分支：`main`
- 每次 push 到 `main` 自动触发 Pages 构建与发布
- 如需预览环境，可开启 PR Preview（Cloudflare 自动生成预览链接）

### 7.5 自定义域名

若你有自定义域名（如仓库 metadata 中的主页域名）：

1. Pages 项目 → **Custom domains**
2. 绑定域名
3. 按提示添加/校验 DNS 记录
4. 等待证书签发与生效（Cloudflare 自动托管 TLS）

---

## 8. 常见问题排查

1. **主题样式丢失**  
   - 检查 `themes/` 是否完整（子模块是否初始化）
   - 检查 `hugo.toml` 中 theme 配置是否正确

2. **Cloudflare 构建失败（Hugo 版本不兼容）**  
   - 在 Pages 环境变量中固定 `HUGO_VERSION`
   - 保证本地与云端版本一致

3. **文章没显示**  
   - 是否仍是 `draft: true`
   - 发布构建是否未使用 `-D`（生产环境通常不应包含草稿）

4. **静态资源 404**  
   - 文件应放在 `static/` 下
   - 引用路径应基于站点根路径检查

---

## 9. 维护建议

- 保持 `content/` 分类清晰，按研究方向分层目录
- 统一论文笔记模板，保证长期可检索性
- 定期升级 Hugo 与主题，但每次升级后先本地完整构建验证
- 发布前本地执行一次 `hugo --minify` 做最终检查

---

## 10. License

项目许可信息见仓库中的：

- `LICENSE`
- `COMMERCIAL_LICENSE.md`

## 11. 作者

**dangerwolf**

- GitHub: [@dangerwolf](https://github.com/dangerwolf)

---


# 友情赞助商

1. [![Powered by DartNode](https://dartnode.com/branding/DN-Open-Source-sm.png)](https://dartnode.com "Powered by DartNode - Free VPS for Open Source")

2. 开发者需要免费服务的，推荐使用每个月免费$5额度的云服务。
[https://console.run.claw.cloud/signin?link=XHJEEP7HEVIR](https://console.run.claw.cloud/signin?link=XHJEEP7HEVIR)

3. [Cloudcone VPS](https://app.cloudcone.com/?ref=12850)

4. [![This Website is Powered by DigitalPlat FreeDomain Get a free domain from DigitalPlat.](https://img.shields.io/badge/DigitalPlat-Get%20a%20free%20domain%20from%20DigitalPlat.-2563eb?style=flat-square&logo=databricks&logoColor=ffffff)](https://dash.domain.digitalplat.org/signup?ref=3lT6CU6HGe)

5. [DMIT](https://www.dmit.io/aff.php?aff=23825)

---

如果你基于本模板创建了新站，欢迎保留来源引用并告知使用场景。祝你创作顺利。
