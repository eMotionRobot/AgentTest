# 专题：技能进化（Skill Evolution）

> 这是 Hermes Agent 相对 OpenClaw 最核心的差异化能力。本文基于 hermes-agent 源码，系统拆解"技能"如何被**创建 → 使用 → 自改进 → 生命周期维护**，并对比 OpenClaw 的技能模型。

---

## 1. 什么是"技能"？——程序性记忆

在 Hermes 中，**技能（Skill）= 程序性记忆（procedural memory）**：它捕获"**如何完成某一类任务**"的、经过实践验证的可复用方法。

与"声明性记忆"对比：

| | 程序性记忆（Skills） | 声明性记忆（MEMORY.md / USER.md） |
|---|---|---|
| 回答 | *怎么做*（how to） | *是什么 / 是谁*（what / who） |
| 粒度 | 窄而可执行 | 宽而陈述性 |
| 形态 | `SKILL.md` + 目录（references/templates/scripts/assets） | 带分隔符的条目 |

技能以 `SKILL.md` 为入口，遵循 [agentskills.io](https://agentskills.io) 开放标准，前置元数据（frontmatter）包含：`name`、`description`、`version`、`author`、`license`、`platforms`（OS 门控）、`metadata.hermes.tags/category/related_skills/config` 等。

> **作者规范（HARDLINE）**：`description` 必须 ≤ 60 字符、单句、以句号结尾——因为过长的描述会膨胀技能清单、稀释模型在加载多技能时的注意力。这是一个很有意思的"为模型注意力做减法"的工程约束。

技能的三个来源/层级：

| 位置 | 含义 |
|------|------|
| `skills/`（仓库内，按类目） | 默认内建、开箱即用 |
| `optional-skills/` | 较重/小众，`hermes skills install official/<cat>/<skill>` 显式安装 |
| `~/.hermes/skills/` | **用户创建 + Agent 自创建**的技能 |

---

## 2. 技能的生命周期：四个阶段

```text
  ① 创建 Create ──▶ ② 使用 Use ──▶ ③ 自改进 Self-improve ──▶ ④ 维护 Curate
   复杂任务后        作为程序性          使用中根据反馈           Curator 后台
   自动沉淀          记忆被加载          打补丁/重写              置顶/归档/合并
       ▲                                                            │
       └──────────────── 闭环：经验持续复利 ◀────────────────────────┘
```

### 阶段 ① 创建：任务后自动沉淀（autonomous skill creation）

Hermes 在**每个对话回合结束后**，可由 `AIAgent.run_conversation` 触发 **后台复盘（background review）**：

- `agent/background_review.py` 派生一个**守护线程**，在一个 *forked* 的 `AIAgent` 中**重放对话快照**，并自问："**是否应该保存/更新某个技能或记忆？**"
- 该 fork **继承父进程的运行时**（provider/model/base_url/凭据/已缓存的系统提示），因此命中同一前缀缓存、复用同一鉴权；
- 它的工具被**白名单限制为"记忆 + 技能管理"**，其余一律在运行时拒绝；
- **主对话与主提示缓存永不被触碰**——这是关键的工程取舍：自我进化在"影子线程"里完成，不污染、不拖慢前台体验。

技能创建/编辑通过 `tools/skill_manager_tool.py`，动作包括：

| 动作 | 作用 |
|------|------|
| `create` | 新建技能（SKILL.md + 目录结构） |
| `edit` | 整体重写某用户技能的 SKILL.md |
| `patch` | 在 SKILL.md 或任意支撑文件内做定向查找替换 |
| `delete` | 删除某用户技能 |
| `write_file` / `remove_file` | 增删支撑文件（references/templates/scripts/assets） |

### 阶段 ② 使用：作为程序性记忆被加载

- 技能可作为斜杠命令暴露（`/skills` 或 `/<skill-name>`）；
- `agent/skill_commands.py` 扫描 `~/.hermes/skills/`，并将技能**作为 user message 注入**（而非系统提示），以**保留前缀缓存**；
- `agent/skill_preprocessing.py` 支持技能内的模板变量替换（`${HERMES_SKILL_DIR}` / `${HERMES_SESSION_ID}`）与**内联 shell**（`` !`date +%Y-%m-%d` ``，并对输出做上限保护，单条出错不会拖垮整个技能）。

### 阶段 ③ 自改进：使用中改进（self-improve during use）

后台复盘的"技能复盘提示词"被设计得**主动、积极**。其要点（源码 `_SKILL_REVIEW_PROMPT`，已转述）：

- "要 **ACTIVE**——大多数会话都应至少产生一次技能更新，哪怕很小。什么都不做是**错失学习机会**，而非中性结果。"
- **目标形态**是 **CLASS-LEVEL（类级）技能**：每个技能有丰富的 SKILL.md + `references/` 目录承载会话级细节；**而非一长串"一会话一技能"的扁平列表**。
- **强信号**（任一即触发更新）：用户纠正了你的风格/语气/格式/冗长度；出现挫败信号如"别再做 X""太啰嗦了""别这样排版""直接给答案""你老是 Y、我很烦"，或明确的"记住这个"——这些都是**一等的技能信号**（不只是记忆信号），应把它嵌入相关技能中。

> 换言之：Hermes 把"用户的不满/纠正"当作**最高优先级的学习触发器**，并把它写进技能，让下次行为直接改变。这正是"进化"的核心闭环。

### 阶段 ④ 维护：Curator 后台生命周期管理

`agent/curator.py` 是一个**辅助模型任务**，**由空闲触发（无 cron 守护）**：当 Agent 空闲、且距上次 Curator 运行超过 `interval_hours` 时，`maybe_run_curator()` 派生一个 fork 的 AIAgent 做技能库审查。

职责与**严格不变量（invariants）**：

| Curator 职责 | 严格约束 |
|--------------|----------|
| 基于活动时间戳自动迁移生命周期状态 | **只触碰 Agent 自创建的技能**（`skill_usage.is_agent_created`） |
| 派生后台审查 Agent：置顶 / 归档 / 合并 / 打补丁 | **永不自动删除，只归档**（归档可恢复） |
| 通过 `skill_manage` 维护技能 | **置顶（pinned）的技能跳过所有自动迁移** |
| 持久化 Curator 状态到 `.curator_state` | **使用辅助客户端，绝不触碰主会话前缀缓存** |

默认时间参数：

| 参数 | 默认值 | 含义 |
|------|--------|------|
| `interval_hours` | 24×7（7 天） | Curator 运行间隔 |
| `min_idle_hours` | 2 | 至少空闲多久才允许运行 |
| `stale_after_days` | 30 | 多久未用判定为"陈旧" |
| `archive_after_days` | 90 | 多久后归档 |

此外 `tools/skill_provenance.py` 用 ContextVar 标记**写入来源**：区分"后台复盘 fork（`background_review`）"写的技能与"前台用户指示"写的技能。**Curator 只整理/裁剪它自己自动创建的技能**；用户让前台 Agent 写的技能属于用户，**绝不会被自动整理**。这是一条非常重要的"边界纪律"。

---

## 3. 技能进化的工程亮点（为什么值得学习）

1. **影子线程隔离**：自我进化在 fork 的 Agent + 守护线程里完成，**不污染主上下文、不破坏前缀缓存**——既"会学习"又"不拖慢"。
2. **来源可溯（provenance）**：用 ContextVar 严格区分"自创建 vs 用户创建"，自动维护只动自己的产物，避免破坏用户资产。
3. **只归档不删除**：所有自动化操作可恢复，降低"自我进化"的风险。
4. **为注意力做减法**：强制 60 字符描述、类级技能结构，避免技能膨胀稀释模型注意力。
5. **挫败即学习**：把用户纠正/不满作为一等学习信号，直接改写行为。

---

## 4. 对比：OpenClaw 的技能模型

| 维度 | Hermes Agent | OpenClaw |
|------|--------------|----------|
| 技能形态 | `SKILL.md` + 目录（agentskills.io 标准） | `SKILL.md`（bundled/managed/workspace） |
| 来源 | 内建 + 可选 + 用户/Agent 自创建 | 内建 + 托管 + 工作区自定义 |
| **自动创建** | ✅ 任务后后台复盘自动沉淀 | ❌ 主要人工编写 |
| **使用中自改进** | ✅ 复盘提示主动更新 | ❌ |
| **后台生命周期维护** | ✅ Curator（置顶/归档/合并/补丁） | ❌ |
| 注册中心 | agentskills.io | ClawHub |
| 组合方式 | 工具集 + 技能注入 | 离散可复用技能拼装 Agent |

OpenClaw 的技能是**强大且模块化的"积木"**——你通过组合离散、可复用的技能来拼装 Agent，每个技能自带工具/提示/能力（[serveravatar](https://serveravatar.com/openclaw-vs-paperclip-vs-hermes/)，内容经转述）。但其进化责任在**人**：用户负责编写、安装、维护。

**一句话总结差异**：
- OpenClaw = "**人来策展**技能库"（手动、透明、可控）；
- Hermes = "**Agent 自我策展 + 人监督**技能库"（自动、闭环、可溯、可恢复）。

> 下一篇：[05-记忆优化.md](./05-记忆优化.md)
