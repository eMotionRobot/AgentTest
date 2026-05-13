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

### 3.1 EdgeFirst Studio™ —— 企业级 MLOps 平台

> 官网：https://www.edgefirst.ai/edgefirststudio
> 文档：https://doc.edgefirst.ai/saas/studio/

EdgeFirst Studio（原名 **Deep View Enterprise**）是整个 EdgeFirst AI 平台的"控制中心"，定位为**面向边缘空间感知的企业级 MLOps 平台**，提供从数据采集、标注、模型训练、对比评估到部署与生命周期管理的全流程多用户 SaaS 工作台。

#### 关键能力

1. **多模态数据支持**：原生支持 RGB 相机、LiDAR、RADAR、ToF 等传感器；数据格式基于 MCAP 容器。
2. **多种标注方式**：2D 框、分割掩码、3D 框、雷达点云标注；内置 AI 辅助自动标注。
3. **多种模型类型**：
   - **Vision 模型**：面向 2D 检测、分割的常规视觉模型（YOLO26 等）。
   - **Fusion 模型**：`EdgeFirst Fusion` 早期融合模型，将雷达 Range-Doppler 原始数据 Cube 与相机图像一起送入网络；支持中期融合（用检测类别给雷达点云"上色"）。
4. **数据集 & 会话管理**：以"项目 (Project) → 数据集 (Dataset) → 训练/验证会话 (Session)"的层级管理资产，方便多人协作与版本对比。
5. **一键部署**：训练完成后可直接把模型部署回 Maivin / Raivin 等设备，并由 `edgefirst-client` CLI/API 驱动。
6. **面向开发者的 7 大功能**（来自官方 [7 Key Features](https://www.edgefirst.ai/7keyfeatures)）：
   - 自动化数据标注与 Ground Truth 生成
   - 原生传感器融合（雷达+视觉，面向雨雾粉尘工况）
   - 丰富的视觉增强（亮度、噪声、模糊、几何变换）
   - 与可视化工具的紧密集成（支持 MCAP 回放）
   - 全流程 API 驱动（CLI + Python SDK）
   - 多用户协作与访问控制
   - 训练/会话对比功能

#### 性能宣称

根据官网与 Edge AI and Vision Alliance 2025-11 的发布稿，EdgeFirst Studio 在典型工作流中可达：

- 标注速度 **20×** 提升
- Ground Truth 生成 **100×** 提升
- 训练与会话对比 **4×** 提升
- 早期用户整体研发周期提升 **20–50×**

#### 商业模式

EdgeFirst Studio 采用 SaaS 订阅制，官网文档明确列出 4 个付费档位：**Individual、Team、Business、Enterprise**。新用户注册后默认获得 20 USD 平台额度（credits）用于试用；订阅与额度绑定到"组织（Organization）"而非个人账号。（资料来源：[doc.edgefirst.ai saas/studio](https://doc.edgefirst.ai/saas/studio/)）

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

## 8. 参考链接（主要）

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
