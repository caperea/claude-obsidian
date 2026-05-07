# claude-obsidian: Agent Instructions

这是一个Claude Code插件，为Obsidian vault提供持续积累的知识wiki系统。基于Karpathy的LLM Wiki模式。

## Skills

| Skill | 触发方式 |
|---|---|
| `wiki` | `/wiki`，wiki状态检查 |
| `wiki-ingest` | ingest，摄入笔记到wiki |
| `wiki-query` | query，从wiki中检索信息 |
| `wiki-lint` | lint the wiki，健康检查 |
| `wiki-fold` | fold the log，日志rollup |
| `save` | /save，保存对话到wiki |
| `autoresearch` | autoresearch，自主研究 |
| `canvas` | /canvas，可视化画布 |
| `defuddle` | defuddle，网页内容清洁 |
| `obsidian-markdown` | Obsidian Markdown语法参考 |
| `obsidian-bases` | Obsidian Bases参考 |

## 核心约定

- **源文件**：vault中除`wiki/`外的所有目录——只读，不得修改
- **wiki**：`wiki/`目录——LLM可自由读写
- **热缓存**：`wiki/hot.md`（会话开始时读取，结束时更新）
- **wiki子目录**：人物/、组织/、项目/、系统/、业务/、概念/、事件/、决策/、目标/、流程/、meta/
- **语言**：中文，中西文之间不加空格

## 启动流程

1. 读取本文件和`CLAUDE.md`
2. 读取`skills/wiki/SKILL.md`了解路由逻辑
3. 如果`wiki/hot.md`存在，读取以恢复最近上下文
