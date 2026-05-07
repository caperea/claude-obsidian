---
name: wiki-lint
description: >
  知识库健康巡检。查找孤立页面、死链接、过时信息、缺失交叉引用、frontmatter缺失、
  空白章节。创建或更新Dataview仪表盘。触发词："lint"、"health check"、"巡检"、
  "clean up wiki"、"check the wiki"、"wiki maintenance"、"find orphans"、"wiki audit"。
---

# wiki-lint：知识库健康巡检

每10-15次摄入后或每周运行一次。自动修复前必须先问。报告输出到`wiki/meta/lint-report-YYYY-MM-DD.md`。

**权限：** `wiki/`可自由读写，其他目录只读。

**写作规范：** 全中文，中西文之间不加空格。

---

## 巡检项目

按顺序逐项检查：

1. **孤立页面**。没有任何入站wikilink指向的wiki页面——存在但无人引用。
2. **死链接**。引用了不存在的页面的wikilink。
3. **过时信息**。老页面中被更新资料推翻或修正的断言。
4. **缺失页面**。在多个页面中被提及但尚未建页的概念或实体。
5. **缺失交叉引用**。页面中提到了某个实体但没有加wikilink。
6. **Frontmatter缺失**。缺少必填字段（type、status、created、updated、tags）的页面。
7. **空白章节**。有标题但下面没有内容。
8. **过时索引条目**。`wiki/index.md`中指向已重命名或已删除页面的条目。

---

## 巡检报告格式

生成到`wiki/meta/lint-report-YYYY-MM-DD.md`：

```markdown
---
type: meta
title: "巡检报告 YYYY-MM-DD"
created: YYYY-MM-DD
updated: YYYY-MM-DD
tags: [meta, 巡检]
status: developing
---

# 巡检报告：YYYY-MM-DD

## 概要
- 扫描页面数：N
- 发现问题数：N
- 自动修复数：N
- 待人工审核：N

## 孤立页面
- [[页面名]]：无入站链接。建议：从[[相关页面]]添加链接，或删除。

## 死链接
- [[缺失页面]]：在[[引用页面]]中被引用但不存在。建议：创建占位页或移除链接。

## 缺失页面
- "概念名"：在[[页面A]]、[[页面B]]、[[页面C]]中被提及。建议：创建对应wiki页。

## Frontmatter缺失
- [[页面名]]：缺少字段：status、tags

## 过时信息
- [[页面名]]：断言"X"可能与更新的来源[[新来源]]矛盾。

## 交叉引用缺失
- [[实体名]]在[[页面A]]中被提及但未加wikilink。
```

---

## 命名规范

巡检时强制执行以下规范：

| 元素 | 规范 | 示例 |
|------|------|------|
| 文件名 | 中文标题 | `张伟.md`、`策略平台.md` |
| 文件夹 | 中文 | `wiki/人物/`、`wiki/概念/` |
| Tags | 中文，分层 | `#团队/安全`、`#方向/治理` |
| Wikilinks | 与文件名完全匹配 | `[[张伟]]`、`[[策略平台]]` |

文件名必须在vault内唯一。文件名唯一时wikilink不需要路径前缀。

---

## 写作风格检查

巡检时标记违反写作规范的页面：

- 非陈述现在时（"X大概做了Y"而非"X负责Y"）
- 有断言但缺少来源标注
- 不确定信息未用`> [!gap]`标记
- 矛盾信息未用`> [!contradiction]`标记
- 中西文之间加了空格

---

## Dataview仪表盘

创建或更新`wiki/meta/dashboard.md`：

````markdown
---
type: meta
title: "仪表盘"
updated: YYYY-MM-DD
---
# Wiki仪表盘

## 最近活动
```dataview
TABLE type, status, updated FROM "wiki" SORT updated DESC LIMIT 15
```

## 种子页面（待完善）
```dataview
LIST FROM "wiki" WHERE status = "seed" SORT updated ASC
```

## 缺少来源的人物页
```dataview
LIST FROM "wiki/人物" WHERE !sources OR length(sources) = 0
```

## 缺少来源的组织页
```dataview
LIST FROM "wiki/组织" WHERE !sources OR length(sources) = 0
```

## 问答页面
```dataview
LIST FROM "wiki" WHERE type = "问答" SORT created DESC
```
````

---

## 自动修复前

必须先展示巡检报告。然后问："要自动修复这些问题，还是逐个审核？"

可以自动修复：
- 补充缺失的frontmatter字段（用占位值）
- 为缺失实体创建占位页
- 为未链接的提及添加wikilink

需要人工审核后才能修复：
- 删除孤立页面（可能是有意独立的）
- 解决矛盾信息（需要人工判断）
- 合并重复页面
