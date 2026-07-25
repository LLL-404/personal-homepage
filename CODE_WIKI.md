# 个人主页 — Code Wiki

> 一个基于 Hugo 的静态个人主页项目，融合字体驱动美学与 Codrops 风格创意动效，支持中英双语、亮暗双主题、Decap CMS 可视化编辑，并通过 GitHub Actions 自动部署至 GitHub Pages。

---

## 目录

- [1. 项目概览](#1-项目概览)
- [2. 技术栈与外部依赖](#2-技术栈与外部依赖)
- [3. 目录结构](#3-目录结构)
- [4. 整体架构](#4-整体架构)
- [5. Hugo 配置详解（hugo.toml）](#5-hugo-配置详解hugotoml)
- [6. 内容层（content/ + i18n/）](#6-内容层content--i18n)
- [7. 布局模板层（layouts/）](#7-布局模板层layouts)
- [8. 前端 JS 模块（关键类与函数）](#8-前端-js-模块关键类与函数)
- [9. 样式系统](#9-样式系统)
- [10. Decap CMS 集成](#10-decap-cms-集成)
- [11. 部署流程（GitHub Actions）](#11-部署流程github-actions)
- [12. 项目运行方式](#12-项目运行方式)
- [13. 已知问题与改进方向](#13-已知问题与改进方向)

---

## 1. 项目概览

**项目定位**：托管在 GitHub 上的个人主页，集合作品集、博客、简历、关于、联系五大模块。

**设计理念**：
- **字体驱动**：衬线字体（Playfair Display）作为视觉主角，超大字号呈现品牌名，引言作副锚点
- **动效克制而惊艳**：GSAP + Three.js + Lenis 组合实现 Codrops 级别交互体验
- **双内容编辑路径**：AI Agent 可直接读写 Markdown 文件；非技术用户可通过 Decap CMS 可视化编辑
- **双语独立 URL**：`/zh/` 与 `/en/` 前缀分离，避免内容耦合

**两套资源体系（重要）**：
项目同时存在两套前端资源组织方式，反映设计演进过程：

| 资源体系 | 位置 | 用途 | 动效驱动方式 |
|---------|------|------|-------------|
| **Hugo 内嵌体系** | `layouts/partials/head.html`、`layouts/partials/scripts.html` | Hugo 构建产物使用 | Lenis + ScrollTrigger 滚动驱动 |
| **独立预览体系** | `public/index.html`、`public/assets/{css,js}/` | 单文件预览 + 拆分资源 | PageTransition + PageAnims 页面切换驱动 |

后者是演进后的版本，引入了页面切换引擎替代纯滚动动画。两套体系在功能上等价但实现路径不同。

---

## 2. 技术栈与外部依赖

### 核心框架
- **Hugo** `0.128.0`（extended 版本，含 Dart Sass 支持）— 静态站点生成器
- **Decap CMS** `^3.4.0` — 基于 Git 的可视化内容管理系统

### 前端运行时依赖（CDN 引入）
所有外部库通过 `layouts/partials/scripts.html` 中的 `<script>` 标签从 CDN 加载，**无 npm 依赖**：

| 库 | 版本 | 用途 |
|----|------|------|
| GSAP | 3.12.5 | 主力动画引擎 |
| ScrollTrigger | 3.12.5 | GSAP 滚动驱动插件 |
| Flip | 3.12.5 | GSAP 布局过渡插件 |
| Lenis | 1.1.13 | 平滑滚动库 |
| Three.js | r128 | Hero 区块粒子背景 |

### 字体（Google Fonts）
- **Playfair Display**（衬线，标题/引言）
- **Inter**（无衬线，正文）
- **JetBrains Mono**（等宽，标签/日期/代码）

### 构建与部署
- **GitHub Actions** — CI/CD
- **GitHub Pages** — 托管

---

## 3. 目录结构

```
个人主页/
├── hugo.toml                    # Hugo 站点配置（双语、菜单、参数）
├── content/                     # Markdown 内容源
│   ├── zh/                      # 中文内容
│   │   ├── _index.md
│   │   ├── about/_index.md
│   │   ├── contact/_index.md
│   │   ├── posts/               # 博客文章（4 篇示例）
│   │   ├── projects/            # 项目（5 个示例）
│   │   └── resume/_index.md
│   └── en/                      # 英文内容（结构对称）
├── i18n/                        # i18n 翻译文件
│   ├── zh.toml
│   └── en.toml
├── layouts/                     # Hugo 模板
│   ├── _default/baseof.html     # 基础布局骨架
│   ├── index.html               # 首页模板
│   ├── partials/                # 可复用片段
│   │   ├── head.html            # <head> 内嵌完整 CSS
│   │   ├── header.html          # 顶部导航
│   │   ├── footer.html          # 页脚
│   │   ├── cursor.html          # 自定义光标 DOM
│   │   └── scripts.html         # <body> 末尾内嵌完整 JS
│   ├── about/single.html
│   ├── contact/single.html
│   ├── posts/{list,single}.html
│   ├── projects/{list,single}.html
│   └── resume/single.html
├── static/                      # 静态资源（原样拷贝至 public/）
│   └── admin/                   # Decap CMS 入口
│       ├── index.html
│       └── config.yml
├── public/                      # 构建产物 + 单文件预览
│   ├── index.html               # 单文件预览版（内联全部 CSS/JS）
│   └── assets/
│       ├── css/                 # 拆分后的样式（6 个文件）
│       └── js/                  # 拆分后的脚本（10 个模块）
├── .github/workflows/
│   └── hugo-deploy.yml          # GitHub Pages 部署工作流
├── .trae/specs/                 # 项目规格说明（Spec/Task/Checklist）
├── .vscode/launch.json          # VSCode 调试配置
└── .gitignore
```

---

## 4. 整体架构

### 4.1 分层架构

```
┌─────────────────────────────────────────────────────┐
│  内容层 (content/)  —  Markdown + Frontmatter       │
│  + i18n 翻译层 (i18n/*.toml)                        │
└─────────────────────────────────────────────────────┘
                          │
                          ▼ Hugo 构建
┌─────────────────────────────────────────────────────┐
│  模板层 (layouts/)  —  Hugo Go Templates            │
│  ├── _default/baseof.html  (骨架)                   │
│  ├── partials/             (复用片段)                │
│  └── {section}/{list,single}.html                   │
└─────────────────────────────────────────────────────┘
                          │
                          ▼ 渲染
┌─────────────────────────────────────────────────────┐
│  表现层  —  内嵌 CSS (head.html) + 内嵌 JS (scripts.html) │
│  运行时依赖：GSAP / Lenis / Three.js (CDN)           │
└─────────────────────────────────────────────────────┘
                          │
                          ▼ 部署
┌─────────────────────────────────────────────────────┐
│  GitHub Pages  —  public/ 产物                       │
└─────────────────────────────────────────────────────┘
```

### 4.2 请求与渲染流程

1. 用户访问 `https://username.github.io/`
2. Hugo 根据 `hugo.toml` 中 `languages.zh.weight = 1` 将 `/` 重定向至中文首页
3. 渲染 `layouts/index.html`（继承 `baseof.html`），注入 `content/zh/_index.md` 内容
4. `baseof.html` 依次加载 partials：`head`（CSS）→ `cursor`（光标 DOM）→ `header`（导航）→ `main` block → `footer` → `scripts`（JS + CDN）
5. 浏览器执行 JS：`DOMContentLoaded` 后按顺序初始化 9 个模块（详见第 8 节）

### 4.3 内容编辑流程

**路径 A — AI Agent / 开发者直接编辑**：
```
修改 content/zh/posts/xxx.md  →  git commit  →  GitHub Actions 触发  →  Hugo 重新构建  →  Pages 更新
```

**路径 B — 非技术用户通过 CMS**：
```
访问 /admin/  →  Decap CMS 加载  →  OAuth 登录 GitHub  →  Web UI 编辑  →  提交到 main 分支  →  触发 CI
```

---

## 5. Hugo 配置详解（hugo.toml）

[hugo.toml](file:///d:/G/github/个人主页/hugo.toml) 是站点核心配置，关键配置块说明如下：

### 5.1 站点基础
```toml
baseURL = "https://username.github.io/"
languageCode = "zh-cn"
title = "Your Name"
theme = ""   # 不使用外部主题，使用自定义布局
```
> 注意：`theme = ""` 表示项目完全依赖 `layouts/` 目录下的自定义模板，无外部主题依赖。

### 5.2 双语配置
```toml
[languages.zh]
languageName = "中文"
weight = 1            # 默认语言
contentDir = "content/zh"

[languages.en]
languageName = "English"
weight = 2
contentDir = "content/en"
```
- `weight` 决定默认语言（中文 weight=1，访问 `/` 即中文版）
- `contentDir` 分离中英内容，互不污染

### 5.3 菜单配置
每语言独立定义 6 项菜单：首页、作品集、博客、简历、关于、联系。英文菜单 URL 带 `/en/` 前缀。

### 5.4 站点参数 `[params]`
```toml
author = "Your Name"
description = "个人主页 - 作品集、博客、简历"
github = "https://github.com/username"
email = "email@example.com"
```
这些参数通过 `.Site.Params.xxx` 在模板中引用，例如首页 Hero 区块的作者名、联系区块的邮箱与 GitHub 链接。

---

## 6. 内容层（content/ + i18n/）

### 6.1 Markdown 文件结构

每个内容文件由 **Frontmatter**（YAML 元信息）+ **Markdown 正文** 组成：

```markdown
---
title: "项目一"                  # 标题（必需）
date: 2024-01-10                 # 发布日期
draft: false                     # 是否草稿
description: "一个很棒的项目"     # 摘要（列表页展示）
tags: ["Python", "Web"]          # 标签
category: "Web应用"              # 分类
tech_stack: ["Python", "Flask"]  # 项目专属：技术栈
link: "https://github.com/..."   # 项目专属：外链
---

正文内容（Markdown）...
```

### 6.2 各 Section 字段差异

| Section | 特有字段 | 列表页展示 |
|---------|---------|-----------|
| `posts` | `tags`, `category` | 日期 + 标题 + 摘要 |
| `projects` | `tech_stack`, `link` | 编号 + 标题 + 摘要 + 标签 |
| `resume` | （正文即简历全文） | 时间线（从正文解析） |
| `about` / `contact` | （正文即页面内容） | 单页渲染 |

### 6.3 `_index.md` 的作用

`_index.md` 表示该 section 的列表页内容。例如 `content/zh/posts/_index.md` 渲染为中文博客列表页。若仅有 `_index.md` 而无子页面（如 `about/`、`contact/`、`resume/`），则该 section 只有单页。

### 6.4 i18n 翻译层

[i18n/zh.toml](file:///d:/G/github/个人主页/i18n/zh.toml) 与 [i18n/en.toml](file:///d:/G/github/个人主页/i18n/en.toml) 定义了 10 个翻译键：

| 键 | 中文 | 英文 |
|----|------|------|
| `nav_home` | 首页 | Home |
| `nav_projects` | 作品集 | Projects |
| `nav_blog` | 博客 | Blog |
| `nav_resume` | 简历 | Resume |
| `nav_about` | 关于 | About |
| `nav_contact` | 联系 | Contact |
| `theme_toggle` | 切换主题 | Toggle theme |
| `language_toggle` | English | 中文 |
| `read_more` | 阅读更多 | Read more |
| `back_to_list` | 返回列表 | Back to list |
| `download_pdf` | 下载 PDF | Download PDF |

> ⚠️ **已知问题**：模板中实际使用的 i18n 键（如 `hero_tagline`、`portfolio_label`、`blog_title` 等）**未在 i18n 文件中定义**。这些键的值目前硬编码在 `scripts.html` 的 `I18n.dict` 对象中（见第 8.2 节），通过 JS 在客户端动态替换。Hugo 的 `i18n` 函数会因找不到键而返回空字符串，导致首屏闪烁。

---

## 7. 布局模板层（layouts/）

### 7.1 模板继承关系

```
baseof.html (骨架)
├── partial: head.html      (内嵌全部 CSS)
├── partial: cursor.html    (光标 DOM)
├── partial: header.html    (导航)
├── block "main"            (各页面覆写)
│   ├── index.html          (首页)
│   ├── posts/list.html     (博客列表)
│   ├── posts/single.html   (博客详情)
│   ├── projects/list.html  (项目列表)
│   ├── projects/single.html(项目详情)
│   ├── resume/single.html  (简历)
│   ├── about/single.html   (关于)
│   └── contact/single.html (联系)
├── partial: footer.html    (页脚)
└── partial: scripts.html   (内嵌全部 JS + CDN)
```

### 7.2 关键模板说明

#### [baseof.html](file:///d:/G/github/个人主页/layouts/_default/baseof.html)
基础骨架，所有页面继承。设置 `<html data-theme="dark">`（默认暗色），引入 Google Fonts，通过 `{{ block "main" . }}` 留出内容插槽。

#### [index.html](file:///d:/G/github/个人主页/layouts/index.html)
首页模板，**单页式**呈现所有区块（不跳转）：
- Hero（粒子背景 + 大字名字 + 引言）
- Portfolio（取前 4 个项目）
- Blog（取前 3 篇文章）
- Resume（时间线）
- About（引言 + 技能）
- Contact（邮箱 + GitHub）

关键模板函数：
- `{{ range first 4 (where .Site.RegularPages "Section" "projects") }}` — 取前 4 个项目
- `{{ i18n "hero_tagline" }}` — 国际化文案
- `{{ printf "%03d" (add .Ordinal 1) }}` — 项目编号格式化为 `001`、`002`

#### [header.html](file:///d:/G/github/个人主页/layouts/partials/header.html)
顶部固定导航，包含：
- 品牌名（`.Site.Params.brand | default "YN."`）
- 菜单链接（`{{ range .Site.Menus.main }}`）
- 语言切换按钮（`#langToggle`）
- 主题切换按钮（`#themeToggle`，含 SVG 图标）

#### [head.html](file:///d:/G/github/个人主页/layouts/partials/head.html)
**内嵌完整 CSS**（约 690 行），包括：
- CSS 变量定义（字体族、间距、过渡曲线）
- 亮/暗主题变量（`[data-theme="light"]` / `[data-theme="dark"]`）
- 基础重置 + 自定义光标样式
- 各区块样式（Hero、Portfolio、Blog、Timeline、About、Contact）
- 响应式适配（1024px / 768px 断点）
- `prefers-reduced-motion` 无障碍支持

#### [scripts.html](file:///d:/G/github/个人主页/layouts/partials/scripts.html)
**内嵌完整 JS**（约 650 行），通过 CDN 引入 GSAP/Lenis/Three.js，然后定义 9 个模块并按序初始化。详见第 8 节。

### 7.3 模板中常用的 Hugo 函数

| 函数 | 用途 | 示例 |
|------|------|------|
| `{{ .Title }}` | 当前页标题 | 渲染 `<title>` |
| `{{ .Site.Params.xxx }}` | 站点参数 | 引用 `author`、`email` |
| `{{ i18n "key" }}` | 国际化 | `{{ i18n "blog_title" }}` |
| `{{ range .Pages }}` | 遍历子页面 | 列表页渲染 |
| `{{ .RelPermalink }}` | 页面相对 URL | 文章链接 |
| `{{ .Date.Format "2006.01.02" }}` | 日期格式化 | 博客日期 |
| `{{ .Content }}` | 渲染 Markdown 正文 | 详情页主体 |
| `{{ .TableOfContents }}` | 自动生成目录 | 博客详情页 |
| `{{ partial "name" . }}` | 引用 partial | 加载头部/尾部 |
| `{{ block "main" . }}` | 定义可覆写块 | baseof 留插槽 |

---

## 8. 前端 JS 模块（关键类与函数）

所有 JS 内嵌在 [layouts/partials/scripts.html](file:///d:/G/github/个人主页/layouts/partials/scripts.html) 中，采用**模块化对象字面量**模式（非 ES Module），每个模块暴露为全局变量。

### 8.1 模块清单与初始化顺序

`DOMContentLoaded` 后按以下顺序初始化：

| 序号 | 模块 | 职责 | 兜底策略 |
|------|------|------|---------|
| 1 | `Theme` | 主题切换 | localStorage 优先，否则 `prefers-color-scheme` |
| 2 | `I18n` | 语言切换 | localStorage 优先，否则默认 `zh` |
| 3 | `Cursor` | 自定义光标 | 移动端自动隐藏 |
| 4 | `HeroName` | Hero 名字动效 | GSAP 失败则直接显示 |
| 5 | `ParticleBg` | Three.js 粒子 | Three.js 失败则跳过 |
| 6 | `SmoothScroll` | Lenis 平滑滚动 | Lenis 失败用原生滚动 |
| 7 | `ScrollAnims` | 滚动动画 | GSAP 失败则 `fallbackReveal()` |
| 8 | `ProjectMask` | 项目卡片遮罩 | 纯 CSS clip-path，无依赖 |

### 8.2 各模块详解

#### `Utils` — 工具函数
```javascript
const Utils = {
    has(libraryName)  // 检测全局库是否加载（CDN 可能失败）
    rafThrottle(fn)   // requestAnimationFrame 节流包装
}
```
所有模块通过 `Utils.has('gsap')` 检测 CDN 加载状态，是整个兜底机制的基础。

#### `I18n` — 国际化模块
```javascript
const I18n = {
    current: 'zh',           // 当前语言
    dict: { zh: {...}, en: {...} },  // 文案字典（约 30 个键）
    apply(lang)              // 应用语言到所有 [data-i18n] 节点
    toggle()                 // 中英切换
    init()                   // 读取 localStorage 或默认 zh
}
```
**工作原理**：扫描所有 `data-i18n="key"` 属性的元素，用 `dict[lang][key]` 替换 `textContent`。同时更新 `<html lang>` 与切换按钮文字，并持久化到 `localStorage`。

> 这套客户端 i18n 与 Hugo 的 `i18n` 函数并行存在，导致部分文案双轨制（详见 6.4 已知问题）。

#### `Theme` — 主题切换
```javascript
const Theme = {
    moonPath, sunPath,       // 月亮/太阳 SVG path
    apply(theme)             // 设置 data-theme 属性 + 更新图标 + 持久化
    toggle()                 // dark ↔ light
    init()                   // localStorage → prefers-color-scheme → 默认 dark
}
```
**默认主题**：暗色（`dark`）。亮色检测依赖 `window.matchMedia('(prefers-color-scheme: light)')`。

#### `Cursor` — 自定义光标
```javascript
const Cursor = {
    el, x, y, targetX, targetY,
    ease: 0.15,              // 缓动系数
    init()                   // 绑定 mousemove + hover 选择器
    render()                 // rAF 循环：缓动跟随
}
```
**实现要点**：
- 通过 `ease` 系数实现缓动跟随（不是即时定位）
- hover 选择器：`a, button, .project-card, .blog-item, .skill-tag, .contact-link`
- 移动端通过 CSS `@media (hover: none)` 隐藏

#### `HeroName` — Hero 名字动效
```javascript
const HeroName = {
    init()        // 拆分字符为 span + GSAP stagger 渐入 + 绑定 3D 倾斜
    bindTilt()    // 鼠标移动 → rotateX/rotateY（最大 8 度）
}
```
**关键逻辑**：
1. 将名字文本拆分为单字 `<span class="char">`
2. GSAP `stagger: 0.06` 实现逐字渐入
3. `Utils.rafThrottle` 节流鼠标移动事件
4. 鼠标相对中心的归一化坐标 `[-1, 1]` × `maxTilt * 2` 计算倾斜角度

#### `ParticleBg` — Three.js 粒子背景
```javascript
const ParticleBg = {
    scene, camera, renderer, particles,
    mouse: { x: 0, y: 0 },
    _frameCount: 0,
    init()              // 创建场景、相机、渲染器、粒子点云
    createParticles()   // 600 个三维随机分布粒子
    bindEvents()        // 鼠标扰动 + 窗口 resize
    animate()           // rAF 循环：旋转 + 鼠标扰动（每 2 帧更新一次）
    updateColor()       // 主题切换时更新粒子颜色
}
```
**性能优化**：
- `_frameCount % 2 === 0` — 每 2 帧才更新粒子位置，减半计算量
- `Math.min(window.devicePixelRatio, 2)` — 限制像素比上限避免高 DPI 设备过载
- 粒子位置缓动逼近目标值（系数 0.04），产生柔和扰动

#### `SmoothScroll` — 平滑滚动
```javascript
const SmoothScroll = {
    lenis: null,
    init()   // 创建 Lenis 实例 + 与 ScrollTrigger 集成
}
```
**集成方式**：将 `lenis.on('scroll', ScrollTrigger.update)` 桥接，并用 `gsap.ticker.add` 驱动 `lenis.raf`，避免双 rAF 竞争。

#### `ScrollAnims` — 滚动动画
```javascript
const ScrollAnims = {
    init()                  // 注册 ScrollTrigger + 调用 5 个子方法
    fallbackReveal()        // CDN 失败兜底：直接显示所有隐藏元素
    animateSectionReveal()  // 通用区块标题渐入
    animateProjectCards()   // 项目卡片 3D 旋转渐入（rotateY 15° → 0°）
    animateBlog()           // 博客列表项渐入
    animateTimeline()       // 时间线：线条 scaleY 生长 + 节点逐个渐入
    animateAbout()          // 关于引言逐行从下往上揭示
    animateContact()        // 联系标题渐入
}
```
**关键技术**：
- `gsap.set()` 预设初始状态 + `gsap.to()` 动画到目标值（避免 `gsap.from()` 的闪烁问题，详见 13 节）
- `scrollTrigger.once: true` — 动画只触发一次
- `scrollTrigger.scrub: 0.5` — 时间线生长与滚动位置绑定

#### `ProjectMask` — 项目卡片遮罩
```javascript
const ProjectMask = {
    init()   // 给每张卡片绑定 mouseenter/mouseleave 的 clip-path 圆形扩散
}
```
**效果**：hover 时 `clip-path: circle(0% → 150%)` 从中心扩散，揭示遮罩层。

### 8.3 资源加载与兜底机制

```
CDN 加载 GSAP/Lenis/Three.js
        │
        ├─ 成功 → Utils.has() 返回 true → 启用完整动效
        │
        └─ 失败 → Utils.has() 返回 false → 各模块走兜底：
                  • HeroName: 直接显示文字
                  • ParticleBg: 跳过初始化
                  • SmoothScroll: 使用原生滚动
                  • ScrollAnims: fallbackReveal() 显示所有内容
```

这一设计确保即使 CDN 被墙或网络异常，页面内容仍可访问，仅丢失动效。

---

## 9. 样式系统

### 9.1 CSS 变量体系

所有样式token集中在 [head.html](file:///d:/G/github/个人主页/layouts/partials/head.html) 顶部的 `:root` 与主题变量块：

```css
:root {
    /* 字体族 */
    --font-serif: 'Playfair Display', serif;
    --font-sans: 'Inter', sans-serif;
    --font-mono: 'JetBrains Mono', monospace;

    /* 间距系统（5 级） */
    --space-xs: 0.5rem;  --space-sm: 1rem;
    --space-md: 2rem;    --space-lg: 4rem;  --space-xl: 8rem;

    /* 过渡曲线 */
    --ease-out: cubic-bezier(0.16, 1, 0.3, 1);
    --ease-in-out: cubic-bezier(0.65, 0, 0.35, 1);
}

[data-theme="light"] { --bg: #fafafa; --fg: #0a0a0a; ... }
[data-theme="dark"]  { --bg: #0a0a0a; --fg: #ededed; ... }
```

### 9.2 主题切换实现

通过 `<html data-theme="dark|light">` 属性切换，所有颜色变量自动联动。`body` 设置过渡：
```css
transition: background-color 0.4s var(--ease-out), color 0.4s var(--ease-out);
```

### 9.3 响应式断点

| 断点 | 调整内容 |
|------|---------|
| `1024px` | 区块内边距减小；博客网格列宽收窄 |
| `768px` | 隐藏导航链接；项目卡片单列；博客项单列；联系链接纵向排列；Hero 字号缩小 |
| `prefers-reduced-motion` | 所有动画时长强制为 `0.01ms` |

### 9.4 性能优化手段

- `content-visibility: auto` + `contain-intrinsic-size` — 区块懒渲染
- `contain: layout style` — 项目卡片样式隔离
- `backdrop-filter: blur(12px)` — 导航毛玻璃效果
- `mix-blend-mode: difference` — 自定义光标在任何背景上可见
- `transform-style: preserve-3d` — Hero 名字 3D 倾斜

### 9.5 独立资源版本（public/assets/css/）

`public/assets/css/` 下拆分为 6 个文件，是单文件预览版的样式拆分：

| 文件 | 职责 |
|------|------|
| `main.css` | 主样式（变量、基础重置） |
| `themes.css` | 亮/暗主题变量 |
| `typography.css` | 字体驱动排版 |
| `components.css` | 组件样式（卡片、按钮、导航） |
| `animations.css` | 动效相关 CSS |
| `responsive.css` | 响应式适配 |

> 这套拆分目前仅用于 `public/index.html` 单文件预览，未接入 Hugo 构建流程。

---

## 10. Decap CMS 集成

### 10.1 入口与配置

- **入口**：[static/admin/index.html](file:///d:/G/github/个人主页/static/admin/index.html) — 加载 Decap CMS CDN
- **配置**：[static/admin/config.yml](file:///d:/G/github/个人主页/static/admin/config.yml)

### 10.2 后端模式
```yaml
backend:
  name: git-gateway     # 通过 GitHub OAuth 代理
  branch: main
local_backend: true     # 支持本地开发模式
```
`git-gateway` 需要 GitHub App 配合；`local_backend: true` 允许运行 `npx decap-server` 本地调试。

### 10.3 集合定义

配置了 **10 个集合**，覆盖中英双语的 5 个 section：

| 集合名 | 目录 | 可创建新条目 |
|--------|------|-------------|
| `zh_posts` / `en_posts` | `content/{zh,en}/posts` | ✅ |
| `zh_projects` / `en_projects` | `content/{zh,en}/projects` | ✅ |
| `zh_resume` / `en_resume` | `content/{zh,en}/resume` | ❌（单页） |
| `zh_about` / `en_about` | `content/{zh,en}/about` | ❌ |
| `zh_contact` / `en_contact` | `content/{zh,en}/contact` | ❌ |

### 10.4 字段定义

CMS 字段与 Frontmatter 一一对应。例如项目集合字段：`title`、`date`、`draft`、`description`、`category`、`tags`、`tech_stack`、`link`、`body`。

---

## 11. 部署流程（GitHub Actions）

工作流文件：[.github/workflows/hugo-deploy.yml](file:///d:/G/github/个人主页/.github/workflows/hugo-deploy.yml)

### 11.1 触发条件
```yaml
on:
  push:
    branches: [main]
  workflow_dispatch:      # 支持手动触发
```

### 11.2 构建作业（build）

```bash
# 1. 安装 Hugo Extended 0.128.0
wget .../hugo_extended_0.128.0_linux-amd64.deb
sudo dpkg -i .../hugo.deb

# 2. 安装 Dart Sass（Hugo extended 编译 SCSS 用）
sudo snap install dart-sass

# 3. Checkout（含 submodule，fetch-depth: 0 用于 lastmod）
# 4. 配置 GitHub Pages
# 5. 安装 Node 依赖（若存在 package-lock.json）
# 6. Hugo 构建
hugo --gc --minify --baseURL "${{ steps.pages.outputs.base_url }}/"

# 7. 上传 ./public 作为 artifact
```

### 11.3 部署作业（deploy）

```yaml
deploy:
  needs: build
  environment:
    name: github-pages
    url: ${{ steps.deployment.outputs.page_url }}
  steps:
    - uses: actions/deploy-pages@v4
```

### 11.4 权限与并发控制
```yaml
permissions:
  contents: read
  pages: write
  id-token: write       # Pages 部署所需

concurrency:
  group: pages
  cancel-in-progress: false   # 不取消进行中的部署，避免中断
```

---

## 12. 项目运行方式

### 12.1 环境准备

**必需**：
- [Hugo Extended](https://github.com/gohugoio/hugo/releases) `≥ 0.128.0`（含 SCSS 支持）
- Git

**可选**：
- Node.js（仅当需要 `npm` 依赖时；当前项目无 `package.json`）
- `npx decap-server`（CMS 本地调试）

### 12.2 本地开发

```powershell
# 在项目根目录执行
hugo server

# 完整参数（推荐）
hugo server --buildDrafts --disableFastRender --logLevel info
```

- 默认地址：`http://localhost:1313/`
- `--buildDrafts`：包含草稿
- `--disableFastRender`：禁用快速渲染，完整刷新（动效调试推荐）
- 修改 `content/`、`layouts/`、`i18n/` 文件会热重载

### 12.3 单文件预览（无需 Hugo）

直接用浏览器打开 `public/index.html` 即可预览首页全部效果。此文件内联了所有 CSS/JS，仅依赖 CDN 加载 GSAP/Lenis/Three.js。

VSCode 调试配置已就绪：[.vscode/launch.json](file:///d:/G/github/个人主页/.vscode/launch.json) 提供 Chrome 启动配置。

### 12.4 生产构建

```powershell
hugo --gc --minify
```

- `--gc`：构建后启用 Go GC 清理内存
- `--minify`：压缩 HTML/CSS/JS
- 产物输出至 `public/`

### 12.5 部署

**自动部署**：推送 `main` 分支即触发 GitHub Actions，自动构建并部署到 GitHub Pages。

**手动部署**：
```powershell
# 1. 本地构建
hugo --gc --minify

# 2. 将 public/ 内容推送至 gh-pages 分支（或用 gh-pages 工具）
# 3. 在 GitHub 仓库 Settings → Pages 配置源
```

### 12.6 CMS 本地调试

```powershell
# 终端 1：启动 Hugo
hugo server

# 终端 2：启动 Decap CMS 本地代理
npx decap-server

# 浏览器访问
http://localhost:1313/admin/
```

### 12.7 内容编辑工作流

**新增博客文章**：
1. 在 `content/zh/posts/` 创建 `my-post.md`
2. 填写 Frontmatter（title、date、description、tags）
3. 编写正文
4. `git commit && git push`
5. 等待 GitHub Actions 部署完成

**通过 CMS 编辑**：
1. 访问 `https://username.github.io/admin/`
2. GitHub OAuth 登录
3. 在 Web UI 中编辑/创建内容
4. 点击「Publish」自动提交到 `main` 分支

---

## 13. 已知问题与改进方向

### 13.1 已知问题

| 问题 | 位置 | 影响 |
|------|------|------|
| i18n 键缺失 | `i18n/{zh,en}.toml` 未定义 `hero_tagline`、`portfolio_label` 等模板中使用的键 | 首屏 Hugo 渲染时文案为空，依赖 JS `I18n.apply()` 兜底，存在闪烁 |
| 双资源体系未统一 | `layouts/partials/` 内嵌 vs `public/assets/` 拆分 | 维护成本翻倍，两套实现可能不一致 |
| `public/assets/js/main.js` 引用了 `PageTransition`、`PageAnims` 模块 | `public/assets/js/` | 这是演进后的版本，与 Hugo 体系（`ScrollAnims`）实现路径不同 |
| 缺失功能 | 见 `.trae/specs/create-personal-homepage/tasks.md` 未完成项 | 阅读进度条、客户端搜索、代码块终端风格、浏览器语言自动重定向、项目筛选未实现 |
| `.gitignore` 仅忽略 `.superpowers/` | [.gitignore](file:///d:/G/github/个人主页/.gitignore) | `public/`、`.trae/`、`.workbuddy/` 未被忽略，可能提交构建产物 |

### 13.2 设计演进记录

从 [tasks.md](file:///d:/G/github/个人主页/.trae/specs/create-personal-homepage/tasks.md) 与代码对比可见，项目经历了**动效策略演进**：

- **v1（Hugo 体系，`scripts.html`）**：Lenis 平滑滚动 + ScrollTrigger 滚动驱动动画
- **v2（预览体系，`public/assets/js/`）**：引入 `PageTransition`（页面切换引擎）+ `PageAnims`（入场动画），**替代滚动驱动**

v2 的设计意图是将单页滚动改为多页切换，每个 section 独立成页。这一演进尚未回流到 Hugo 模板体系。

### 13.3 改进建议

1. **统一 i18n**：将 `I18n.dict` 中的键迁移至 `i18n/*.toml`，让 Hugo 在服务端渲染时即输出正确文案，JS 仅负责切换
2. **统一资源体系**：选择内嵌或拆分其一，避免双轨维护。推荐拆分（用 Hugo Pipes 或 `assets/` 目录）
3. **完善 `.gitignore`**：添加 `public/`、`.trae/`、`.workbuddy/`、`resources/_gen/`
4. **补齐缺失功能**：按 `tasks.md` 中未完成项优先级实现
5. **回流动效演进**：若决定采用 v2 的页面切换模式，需将 `PageTransition`/`PageAnims` 模块整合回 `scripts.html`

---

## 附录：关键文件速查

| 文件 | 行数 | 说明 |
|------|------|------|
| [hugo.toml](file:///d:/G/github/个人主页/hugo.toml) | 89 | 站点配置 |
| [layouts/_default/baseof.html](file:///d:/G/github/个人主页/layouts/_default/baseof.html) | 23 | 基础骨架 |
| [layouts/index.html](file:///d:/G/github/个人主页/layouts/index.html) | 96 | 首页模板 |
| [layouts/partials/head.html](file:///d:/G/github/个人主页/layouts/partials/head.html) | 693 | 内嵌完整 CSS |
| [layouts/partials/scripts.html](file:///d:/G/github/个人主页/layouts/partials/scripts.html) | 648 | 内嵌完整 JS + CDN |
| [layouts/partials/header.html](file:///d:/G/github/个人主页/layouts/partials/header.html) | 19 | 顶部导航 |
| [static/admin/config.yml](file:///d:/G/github/个人主页/static/admin/config.yml) | 152 | Decap CMS 配置 |
| [.github/workflows/hugo-deploy.yml](file:///d:/G/github/个人主页/.github/workflows/hugo-deploy.yml) | 72 | 部署工作流 |
| [.trae/specs/create-personal-homepage/spec.md](file:///d:/G/github/个人主页/.trae/specs/create-personal-homepage/spec.md) | 154 | 项目规格说明 |
| [.trae/specs/create-personal-homepage/tasks.md](file:///d:/G/github/个人主页/.trae/specs/create-personal-homepage/tasks.md) | 125 | 任务清单与完成状态 |

---

*文档生成时间：2026-07-25 · 基于仓库当前状态分析*
