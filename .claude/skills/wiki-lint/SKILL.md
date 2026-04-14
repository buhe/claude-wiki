---
name: wiki-lint
description: 在审核 wiki 健康问题时使用 —— 页面间的矛盾、孤立页面、损坏的交叉引用、过时声明、缺失页面或覆盖缺口。每进行 5-10 次摄入后运行一次。
---

# Wiki Lint

审核 wiki。生成分类报告。提供具体修复方案。记录操作日志。

## 前置条件

查找 `SCHEMA.md`（从当前工作目录向上搜索，或 `~/wikis/`）。如果未找到，告诉用户先运行 `wiki-init`。读取它以获取 wiki 根路径和约定。

## 处理流程

### 1. 构建页面清单

读取 `wiki/index.md`、`wiki/overview.md` 以及 `wiki/pages/` 中的所有文件。构建映射：
- 所有现有的 slug（不带 `.md` 的文件名）
- 在任何页面中找到的所有 `[[slug]]` 引用
- frontmatter 中列出的所有 `sources`

### 2. 运行所有检查

**🔴 错误（必须修复）**

- **损坏的链接** — `[[slug]]` 引用对应的 `wiki/pages/<slug>.md` 不存在
- **缺失 frontmatter** — 页面缺少必需的 `title`、`tags`、`sources` 或 `updated` 字段

**🟡 警告（应该修复）**

- **孤立页面** — 没有任何其他页面的入站 `[[slug]]` 链接的页面（不包括 index.md 和 overview.md）
- **矛盾** — 一个页面中的声明与另一个页面中的声明直接冲突（查找被不同描述的同一实体：日期、计数、名称、关系）
- **过时声明** — 90 天内未更新的页面，包含 "current"、"latest"、"recent"、"state-of-the-art" 或两年以上的年份字面量

**🔵 信息（考虑处理）**

- **缺失概念页面** — `[[slug]]` 引用在 wiki 中出现 3 次以上但没有专门页面
- **覆盖缺口** — `overview.md` 中列出的开放问题，可以通过网络搜索或新的摄入来回答
- **缺失交叉引用** — 两个页面讨论同一实体但未相互链接

### 3. 编写 lint 报告

编写 `wiki/pages/lint-<today>.md`（无需请求许可 —— 始终写入此文件）：

```markdown
---
title: Lint Report <today>
tags: [lint, maintenance]
sources: []
updated: <today>
---

# Lint Report — <today>

## Summary
- 🔴 Errors: N
- 🟡 Warnings: N
- 🔵 Info: N

## 🔴 Broken Links
- [[source-page]] references [[missing-slug]] — does not exist
  Fix: create the page or remove the reference

## 🔴 Missing Frontmatter
- [[page]] is missing: title, updated

## 🟡 Orphan Pages
- [[slug]] — no inbound links
  Fix: add link from [[related-page]], or delete if no longer relevant

## 🟡 Contradictions
- [[page-a]] says: "<claim>"
- [[page-b]] says: "<conflicting claim>"
  Recommendation: <which to trust, or "investigate further">

## 🟡 Stale Claims
- [[page]] last updated <date>, contains "latest" — may be outdated
  Fix: re-verify claims or add a "as of <date>" qualifier

## 🔵 Missing Concept Pages
- [[slug]] referenced N times but no page exists
  Fix: run wiki-ingest or create a stub

## 🔵 Coverage Gaps
- Open question from overview.md: "<question>"
  Suggestion: search for <X> or ingest <source type>

## 🔵 Missing Cross-References
- [[page-a]] and [[page-b]] both discuss <entity> but don't link to each other
```

将 lint 报告添加到 `wiki/index.md` 的 Maintenance 类别下（如果不存在则创建）。

### 4. 提供具体修复方案

对于每个可修复的类别，提供：
- **损坏的链接：** "删除损坏的 `[[slug]]` 引用？（写入前我会展示每次更改）"
- **缺失交叉引用：** "在这些页面对之间添加缺失的链接？"
- **孤立页面标签：** "在孤立页面的 frontmatter 中添加 `status: orphan`？"
- **缺失 frontmatter：** "添加带有占位符值的缺失 frontmatter 字段？"

写入前展示每次更改的确切差异。仅在确认后应用。

### 5. 追加到 `wiki/log.md`

始终追加 —— 无需请求许可：
```
## [<today>] lint | <N errors> errors, <N warnings> warnings, <N info> info
Report: [[lint-<today>]]
Fixed: <list what was auto-fixed, or "none">
```
