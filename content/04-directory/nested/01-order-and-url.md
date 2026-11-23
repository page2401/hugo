---
title: "嵌套子页 A：排序前缀与 URL 纯净度权衡"
weight: 1
---

# 嵌套子页 A：排序前缀与 URL 纯净度权衡

在母版目录 `docs/04-directory/nested/` 中，文件名使用 `01-order-and-url.md` 以保证在 Git 与代码编辑器中按阅读先后顺序排列。在 Hugo 中，既可以通过 Frontmatter 的 `weight: 1` 精确控制排序，也可以在需要净化线上地址时为单页单独补充 `slug: "order-and-url"` 覆写末端 URL，兼顾本地有序与路由灵活性。

- [返回《04. 目录结构与路由取舍》](../../)
- [下一篇：《嵌套子页 B：多级子目录与侧边栏同步机制》](../02-sidebar-sync/)
