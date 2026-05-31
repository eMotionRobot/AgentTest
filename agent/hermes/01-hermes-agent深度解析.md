# Hermes Agent 深度解析

> 仓库：[NousResearch/hermes-agent](https://github.com/NousResearch/hermes-agent) · 许可证：MIT · 出品方：Nous Research
> 定位：**「The self-improving AI agent」——唯一内建学习闭环的 Agent**：从经验中创建技能、在使用中改进技能、主动沉淀知识、检索自身历史会话、跨会话构建对你的认知模型。

---

## 1. 设计哲学：自我进化（Self-Improving）

Hermes 的核心主张是 **闭环学习（closed learning loop）**。它不仅"执行任务"，还在每次交互后**反思并把经验固化下来**，因此"越用越懂你"。这一闭环由四个机制支撑：

1. **Agent 自策展记忆（agent-curated memory）** + 周期性"提醒（nudge）"持久化知识；
2. **复杂任务后自动创建技能（autonomous skill creation）**；
3. **技能在使用中自我改进（self-improve during use）**；
4. **FTS5 会话全文检索 + LLM 摘要** 实现跨会话回忆；外加 [Honcho](https://github.com/plastic-labs/honcho) 辩证式用户建模。

> 这些机制的详细拆解分别见 [04-skill技能进化.md](./04-skill技能进化.md) 与 [05-记忆优化.md](./05-记忆优化.md)。

---

## 2. 整体架构

Hermes 是 **Python 后端 + TypeScript(Ink/React) 终端 UI** 的双层结构，二者通过基于 stdio 的换行分隔 JSON-RPC 通信。

```text
                       ┌──────────────────────────────────────────┐
   入口                │  hermes (CLI/TUI)   hermes gateway (消息)   │
                       └───────────────┬──────────────────────────┘
                                       │
            ┌──────────────────────────┴───────────────────────────┐
            │                  AIAgent（run_agent.py）               │
            │   核心同步对话循环 · 工具编排 · 迭代预算 · 中断/重定向    │
            └──────┬───────────────┬──────────────┬─────────────────┘
                   │               │              │
        ┌──────────▼───┐  ┌────────▼───────┐  ┌───▼─────────────┐
        │ 工具系统      │  │ 记忆/技能子系统  │  │ 上下文压缩       │
        │ tools/*.py   │  │ memory/skills   │  │ context_compr.  │
        │ + registry   │  │ + curator       │  │ + FTS5 检索      │
        └──────┬───────┘  └────────────────┘  └─────────────────┘
               │
        ┌──────▼───────────────────────────────────────────────┐
        │ 终端后端（environments）：local / docker / ssh /        │
        │                          singularity / modal / daytona│
        └────────────────────────────────────────────────────────┘
```

### 2.1 关键模块（源码视角）

| 文件 / 目录 | 职责 |
|------------|------|
| `run_agent.py` | `AIAgent` 类，核心对话循环（约 12k 行），`run_conversation()` / `chat()` |
| `model_tools.py` | 工具编排，`discover_builtin_tools()`、`handle_function_call()` |
| `toolsets.py` | 工具集定义，`_HERMES_CORE_TOOLS` |
| `hermes_state.py` | `SessionDB`——基于 SQLite 的会话存储，FTS5 全文检索 |
| `agent/` | Agent 内部：供应商适配、记忆、压缩、缓存、后台复盘、Curator 等 |
| `tools/` | 工具实现，经 `tools/registry.py` 自动发现 |
| `tools/environments/` | 6 种终端后端 |
| `gateway/` | 消息网关 + 各平台适配器（`platforms/`） |
| `plugins/` | 插件体系（memory / model-providers / context_engine / kanban 等） |
| `skills/` · `optional-skills/` | 内建技能 / 可选技能 |
| `cron/` | 定时调度器 |
| `acp_adapter/` | ACP 服务（VS Code / Zed / JetBrains 集成） |

---

## 3. Agent Loop（对话主循环）

核心循环位于 `run_conversation()`，**完全同步**，带中断检查、预算追踪与"宽限调用"：

```python
while (api_call_count < self.max_iterations and self.iteration_budget.remaining > 0) \
        or self._budget_grace_call:
    if self._interrupt_requested:
        break
    response = client.chat.completions.create(model=model, messages=messages, tools=tool_schemas)
    if response.tool_calls:
        for tool_call in response.tool_calls:
            result = handle_function_call(tool_call.name, tool_call.args, task_id)
            messages.append(tool_result_message(result))
        api_call_count += 1
    else:
        return response.content
```

要点：
- 消息遵循 OpenAI 格式（`system/user/assistant/tool`），推理内容存于 `assistant_msg["reasoning"]`；
- `max_iterations` 默认 90，并与子代理共享；`iteration_budget` 控制整体工具调用预算；
- 支持 **中断并重定向**（`Ctrl+C` 或直接发新消息），适合长任务交互。

---

## 4. 工具系统（40+ 工具）

- **自动发现**：任何 `tools/*.py` 中含顶层 `registry.register()` 的文件会被自动导入注册；但要"暴露给模型"，仍需将工具名加入 `toolsets.py` 的某个工具集（如 `_HERMES_CORE_TOOLS`）——这是有意为之的手动步骤。
- **工具集（toolsets）**：按平台/场景分组启用，控制注入到模型 schema 的工具范围，避免 schema 膨胀。
- **Agent 级工具**：如 `todo`、`memory` 在 `run_agent.py` 中被提前拦截处理（不走通用分发）。
- 所有 handler 必须返回 JSON 字符串；registry 统一负责 schema 收集、分发、可用性检查与错误包装。

代表性工具（与本研究相关）：

| 工具文件 | 作用 |
|---------|------|
| `tools/memory_tool.py` | 持久化记忆（MEMORY.md / USER.md） |
| `tools/skill_manager_tool.py` | 技能创建/编辑/补丁/删除 |
| `tools/delegate_tool.py` | 子代理委派（单任务/批量并行） |
| `tools/kanban_tools.py` | Kanban 多 Agent 看板编排 |
| `tools/mixture_of_agents_tool.py` | Mixture-of-Agents 多模型协作 |
| `tools/todo_tool.py` | 任务清单（上下文内规划） |

---

## 5. 插件体系（高可扩展）

Hermes 的插件面非常丰富，且有明确的**架构纪律**：插件不得修改核心文件，若需新能力应扩展通用插件接口（新钩子 / 新 ctx 方法）。

- **通用插件**：`register(ctx)`，可注册生命周期钩子（`pre/post_tool_call`、`pre/post_llm_call`、`on_session_start/end`）、新工具、CLI 子命令。
- **记忆插件**（`plugins/memory/`）：实现 `MemoryProvider` ABC；内建 honcho、mem0、supermemory、byterover、hindsight、holographic、openviking、retaindb 等；**同一时刻只允许一个外部记忆后端**（防止 schema 膨胀与冲突）。新后端须以独立插件仓库形式发布。
- **模型供应商插件**（`plugins/model-providers/`）：openrouter、anthropic、gmi、deepseek、nvidia 等，每个在加载时调用 `register_provider(ProviderProfile(...))`，惰性发现。
- **上下文引擎 / 图像生成 / 看板** 等插件目录遵循相同模式（ABC + 编排器 + 每插件目录）。

---

## 6. 部署形态（"不绑定你的笔记本"）

Hermes 强调**可在任意环境运行**，提供 6 种终端后端：

| 后端 | 说明 |
|------|------|
| local | 本地直接执行 |
| docker | 容器隔离 |
| ssh | 远程主机 |
| singularity | HPC/科研集群常用 |
| **modal** | Serverless，闲置休眠、按需唤醒，几乎零成本 |
| **daytona** | Serverless 持久化环境 |

配合消息网关，你可以在 Telegram 上与一台云端 VM 上的 Agent 对话——它不依附于你的本地机器。可跑在 5 美元 VPS、GPU 集群或近乎免费的 Serverless 上。

---

## 7. 多渠道网关与自动化

- **网关（gateway）**：单一进程接入 Telegram、Discord、Slack、WhatsApp、Signal、Email 等，支持语音备忘转写、跨平台会话连续性。
- **Cron 调度**：自然语言描述的定时任务（日报、夜间备份、周度审计），可投递到任意平台。
- **模型自由**：300+ 模型（Nous Portal / OpenRouter / NovitaAI / NVIDIA NIM / 智谱 GLM / Kimi / MiniMax / HuggingFace / OpenAI / 自建端点），`hermes model` 一键切换，无锁定。
- **MCP**：一等公民，可接入任意 MCP server 扩展能力。

---

## 8. 研究/训练取向（Nous 特色）

作为研究机构出品，Hermes 还面向"训练下一代工具调用模型"：
- **批量轨迹生成**（`batch_runner.py`）；
- **轨迹压缩**（`trajectory_compressor.py`）用于训练数据生产。

这使 Hermes 不只是产品，也是 Agent 行为数据的生产管线。

---

## 9. 小结

Hermes Agent 的差异化在于把"学习"做成了**系统级的后台闭环**：记忆与技能不是被动存储，而是由 Agent 自身在后台复盘、创建、改进、并由 Curator 维护其生命周期。叠加 FTS5 跨会话检索、可插拔记忆后端、6 种终端后端与 Serverless 持久化，Hermes 在"长期运行、随用随长"的个人智能体方向上走得更深。

> 下一篇：[02-openclaw深度解析.md](./02-openclaw深度解析.md)
