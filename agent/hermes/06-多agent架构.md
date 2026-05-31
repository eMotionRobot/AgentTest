# 专题：多 Agent 架构（Multi-Agent）

> 本文拆解 Hermes Agent 的三种多 Agent 能力——**子代理委派（delegate）、Kanban 看板编排、Mixture-of-Agents 多模型协作**——并对比 OpenClaw 的"渠道→隔离 Agent 路由"模型。

---

## 1. Hermes 的三种多 Agent 形态

```text
   ┌────────────────────────────────────────────────────────────┐
   │ A. 子代理委派 delegate_tool        —— 纵向：父派子，隔离并行  │
   │ B. Kanban 看板 kanban_tools        —— 横向：编排者/工人协作   │
   │ C. Mixture-of-Agents               —— 纵深：多模型分层共识    │
   └────────────────────────────────────────────────────────────┘
```

---

## 2. A. 子代理委派（delegate_tool.py）

**目的**：派生子 `AIAgent` 实例，拥有**隔离上下文、受限工具集、独立终端会话**。支持**单任务**与**批量（并行）**模式，父代理**阻塞直到所有子代理完成**。

每个子代理获得：

| 属性 | 说明 |
|------|------|
| 全新对话 | **不含父代理历史**（fresh conversation） |
| 独立 `task_id` | 自己的终端会话、文件操作缓存 |
| 受限工具集 | 可配置；被封禁的工具始终被剥离 |
| 聚焦系统提示 | 由"委派目标 + 上下文"构建 |

**关键设计——上下文防火墙**：父代理的上下文**只看到委派调用本身与子代理返回的摘要结果**，**永远看不到子代理的中间工具调用与推理过程**。

> 这是一个非常重要的**上下文经济学（context economics）**设计：把一个多步子流程"坍缩"成父上下文里的一次调用 + 一段摘要，**几乎零上下文成本**。README 称之为"collapsing multi-step pipelines into zero-context-cost turns"。

**并行批量**：用线程池并发执行多个子任务（`ThreadPoolExecutor`，带超时控制），适合"分头调研/并行处理"类工作流。

```text
        父 Agent
          │  delegate(batch=[t1, t2, t3], toolset=restricted)
          ├──────────────┬──────────────┬───────────────
          ▼              ▼              ▼
       子Agent t1     子Agent t2     子Agent t3      （各自隔离上下文/终端/工具）
          │              │              │
          └──── 仅摘要 ───┴──── 仅摘要 ──┘
          ▼
   父上下文只见：调用 + 各子任务摘要（中间过程不可见）
```

---

## 3. B. Kanban 看板编排（kanban_tools.py + plugins/kanban）

**目的**：以**看板（Kanban board）**为协调中枢，支撑 **编排者（orchestrator）/ 工人（worker）** 多 Agent 协作。状态存于 `~/.hermes/kanban.db`。

### 为何用"工具"而非直接 shell 调 `hermes kanban`？（源码注释，已转述）

1. **后端可移植**：工人的终端可能指向 Docker/Modal/Singularity/SSH，容器里没装 `hermes`、也没挂载 DB；而工具运行在 Agent 的 Python 进程内，**无论终端后端如何都能访问** `~/.hermes/kanban.db`。
2. **无 shell 引号陷阱**：结构化工具参数避免 `--metadata '{"x":[...]}'` 经 shlex+argparse 的脆弱性。
3. **更好的错误**：工具失败返回结构化 JSON，模型可据此推理，而非解析 stderr。

> 人类继续用 CLI（`hermes kanban …`）、Dashboard（`hermes dashboard`）、斜杠命令（`/kanban …`）——三者都绕过 Agent。工具则专供**调度器派生的工人交接**与**配置了 kanban 工具集的编排者**。普通 `hermes chat` 会话默认**看不到任何 kanban 工具**。

### 任务生命周期状态

```text
  triage ──▶ todo ──▶ ready ──▶ running ──▶ done ──▶ archived
                                   │
                                   └──▶ blocked ──(unblock)──▶ ready
```

| 工具/动作 | 作用 |
|-----------|------|
| `kanban_complete` | 将当前任务标记完成，附**结构化交接（handoff）**；校验 `created_cards`（已创建卡片需被认领，或显式置空跳过） |
| `kanban_block` | 转为 `blocked` 并给出**人类可读的原因** |
| `kanban_heartbeat` | 心跳：延长认领 TTL（`heartbeat_claim`）+ 记录工人心跳事件 |
| unblock | 把 blocked 任务转回 ready |

### 认领与防失联（claim / heartbeat）

- 工人通过**认领（claim）**锁定任务，并定期心跳延长认领 TTL；
- `release_stale_claims` 会回收超过 `DEFAULT_CLAIM_TTL_SECONDS` 仍无心跳的"失联"任务；
- 长耗时操作需主动 `heartbeat_claim`，否则即便在运行也会被回收（源码注释明确警告这个"陷阱"）；
- 环境变量 `HERMES_KANBAN_TASK`（标记运行在调度器下）、`HERMES_KANBAN_CLAIM_LOCK`（认领锁）协同工作。

### 角色分工

- **编排者**：可创建任务、重开 blocked 任务；
- **工人**：被**窄范围限定**于其任务，命名如 `researcher-a` / `reviewer` / `writer`；
- `plugins/kanban/` 提供 **dashboard 插件 API** 与 **systemd 调度器服务**（`hermes-kanban-dispatcher.service`），用于让看板调度器作为后台服务常驻派发任务。

```text
        ┌──────────────┐         认领/心跳        ┌────────────┐
        │  编排者 Agent  │◀───────────────────────▶│ kanban.db  │
        │ 建任务/重开    │                          │ (SQLite)   │
        └──────┬───────┘                          └─────┬──────┘
               │ 调度器派发                               │
        ┌──────▼──────┐   ┌──────────────┐   ┌──────────▼─────┐
        │ worker:研究  │   │ worker:写作   │   │ worker:评审     │
        │ researcher-a │   │ writer        │   │ reviewer        │
        └─────────────┘   └──────────────┘   └────────────────┘
```

---

## 4. C. Mixture-of-Agents（mixture_of_agents_tool.py）

**目的**：用**多个前沿 LLM 的分层架构**汇聚集体优势，攻克需要高强度推理的难题（编码/数学/复杂分析）。实现基于论文 *"Mixture-of-Agents Enhances Large Language Model Capabilities"*（[arXiv:2406.04692](https://arxiv.org/abs/2406.04692)）。

架构：

```text
  查询 ─┬─▶ 参考模型A ─┐
        ├─▶ 参考模型B ─┤
        ├─▶ 参考模型C ─┼─▶ 聚合模型（aggregator）─▶ 高质量综合输出
        └─▶ 参考模型D ─┘
        （并行生成多样化初稿）      （合成/择优/纠错）
```

- **参考模型**并行生成多样化初始回答（如 Claude Opus / Gemini Pro / GPT 系列 / DeepSeek 等，经 OpenRouter）；
- **聚合模型**（取最高能力模型）综合这些回答为高质量输出；
- 可多层迭代精炼。

> 与 delegate / kanban 是"任务分解协作"不同，MoA 是"**同一问题、多模型共识**"，本质是**用多样性换正确性**。

---

## 5. 对比：OpenClaw 的多 Agent 模型

OpenClaw 的多 Agent 走的是**「渠道 → 隔离 Agent 路由」**路线：

- **多 Agent 路由**：把入站的**渠道 / 账号 / 对端（channels/accounts/peers）路由到隔离的 Agent**，每个 Agent 有**独立工作区 + 独立会话**（[官方 README/配置文档](https://docs.openclaw.ai/gateway/configuration)，内容经转述）。有评测转述为"每个 Agent 拥有自己的渠道与人格"（[kilo.ai](https://kilo.ai/kiloclaw/openclaw-vs-hermes)，内容经转述）。
- **会话派生工具**：`sessions_spawn`（派生子会话）、`sessions_list`、`sessions_history`、`sessions_send`——可由一个会话派生/查询/向另一个会话发送，构成轻量的多会话协作。
- **沙箱**：非 `main` 会话可在 Docker/SSH/OpenShell 沙箱中运行，天然适合"多 Agent + 隔离"。

### 横向对比

| 维度 | Hermes Agent | OpenClaw |
|------|--------------|----------|
| 子代理派生 | ✅ `delegate`（隔离上下文/工具，批量并行，仅回摘要） | ✅ `sessions_spawn`（派生会话） |
| 上下文隔离 | ✅ 父只见摘要，中间过程不可见 | ✅ 每 Agent 独立工作区/会话 |
| 显式编排器 | ✅ Kanban 看板（状态机 + 认领/心跳 + 调度器服务） | ⚠️ 以渠道路由为主，无内建看板状态机 |
| 多模型共识 | ✅ Mixture-of-Agents（分层聚合） | ⚠️ 主要是模型失败回退 |
| 路由维度 | 工具集/任务驱动 | **渠道/账号/对端 → Agent** |
| 持久协作存储 | `~/.hermes/kanban.db`（SQLite） | 会话模型 |
| 后台调度 | systemd dispatcher + 空闲触发 | 守护进程 + cron/heartbeat |

---

## 6. 总结：两种多 Agent 哲学

- **Hermes**：偏"**任务工程**"——把复杂工作**分解（delegate）、编排（kanban）、共识（MoA）**，强调**上下文经济学**（子代理只回摘要）与**可观测的协作状态机**（看板）。更适合"一个用户、复杂工作流并行推进"。
- **OpenClaw**：偏"**身份/渠道工程**"——把不同**渠道/账号/对端**映射到不同**人格 + 工作区**的隔离 Agent，再用 `sessions_*` 做轻量会话协作。更适合"多渠道、多人格、多入口"的个人助理矩阵。

两者并不互斥：Hermes 解决"**一个任务如何被多个 Agent 高效完成**"，OpenClaw 解决"**多个入口如何对应多个 Agent 身份**"。

> 返回：[README.md](./README.md)
