---
name: save
description: >
  将当前对话、回答或洞察保存到Obsidian wiki中。分析对话内容，判定类型（人物/组织/项目/系统/业务/概念/事件/决策/目标/流程），
  创建frontmatter，归档到正确的wiki子目录，更新index、log和hot cache。
  Triggers on: "save this", "save that answer", "/save", "file this",
  "save to wiki", "save this session", "file this conversation", "keep this",
  "save this analysis", "add this to the wiki", "保存", "存一下".
allowed-tools: Read Write Edit Glob Grep
---

# save：对话沉淀到Wiki

好的回答和洞察不应消失在对话历史里。本技能把刚才讨论的内容归档为持久的wiki页面。

wiki靠积累产生复利。多保存。

---

## 类型判定

根据对话内容选择最合适的类型和目录：

| 类型 | 目录 | 使用场景 |
|------|------|----------|
| 综合分析 | wiki/概念/ | 多步骤分析、对比、回答一个具体问题 |
| 概念 | wiki/概念/ | 解释一个想法、模式或框架 |
| 人物 | wiki/人物/ | 关于人的分析和洞察 |
| 组织 | wiki/组织/ | 关于团队或组织的分析 |
| 项目 | wiki/项目/ | 项目相关的讨论和决定 |
| 系统 | wiki/系统/ | 技术系统相关的讨论 |
| 业务 | wiki/业务/ | 业务领域知识 |
| 决策 | wiki/决策/ | 做出的技术、组织或战略决策 |
| 目标 | wiki/目标/ | 目标、指标、KPI相关 |
| 流程 | wiki/流程/ | 流程、Runbook、操作规范 |
| 事件 | wiki/事件/ | 故障、复盘、关键事件 |
| meta | wiki/meta/ | 完整会话摘要 |

如果用户指定了类型就用指定的。否则根据内容选最合适的。拿不准时用`综合分析`。

---

## 保存流程

1. **扫描**当前对话，识别最有价值的内容。
2. **命名**（如果还没名字）：问用户"叫什么名字？"保持简短。
3. **判定类型**，用上面的表格选择目录。
4. **提取**对话中的相关内容。用陈述句现在时改写（不是"用户问了X"而是X本身）。
5. **创建**笔记到对应目录，包含完整frontmatter。
6. **收集链接**：对话中提到的wiki页面加入frontmatter的`related`字段。
7. **更新**`wiki/index.md`，在相应分区添加条目。
8. **追加**`wiki/log.md`（新条目在顶部）：
   ```
   ## [YYYY-MM-DD] save | 标题
   - 类型：[类型]
   - 位置：wiki/[目录]/标题.md
   - 来源：关于[简要描述]的对话
   ```
9. **更新**`wiki/hot.md`。
10. **确认**："已保存为[[标题]]到wiki/[目录]/。"

---

## Frontmatter模板

```yaml
---
type: <人物|组织|项目|系统|业务|概念|事件|决策|目标|流程|meta>
title: "标题"
created: YYYY-MM-DD
updated: YYYY-MM-DD
tags:
  - <领域标签>
status: 发展中
related:
  - "[[相关页面]]"
sources:
  - "[[源文件路径]]"
---
```

综合分析类型额外字段：
```yaml
question: "原始问题"
answer_quality: solid
```

决策类型额外字段：
```yaml
decision_type: <技术选型|组织调整|优先级取舍|架构变更>
decision_date: YYYY-MM-DD
decided_by: "决策人"
options:
  - "选项A"
  - "选项B"
outcome: "最终结论"
```

目标类型额外字段：
```yaml
goal_type: <OKR|KPI|里程碑>
owner: "负责人"
period: "2026-Q2"
target_value: "≤0.5‰"
current_value: "0.8‰"
unit: "事件率"
```

流程类型额外字段：
```yaml
process_type: <Runbook|审批|应急|日常>
owner: "负责人"
trigger: "触发条件"
frequency: <按需|日|周|月>
```

---

## 写作风格

- 陈述句、现在时。写知识本身，不写对话过程。
- 不要："用户问了X，Claude解释了..."
- 要："X的工作方式是Y。关键洞察是Z。"
- 包含所有相关上下文。未来session应该能独立阅读这个页面。
- 用wikilink链接所有提及的概念、实体和wiki页面。
- 标注来源：`（来源：[[页面名]]）`。
- 中西文之间不加空格。

---

## 保存vs跳过

保存：
- 非显而易见的洞察或综合分析
- 带理由的决策
- 花了大量精力的分析
- 可能被反复引用的对比
- 研究发现
- 新的目标/指标定义
- 新的流程或Runbook

跳过：
- 机械问答（一查就知道的答案）
- 已经记录在别处的配置步骤
- 没有持久洞察的临时调试
- wiki里已有的内容

如果wiki里已有相关页面，更新已有页面而不是创建新的。
