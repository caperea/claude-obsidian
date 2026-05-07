# claude-obsidian — 工作知识Wiki

这是一个Claude Code插件，为Obsidian vault提供持续积累的知识wiki系统。

**插件名称：** `claude-obsidian`
**Skills：** `/wiki`, `/wiki-ingest`, `/wiki-query`, `/wiki-lint`, `/save`, `/autoresearch`, `/canvas`

## Vault结构

```
vault根目录/
├── 0-9各目录/     源文件——只读，不得修改
├── wiki/          LLM生成的知识wiki——可自由读写
└── _templates/    Obsidian Templater模板
```

## 使用方式

- 指定一个源文件，让Claude摄入：`ingest 班子建设追踪.md`
- 提问：Claude先读索引，再深入相关页面
- 巡检：`lint the wiki`，每10-15次摄入后运行一次

## Wiki子目录

| 目录 | 内容 |
|------|------|
| `wiki/人物/` | 团队成员、候选人、stakeholder画像 |
| `wiki/组织/` | 团队结构、能力分布、资源配置 |
| `wiki/项目/` | 在跑的项目、专项、OKR追踪 |
| `wiki/系统/` | 技术系统、架构、依赖关系 |
| `wiki/业务/` | 安全/治理业务领域知识 |
| `wiki/概念/` | 技术框架、管理方法论 |
| `wiki/事件/` | 线上故障、复盘、关键事件 |
| `wiki/决策/` | 技术选型、组织调整、优先级取舍 |
| `wiki/目标/` | OKR、关键指标、里程碑 |
| `wiki/流程/` | Runbook、审批链路、应急响应 |
| `wiki/meta/` | lint报告、dashboard |

## 权限规则

- `wiki/`目录：LLM拥有完全读写权限
- 其他所有目录：只读，不得修改源文件
- 中西文之间不加空格
