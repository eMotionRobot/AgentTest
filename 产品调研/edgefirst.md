# EdgeFirst（Au-Zone Technologies）公司与产品调研

> 调研对象：[EdgeFirst AI 官网](https://www.edgefirst.ai/)
> 调研时间：2026-05
> 资料来源：EdgeFirst/Au-Zone 官网、文档站（doc.edgefirst.ai）、NXP 博客、Edge AI and Vision Alliance、Crunchbase、CB Insights、Hugging Face 等公开资料

---

## 1. 公司概览

### 1.1 公司基本信息

| 项目 | 内容 |
| --- | --- |
| 公司名称 | Au-Zone Technologies, Inc. |
| 产品品牌 | **EdgeFirst™**（前身 Deep View / DeepView Enterprise） |
| 官网 | https://www.edgefirst.ai/ （产品站） / https://www.embeddedml.com/ （公司站） |
| 总部 | 加拿大 艾伯塔省 卡尔加里（Calgary, Alberta, Canada） |
| 成立年份 | 2001 年 |
| 创始人 | Brad Scott |
| 员工规模 | 11–50 人（Crunchbase 数据） |
| 所处行业 | 嵌入式 AI、视觉感知、机器学习、自主机器、机器人 |
| 融资状态 | 私营企业，曾获 NXP 半导体战略投资 |
| 文档站 | https://doc.edgefirst.ai/ |
| 模型发布 | https://huggingface.co/EdgeFirst |

### 1.2 公司定位与愿景

Au-Zone Technologies 是一家拥有 20 多年嵌入式视觉和边缘 AI 经验的加拿大科技公司，长期为 NXP 等 MCU/应用处理器厂商提供机器学习工具链（早期产品为 **DeepView ML Tool Suite**）。公司定位于"边缘优先（Edge-First）的 3D/4D 空间感知（Spatial Perception）技术提供商"，目标客户为 OEM（主机厂）与系统集成商（Tier 1/系统供应商）。

公司愿景：在非结构化、恶劣、不确定的作业环境（越野车辆、工程机械、矿山、农业、机器人）中，让机器拥有可信赖、精确、可靠的感知能力，设定安全、视觉导航与自主作业的新标准。

资料来源：[Au-Zone 官网 About Us](https://www.embeddedml.com/about-us)，[Crunchbase](https://www.crunchbase.com/organization/au-zone-technologies)，[CB Insights](https://www.cbinsights.com/company/au-zone-technologies/)

### 1.3 发展沿革（关键节点）

- **2001**：Au-Zone Technologies 成立，提供嵌入式视觉开发工具与设计服务。
- **2020-10**：NXP 半导体与 Au-Zone 建立排他性战略合作，以投资方式把 **DeepView ML Tool Suite** 整合进 NXP 的 **eIQ** 机器学习开发环境，用于 i.MX 应用处理器、i.MX RT 跨界 MCU 等的 ML 模型导入、训练与部署。（资料来源：[eeNews Europe 报道](https://www.eenewseurope.com/en/nxp-expands-ml-offering-with-au-zone-partnership/)、[IoT Evolution 报道](https://www.iotevolutionworld.com/fog/articles/447050-nxp-expands-scalable-machine-learning-portfolio-capabilities.htm)）
- **2023–2024**：推出 **Maivin** AI 视觉启动套件（基于 NXP i.MX 8M Plus）与 **Raivin** 雷达-视觉融合模组，产品成熟进入量产。
- **2025-01**：与 **The Ocean Cleanup** 合作的 Automated Debris Imaging System（ADIS）公开报道，EdgeFirst 助力海洋塑料漂浮物的自动化检测。（资料来源：[Edge AI and Vision Alliance](https://www.edge-ai-vision.com/2025/01/how-au-zone-technologies-plays-a-key-role-in-the-ocean-cleanup-automated-debris-imaging-system/)）
- **2025-11-19**：Au-Zone 正式对外扩大 **EdgeFirst Studio™** 的通用访问（general access），定位为"为边缘空间感知打造的企业级 MLOps 平台"，早期用户报告开发周期可缩短 20–50 倍。（资料来源：[Edge AI and Vision Alliance](https://www.edge-ai-vision.com/2025/11/au-zone-technologies-expands-edgefirst-studio-access/)）
- **2026-03**：与 NXP 共同发布 "Conversations at the Edge" 视频系列，强调 Raivin 多模态（vision+radar+lidar）感知的产品价值。

### 1.4 战略合作伙伴

- **NXP 半导体**：最深度合作伙伴，Au-Zone 的软件栈是 NXP eIQ 生态的一部分；Maivin/Raivin 均以 NXP i.MX 8M Plus 为主算力平台，未来也覆盖 i.MX 93 / i.MX 95 / Ara240 等。
- **Toradex、Vision Components**：Maivin AI Vision Starter Kit 的联合参考设计方。
- **smartmicro**：Raivin 内置的 **DRVEGRD-169** 汽车级 4D 雷达模组提供方。
- **Hugging Face 生态**：EdgeFirst 组织下发布 YOLO26 等在 i.MX 8M Plus / i.MX 93 / i.MX 95 / Ara240 / Jetson / RPi5+Hailo 等硬件上的预编译模型。

---

## 2. 产品与技术体系总览

EdgeFirst AI 是一个从 **数据 → 模型 → 中间件 → 硬件** 的端到端技术栈，面向越野、工业与移动机器人应用。整体架构大致可分为 4 个层次：

```
┌───────────────────────────────────────────────┐
│  应用 (AMR/UGV、矿山、农业、建筑、国防、海洋) │
├───────────────────────────────────────────────┤
│  EdgeFirst Studio (SaaS MLOps 平台)           │
│    ├─ Datasets / Data Pipelines               │
│    ├─ Auto-Labeling / Annotation              │
│    ├─ Vision / Fusion 模型训练与对比          │
│    └─ 模型部署与生命周期管理                  │
├───────────────────────────────────────────────┤
│  EdgeFirst Perception Engine / Middleware     │
│  (运行在设备端的推理与传感器融合中间件)        │
├───────────────────────────────────────────────┤
│  Field-Ready Modules                          │
│    ├─ Maivin  (IP66/67 视觉模组)              │
│    └─ Raivin  (视觉+雷达/LiDAR 融合模组)      │
└───────────────────────────────────────────────┘
```

下面分别介绍每一条产品线。

---

## 3. 核心产品详解

### 3.1 EdgeFirst Studio™ —— 企业级 MLOps 平台（深度分析）

> 官网：https://www.edgefirst.ai/edgefirststudio
> 文档：https://doc.edgefirst.ai/saas/studio/
> 官方 PDF（7 Key Features 1-Pager）：[EdgeFirst Studio 7 Powerful Features](https://30712b24-5b58-49b8-a906-e42ddbb7eacb.filesusr.com/ugd/c11122_8f471536a35541108cc5d5a0c330136c.pdf)

EdgeFirst Studio（原名 **Deep View Enterprise**）是整个 EdgeFirst AI 平台的"控制中心"，定位为**面向边缘空间感知的企业级 MLOps 平台**，提供从数据采集、标注、模型训练、对比评估到部署与生命周期管理的全流程多用户 SaaS 工作台。

#### 3.1.1 核心功能模块详解

##### A. Automatic Ground Truth Generation（AGTG）—— 自动标注管线

- **核心原理**：以检测模型（如 ModelPack / YOLO）作为"提示器"，驱动 **SAM-2**（Segment Anything Model 2）对视频帧中的目标进行自动分割与跟踪，并从分割掩码中生成 2D/3D Bounding Box 标注。
- **两种调用模式**：
  - **全自动（Fully Automatic）**：在导入数据集（Snapshot）时作为后台任务自动触发，整个数据集一次性完成。
  - **半自动（Semi-Automatic）**：用户在数据集 Gallery 中手动触发 AI 辅助标注，支持对单帧/连续帧进行逐目标级精调。
- **输出格式**：生成 2D BBox、Segmentation Mask，以及 3D BBox（如果数据含有 LiDAR / Radar 点云）。
- **价值**：相比手工逐帧画框，官方宣称 Ground Truth 生成速度提升 **100×**，大幅降低数据标注的人力成本。

（资料来源：[doc.edgefirst.ai/saas/studio/agtg](https://doc.edgefirst.ai/saas/studio/agtg/)、[doc.edgefirst.ai/saas/datasets/tutorials/annotations/automatic](https://doc.edgefirst.ai/saas/datasets/tutorials/annotations/automatic/)）

##### B. ModelPack —— 边缘实时检测与分割模型族

- **定义**：ModelPack 是 EdgeFirst Studio 内置的一组可训练模型架构，提供从 Nano / Small / Medium / Large 等多尺寸变体。
- **任务类型**：同时支持 **目标检测（Object Detection）** 与 **语义分割（Semantic Segmentation）**。
- **硬件要求**：仅需 **0.5 TOPS 及以上** 的 AI 加速器即可运行，适配 NXP i.MX 8M Plus（2 TOPS NPU）等低功耗嵌入式平台。
- **端到端体验**：官方教程展示"从数据采集到部署的实时检测/分割"可在一小时内完成。
- **基准测试**：在 ImageNet（分类）、PlayingCards（检测）、COCO（检测 & 分割）等公开数据集上提供 Benchmark 对比。

（资料来源：[doc.edgefirst.ai/saas/models/modelpack](https://doc.edgefirst.ai/saas/models/modelpack/)、[doc.edgefirst.ai/saas/models/modelpack/benchmarks](https://doc.edgefirst.ai/saas/models/modelpack/benchmarks/)）

##### C. Fusion Model —— 雷达-视觉融合模型

- **早期融合（Early Fusion）**：将雷达 **Range-Doppler 原始数据 Cube** 与 RGB 相机帧同时送入网络，端到端学习跨模态特征，适用于雨雾粉尘等降级场景。
- **中期融合（Mid-Level Fusion）**：使用雷达点云（PCD）并由检测模型为点云"上色"分配语义类别，作为高层传感器抽象的输入。
- **配套中间件**：包含数据集采集（MCAP Recording）、标定工具（Calibration）、Fusion 运行时等组件。
- **训练流程**：在 Studio 中创建含相机+雷达的联合数据集 → 标注 2D + 3D → 启动 Fusion Training Session → 评估并部署到 Raivin。

（资料来源：[doc.edgefirst.ai/saas/models/fusion](https://doc.edgefirst.ai/saas/models/fusion/)、[doc.edgefirst.ai/test/models/training/fusion](https://doc.edgefirst.ai/test/models/training/fusion/)）

##### D. 数据采集与回放（MCAP Recording & Replay）

- Maivin / Raivin 设备内置 **MCAP Recording Service**，可在现场一键录制多传感器同步数据流（相机、雷达、LiDAR 时间对齐），形成 .mcap 文件。
- **Replay Service** 允许在设备端或 Studio 中回放已录制数据，并叠加实时 Fusion / Vision 模型推理输出用于对比验证。
- 录制的 MCAP 可通过 `edgefirst-client` 自动上传到 Studio，转化为 Snapshot → 数据集 → 训练素材，形成数据飞轮。

（资料来源：[doc.edgefirst.ai/saas/platforms/recording](https://doc.edgefirst.ai/saas/platforms/recording/)、[doc.edgefirst.ai/saas/platforms/replay](https://doc.edgefirst.ai/saas/platforms/replay/)）

##### E. Foxglove Studio 集成 —— 可视化与调试

- EdgeFirst Studio 与 **Foxglove Studio**（开源/商用的机器人数据可视化工具）深度集成。
- 支持 3D 点云叠加、Camera + Radar Grid 联合可视化、Lidar View 等面板。
- 方便开发者在图形化界面中直觉地检查感知管线的输入-输出对齐情况。

##### F. 数据增强（Advanced Data Augmentation）

- 提供丰富的可配置视觉增强方法：亮度调节、噪声注入、模糊、几何变换等。
- 目标是训练出能够应对真实世界变化性的鲁棒模型，对于粉尘、光照剧变的越野环境尤为关键。

##### G. API-First 设计（edgefirst-client）

- 提供 **Python API**（`edgefirst_client` 模块）与 **CLI** 两种接入方式。
- 主要 Class：`Client`，支持 Token / 用户名密码认证。
- 可通过 API 完成：MCAP 上传、Snapshot 创建、数据集转换、模型训练触发、验证、部署等全部操作。
- 适合集成到客户自己的 CI/CD 管道或定制化开发者工作流。

（资料来源：[doc.edgefirst.ai/saas/perception/api/studio](https://doc.edgefirst.ai/saas/perception/api/studio/)、[doc.edgefirst.ai/saas/perception/studio](https://doc.edgefirst.ai/saas/perception/studio/)）

#### 3.1.2 典型工作流

EdgeFirst Studio 针对不同硬件和用户能力提供多条预定义 Workflow：

| Workflow | 适配硬件 | 流程概述 |
| --- | --- | --- |
| **Maivin Workflow** | Maivin 模组 | 录制 MCAP → 上传 → 标注 2D（BBox + Segmentation） → 训练 Vision 模型 → 验证 → 部署回 Maivin |
| **Raivin Workflow** | Raivin 模组 | 录制 MCAP（相机+雷达） → 上传 → 2D + 3D 标注 → 训练 Fusion 模型 → 验证 → 部署回 Raivin |
| **Web/Mobile Workflow** | 手机 / PC（无硬件） | 手机拍视频/照片 → 上传到 Studio → 标注 → 训练 → 部署到任意支持设备或 PC |
| **Tourist Workflow** | PC（入门体验） | 使用内置 Sample Project（Coffee Cup 数据集） → 标注 → 训练 → 部署到 PC，零硬件快速体验 |

#### 3.1.3 性能宣称（汇总）

| 维度 | 提升倍数 | 说明 |
| --- | --- | --- |
| 标注速度 | **20×** | 对比传统手工逐帧标注 |
| Ground Truth 生成 | **100×** | AGTG（SAM-2 驱动自动标注）vs 人工 |
| 训练与会话对比 | **4×** | 多 Session 对比评估效率 |
| 端到端研发周期 | **20–50×** | 早期客户反馈（含数据收集到部署） |
| ModelPack 首次部署 | **< 1 小时** | 从数据采集到部署实时检测/分割 |

#### 3.1.4 商业模式与定价

EdgeFirst Studio 采用 **SaaS 订阅制 + Credits 消耗** 双模式：

| 层级 | 定位 | 说明 |
| --- | --- | --- |
| **Individual** | 个人开发者/评估 | 注册即获得 20 USD Credits；适合快速体验 |
| **Team** | 小型团队 | 多用户协作；更多计算额度 |
| **Business** | 中型企业 | 更高并发 Session、优先支持 |
| **Enterprise** | 大型 OEM / 系统集成商 | 定制 SLA、专属部署选项、大规模设备管理 |

- **Credits** 绑定到"组织（Organization）"而非个人账号。
- Studio 同时提供 `edgefirst-client` CLI/SDK，方便自动化消耗管理。

（资料来源：[doc.edgefirst.ai/saas/studio](https://doc.edgefirst.ai/saas/studio/)）

#### 3.1.5 EdgeFirst Studio 产品在 7 Key Features PDF 中的官方总结

根据 [官方 1-Pager PDF](https://30712b24-5b58-49b8-a906-e42ddbb7eacb.filesusr.com/ugd/c11122_8f471536a35541108cc5d5a0c330136c.pdf) 的 7 大功能摘要如下：

1. **Automatic Ground Truth Generation (AGTG)**：AI 驱动的自动标注，减少人工标注的繁琐与错误。
2. **ModelPack for Real-Time Detection & Segmentation at the Edge**：仅需 0.5 TOPS 即可实时运行的检测+分割模型族。
3. **Fusion Model Support**：原生支持雷达-视觉融合训练，应对雨雾粉尘恶劣工况。
4. **Built-in Middleware and Dataset Collection Tools**：MCAP 录制与回放服务，多传感器同步采集与调试。
5. **Foxglove Studio Integration for Visualization**：图形化 3D 空间感知可视化与管线调试。
6. **Advanced Data Augmentation for Vision Models**：丰富的增强策略（亮度/噪声/模糊/几何变换）提升模型鲁棒性。
7. **API-First Design for Automation and Customization**：全流程 API 驱动，方便集成到企业自有管线。

> 官方口号："我们的平台旨在让你快速上手，最高可达 **10× 更快** 地将视觉感知部署到边缘。"

---

### 3.2 EdgeFirst Field-Ready Modules —— 现场级硬件模组

> 官网：https://www.edgefirst.ai/edgefirstmodules

EdgeFirst 提供两款面向越野/工业/机器人场景、可直接部署于户外现场的硬件模组，均具备 IP66/67 防护和 M12 工业连接器，内置 EdgeFirst Perception Engine 感知中间件。

#### 3.2.1 Maivin —— AI 视觉模组

| 维度 | 规格 |
| --- | --- |
| 核心 SoC | NXP **i.MX 8M Plus**（4× Cortex-A53 + 2 TOPS NPU） |
| 推理后端 | 默认 NPU，可切换至 CPU / GPU |
| 传感器 | 1× 工业相机模组（带 ISP），可扩展第二相机 |
| 网络 | 千兆以太网、Wi-Fi，可选 LTE 调制解调器 |
| 供电/连接器 | 5 m M12 公头到 2.1×5.5 mm 桶形适配器；多国插头适配器 |
| 防护等级 | IP66/67 防水防尘 |
| 生态合作 | 参考设计来自 Au-Zone + Toradex + Vision Components，作为 NXP 官方 AI 视觉 Starter Kit（MAVK-SMART-8MPLUS-CAMERA） |
| 典型用途 | 纯视觉的 2D 检测/分割，如智慧城市、停车监控、作业区域监控 |

Maivin 可以看作"工业级 AI 智能相机"，是 EdgeFirst 进入视觉感知的入门硬件。

#### 3.2.2 Raivin —— 视觉+雷达（+LiDAR）融合模组

Raivin 是 Maivin 的上位版本，在同样的 i.MX 8M Plus 计算平台上增加了 **4D 雷达**（与 NXP 合作推广的关键差异化产品），并且可选集成 LiDAR。

| 维度 | 规格 |
| --- | --- |
| 核心 SoC | NXP i.MX 8M Plus + 2 TOPS NPU |
| 雷达模组 | smartmicro **DRVEGRD-169**（4D 汽车级雷达），通过 CAN 总线与主板通信，接收内部供电 |
| LiDAR（可选） | 通过千兆以太网 + PoE 接入 |
| 感知栈 | EdgeFirst **Perception Middleware + RadarExp Fusion Model**（低层雷达数据与视觉早期融合） |
| 结构/防护 | 与 Maivin 同级别，IP66/67 |
| 典型能力 | 低光、雨雾、粉尘等恶劣工况下的目标检测、可行驶区域、3D 目标/距离、速度估计；超越纯光学感知 |
| 典型用途 | 越野 AMR/UGV、矿山设备、越野车辆、港口/物流设备 |

在 NXP 官方博客中，Raivin 被描述为"把雷达感知、视觉处理与边缘 AI 推理融合到单个可量产单元中的 3D 感知系统"，面向复杂作业环境的实时决策。（资料来源：[NXP Smarter World Blog](https://www.nxp.com/company/about-nxp/smarter-world-blog/BL-AUTONOMOUS-MACHINES-NXP-AND-AUZONES-LEAP)）

---

### 3.3 EdgeFirst Datasets —— 生产级数据集与数据管道

> 官网：https://www.edgefirst.ai/edgefirstdatasets

EdgeFirst Datasets 提供面向越野空间感知的"生产级"视觉与传感器融合数据集，配套可扩展的 **EdgeFirst Data Pipelines** 用于客户侧快速定制。特征：

- 全标注，适配 Vision 与 Fusion 两类模型训练；
- 包含相机图像、雷达点云、雷达 Data Cube、LiDAR 数据等多模态格式；
- 与 EdgeFirst Studio 原生集成，可直接用于训练与评估。

---

### 3.4 EdgeFirst Perception Engine / Middleware —— 设备端感知中间件

Perception Engine / Perception Middleware 是部署在 Maivin / Raivin 设备上的边缘运行时，负责：

- 多传感器时间同步与标定；
- 雷达/视觉/LiDAR 数据预处理；
- 调用训练好的 Vision / Fusion 模型做实时推理；
- 输出可行驶区域、目标框、类别、3D 位置、速度等高层语义；
- 将数据以 MCAP 形式上传到 EdgeFirst Studio 形成数据闭环（Data Flywheel）。

这使得"Studio 训练 → 设备部署 → 设备回传数据 → Studio 再训练"形成完整 MLOps 闭环。

---

### 3.5 开源/开放模型资产

Au-Zone 在 [Hugging Face EdgeFirst 组织](https://huggingface.co/EdgeFirst) 下发布多款针对边缘硬件优化的模型，例如：

- **YOLO26-det**：涵盖 Nano 到 XLarge 多尺寸，提供 ONNX FP32 + TFLite INT8，并针对 NXP i.MX 8M Plus / i.MX 93 / i.MX 95 / Ara240、NVIDIA Jetson、Raspberry Pi 5 + Hailo-8/8L 等平台提供预编译版本以启用 NPU 加速。

这表明 EdgeFirst 虽然在商业化平台是闭源 SaaS，但在"模型 + 推理后端"层面愿意与开源社区与多硬件生态共存。

---

## 4. 目标行业与典型应用

官网 [Applications 页面](https://www.edgefirst.ai/applications) 把 EdgeFirst 的能力概括为"视觉定位、地图构建、操作员辅助与自动化、全自主"四大场景。具体行业：

| 行业 | 典型诉求 | EdgeFirst 解决方案 |
| --- | --- | --- |
| **AMR / UGV（移动机器人/无人地面车辆）** | 复杂地形的避障、动态环境导航、实时自主决策；常见于国防、矿业、物流 | 4D 感知 + Raivin 模组 + Fusion 模型，实现粉尘/低光下的目标识别与避障（[amr-ugv 页面](https://www.edgefirst.ai/amr-ugv)） |
| **矿山设备** | 降低安全风险、减少设备损伤、在偏远/极端矿场保持连续作业 | Raivin 模组 + 感知中间件用于碰撞预警、操作员辅助、作业效率优化（[mining-equipment 页面](https://www.edgefirst.ai/mining-equipment)） |
| **土木/建筑机械** | 可行驶表面识别、实时行人检测、避撞、安全与效率监控、全自主作业 | Civil Construction Case Study 显示 EdgeFirst 支持"可行驶面判断、人员检测、避撞、安全监控、生产力跟踪、全自主" |
| **农业机械** | 作物制图、自主导航、自动除草 | EdgeFirst Studio 支撑田间自主导航与地图化 |
| **海洋/环境**（海洋清洁） | 海上漂浮塑料垃圾检测 | 与 **The Ocean Cleanup** 合作的 **ADIS**（Automated Debris Imaging System）智能相机网络，覆盖 UN "Ocean Decade" 挑战 1/8/9 |
| **国防 & 物流** | UGV 在对抗/危险环境下的态势感知 | 4D 感知 + 模组的恶劣环境鲁棒性 |

---

## 5. 差异化优势

结合官网宣称与第三方资料（NXP 博客、Edge AI and Vision Alliance、Venturelab 等），EdgeFirst 的差异化主要体现在：

1. **"越野优先"的定位**：不同于大量聚焦乘用车 ADAS 的感知栈，EdgeFirst 明确面向 off-road、非结构化、粉尘/雨雾等恶劣环境，填补工程机械、矿山、农机、AMR 的中高端感知需求。
2. **端到端打通**：同时提供 **数据集 + MLOps 平台 + 感知中间件 + 硬件模组**，OEM 不需要在多个供应商之间拼接感知栈。
3. **早期雷达-视觉融合（4D Fusion）**：支持把雷达 Range-Doppler 原始数据 Cube 直接送入网络进行早融合，兼顾纯光学不可用工况下的鲁棒性与短开发周期。
4. **与 NXP 深度绑定的硬件路径**：依托 i.MX 8M Plus / i.MX 93 / i.MX 95 与 eIQ 工具链，对已经采用 NXP 的客户几乎"开箱即用"。
5. **20 年嵌入式视觉积累 + 产线级模组**：Maivin/Raivin 以 IP66/67、M12、-宽温工业接口交付，直接进入 OEM 的产品化节奏，而非停留在开发板阶段。
6. **MLOps 工作流加速**：官方宣称在标注、GT 生成、训练对比等关键步骤有 4×–100× 的效率提升，早期客户整体研发周期可缩短 20–50×。

---

## 6. 商业模式小结

- **硬件**：以 Maivin / Raivin 模组销售或参考设计授权的方式进入 OEM 量产链；与 Toradex、Vision Components 等合作方形成供应链。
- **软件**：EdgeFirst Studio 以 SaaS 订阅销售（Individual / Team / Business / Enterprise 四档），并配合按量消耗的 credits。
- **数据**：提供可直接使用的越野行业数据集与定制化 Data Pipelines 服务。
- **生态**：与 NXP、Hugging Face、smartmicro 等深度合作，形成算法-芯片-传感器-工具链的闭环。
- **服务**：基于 20 年嵌入式设计经验，面向关键客户提供工程咨询与定制开发。

---

## 7. 主要风险与需要关注的问题

1. **硬件路线较集中**：当前主要算力绑定 NXP i.MX 家族，若客户要求 NVIDIA Jetson / Qualcomm / Hailo 等其他平台，需要依赖 Studio 跨平台模型编译能力（已在 Hugging Face 侧观察到此类适配）。
2. **公司规模与融资披露有限**：Crunchbase 记录员工 11–50 人，未见大额公开融资，抗击头部玩家（Tier1、自动驾驶公司自建感知团队）的节奏需持续观察。
3. **数据集与行业覆盖广度**：矿山、农业、建筑的真实数据集难以规模化采集；EdgeFirst 在这些垂直领域的标注数据深度将直接决定长期竞争力。
4. **评估性能仍依赖厂家自陈**：20–50× 开发周期提升、20× 标注加速等指标来自官方与早期客户描述，实际 POC 需要客户在自身场景中复现。

---

## 8. EdgeFirst 业务场景中的 OEM 与 ODM 模式分析

### 8.1 OEM 与 ODM 概念全称及基本定义

| 缩写 | 全称 | 基本含义 |
| --- | --- | --- |
| **OEM** | Original Equipment Manufacturer（原始设备制造商） | **客户自行设计产品**，委托制造商按照客户提供的规格/图纸进行生产制造。客户拥有知识产权，以自有品牌销售。 |
| **ODM** | Original Design Manufacturer（原始设计制造商） | **制造商同时负责设计与生产**，提供现成的产品平台/方案，客户可在此基础上贴牌（White Label）或进行有限定制后以自有品牌销售。设计 IP 通常属于 ODM 方。 |

> 简单类比：OEM = "你设计，我生产"；ODM = "我设计 + 我生产，你贴牌卖"。

### 8.2 EdgeFirst 业务中 OEM 与 ODM 的角色映射

在 EdgeFirst 所服务的"越野车辆 / 工程机械 / 移动机器人"这类行业中：

| 角色 | 典型企业举例 | 与 EdgeFirst 的关系 |
| --- | --- | --- |
| **OEM（品牌主机厂）** | Caterpillar（卡特彼勒）、John Deere（约翰迪尔）、Komatsu（小松）、大型矿业机器人公司 | 拥有整机品牌、系统架构与核心需求定义权。使用 EdgeFirst Studio 自研感知模型，或采购 Raivin/Maivin 模组集成到自有产品中。 |
| **ODM（方案/系统供应商）** | Tier-1 零部件商、专业感知方案公司、System Supplier | 提供"交钥匙"感知子系统（含硬件+算法+中间件），OEM 按需选购后贴入自有整机。Au-Zone/EdgeFirst 自身也可扮演 ODM 角色——提供 Raivin 参考设计 + Perception Engine 供客户贴牌使用。 |

EdgeFirst 官网即将其目标客户描述为 **"OEMs and System Suppliers"**，这两类客户对感知技术的采购/开发策略有本质差异。

### 8.3 OEM 与 ODM 的核心差异对比

| 维度 | OEM（主机厂） | ODM（方案供应商 / 系统供应商） |
| --- | --- | --- |
| **产品设计权** | 自行定义系统架构、功能需求、性能指标；拥有最终产品设计 IP | 拥有子系统/模块的设计 IP；按照市场通用需求或特定 OEM 要求做方案 |
| **品牌与渠道** | 以自有品牌面向终端客户（矿业公司、施工方等） | 通常不面向最终用户，以 B2B 方式供货给 OEM |
| **知识产权归属** | 系统级 IP 属于 OEM，可能使用外购模块 | 模块/子系统 IP 属于 ODM，授权或销售给多个 OEM |
| **定制深度** | 深度定制：传感器选型、算法定制、安全等级、整机集成 | 提供标准平台 + 有限定制（外观、接口、参数调优） |
| **开发周期** | 较长（通常 1–3 年），需要完整的 V&V 验证 | 较短，因为核心设计已就绪，客户选购后快速集成 |
| **成本结构** | 前期投入大（NRE）、单位成本取决于量产规模 | 研发成本分摊给多个客户，单模组价格明确 |
| **风险分配** | OEM 承担产品级系统风险与法规认证 | ODM 承担模块级质量风险 |
| **灵活性** | 高度灵活，但需要自有工程团队支撑 | 灵活性有限，受限于 ODM 的平台能力边界 |
| **上市速度** | 较慢 | 较快（利用现成平台） |

### 8.4 对 OEM 厂商的能力要求（使用 EdgeFirst 平台）

OEM 厂商如果要深度使用 EdgeFirst 生态进行自研感知系统开发，需要具备以下关键能力：

| 能力维度 | 具体要求 |
| --- | --- |
| **系统架构设计** | 能定义整机感知需求（覆盖范围、工况、安全等级）、选择传感器组合（Camera / Radar / LiDAR）、规划数据流 |
| **AI/ML 工程团队** | 能使用 EdgeFirst Studio 进行数据标注、模型训练、调参、验证与持续迭代；需要了解目标检测/分割/融合的基本原理 |
| **嵌入式集成能力** | 将 Maivin/Raivin 模组或自定义硬件集成到整机中，包括机械安装、电气接口（M12、CAN、Ethernet）、供电管理 |
| **数据采集与管理** | 能在真实作业现场持续采集高质量多模态数据，并通过 MCAP Pipeline 上传到 Studio 形成数据飞轮 |
| **功能安全（FuSa）** | 对于矿山/建筑/国防等安全关键场景，OEM 需自行负责 ISO 13849 / IEC 61508 / ISO 25119 等认证 |
| **长期维护与 OTA** | 建立模型版本管理、Over-the-Air 更新机制，利用 Studio 的 Lifecycle Management 功能 |
| **项目管理与预算** | NRE 投入较大，需要评估 EdgeFirst Studio Enterprise 订阅 + 硬件采购 + 内部工程团队的总体拥有成本 |

**OEM 使用 EdgeFirst 的典型路径**：
```
评估（Individual Tier + Tourist Workflow）
→ 概念验证（Team Tier + Maivin/Raivin Dev Kit）
→ 定制开发（Business/Enterprise Tier + 自有数据 + Fusion 训练）
→ 量产集成（批量采购模组 + 生产环境部署）
→ 持续迭代（Data Flywheel + 模型更新）
```

### 8.5 对 ODM 厂商（系统供应商）的能力要求

ODM / System Supplier 如果要基于 EdgeFirst 平台向 OEM 提供"交钥匙"感知子系统方案，则需具备：

| 能力维度 | 具体要求 |
| --- | --- |
| **平台化产品设计** | 在 EdgeFirst 硬件（Maivin/Raivin）或自研硬件上建立可复用的感知方案平台，可同时服务多个 OEM |
| **深度算法定制** | 不仅使用 ModelPack，还需要训练领域特定模型（如矿山专用、农业专用），建立私有数据集与行业 Know-how |
| **多客户数据隔离** | 通过 Studio 的 Organization / Project 机制，为不同 OEM 客户隔离数据与模型资产 |
| **系统验证与认证** | 对子系统级性能负责，提供 API 接口文档、性能 Benchmark、MTBF 可靠性数据 |
| **供应链管理** | 管理 NXP SoC、smartmicro 雷达、相机镜头、PCB、外壳等 BOM 采购与质量 |
| **技术支持能力** | 为 OEM 客户提供现场调试、模型调优、集成咨询等 Tier-2 级支持 |
| **规模化生产** | 具备小批量到中大批量（数百至数千台）的硬件生产与测试交付能力 |
| **快速交付** | 利用现成参考设计与预训练模型，将客户从需求到首批交付的周期压缩到数周 |

**ODM 使用 EdgeFirst 的典型路径**：
```
与 Au-Zone 建立合作（获取参考设计 + 硬件 BOM + Studio Enterprise 接入）
→ 建立行业垂直方案（基于 Raivin 训练特定行业 Fusion 模型）
→ 面向 OEM 推广方案（提供"感知模组 + 中间件 + 标定工具"打包方案）
→ 为每个 OEM 定制化适配（接口协议、安装形态、模型微调）
→ 批量交付 + 持续模型更新服务
```

### 8.6 OEM vs ODM 选择 EdgeFirst 的决策矩阵

| 决策因素 | 倾向 OEM 自研 | 倾向采购 ODM 方案 |
| --- | --- | --- |
| 核心竞争力 | 感知是差异化核心 → 自研 | 感知非核心，快速上市更重要 → 外购 |
| 工程团队规模 | 有 AI/嵌入式团队 → 自研 | 团队有限，缺乏 ML 能力 → 外购 |
| 数据资产 | 有大量自有现场数据 → 自研可持续迭代 | 缺乏数据 → 依赖 ODM 的预训练模型 |
| 上市时间压力 | 可接受 12–24 个月 → 自研 | 需 3–6 个月交付 → 外购 ODM 标准品 |
| 定制深度 | 需要深度定制传感器布局与算法 → 自研 | 通用方案可满足需求 → 外购 |
| 量产规模 | 大规模（万台+）→ 自研成本摊薄 | 小批量（数百台）→ 外购更经济 |
| IP 控制 | 要求完全掌控算法 IP → 自研 | 可接受 IP 在供应商处 → ODM |

### 8.7 EdgeFirst / Au-Zone 在 OEM-ODM 生态中的定位

Au-Zone / EdgeFirst 实际上同时服务 OEM 与 ODM 两类客户，并且自身在特定场景下也扮演 ODM 角色：

```
┌─────────────────────────────────────────────────┐
│               Au-Zone / EdgeFirst               │
│  ┌──────────┐  ┌──────────┐  ┌───────────────┐ │
│  │ 对 OEM   │  │ 对 ODM   │  │ 自身为 ODM   │ │
│  │ 卖平台   │  │ 卖平台   │  │ 卖整体方案   │ │
│  │ +硬件    │  │ +参考设计│  │ 给小型 OEM   │ │
│  └──────────┘  └──────────┘  └───────────────┘ │
└─────────────────────────────────────────────────┘
        │                │               │
   大型 OEM         系统供应商       中小型 OEM
 (Caterpillar等)    (Tier-1)       (缺乏AI团队)
   深度自研         平台化交付      直接贴牌使用
```

- **对大型 OEM**：EdgeFirst 提供 Studio (Enterprise) + 硬件模组 + 数据集，OEM 自有团队主导定制开发。
- **对 ODM / 系统供应商**：EdgeFirst 提供参考设计 + 白标硬件 + 平台接入，ODM 在此基础上做行业打包方案。
- **对中小型 OEM**（自身无 AI 团队）：Au-Zone 直接以"方案供应商"（即 ODM 角色）身份提供 Raivin + Perception Engine + 预训练模型的交钥匙方案，客户贴牌集成即可。

这种"平台+方案"的双轨商业模式使 Au-Zone 能覆盖不同规模和能力层次的客户群体。

---

## 9. 参考链接（主要）

- EdgeFirst 产品站：https://www.edgefirst.ai/
- EdgeFirst Studio：https://www.edgefirst.ai/edgefirststudio
- EdgeFirst AI Platform：https://www.edgefirst.ai/edgefirstaiplatform
- Field-Ready Modules：https://www.edgefirst.ai/edgefirstmodules
- Datasets：https://www.edgefirst.ai/edgefirstdatasets
- 7 Key Features：https://www.edgefirst.ai/7keyfeatures
- AMR/UGV 应用：https://www.edgefirst.ai/amr-ugv
- 矿山应用：https://www.edgefirst.ai/mining-equipment
- 建筑案例：https://www.edgefirst.ai/civilconstructioncasestudy
- 海洋清洁案例：https://www.edgefirst.ai/the-ocean-cleanup-casestudy
- 雷达-视觉融合白皮书：https://www.edgefirst.ai/technical-brief-radar-vision-fusion
- 官方文档站：https://doc.edgefirst.ai/
- Au-Zone 公司站：https://www.embeddedml.com/
- Hugging Face 模型组织：https://huggingface.co/EdgeFirst
- NXP 博客（Raivin 3D 融合）：https://www.nxp.com/company/about-nxp/smarter-world-blog/BL-AUTONOMOUS-MACHINES-NXP-AND-AUZONES-LEAP
- Edge AI and Vision Alliance（Studio 通用访问发布）：https://www.edge-ai-vision.com/2025/11/au-zone-technologies-expands-edgefirst-studio-access/
- Edge AI and Vision Alliance（Ocean Cleanup 案例）：https://www.edge-ai-vision.com/2025/01/how-au-zone-technologies-plays-a-key-role-in-the-ocean-cleanup-automated-debris-imaging-system/
- Crunchbase 公司页：https://www.crunchbase.com/organization/au-zone-technologies
- CB Insights 公司页：https://www.cbinsights.com/company/au-zone-technologies/
- NXP i.MX 8M Plus 产品页：https://www.nxp.com/products/processors-and-microcontrollers/arm-processors/i-mx-applications-processors/i-mx-8-applications-processors/maivin-ai-vision-starter-kit:MAVK-SMART-8MPLUS-CAMERA

> 注：本报告的外部引用内容均经过概括与改写，未超过单一来源 30 字的原文，以符合内容合规要求。
