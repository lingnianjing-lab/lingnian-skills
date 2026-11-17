---
name: hive-swarm
description: >-
  Orchestrates multi-agent swarm collaboration using a Queen-Worker-Auditor
  hive architecture (蜂巢/蜂群协同模式). Combines dependency-wave planning, file
  ownership isolation, single-channel hardware/MCP mutex queues, and mandatory
  visual/metric quality gates via native subagent tools (define_subagent,
  invoke_subagent, send_message, manage_subagents). Use when the user mentions
  "蜂巢", "蜂群", "启动蜂巢", "hive", "swarm", "多Agent协作", or when tackling
  multi-module app development, full-stack features, or multi-stage media/data
  pipelines that benefit from parallel subagents under strict reviewer supervision.
---

# Hive-Swarm: 蜂巢多智能体协同编排体系 (Queen-Worker-Auditor)

本技能将多智能体协同拆解为 **「蜂王统筹 + 依赖波次并发 + 单通道资源互斥锁 + 零信任督察门禁」** 四大核心机制，专为具备原生子智能体调度接口（`define_subagent`、`invoke_subagent`、`send_message`、`manage_subagents`）的 AI 编程代理设计：

- **依赖波次与文件归属隔离（Dependency Waves & File Ownership）**：显式构建有向无环图（**DAG / Parallel Waves**）与文件归属白名单（**File Ownership Matrix**），从根源杜绝多工蜂并发覆写同一文件。
- **蜂王-工蜂-督察三级拓扑（Queen-Worker-Auditor Topology）**：主会话担任 **蜂王（Queen Coordinator）**，按职责派生只读侦察蜂、并发建造工蜂与独立督察蜂，砍掉冗余的串行文书传递。
- **单通道资源互斥锁（Single-Channel Resource Mutex）**：将无状态纯代码/资产生成任务设为并发扇出（`[PARALLEL]`），将单实例硬件、远程连接或全局锁文件设为串行互斥队列（`[MUTEX:<resource>]`）。
- **零信任验收与有状态原位返工（Zero-Trust Gate & Stateful Rework）**：不轻信工蜂口头回报的“已完成”，必须通过真实测试、构建校验或视觉接触表抽检；发现缺陷时通过 `send_message` 原路打回原工蜂会话修复，保留完整上下文。

---

## 一、角色拓扑与原生工具映射 (Hive Topology)

| 蜂巢层级 | 角色代号 | 对应原生工具 / Subagent | 核心职责与权限边界 |
| :--- | :--- | :--- | :--- |
| **统筹层** | **蜂王 (Queen Coordinator)** | 当前主 Agent 会话 | 拆解波次计划、维护接口契约、掌管单通道互斥锁（Mutex）、执行最终集成合流与质量签字。 |
| **侦察层** | **侦察蜂 (Scout Bee)** | `invoke_subagent(TypeName="research")` | 只读并发扫描代码库、接口定义、外部文档与依赖关系，严禁修改任何文件。 |
| **建造层** | **工蜂 (Worker Bee)** | `invoke_subagent(TypeName="self")` 或 `define_subagent` 定制工种 | 在指定的独占文件边界或隔离工作区（`Workspace="share"` / `"branch"`）内并行编写代码、生成脚本或合成资产。 |
| **督察层** | **督察蜂 (Auditor Bee)** | `code-reviewer` / `test-engineer` / `security-auditor` / `web-performance-auditor` 或蜂王直审 | 对工蜂产出执行五维代码审查、自动化冒烟测试或视觉/音视频硬指标抽检（如接触表 `read_file`、峰值 dBFS、`ffprobe`）。 |

---

## 二、四步标准作业流 (Four-Stage Hive SOP)

### Stage 1: 侦察与波次规划 (Scout & Wave Planning)

在派出任何具有写权限的工蜂前，蜂王必须先建立 **蜂巢波次作战表（Hive Dispatch Table）**：

1. **按需并发侦察**：若任务涉及多个未知模块，可先并行调用 2–3 个只读 `research` 子代理快速摸清目录结构与接口签名。
2. **划分并发波次（Parallel Waves）与互斥标签（Mutex Tags）**：
   - **`[PARALLEL]`（纯并发任务）**：互不依赖且写入**完全不相交文件路径**的任务（例如：主进程模块 vs. 渲染层界面 vs. 端到端测试脚本；或图表渲染脚本 vs. 音频合成脚本 vs. 字幕时间轴计算）。
   - **`[MUTEX:<resource>]`（单通道独占任务）**：凡涉及唯一状态连接或不可并发抢占的资源，必须打上互斥标签，例如：
     - `[MUTEX:gpu-mcp]`：单连接远程 GPU / Notebook 运行时（受单次执行超时或 WebSocket 帧大小上限约束）。
     - `[MUTEX:remote-deploy]`：远程服务器服务重启、反向代理重载或线上配置变更。
     - `[MUTEX:packager-lock]`：桌面端打包器（如 Electron Builder）、数据库 Schema 迁移或全局锁文件（如 `package-lock.json`）。
   - **调度铁律**：`[PARALLEL]` 任务交由多个工蜂在同一波次同时后台执行；`[MUTEX]` 资源由工蜂先产出脚本/配置，再交由蜂王统一排队串行触发，严禁多个工蜂同时直连同一个单通道硬件或写同一个锁文件。

### Stage 2: 契约派单与并发扇出 (Contract Fan-Out)

蜂王在同一轮工具调用中并行调用多个 `invoke_subagent` 启动当前波次的所有工蜂：

1. **工作区模式选择（Workspace Mode）**：
   - 当各工蜂修改**完全不相交的文件路径**时：优先使用默认 `Workspace: "inherit"`（零克隆开销，各工蜂直接落盘各自负责的文件）。
   - 当各工蜂需要运行可能互相干扰的编译构建或实验性分支时：使用 `Workspace: "share"`（共享底层版本库但拥有独立工作树）或 `Workspace: "branch"`。
2. **模型参数规范**：除非用户明确指定特定模型，`Model` 参数一律保持默认 `"inherit"`。
3. **工蜂派单四要素（详见 [dispatch-templates.md](./references/dispatch-templates.md)）**：
   - **目标与已知上下文（Context & Goal）**：直接将蜂王已掌握的架构、接口签名、行号定位喂给工蜂，避免工蜂重复扫描代码库浪费 Token 与时间。
   - **独占文件白名单（Allowed Files）**：明确列出该工蜂**唯一允许创建或修改**的文件路径，严禁越界修改其他工蜂的文件。
   - **自测要求（Self-Verification）**：工蜂交活前必须跑通的最小验证命令或语法检查。
   - **返回格式（Return Payload）**：要求工蜂返回修改文件清单、实测输出摘要及遗留边界风险。
4. **响应式唤醒（Reactive Wakeup）**：
   - 派发完当前波次的子代理后，**严禁编写死循环轮询（Polling）或调用 `sleep`**。直接完成其他非冲突准备工作或结束当前轮次，等待系统在工蜂完成时自动唤醒蜂王。

### Stage 3: 独立督察与原路打回返工 (Audit Gate & Targeted Rework)

当所有工蜂完成当前波次并唤醒蜂王后，进入强制审核门禁：

1. **零信任实测（Zero-Trust Verification）**：
   - **应用与工程代码交付**：运行自动化冒烟测试、单元测试或类型/构建检查，必要时唤醒 `code-reviewer` / `test-engineer` 做交叉审查。
   - **多媒体、UI 与图表交付**：生成轻量级压缩预览接触表（Contact Sheet，如 `< 120 KB` JPEG 拼图），由蜂王亲自调用 `read_file` 逐帧目视检查（排查文字贴边裁切、色彩对比度不足、物理结构幻觉、底部字幕遮挡），并结合 `ffprobe` 或音频分贝检测核对硬性指标。
2. **有状态原位返工（Stateful Rework via `send_message`）**：
   - 若发现某工蜂产出存在缺陷，**切勿新建子代理从头再来**，而是直接调用 `send_message(Recipient=<该工蜂的 conversationId>, Message=<具体缺陷坐标与修复指令>)`，利用该工蜂已有的上下文记忆精准返工，直至复检全绿。
3. **会话清理**：确认某个分支工蜂任务彻底结束或不再需要时，使用 `manage_subagents` 回收异常残留会话。

### Stage 4: 集成合流与结构化复盘 (Integration & Evidence Report)

所有并发波次通过督察门禁后，蜂王执行最终合流（如全量集成测试、桌面端打包或音视频混流封装），并输出结构化交付报告：
1. **结论先行**：开门见山汇报各波次完成状态与最终产物路径。
2. **数据支撑**：附上实测耗时、产物体积、测试通过率、音频峰值或性能指标。
3. **事实与推测分离**：明确区分“实测查到的结果（Observed）”与“推断结论（Inferred）”。
