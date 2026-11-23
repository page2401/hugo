---
title: "00. 概览与版本档案 (Hugo Extended)"
description: "Go 语言单二进制极速静态站点生成器 Hugo Extended 0.170.1 基准实测子站。"
weight: 1
---

# Hugo Extended (`0.170.1`) 基准实测子站

本子模块由主仓 `page2401.github.io/docs/` 标准 GFM 母版**最小化派生适配**而来，实测 Go 语言生态标杆单二进制生成器 **Hugo Extended `v0.170.1`**（内置 `Goldmark` 解析内核 + Render Hooks 钩子机制 + Page Bundles 目录模型）的真实工程表现。

<div class="hero-grid">
  <div class="hero-card">
    <h3><a href="./01-annotations/">01. 专属标注与元数据 →</a></h3>
    <p>实测 Shortcode <code>&#123;&#123;&lt; callout &gt;&#125;&#125;</code> 与 Goldmark <code>render-blockquote.html</code> 原生直通 GFM Alerts。</p>
  </div>
  <div class="hero-card">
    <h3><a href="./02-diagrams/">02. 流程图与时序图 →</a></h3>
    <p>通过 1 行 <code>render-codeblock-mermaid.html</code> 钩子直连本地锁定的 <code>mermaid@12.3.0</code>。</p>
  </div>
  <div class="hero-card">
    <h3><a href="./03-math/">03. 数学公式排版 →</a></h3>
    <p>开启 Goldmark <code>passthrough</code> 定界符透传 + <code>render-passthrough.html</code> 零转义损耗接入 <code>katex@0.19.1</code>。</p>
  </div>
  <div class="hero-card">
    <h3><a href="./04-directory/">04. 目录结构与路由取舍 →</a></h3>
    <p>深度剖析 Hugo 独有的 <code>_index.md</code>（Branch Bundle）与 <code>index.md</code>（Leaf Bundle）吞噬子路由陷阱。</p>
  </div>
</div>

## 精确锁定版本档案（`ssg-lock.json`）

Hugo 作为 Go 静态编译单文件引擎，无需 `node_modules` 或虚拟环境，全站构建毫秒级完成；前端数学与图表运行时完全本地化托管于 `static/vendor/`，零外部 CDN 依赖：

| 模块 / 组件 | 精确锁定版本 | 在本子站中的职责与接入方式 |
| :--- | :--- | :--- |
| `hugo` (Extended) | `0.170.1` | Go 单二进制核心编译器（内置 SCSS/SASS 与 Goldmark AST 钩子） |
| `goldmark` | `builtin` | 内置 CommonMark / GFM 解析器，开启 `unsafe = true` 与 `passthrough` 数学透传 |
| `katex` | `0.19.1` | 托管于 `static/vendor/`，配合 `render-passthrough.html` 精确渲染行内与块级公式 |
| `mermaid` | `12.3.0` | 托管于 `static/vendor/`，配合 `render-codeblock-mermaid.html` 与 `data-theme` 联动深浅色绘图 |

## 五大标准基准章节对照

| 章节路径 | 测试维度 | 相对母版 `docs/*.md` 的核心派生改造点 |
| :--- | :--- | :--- |
| [`01-annotations`](./01-annotations/) | 专属标注与元数据 | 追加 `weight: 2`；第 2 节改用 Hugo 原生 Shortcode，第 4 节靠 `render-blockquote.html` 100% 零改动直通 `> [!NOTE]` |
| [`02-diagrams`](./02-diagrams/) | 流程图与时序图 | 正文代码块零改动，仅由 `layouts/_default/_markup/render-codeblock-mermaid.html` 接管输出 |
| [`03-math`](./03-math/) | 数学公式排版 | 正文公式零双反斜杠改写，靠 `markup.goldmark.extensions.passthrough` 阻断 Markdown 提前转义 `\\` 与 `\_` |
| [`04-directory/`](./04-directory/) | 目录结构与路由取舍 | 目录入口必须重命名为 `_index.md`（Branch Bundle），否则 `index.md`（Leaf Bundle）会将子目录降级为静态资源导致 404 |
