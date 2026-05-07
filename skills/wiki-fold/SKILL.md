---
name: wiki-fold
description: "将wiki/log.md中的操作记录批量归档为摘要页面。支持按数量（n=10）或时间范围（from/to）选择条目。提取式摘要，不编造。Dry-run模式默认只输出不写入。触发词：fold the log, 归档日志, run a fold, log rollup, 日志归档, fold n=, fold from=."
---

# wiki-fold: 日志归档摘要

把`wiki/log.md`里的操作记录批量归档成一个摘要页面。log.md是流水账，fold是阶段总结。

fold是**只加不改**的：被归档的log条目和它们引用的页面不会被修改或删除。fold是**提取式**的：摘要中的每个结论都必须可以追溯到具体的log条目，不编造。

---

## 范围

本skill只做：
- 对log.md条目的扁平归档（按数量或时间范围）
- 通过确定性fold ID实现结构幂等
- 带计数校验的提取式摘要

不做：
- 折叠的折叠（hierarchical stacking，留给未来）
- 自动触发（当前版本全部人工发起）

---

## Modes

| 模式 | 写文件？ | 调用方式 |
|------|----------|----------|
| **dry-run（默认）** | **不调用Write。** 通过Bash `cat`/`heredoc`输出到stdout。 | `fold the log, dry-run n=10` 或 `fold the log, dry-run from=2026-04-01 to=2026-04-30` |
| **commit** | 调用Write/Edit。每次Write会触发PostToolUse hook自动提交。先组装完整内容，再顺序写入。 | `fold the log, commit n=10`（先跑一次dry-run确认无误） |

**Why stdout-only in dry-run**: the repo's `hooks/hooks.json` PostToolUse hook fires on any `Write|Edit` and runs `git add wiki/ .raw/`. Writing to `/tmp` does not stage /tmp, but it still triggers the hook, which will commit *any pending wiki changes* under a generic message. Dry-run must leave zero residue. Bash stdout does not fire the hook.

---

## 确定性fold ID

每个fold的ID由输入决定：

```
fold-from-{EARLIEST-DATE}-to-{LATEST-DATE}-n{COUNT}
```

示例：`fold-from-2026-04-10-to-2026-04-23-n8`。

commit模式下文件名为`wiki/folds/{FOLD-ID}.md`。文件名中不含创建日期，标题中不含时间戳。

**重复检测（必须）**：输出前检查`wiki/folds/{FOLD-ID}.md`是否已存在。如果存在，报告"Fold已存在于wiki/folds/{FOLD-ID}.md。用--force覆盖，或选择其他范围。"然后停止。这是幂等保证——内容不保证逐字相同（LLM输出有变化），但文件名和范围是固定的。

---

## 参数

两种选择范围的方式（二选一）：

**按数量**：`n=12`，取log.md最近的12条记录。
- 如果log中不足n条，报告不足并停止，不做部分归档。

**按时间范围**：`from=2026-04-01 to=2026-04-30`，取这个日期区间内的所有记录。
- 如果区间内没有记录，报告为空并停止。

两种都不指定时，默认`n=10`。

其他参数：
- `--force`：覆盖已有的同ID fold。默认不覆盖。
- `--commit`：写入wiki/。不加则为dry-run，只输出到stdout。

---

## Procedure

### 1. 解析log条目

**按数量模式**：
```
grep -n "^## \[" wiki/log.md | head -n {N}
```

**按时间范围模式**：
```
grep -n "^## \[" wiki/log.md
```
然后按日期过滤，只保留`from`到`to`区间内的条目。

记录每条的：行号、日期、操作类型、标题，以及后续的bullet行（到下一个`## [`或文件末尾）。

### 2. Extract child page identifiers

From each entry's bullet list, extract:
- `Location: wiki/path/to/page.md` (the primary page)
- `[[Wikilinks]]` inline
- `Pages created:` and `Pages updated:` lists

Build a structured children list:
```yaml
children:
  - date: "2026-04-23"
    op: "save"
    title: "DragonScale Memory v0.2 — post-adversarial-review"
    page: "[[DragonScale Memory]]"
  - ...
```

One record per log entry. Do not dedupe by page: if two entries both point to `[[DragonScale Memory]]`, both records appear, distinguishable by date and title.

### 3. Read referenced pages (bounded)

Read only the pages that are not already captured fully in the log entry's bullets. Budget: 0-10 page reads. Hard ceiling: 15. If an entry's referenced page is missing, record `page_missing: true` and proceed.

### 4. Extractive summarization with count checks

Write the fold body per `references/fold-template.md`. **Rules**:

- **Extractive only.** Every outcome bullet and theme bullet must cite a specific child entry (e.g., `(from 2026-04-14 session)`) or a quoted line from that entry. Do not introduce events, counts, or interpretations not present in a child entry.
- **Log entry is the primary source.** If the log entry's bullets and the referenced meta-page disagree on a fact (e.g., a count), prefer the log-entry bullets and flag the mismatch as "source mismatch: log says X, meta says Y."
- **Count checks.** If you write "N concept pages" or "M repos updated," grep the source entries for the number and verify. Numeric mismatches are dry-run blockers.
- **No merging across entries without naming them.** A theme that spans multiple entries must name each contributing entry inline.
- **Uncertainty is a feature.** If an entry is ambiguous, say "ambiguous in source: [[Entry]]" rather than picking one interpretation.

### 5. Self-check before emitting

Before printing output, verify:
- Every child in `children:` frontmatter appears exactly once in the Child Entries table.
- Every entry in the table appears in the `children:` frontmatter.
- Every numeric claim in Key Outcomes is grep-verifiable against a child entry.
- The fold ID is deterministic and the file does not already exist (or `--force` is set).

If any check fails, abort and report the specific failure.

### 6. Emit

**Dry-run**: use Bash `cat <<'EOF' ... EOF` to stdout. Do not use Write. Print the fold ID and a one-line summary of what the commit step would do.

**commit**（用户确认后才执行）：
1. `Write`fold页面到`wiki/folds/{FOLD-ID}.md`。（PostToolUse hook会自动提交。）
2. `Edit` `wiki/index.md`，在`## Folds`分区下添加链接（分区不存在则创建）。（hook自动提交。）
3. `Edit` `wiki/log.md`，在顶部追加一条：
   ```
   ## [YYYY-MM-DD] fold | 归档N条记录
   - 位置：wiki/folds/{FOLD-ID}.md
   - 范围：{EARLIEST-DATE}至{LATEST-DATE}
   - 子条目：N条log记录
   ```
   （hook自动提交。）

会产生三次自动提交。这是预期行为，不要试图绕过hook。

---

## Output schema

See `references/fold-template.md` for the canonical frontmatter and body layout.

---

## 不变量

1. **结构幂等**：相同范围 → 相同fold ID → 重复检测阻止重复写入。LLM输出的文字可能不同，但文件名和范围是固定的。
2. **只加不改**：被归档的条目和页面不会被修改。
3. **有界读取**：每次fold最多读0-15个子页面。
4. **提取式**：不编造。所有数值必须经过计数校验。
5. **不链式调用**：wiki-fold不会触发wiki-lint、wiki-ingest、autoresearch或save。

---

## 不要做

- dry-run时不要调用Write/Edit，只用Bash stdout。
- 文件名和标题中不要包含创建日期，用子条目的日期范围。
- 不要按页面标题去重children。一条log记录对应一条children记录。
- 不要写没有具名来源条目的"涌现主题"。
- 不要声称逐字节幂等。实际保证的是结构幂等。
- 不要绕过PostToolUse自动提交hook。
- 不要更新`wiki/hot.md`，那是save/ingest skill的职责。

---

## 撤销

已提交的fold撤销（三步，按顺序执行）：
1. 删除log.md中的fold条目。
2. 删除index.md中的条目。
3. 删除fold页面文件。

或者：`git revert`那三次自动提交。无论哪种方式，子页面都不受影响。

---

## 示例

### 按数量：`fold the log, dry-run n=8`

1. 解析`wiki/log.md`最近8条记录。
2. 构建children列表（8条）。
3. 按需读取0-10个引用页面。
4. 生成fold ID：`fold-from-2026-04-10-to-2026-04-23-n8`。
5. 检查`wiki/folds/fold-from-2026-04-10-to-2026-04-23-n8.md`不存在。
6. 按模板生成fold内容。
7. 自检（frontmatter/表格一致性、计数校验）。
8. 通过`cat <<'EOF' ... EOF`输出到stdout。
9. 报告："Dry-run完成。Fold ID: {FOLD-ID}。确认后执行：'commit the fold'。"

### 按时间：`fold the log, dry-run from=2026-04-01 to=2026-04-30`

1. 解析`wiki/log.md`中所有条目。
2. 过滤出2026-04-01至2026-04-30之间的条目（假设12条）。
3. 后续步骤同上，fold ID为`fold-from-2026-04-01-to-2026-04-30-n12`。
