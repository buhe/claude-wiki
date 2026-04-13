---
name: wiki-init
description: 用于为任何知识领域初始化新的个人wiki时使用 —— 研究文档、代码库文档、阅读笔记、竞品分析或任何长期知识积累项目。
---

# Wiki 初始化

在用户指定的路径下初始化一个新的由LLM维护的wiki。

## 准备工作

检查附近是否已存在 `SCHEMA.md`。如果存在，询问用户是要重新初始化还是继续使用现有wiki。

## 流程

### 1. 配置

配置项：
1. **当前目录为** `wiki-root`
2. **`index.md` 分类为** `原则 | 普通股 | 债券 | 另类资产`

### 2. 创建目录结构

```
<wiki-root>/
├── SCHEMA.md         ← 约定规范 + 绝对路径（其他技能通过它找到wiki）
├── raw/              ← 不可变的源文档（用户添加，LLM从不修改）
├── wiki/
│   ├── index.md      ← 内容目录：每个页面、单行摘要、按分类
│   ├── log.md        ← 仅追加的操作日志
│   ├── overview.md   ← 所有已知内容的演进式综合
│   └── pages/        ← 所有wiki页面，扁平结构，slug命名（无子目录）
└── assets/           ← 下载的图片、PDF、附件
```

**关键点：** `wiki/pages/` 是扁平结构。所有页面都以 `<slug>.md` 形式存放在此。无子目录。slug使用小写、连字符分隔。

### 3. 编写 `SCHEMA.md`

```markdown
# Wiki Schema

## Identity
- **Path:** <wiki-root的绝对路径>
- **Domain:** <用户的领域描述>
- **Source types:** <列表>
- **Created:** <YYYY-MM-DD>

## Page Frontmatter
每个wiki页面必须以以下内容开头：
---
title: <页面标题>
tags: [tag1, tag2]
sources: [source-slug1]
updated: YYYY-MM-DD
---

## Cross-References
使用 `[[slug]]`，其中 slug = 去掉 `.md` 的文件名
示例：`[[transformer-architecture]]` → `wiki/pages/transformer-architecture.md`

## Log Entry Format
## [YYYY-MM-DD] <operation> | <title>
操作类型：init, ingest, query, update, lint

## Index Categories
<每行一个，与用户选择的分类体系一致>

## Conventions
- raw/ 不可变 —— 技能从不修改它
- log.md 仅追加 —— 永不重写，只能追加
- index.md 在每次添加或更改页面时更新
- 所有页面扁平存放在 wiki/pages/ —— 无子目录
- overview.md 反映所有来源的当前综合理解
```

### 4. 编写 `wiki/index.md`

```markdown
# Wiki 索引 — <domain>

<对每个分类>
### <分类名称>
<!-- 条目由 wiki-ingest 添加 -->
```

### 5. 编写 `wiki/log.md`

```markdown
# Wiki 日志

仅追加。格式：`## [YYYY-MM-DD] <operation> | <title>`
最近条目：`grep "^## \[" log.md | tail -10`

---

## [<今天>] init | <domain>
```

### 6. 编写 `wiki/overview.md`

```markdown
---
title: Overview
tags: [overview, synthesis]
sources: []
updated: <今天>
---

# <Domain> — 概述

> wiki中所有内容的演进式综合。当来源改变理解时由wiki-ingest更新。

## 当前理解

*尚未摄入任何来源。*

## 待解决问题

*问题出现时在此添加。*

## 关键实体 / 概念

*页面创建时填充。*
```

### 7. 确认

告知用户：
- Wiki已初始化于 `<path>`
- 手动向 `raw/` 添加来源，或直接使用URL或文件路径运行 `wiki-ingest`
- 定期运行 `wiki-lint` 保持wiki健康
- `SCHEMA.md` 是所有其他技能定位此wiki的方式 —— 请勿移动或删除
