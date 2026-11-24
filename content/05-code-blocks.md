---
title: "05. 高阶代码块与语法高亮"
description: "标准 GFM 母版派生：实测 Hugo Extended 0.170.1 内置 Go Chroma 高亮器的 {hl_lines=[3,\"7-9\"], linenos=table} 属性、无脚本构建期词法染色与 diff 着色。"
weight: 5
---

# 05. 高阶代码块与语法高亮（Hugo + Go Chroma 引擎）

技术文档中超过 40% 的篇幅由代码块构成，但标准 CommonMark / GitHub Flavored Markdown **仅定义了围栏语言标识（```` ```ts ````）**，并未定义文件名标题栏、指定行高亮、带语法着色的行级 Diff 以及多语言 Tab 代码组。本文件由原始 GFM 母版派生适配至 **Hugo Extended (`0.170.1`)**，集中检验其内置 **Go Chroma** 语法高亮引擎的原语边界。

## 1. 代码块文件名标题栏与基础语法高亮（Go Chroma 毫秒级编译）

Hugo 没有像 Node.js 系框架那样引入庞大的 TextMate WASM（Shiki），而是将用纯 Go 移植自 Python Pygments 的 **Chroma** 编译器直接静态链接进单二进制内核。在 `hugo.toml` 开启 `[markup.highlight] noClasses = false` 后，Chroma 在毫秒级构建期内直接输出语义化 `.chroma` Token 类名（零客户端 JS 水合）；对于文件名标题栏，默认无需编写额外 Render Hook 即可沿用母版路径提示：

**`scripts/resolve-route.ts`**

```ts
export interface RouteContext {
  basePath: string
  slug: string
  cleanUrls: boolean
}

export function formatDocRoute(ctx: RouteContext): string {
  const prefix = ctx.basePath.endsWith('/') ? ctx.basePath : `${ctx.basePath}/`
  const normalized = ctx.slug.replace(/^\/+|\/+$/g, '')
  if (!normalized || normalized === 'index') return prefix
  return ctx.cleanUrls ? `${prefix}${normalized}/` : `${prefix}${normalized}.html`
}
```

## 2. 关键行聚焦与表格行号（`{hl_lines=[3,"7-9"], linenos=table}`）

Hugo 的 Goldmark 解析器原生支持在代码块围栏信息串末尾追加花括号属性表 `{...}`，直接向底层 Chroma 传递渲染参数：

- **`hl_lines=[3,"7-9"]`**：精确为第 `3` 行与第 `7–9` 行注入 `<span class="line hl">`，无需任何源码魔法注释；
- **`linenos=table`**：以双列 `<table class="lntable">` 分离左侧行号列（`user-select: none`）与右侧代码列，确保读者双击或拖拽复制代码时**绝不会把行号数字一并粘走**！

```json {hl_lines=[3,"7-9"], linenos=table}
{
  "id": "ssg-benchmark",
  "routeStyle": "clean",
  "strictZeroCdn": true,
  "markdown": {
    "mathEngine": "katex",
    "diagramEngine": "mermaid",
    "lineHighlight": true,
    "codeTransformers": ["diff", "focus"]
  }
}
```

## 3. 代码增删对比着色（Chroma `diff` 词法分析器）

与 Pygments 血脉同源，Go Chroma 内置独立的 `diff` 词法器，自动将 `-` 删除行包裹为 `<span class="gd">`（GenericDeleted）、将 `+` 新增行包裹为 `<span class="gi">`（GenericInserted）。其优势是 100% 兼容标准 GitHub Markdown `diff` 代码块，取舍则是与 Pygments/Rouge 相同——独立 `diff` 词法器不再二次解析内部 TypeScript 关键字：

```diff
 export function mountSubsite(baseDir: string, subPath: string): string | null {
-  const cleanRel = decodeURIComponent(subPath).replace(/\.\./g, '')
-  return `${baseDir}/${cleanRel}`
+  const resolvedBase = path.resolve(baseDir)
+  const target = path.resolve(resolvedBase, decodeURIComponent(subPath).replace(/^\/+/, ''))
+  return target.startsWith(`${resolvedBase}${path.sep}`) ? target : null
 }
```

## 4. 多语言运行时命令组（原生 `<details>` 零脚本折叠组）

Hugo 内核本身不绑定客户端 UI 组件库（除非引入 Docsy / Hextra 等主题的 Tabs Shortcode）。在保持零额外 JS 依赖的前提下，得益于 `markup.goldmark.renderer.unsafe = true`，使用 HTML5 原生 `<details name="hugo-cmd-group">` 互斥手风琴即可在零脚本下优雅收纳跨语言构建命令：

<details name="hugo-cmd-group" open>
<summary>Go 单体二进制（<code>hugo.toml</code> · 当前子站）</summary>

```bash
hugo --minify
```

</details>

<details name="hugo-cmd-group">
<summary>Node.js（<code>package.json</code>）</summary>

```bash
npm ci && npm run build
```

</details>

<details name="hugo-cmd-group">
<summary>Deno 2（<code>deno.json</code>）</summary>

```bash
deno task build
```

</details>

<details name="hugo-cmd-group">
<summary>Python（<code>requirements.txt</code>）与 Ruby（<code>Gemfile</code>）</summary>

```bash
pip install -r requirements.txt && sphinx-build -b dirhtml . _build/dirhtml
bundle install && bundle exec jekyll build
```

</details>

## 5. 四大底层高亮引擎能力对照小结

| 高亮引擎代表 | 典型采用框架 | 构建期零运行时 JS | 原生行号 / 行高亮 | 保留词法着色的 Diff |
| :--- | :--- | :--- | :--- | :--- |
| **Shiki / Expressive Code** | VitePress、Starlight、Fumadocs、Nextra、Rspress、Nuxt、Quartz | ✅ 纯静态内联样式 | ✅ `{1,3-5}` / `[!code highlight]` | ✅ `// [!code ++]` |
| **Prism.js** | Docusaurus、VuePress、Eleventy (plugin)、Gatsby | ✅ 静态 Token 类名 | ✅ `{1,3-5}` / 魔法注释 | ⚠️ 需配 diff-highlight 插件 |
| **Pygments / Rouge** | MkDocs Material、Sphinx、Pelican、Jekyll | ✅ Python/Ruby 编译直出 | ✅ `hl_lines` / `emphasize-lines` | ⚠️ 仅标准 `diff` 块红绿 |
| **Chroma / Giallo / hljs** | **Hugo (Go Chroma)**、Zola (Rust Giallo)、mdBook (hljs) | ✅ (mdBook 为客户端 hljs) | ✅ **Hugo 原生 `{hl_lines=[3,"7-9"], linenos=table}`** | ⚠️ 仅标准 `diff` 块红绿 (`.gd` / `.gi`) |
