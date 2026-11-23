---
title: "04. 目录结构与路由取舍"
description: "实测 Hugo Page Bundles 核心机制：_index.md (Branch Bundle) 与 index.md (Leaf Bundle) 的致命差异。"
weight: 5
---

# 04. 目录结构与路由取舍（Hugo `_index.md` Branch Bundle 模型）

将标准 GFM 母版 `docs/04-directory/index.md` 迁移到 Hugo `content/04-directory/` 时，会迎头撞上 Hugo 最著名、也最容易让新手踩坑的 **Page Bundles（页面束）双轨约定**：

> [!CAUTION]
> **致命陷阱：`_index.md`（Branch Bundle） vs `index.md`（Leaf Bundle）**
> - 如果保留母版的文件名 `04-directory/index.md`，Hugo 会将该目录判定为 **Leaf Bundle（叶子束/终点单页）**——同目录及下级 `nested/` 中的所有 `.md` 文件都会被降级为“非独立路由的附属页面资源（Page Resources）”，**不再生成独立网页 HTML，直接访问全部报 404**！
> - 必须将目录入口改名为带前导下划线的 **`_index.md`**（包括子目录 `nested/_index.md`），将其声明为 **Branch Bundle（分支章节束）**，子页面才能正常编译路由并出现在 `.Pages.ByWeight` 侧边栏树中！

## Hugo 目录体系优劣权衡

1. **工程隔离度极佳**：所有正文严格收拢在 `content/` 目录，静态文件收拢在 `static/`，模板收拢在 `layouts/`，仓库根 `README.md` 绝不会被误编译。
2. **原生树形自动侧边栏 + 中文标题自由**：无需手写配置文件数组，也不像 Starlight `autogenerate` 那样暴露英文目录名 `nested`——只需在 `nested/_index.md` 中声明 `title: "嵌套子章节 (nested)"` 与 `weight`，模板遍历 `.Pages.ByWeight` 即可自动渲染多级中文导航。
3. **代价：`_index.md` 散落与 Pretty URL 相对路径偏移**：每个包含子页面的层级都必须放置 `_index.md`；且 Hugo 默认将 `01-order-and-url.md` 输出为目录式路由 `/04-directory/nested/01-order-and-url/`，子页间的相对跳转需按尾部斜杠目录层级书写。

## 嵌套子文档入口

- [进入嵌套分组页：`nested/_index.md`](./nested/)
- [进入子文档 A：排序前缀与 URL 纯净度权衡](./nested/01-order-and-url/)
- [进入子文档 B：多级子目录与侧边栏同步机制](./nested/02-sidebar-sync/)
