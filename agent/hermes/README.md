# Hermes Agent 与 OpenClaw 框架对比研究

> 本目录是对两款当下最受关注的「个人/自主 AI Agent」开源框架的深度调研与对比分析。
> 重点剖析 **框架架构、功能矩阵、技能进化（Skill Evolution）、记忆优化（Memory Optimization）、多 Agent 协作（Multi-Agent）** 五个维度。

研究对象：

| 项目 | 仓库 | 出品方 | 主语言 | 许可证 | 一句话定位 |
|------|------|--------|--------|--------|-----------|
| **Hermes Agent** | [NousResearch/hermes-agent](https://github.com/NousResearch/hermes-agent) | Nous Research | Python（后端）+ TypeScript/React-Ink（TUI） | MIT | 「会随你成长」的自我进化型 AI Agent |
| **OpenClaw** | [openclaw/openclaw](https://github.com/openclaw/openclaw) | Peter Steinberger 及社区（原名 Clawdbot） | Node.js / TypeScript | MIT | 跑在你自己设备上的「个人 AI 助理」 |

> 说明：本研究基于对 hermes-agent 源码仓库的实际克隆与源码阅读，以及 OpenClaw 官方仓库 README、官方文档与公开技术文章。第三方文章中的观点已做转述并标注来源，内容经改写以符合授权要求。

---

## 文档导航

| 文档 | 内容 |
|------|------|
| [01-hermes-agent深度解析.md](./01-hermes-agent深度解析.md) | Hermes Agent 架构、Agent Loop、工具系统、插件体系、部署形态 |
| [02-openclaw深度解析.md](./02-openclaw深度解析.md) | OpenClaw 网关架构、Agentic Loop、工作区文件、渠道与节点、安全模型 |
| [03-框架与功能对比.md](./03-框架与功能对比.md) | 两者设计哲学、功能矩阵、性能/部署/生态横向对比与选型建议 |
| [04-skill技能进化.md](./04-skill技能进化.md) | **技能进化**：技能即程序性记忆、自创建、自改进、Curator 生命周期管理 |
| [05-记忆优化.md](./05-记忆优化.md) | **记忆优化**：分层记忆、冻结快照、上下文压缩、FTS5 会话检索、用户建模 |
| [06-多agent架构.md](./06-多agent架构.md) | **多 Agent**：子代理委派、Kanban 编排、Mixture-of-Agents、渠道路由隔离 |

---

## 30 秒速览：核心差异

```text
设计哲学
  Hermes  = 「自我进化的智能体」：内建闭环学习，越用越懂你
  OpenClaw= 「网关即平台」：以 Gateway 控制面为核心，模型可插拔，工作区原生

最大区分点
  Hermes  独有：① 后台自我复盘（写记忆/造技能）② Curator 技能生命周期管理
                ③ FTS5 跨会话检索 ④ 6 种终端后端 + Serverless 持久化
  OpenClaw独有：① 多渠道数量更多 + 移动端 Node ② Voice Wake / Live Canvas
                ③ 更成熟的生态（200k+ stars）④ 网关优先、工作区原生的手动控制
```

更详细的对比见 [03-框架与功能对比.md](./03-框架与功能对比.md)。

---

## 快速对比表（精简版）

| 维度 | Hermes Agent | OpenClaw |
|------|--------------|----------|
| 核心抽象 | 自进化 Agent（AIAgent 循环） | Gateway 控制面 + Agentic Loop |
| 实现语言 | Python + Ink/React TUI | Node.js / TypeScript |
| 技能自创建 | ✅ 任务后自动沉淀技能 | ⚠️ 主要靠人工编写 SKILL.md |
| 技能自改进 | ✅ 使用中改进 + Curator 后台维护 | ❌ 无内建自动维护 |
| 记忆持久化 | MEMORY.md / USER.md + 可插拔记忆后端 | SOUL.md / MEMORY.md（Markdown 文件） |
| 跨会话检索 | ✅ SQLite FTS5 + 三元组（CJK 友好） | ⚠️ 依赖会话历史，无内建全文检索 |
| 上下文压缩 | ✅ 辅助模型摘要 + 头尾保护 | ✅ `/compact` 命令 |
| 子代理/并行 | ✅ delegate 子代理 + 批量并行 | ✅ `sessions_spawn` 派生会话 |
| 多 Agent 编排 | ✅ Kanban 看板 + MoA 多模型 | ✅ 渠道→隔离 Agent 路由 |
| 消息渠道 | Telegram/Discord/Slack/WhatsApp/Signal 等 | 20+ 渠道 + macOS/iOS/Android Node |
| 定时/自动化 | ✅ Cron 调度 | ✅ Cron + Heartbeat + 事件钩子 |
| 模型供应 | 300+（Portal/OpenRouter/自建等） | 可插拔（OpenAI OAuth 等） |
| MCP 支持 | ✅ 一等公民 | ✅ 支持 |
| 迁移工具 | ✅ `hermes claw migrate`（从 OpenClaw 导入） | — |

> 表中 ✅/⚠️/❌ 为相对程度评估，并非绝对；以各项目最新版本为准。
