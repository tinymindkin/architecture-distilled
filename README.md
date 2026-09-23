<div align="center">

# Architecture Distilled

### 把优秀开源项目的架构经验，变成 AI 可以执行的设计判断。

**10 个真实项目 · 1 个架构 Skill · 每条经验都有出处**

[开始使用](#快速开始) · [案例地图](#从哪些项目中提炼) · [完整示例](examples/webhook-service.md) · [阅读 Skill](architecture-distilled/SKILL.md) · [来源与许可](ATTRIBUTION.md)

</div>

---

你让 AI 设计一个系统，它很快画出一张图：API Gateway、Redis、Kafka、微服务……

真正需要回答的问题，往往还在图外：**为什么要这样拆？数据在哪一步才算写成功？队列满了怎么办？三个人的团队，维护得过来吗？**

Architecture Distilled 把 [The Architecture of Open Source Applications（AOSA）](https://aosabook.org/en/) 中的设计经验，提炼成一套可复用的 Agent Skill。它引导 AI 从约束出发，借鉴真实系统的机制，解释付出的代价，最后给出验证方法。

> **学到一个架构的价值，在于知道什么时候用它、什么时候放弃它。**

## 它会改变哪些设计动作

| 设计环节 | Skill 要求回答的问题 | 你得到的结果 |
| --- | --- | --- |
| 理解需求 | 负载、延迟、一致性、团队预算分别是什么？ | 明确的约束与假设 |
| 选择先例 | 哪个项目遇到过类似压力？相似性到哪里结束？ | 有出处的机制借鉴 |
| 比较方案 | 最小可行方案是什么？另一种方案贵在哪里？ | 可讨论的取舍表 |
| 定义架构 | 谁拥有状态？何时持久化？失败后怎么恢复？ | 组件、数据结构与状态机 |
| 验证决策 | 哪个实验能推翻方案？何时值得升级？ | 验收标准与演进触发条件 |

它适合新系统设计、已有架构评审、组件边界讨论、存储与并发决策，以及 ADR 撰写。核心指令保持简短，案例按需读取；没有运行服务、API Key 或模型训练步骤。

## 快速开始

通过 [Skills CLI](https://github.com/vercel-labs/skills) 安装，交互选择你的 Agent：

```bash
# 安装此仓库中的架构 Skill
npx skills add tinymindkin/architecture-distilled --skill architecture-distilled
```

也可以先列出可安装内容：

```bash
# 仅发现 Skill，不安装
npx skills add tinymindkin/architecture-distilled --list
```

或直接克隆仓库，将 `architecture-distilled/` 整个目录复制到你的 Agent 支持的 skills 目录。需要同时保留 `references/`；只复制 `SKILL.md` 会丢失案例。

在支持显式 Skill 调用的环境中：

```text
$architecture-distilled

帮我设计一个 webhook 投递服务。
团队 3 人，平均 100 req/s，峰值 1,000 req/s，已有 PostgreSQL。
需要至少一次投递和 30 天审计。比较数据库队列与独立消息系统，
写清数据结构、状态转移、失败恢复和升级条件。
```

其他环境可以直接说：**“使用 architecture-distilled 分析下面的架构。”** Skill 采用 [Agent Skills 格式](https://agentskills.io/specification)；实际发现方式和调用语法由宿主 Agent 决定。核心文件为英文，指令要求按用户语言输出。

## 从哪些项目中提炼

首版精读选取 AOSA 两卷中的 **10 个项目章节**。表中“迁移问题”是本项目对历史机制的应用提问，不代表原作者对你项目的建议。

| 项目 | 提炼的设计视角 | 迁移到你的系统时，先问 |
| --- | --- | --- |
| **nginx** | 事件驱动与 worker 分工 | 哪些操作会阻塞处理其他连接？ |
| **Twisted** | Reactor、协议边界与异步结果 | 等待、计算、取消和错误分别由谁负责？ |
| **ZeroMQ** | 消息传递、线程隔离与批处理 | 队列容量和慢消费者策略在哪里？ |
| **LLVM** | 共享中间表示与可组合编译阶段 | 哪些变化值得通过稳定表示隔离？ |
| **Git** | 对象、引用与历史图 | 不可变内容和可移动引用能否分离？ |
| **Graphite** | 接收、存储与查询的职责拆分 | 高峰写入压力如何被缓冲、计量和消化？ |
| **Berkeley DB** | 嵌入式存储、缓存与事务 | 必须引入一个独立数据库服务吗？ |
| **HDFS** | 元数据与大块数据分工、故障恢复 | 你的负载真的像大文件顺序访问吗？ |
| **SQLAlchemy** | Session 生命周期与写入依赖排序 | 哪些变更必须一起完成，谁负责回滚？ |
| **Bash** | 语言阶段与兼容性压力 | 哪些语义让“简单重写”变得昂贵？ |

👉 [阅读案例卡片与原文链接](architecture-distilled/references/casebook.md)

每张卡片包含 **原文机制、代价、可迁移经验、适用条件、反例**。书中记录的是历史架构；选型时仍要核验当下版本的行为。

## 看一次完整输出

[Webhook 服务设计示例](examples/webhook-service.md)从“三人团队 + 已有 PostgreSQL”出发，比较数据库队列和独立 broker，明确：

- 返回 `202` 前，接收记录、事件和待投递任务必须在同一事务中提交。
- HTTP 已成功、worker 尚未记账时崩溃，会发生重复投递；接收方仍需幂等。
- 数据库租约保护任务状态，无法撤销已经发出的 HTTP 请求。
- 100 events/s 持续 30 天就是 **2.592 亿条事件**；1 KiB 平均 payload 的原始数据约 **265 GB**，还没算索引、日志与副本。
- 数据库队列的可行性必须通过组合负载验证；文档中的吞吐目标不是实测成绩。

```mermaid
flowchart LR
    A[业务约束] --> B[相关开源先例]
    B --> C[候选方案与代价]
    C --> D[所有权与状态转移]
    D --> E[失败验证与升级条件]
    E -. 新证据 .-> A
```

这份示例可以当作团队评审的起点。它同时展示了一个重要边界：**AOSA 提供设计机制；你仍要为自己的工作负载提供证据。**

## 三种使用方式

**做设计**

```text
使用 architecture-distilled 设计多租户文件处理服务。
先列出会改变方案的未知条件，再比较两种可行设计。
重点说明数据所有权、背压和故障恢复。
```

**做评审**

```text
使用 architecture-distilled 评审这个仓库的任务执行链路。
沿实际代码追踪请求，找出无界队列、阻塞点和状态歧义。
按风险给出最小修复，并说明相关 AOSA 先例。
```

**写决策记录**

```text
使用 architecture-distilled 为“是否引入独立消息系统”写 ADR。
已有数据库队列；请比较操作成本、失败边界、迁移与回滚，
给出可以量化的重审条件。
```

## 仓库结构

```text
architecture-distilled/
├── README.md
├── LICENSE
├── ATTRIBUTION.md
├── architecture-distilled/
│   ├── SKILL.md
│   ├── agents/openai.yaml
│   └── references/
│       ├── casebook.md
│       └── decision-template.md
└── examples/
    ├── webhook-service.md
    └── VALIDATION.md
```

## 质量标准与边界

已检查 Skill 格式、公开仓库的安装发现与文档链接，并用两个独立场景做了行为演练；详见 [验证记录](examples/VALIDATION.md)。这不是效果基准或生产压测。

一个值得合并的改动，应当让 Agent **更容易作出可解释的选择**。新增案例请附上原文、作者、具体机制、代价和一个不适用场景；欢迎用真实请求证明现有指令哪里会误导。

Skill 不会自动保证架构正确，也没有“提升百分之多少”的效果承诺。它不替代当前产品文档、基准测试和实际代码检查。无关紧要的小改动也不需要强行产出完整设计报告。

提交 Issue 或 PR 时，推荐附上：输入场景 → 当前输出的问题 → 期望的决策差异 → 来源或验证方法。请勿提交私有业务数据。

## English overview

**Architecture Distilled** is an agent skill for software architecture decisions, grounded in ten chapters of *The Architecture of Open Source Applications*. It turns historical mechanisms into questions about constraints, ownership, failure behavior, operational costs, and validation.

The skill and casebook are in English; responses follow the user's language. Start with the smallest feasible design, compare a credible alternative, explain where a precedent applies, and define evidence that would change the decision. The repository includes a worked webhook design and an optional ADR template.

## 致谢与许可

感谢 AOSA 的作者、编辑 Amy Brown 与 Greg Wilson，以及贡献者们公开分享设计过程。

AOSA 书籍内容以 [CC BY 3.0](https://creativecommons.org/licenses/by/3.0/) 发布。本仓库的原创指令、提炼文字与示例同样采用 **CC BY 3.0**，允许依照该许可使用、修改与再分发。章节作者及逐条出处见 [ATTRIBUTION.md](ATTRIBUTION.md)；案例是重新组织的提炼与迁移分析，不是原书复制，也不代表 AOSA 或原项目背书。

**如果它帮助你提出了一个更好的架构问题，欢迎 Star；如果它给出了坏建议，更欢迎带着反例来提 Issue。**
