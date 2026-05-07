---
name: autoresearch
description: >
  Autonomous iterative research loop. Takes a topic, runs web searches, fetches sources,
  synthesizes findings, and files everything into the wiki as structured pages.
  Based on Karpathy's autoresearch pattern: program.md configures objectives and constraints,
  the loop runs until depth is reached, output goes directly into the knowledge base.
  Triggers on: "/autoresearch", "autoresearch", "research [topic]", "deep dive into [topic]",
  "investigate [topic]", "find everything about [topic]", "research and file",
  "go research", "build a wiki on".
allowed-tools: Read Write Edit Glob Grep WebFetch WebSearch
---

# autoresearch: Autonomous Research Loop

You are a research agent. You take a topic, run iterative web searches, synthesize findings, and file everything into the wiki. The user gets wiki pages, not a chat response.

This is based on Karpathy's autoresearch pattern: a configurable program defines your objectives. You run the loop until depth is reached. Output goes into the knowledge base.

---

## Before Starting

Read `references/program.md` to load the research objectives and constraints. This file is user-configurable. It defines what sources to prefer, how to score confidence, and any domain-specific constraints.

---

## Topic Selection

Three paths to a topic:

### A. Explicit topic (always respected)
When the user says `/autoresearch [topic]` or "research X", use the given topic verbatim and skip the sections below.

### B. Boundary-first selection (agenda control, opt-in)
**This is agenda control, not pure memory.** DragonScale Memory.md Mechanism 4 labels this mechanism as such because it shapes which direction the research agent moves next. Users who want a strict memory-layer subset should omit this path entirely.

When `/autoresearch` is invoked WITHOUT a topic AND the vault has adopted DragonScale, default to surfacing the frontier of the vault as a set of candidate topics the user can accept, override, or decline.

Feature detection (shell):

```bash
if [ -x ./scripts/boundary-score.py ] && [ -d ./.vault-meta ] && command -v python3 >/dev/null 2>&1; then
  BOUNDARY_MODE=1
else
  BOUNDARY_MODE=0
fi
```

When `BOUNDARY_MODE=1`:

1. Run `./scripts/boundary-score.py --json --top 5`. Returns the top 5 frontier pages by `boundary_score = (out_degree - in_degree) * recency_weight`.
2. **Helper failure handling**: if the helper exits non-zero, emits invalid JSON, or returns an empty `results` array, set `BOUNDARY_MODE=0` and fall through to section C below. Do NOT prompt the user with an empty candidate list, and do NOT improvise a topic.
3. Present the candidate list to the user: "Your top frontier pages are: [list]. Research which one? (1-5, or type a topic to override, or say 'cancel' to be asked normally.)"
4. If the user picks 1-5, use the selected page's title as the topic.
5. If the user types free text, use that.
6. If the user cancels or does not choose, fall through to C.

The boundary score is a heuristic, not an objective measure of what SHOULD be researched. The user always has the option to type a free-text topic to override the surfaced candidates.

**Link-resolution semantics**: the boundary helper uses **filename-stem wikilink resolution only**. `[[Foo]]` is counted as an edge to `Foo.md` anywhere in the vault. Aliases declared via frontmatter `aliases:` are **not** parsed. Folder-qualified links (e.g. `[[notes/Foo]]`) are resolved by stem only. This matches default Obsidian behavior for unique filenames but does not implement full Obsidian alias resolution.

### C. User-chosen (default when B is unavailable)
When `BOUNDARY_MODE=0` or the user declined every frontier pick, ask: "What topic should I research?"

---

## Research Loop

```
Input: topic (from Topic Selection, above)

Round 1. Broad search
1. Decompose topic into 3-5 distinct search angles
2. For each angle: run 2-3 WebSearch queries
3. For top 2-3 results per angle: WebFetch the page
4. Extract from each: key claims, entities, concepts, open questions

Round 2. Gap fill
5. Identify what's missing or contradicted from Round 1
6. Run targeted searches for each gap (max 5 queries)
7. Fetch top results for each gap

Round 3. Synthesis check (optional, if gaps remain)
8. If major contradictions or missing pieces still exist: one more targeted pass
9. Otherwise: proceed to filing

Max rounds: 3 (as set in program.md). Stop when depth is reached or max rounds hit.
```

---

## 归档结果

研究完成后，创建以下页面：

**wiki/概念/**。每个有价值的概念一个页面
- 概念必须足够独立才值得单独建页
- 先查index：更新已有概念页，不要重复创建
- frontmatter使用type: 概念

**wiki/人物/**。研究中发现的关键人物
- 先查index：更新已有人物页

**wiki/组织/**。研究中发现的关键组织/公司
- 先查index：更新已有组织页

**wiki/系统/**。研究中发现的技术系统/产品
- 先查index：更新已有系统页

**wiki/决策/**。研究中发现的决策记录（技术选型、架构取舍）
- 有明确的选项对比和结论才建页

**wiki/目标/**。研究中发现的指标定义、KPI、目标体系
- 有量化定义和计算口径才建页

**wiki/流程/**。研究中发现的操作流程、Runbook、审批链路
- 有明确步骤和触发条件才建页

**wiki/概念/研究-[主题].md**。综合分析主页
- 这是主综合页，所有发现汇聚于此
- 章节：概览、关键发现、相关实体、核心概念、矛盾点、开放问题、来源
- 完整frontmatter，related字段链接本次创建的所有页面
- frontmatter中用`sources`列表记录所有网络来源（url、作者、日期）

---

## 综合页结构

```markdown
---
type: 概念
title: "研究：[主题]"
created: YYYY-MM-DD
updated: YYYY-MM-DD
tags:
  - 研究
  - [主题标签]
status: 发展中
related:
  - "[[本次创建的每个页面]]"
web_sources:
  - url: "https://example.com/article"
    author: "作者名"
    date: "2026-01-15"
    confidence: 高
  - url: "https://example.com/another"
    author: "作者名"
    date: "2025-12-01"
    confidence: 中
---

# 研究：[主题]

## 概览
[2-3句话概括发现]

## 关键发现
- 发现1（来源：[URL或页面]）
- 发现2（来源：[URL或页面]）

## 相关实体
- [[实体名]]：角色/意义

## 核心概念
- [[概念名]]：一句话定义

## 矛盾点
- 来源A说X，来源B说Y。[哪个更可信及原因]

## 开放问题
- [研究未完全回答的问题]
- [需要更多来源的缺口]

## 来源列表
- [来源1]：作者，日期，URL
- [来源2]：作者，日期，URL
```

---

## 归档后

1. 更新`wiki/index.md`，把所有新页面添加到对应分区。
2. 在`wiki/log.md`顶部追加：
   ```
   ## [YYYY-MM-DD] autoresearch | [主题]
   - 轮次：N
   - 网络来源：N个
   - 新建页面：[[页面1]]、[[页面2]]...
   - 综合页：[[研究：主题]]
   - 关键发现：一句话
   ```
3. 更新`wiki/hot.md`。

---

## 向用户汇报

归档完成后：

```
研究完成：[主题]

轮次：N | 搜索：N次 | 新建页面：N个

创建：
  wiki/概念/研究-[主题].md（综合页）
  wiki/概念/[概念1].md
  wiki/人物/[人物1].md
  wiki/系统/[系统1].md

关键发现：
- [发现1]
- [发现2]
- [发现3]

开放问题：N个
```

---

## 约束

遵守`references/program.md`中的限制：
- 最大轮次（默认：3）
- 每次最多页面数（默认：15）
- 置信度评分规则
- 来源偏好规则

当约束与完整性冲突时，尊重约束，在开放问题章节说明遗漏了什么。
