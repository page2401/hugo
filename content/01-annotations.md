---
title: "01. 专属标注与元数据"
description: "测试 Hugo 0.170.1 的 YAML/TOML Frontmatter (weight)、Shortcode 调用与 Goldmark GFM Alerts 原生钩子。"
weight: 2
---

# 01. 专属标注与元数据（Hugo Shortcode + Goldmark 钩子）

不同静态网站生成器对“提示框 / 标注（Callouts / Admonitions）”和页面元数据的语法约定存在显著分歧。本文件由标准 GFM 母版派生适配至 **Hugo Extended (`0.170.1`)**。

## 1. Frontmatter 元数据机制

Hugo 原生支持 YAML（`---`）、TOML（`+++`）与 JSON 三种 Frontmatter 格式，且最重要的页面排序字段为整型的 `weight`（数值越小越靠前）：

- **当前页标题字段**：`title: "01. 专属标注与元数据"`
- **页面摘要字段**：`description: "..."`
- **章节排序权重**：`weight: 2`（Hugo 侧边栏通过 `.Pages.ByWeight` 自动按权重升序排列，无需手写外部导航数组）

## 2. 标准提示语义块（Hugo 传统采用 Shortcode 语法）

Hugo 内置的 Goldmark 不像 `markdown-it-container` 那样采用 `::: warning` 三冒号语法，Hugo 生态最经典的自定义组件方式是编写 `layouts/shortcodes/callout.html` 并在正文中调用 `{{</* callout type="..." title="..." */>}}` 短代码：

{{< callout type="info" title="默认信息提示块" >}}
静态站点对比平台的每个子模块均独立维护 `main`（源码）与 `page`（构建产物）双分支，互不污染提交历史。
{{< /callout >}}

{{< callout type="tip" title="性能优化建议（自定义标题语义）" >}}
在 Hugo 短代码模板中，通过 `.Inner | .Page.RenderString` 管道即可让容器内部正常解析行内代码与 **加粗强调**，但双花括号短代码标记会导致文档无法直接在其他生成器原样复用。
{{< /callout >}}

{{< callout type="warning" title="版本边界注意" >}}
若在 Markdown 代码块中演示短代码字面量，必须使用双花括号加 `/* ... */` 注释转义，否则 Hugo 预处理器会在代码块解析前强行展开（如本段若直接内嵌裸短代码标记甚至会触发二次解析报错）！
{{< /callout >}}

{{< callout type="danger" title="不可逆分支覆盖警告" >}}
自动化流水线在向 `page` 分支推送构建产物时会启用 `force_orphan: true`，切勿在 `page` 分支直接手工修改代码！
{{< /callout >}}

## 3. 可折叠详情容器（开启 Goldmark `unsafe = true` 直通 HTML `<details>`）

Hugo 默认出于安全策略会过滤 Markdown 中的原始 HTML 标签（打印 `<!-- raw HTML omitted -->`）。在 `hugo.toml` 开启 `markup.goldmark.renderer.unsafe = true` 后，即可完美直通母版原生 `<details>` 与内部公式：

<details>
<summary>点击展开查看 Hugo 短代码与母版折叠容器源码结构</summary>

```markdown
{{</* callout type="tip" title="性能优化建议" */>}}
Hugo 短代码语法与纯正 GFM 引用警报并存对比。
{{</* /callout */>}}
```

除了代码块，折叠容器内同样支持列表、表格与数学公式 $f(x) = x^2 + 1$。

</details>

## 4. GitHub 风格引用警报（GFM Alerts）跨框架直通测试

以下三段**保持母版原文一字不改**！得益于现代 Hugo 的 `layouts/_default/_markup/render-blockquote.html` 渲染钩子（内置识别 `.Type == "alert"` 与 `.AlertType`），无需任何三方插件即可将全部 5 类 GitHub Alerts 原生渲染为语义化警报框：

> [!NOTE]
> 这是一段标准的 GitHub Flavored Markdown `[!NOTE]` 警报文本，用于检验各生成器对母版语法的零改造直通能力。

> [!IMPORTANT]
> 跨平台迁移 Markdown 文档时，是否原生兼容 `> [!IMPORTANT]` 与 `> [!WARNING]` 直接决定了母版文档的派生改造成本。

> [!CAUTION]
> 若目标生成器未内置 GFM Alert 解析器（如 VuePress v1、Starlight 默认配置、Docsify v4 旧插件），上方标记会退化为普通引用块并原样露出 `[!CAUTION]` 字符串。
