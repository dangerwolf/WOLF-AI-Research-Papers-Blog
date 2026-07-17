# wolf-hugo-template-base

一个基于 Hugo 的博客/资料站模板仓库。  
本模板保留了主题能力、目录架构和核心配置，已剥离历史内容（文章、分类、标签、图片等），用于快速创建新站点。

> 来源说明：该模板从个人项目骨架中抽离而来，目标是“可复用、可扩展、可快速初始化”。

---

## 目录

- [项目定位](#项目定位)
- [功能特性](#功能特性)
- [项目结构](#项目结构)
- [快速开始](#快速开始)
- [模板使用方式](#模板使用方式)
- [初始化检查清单](#初始化检查清单)
- [内容与主题开发](#内容与主题开发)
- [部署建议](#部署建议)
- [许可授权（重要）](#许可授权重要)
- [商业授权](#商业授权)
- [贡献说明](#贡献说明)

---

## 项目定位

这个仓库不是现成内容站，而是一个 **Hugo 模板基座**。  
你可以基于它快速生成一个新的博客/Wiki站点，重点复用：

- 主题与视觉结构
- Hugo 配置体系
- 常用插件/扩展设定
- 目录组织与工程化习惯

---

## 功能特性

- 保留 Hugo 项目骨架与主题能力
- 保留可复用配置（菜单、参数、构建相关）
- 清空历史业务内容，避免“旧文章遗留”
- 使用统一占位符（`__TEMPLATE_*__`）进行个性化初始化
- 适合通过 GitHub Template 一键创建新仓库

---

## 项目结构

> 以下为模板推荐结构示例（实际以仓库当前文件为准）：

```text
.
├── archetypes/                # Hugo 内容原型（new 命令模板）
├── assets/                    # Hugo Pipe 资源（SCSS/TS/图片等）
├── config/                    # 分环境或分模块配置（可选）
│   └── _default/              # 默认配置集合（推荐）
├── content/                   # 站点内容（模板中应为“空壳”）
│   ├── _index.md
│   ├── posts/
│   ├── page/
│   └── wiki/
├── data/                      # 数据驱动配置（作者、链接等）
├── layouts/                   # 自定义布局模板（覆盖主题）
├── static/                    # 静态资源（favicon、固定文件）
├── themes/                    # 主题（内置或子模块）
├── .github/workflows/         # CI/CD（可选）
├── TEMPLATE_SETUP.md          # 模板初始化说明
├── LICENSE                    # 许可协议（非商业源码许可）
├── COMMERCIAL_LICENSE.md      # 商业授权说明
└── README.md
```

---

## 快速开始

### 1) 本地运行

先确保安装 Hugo（建议 extended 版本）：

```bash
hugo version
```

启动开发服务器：

```bash
hugo server -D
```

默认访问地址：

- `http://localhost:1313`

### 2) 构建发布文件

```bash
hugo --minify
```

生成目录通常为：

- `public/`

---

## 模板使用方式

### 方式 A：GitHub 模板（推荐）

1. 点击仓库页面上的 **Use this template**
2. 创建你的新仓库
3. 克隆到本地并进入目录
4. 按 `TEMPLATE_SETUP.md` 替换所有占位符
5. 本地运行确认无误后开始写作

### 方式 B：手动复制骨架

1. 克隆本仓库
2. 删除原 git 历史并重新初始化（可选）
3. 替换 `__TEMPLATE_*__` 字段
4. 提交到你自己的新仓库

---

## 初始化检查清单

创建新站后，请全局搜索占位符：

```bash
grep -R "__TEMPLATE_" -n .
```

至少替换以下字段：

- `__TEMPLATE_BASE_URL__`：站点地址
- `__TEMPLATE_SITE_TITLE__`：站点名称
- `__TEMPLATE_SITE_DESCRIPTION__`：站点描述
- `__TEMPLATE_AUTHOR_NAME__`：作者名
- `__TEMPLATE_AUTHOR_EMAIL__`：作者邮箱
- `__TEMPLATE_AVATAR_URL__`：头像链接
- `__TEMPLATE_HOME_TITLE__`：首页标题
- `__TEMPLATE_BUSINESS_EMAIL__`：商业授权联系邮箱
- `__TEMPLATE_CONTACT__`：备用联系方式
- `__TEMPLATE_RESPONSE_TIME__`：响应时间说明

---

## 内容与主题开发

### 新建文章

```bash
hugo new posts/my-first-post.md
```

### 常见内容目录建议

- `content/posts/`：博客文章
- `content/page/`：独立页面（关于、友链等）
- `content/wiki/`：知识条目/文档型内容

### 主题定制建议

- 优先在 `layouts/` 覆盖主题模板，避免直接改 `themes/`
- 样式优先放在 `assets/`，通过 Hugo Pipe 处理
- 保持 `config` 与 `data` 的职责边界（配置 vs 内容数据）

---

## 部署建议

可部署到：

- GitHub Pages
- Netlify
- Vercel
- 自托管 Nginx/对象存储

部署前建议：

1. 确认 `baseURL` 已替换
2. 本地 `hugo --minify` 成功
3. 检查无 `__TEMPLATE_*__` 残留
4. 检查 robots/sitemap/analytics/comment 等配置是否符合预期

---

## 许可授权（重要）

本项目采用：

- **Personal Non-Commercial Source License (PNSL) v1.0**（见 [LICENSE](./LICENSE)）

### 你可以做什么

- 个人学习、研究、非商业用途下免费使用与修改。

### 你不可以做什么（未经书面授权）

- 任何商业使用（包括直接商用、企业使用、客户交付、SaaS/托管、广告或付费变现）
- 二次开发后商用（仍属于受限行为）
- 删除或篡改版权与许可声明

> 说明：本项目为“源码可见（source-available）”，并非 OSI 定义的开源许可。

---

## 商业授权

凡涉及商业目的，请先取得书面授权并完成付费许可。  
请查看并按模板提交申请：

- [COMMERCIAL_LICENSE.md](./COMMERCIAL_LICENSE.md)

未获书面商业授权前，任何商业使用均不被许可。

---

## 贡献说明

欢迎提交 Issue / PR（Bug 修复、文档改进、模板优化）。  
提交贡献即表示你有权提交该内容，且同意项目在许可框架内使用你的贡献。

## 👤 作者

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
