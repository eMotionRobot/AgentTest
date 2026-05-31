# OpenClaw 深度解析

> 仓库：[openclaw/openclaw](https://github.com/openclaw/openclaw) · 许可证：MIT · 作者：Peter Steinberger 及社区（前身 Clawdbot）
> 定位：**「Personal AI Assistant」——一个跑在你自己设备上的个人 AI 助理**。它通过你已经在用的消息渠道回应你，可在 macOS/iOS/Android 上语音收发，并能渲染由你控制的实时 Canvas。

> 资料来源：OpenClaw 官方仓库 README、官方文档（docs.openclaw.ai）及公开技术文章。第三方观点已转述并标注来源，内容经改写以符合授权要求。

---

## 1. 设计哲学：网关即平台（Gateway-First）

OpenClaw 的核心抽象是 **Gateway（网关）**——一个常驻进程，作为人与 Agent 之间的**控制面（control plane）**，统一管理：会话（sessions）、渠道（channels）、工具（tools）、事件（events）与路由（routing）。

正如官方 README 所述：「**The Gateway is just the control plane — the product is the assistant**（网关只是控制面，产品是助理本身）」。模型是可插拔的。多篇技术分析也指出 OpenClaw 把"会回答的聊天机器人"变成了"会行动的 Agent"——一个常驻在你自有硬件上、通过你熟悉的 App 访问的持久助理（[ppaolo, Substack](https://ppaolo.substack.com/p/openclaw-system-architecture-overview)，内容经转述）。

---

## 2. 整体架构

```text
   渠道（Channels）                         设备节点（Nodes）
   WhatsApp/Telegram/Slack/Discord/         macOS App / iOS / Android
   Signal/iMessage/Matrix/WeChat/...        （Voice Wake / Canvas / 摄像头/屏幕）
        │                                          │
        └───────────────┬──────────────────────────┘
                        ▼
        ┌───────────────────────────────────────────┐
        │            Gateway（控制面/常驻进程）        │
        │  会话路由 · 权限 · 渠道适配 · 技能分发 · 事件 │
        └───────────────┬───────────────────────────┘
                        ▼
        ┌───────────────────────────────────────────┐
        │         Agent Runtime（Agentic Loop）       │
        │   提出并执行工具调用，链式自主推进直到完成    │
        └───────────────┬───────────────────────────┘
                        ▼
        ┌───────────────────────────────────────────┐
        │  工具/技能（permissioned，可选沙箱）          │
        │  bash/process/read/write/edit/browser/      │
        │  canvas/nodes/cron/sessions/...             │
        └───────────────────────────────────────────┘
```

一篇生产经验文章将其概括为：网关拥有渠道适配器、会话与到 Agent 运行时的路由；Agent 循环负责提出并执行工具；工具/技能被赋权并可选择性沙箱化（[skywork.ai](https://skywork.ai/blog/ai-agent/clawdbot-developer-lessons/)，内容经转述）。

---

## 3. Agentic Loop（自主循环）

OpenClaw 区别于普通聊天机器人的关键是 **Agentic Loop**：它会把多次工具调用**链式串联**，自主推进直到任务完成，而**不需要每一步都给提示**（[roborhythms.com](https://www.roborhythms.com/how-openclaw-ai-agent-works/)，内容经转述）。整条管线大致为：消息经 Gateway → Agent Runner → Agentic Loop → Response Path → 返回到对应渠道。

---

## 4. 工作区与注入文件（Workspace）

OpenClaw 几乎完全通过**纯文本 Markdown 文件**来配置 Agent：

- **工作区根目录**：`~/.openclaw/workspace`（可由 `agents.defaults.workspace` 配置）。
- **注入到提示词的文件**：
  - `AGENTS.md` —— 工作区级指令；
  - `SOUL.md` —— 定义 Agent 的人格、价值观、语气与行为边界，是每次会话开始**最先注入**上下文的文件；
  - `TOOLS.md` —— 工具相关说明。
- 还有社区约定的 `HEARTBEAT.md` 等文件（用于自动化/心跳）。

> SOUL.md 已发展成一个小生态（如 [soul.md](https://soul.md)、`awesome-openclaw-agents` 收录了 162 个跨 19 类的 SOUL.md 模板），可让 Agent "拥有灵魂/人格"。

---

## 5. 技能系统（Skills）

- 技能位于 `~/.openclaw/workspace/skills/<skill>/SKILL.md`。
- 三类来源：**bundled（内建）/ managed（托管）/ workspace（工作区自定义）**。
- 技能注册中心：**[ClawHub](https://clawhub.ai)**。
- 每个技能是自包含单元，拥有自己的工具、提示与能力；你通过组合离散、可复用的技能来"拼装" Agent（[serveravatar.com](https://serveravatar.com/openclaw-vs-paperclip-vs-hermes/)，内容经转述）。

> 与 Hermes 不同，OpenClaw 的技能**主要靠人工编写/安装**，没有内建的"任务后自动造技能 + 后台自动维护"闭环（详见 [04-skill技能进化.md](./04-skill技能进化.md)）。

---

## 6. 渠道、设备节点与多模态

OpenClaw 在"接入面"上非常强：

- **20+ 消息渠道**：WhatsApp、Telegram、Slack、Discord、Google Chat、Signal、iMessage、IRC、Microsoft Teams、Matrix、Feishu(飞书)、LINE、Mattermost、Nextcloud Talk、Nostr、Synology Chat、Tlon、Twitch、Zalo、WeChat(微信)、QQ、WebChat 等。
- **设备节点（Nodes）**：macOS 菜单栏 App、iOS / Android 节点（通过网关 WebSocket 配对）。
- **Voice Wake + Talk Mode**：macOS/iOS 唤醒词、Android 持续语音（ElevenLabs + 系统 TTS 兜底）。
- **Live Canvas**：由 Agent 驱动、你可控制的可视化工作区（A2UI）。

---

## 7. 自动化（Automation）

OpenClaw 通过多种机制在后台运行工作：**tasks（任务）、scheduled jobs（定时作业/cron）、inferred commitments（推断出的承诺）、event hooks（事件钩子）、standing instructions（常驻指令）**（[官方文档 automation](https://clawdhub.mintlify.app/automation)，内容经转述）。其中 **cron 与 heartbeat** 是两类典型后台触发方式。

---

## 8. 安全模型

OpenClaw 直连真实消息渠道，因此把入站 DM 视为**不可信输入**：

- **默认**：`main` 会话的工具在宿主机上运行——当只有你自己时拥有完整访问权限。
- **群组/渠道安全**：设 `agents.defaults.sandbox.mode: "non-main"`，让非 `main` 会话在沙箱中运行；默认沙箱后端为 **Docker**，也支持 **SSH 与 OpenShell**。
- **典型沙箱白名单**：允许 `bash/process/read/write/edit/sessions_*`，拒绝 `browser/canvas/nodes/cron/discord/gateway`。
- **DM 配对（pairing）**：陌生发件人收到配对码，需 `openclaw pairing approve` 批准后才进入本地白名单；公开入站 DM 需显式 opt-in。
- `openclaw doctor` 可体检风险配置。

---

## 9. 安装与运行

```bash
npm install -g openclaw@latest        # 运行时建议 Node 24（或 22.19+）
openclaw onboard --install-daemon     # 引导式安装并以守护进程常驻
openclaw gateway status               # 查看网关状态
openclaw agent --message "Ship checklist" --thinking high
```

- 推荐用 `openclaw onboard` 引导式完成网关/工作区/渠道/技能配置；
- 最小配置 `~/.openclaw/openclaw.json` 仅需指定模型；
- 支持 npm / pnpm / bun；源码开发用 pnpm workspace。

---

## 10. 小结

OpenClaw 是一个**网关优先、工作区原生**的个人助理平台。它的强项在于：极其丰富的渠道与移动端接入、语音与 Canvas 多模态、纯 Markdown 文件驱动的透明配置、以及围绕 SOUL.md / ClawHub 形成的活跃生态（200k+ stars）。它给予用户**更紧的手动控制**，但把"学习与自我进化"留给了用户去维护，而非系统自动完成。

> 下一篇：[03-框架与功能对比.md](./03-框架与功能对比.md)
