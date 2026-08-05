# deepFlow

> 面向 WorkBuddy 的深度研究入口 Skill · 开发中

deepFlow 是安装在 WorkBuddy 上的深度研究 Skill。它位于用户交互层与 deepGraph 研究后端之间，负责理解用户的研究意图、加载工作流配置与研究计划、整理和校验参数，并将自然语言请求翻译为 deepGraph API 可以稳定执行的结构化命令。

deepFlow **不是完整的深度研究系统，也不承担 LangGraph 编排本身**。它的核心价值是提供一个稳定、无状态、可复用的 IO 与协议适配层。

## 项目定位

deepFlow 负责回答一个问题：

> 用户在 WorkBuddy 中表达的研究意图，应该如何被转换为后端可以明确执行的研究任务？

完整工作流由三个相互独立的项目共同组成：

| 项目 | 定位 | 主要职责 |
| --- | --- | --- |
| [MyWorkflow/deepFlow](https://github.com/zkzchb/MyWorkflow/tree/main/deepFlow) | 工作流配置与研究计划中心 | 保存关键配置、研究计划、主题定义和工作流规范 |
| **deepFlow** | WorkBuddy Skill | 意图理解、配置加载、参数结构化、API 调用与结果回传 |
| [deepGraph](https://github.com/zkzchb/deepGraph) | LangGraph 后端 | 研究流程编排、任务状态、工具路由与实际执行 |

MyWorkflow 中的 deepFlow 工作流介绍页：

```text
https://my-work-flow.pages.dev/deepFlow/
```

## 总体链路

```text
用户
  ↓
WorkBuddy
  ↓
deepFlow Skill
  ├─ 理解研究意图
  ├─ 识别任务 / 研究计划
  ├─ 加载 MyWorkflow/deepFlow 配置
  ├─ 参数补全、规范化与校验
  └─ 生成标准化 Command
  ↓
deepGraph API
  ↓
LangGraph 编排与多途径工具
  ↓
任务状态 / 阶段进展 / 最终结果
  ↓
deepFlow Skill
  ↓
WorkBuddy
```

用户始终以 WorkBuddy 作为主要交互入口，不需要直接操作 deepGraph、LangGraph 或具体研究工具。

## 核心职责

### 1. 理解意图

识别用户当前希望执行的研究动作，例如发起研究、运行既有研究计划、查询进展或继续既有任务。具体操作名与 Command Schema 将在 API 契约确定后冻结。

### 2. 加载工作流配置

deepFlow 自身不保存个人配置和研究项目。运行所需的工作流配置、研究计划和主题定义由 [MyWorkflow/deepFlow](https://github.com/zkzchb/MyWorkflow/tree/main/deepFlow) 提供。

未来计划将系统配置与具体研究计划分离：

```text
MyWorkflow/deepFlow/
├── v1/
│   └── manifest.json
└── plans/
    ├── index.json
    └── <plan-id>/
        └── plan.md
```

实际目录和字段结构将在 v1 数据模型确定后正式冻结。

### 3. 将意图编译为结构化命令

deepFlow 将用户输入、WorkBuddy 提供的上下文、工作流 Manifest、Research Plan 以及本次运行的临时要求合并为后端可验证的标准请求。

这一层重点处理：

- 任务和计划识别；
- 必要参数补全；
- 默认值与本次临时覆盖值合并；
- 参数类型和取值校验；
- 输出要求规范化；
- API 请求构造。

### 4. 调用 deepGraph API

deepFlow 只通过稳定 API 与研究后端交互，不复制 LangGraph 的节点、边、路由或工具执行逻辑。

### 5. 将状态与结果返回 WorkBuddy

后端返回的任务 ID、运行状态、阶段进展、最终结果或错误信息，由 deepFlow 转换为适合 WorkBuddy 继续处理和展示的响应。

## 无状态原则

deepFlow Skill 保持无状态：

- 不保存具体研究项目；
- 不保存 Research Plan；
- 不保存 LangGraph checkpoint；
- 不保存任务历史或最终研究成果；
- 不依赖上一次 Skill 调用留下的本地状态。

一次调用所需的信息应来自当前 WorkBuddy 上下文、MyWorkflow/deepFlow 配置以及 deepGraph API 返回的任务标识和状态。

这使 deepFlow 可以独立安装、升级和替换，而不会把个人研究数据绑定到 Skill 本身。

## 不属于 deepFlow 的职责

以下能力明确不放入本项目：

- 具体研究主题和长期研究计划；
- 用户个人工作流配置；
- LangGraph 图与研究步骤编排；
- checkpoint、队列和运行时持久化；
- 搜索、抓取、RAG、分析模型等工具的具体执行逻辑；
- 研究成果的长期存储。

这些能力分别属于 MyWorkflow/deepFlow、deepGraph 或独立的数据与成果存储层。

## IO 契约

deepFlow 与 deepGraph 之间需要建立稳定的 Command / Result Contract。当前仅确定概念边界，具体字段尚未冻结。

输入侧至少需要能够表达：

- 用户意图与本次指令；
- Research Plan 标识或临时研究目标；
- 任务上下文与运行参数；
- 本次执行的覆盖条件；
- 期望的输出形式。

输出侧至少需要能够表达：

- 任务标识；
- 当前状态；
- 阶段进展；
- 最终结果；
- 可恢复的错误信息。

正式 Schema 将在 MyWorkflow/deepFlow 数据模型与 deepGraph API 设计完成后共同确定。

## 安全边界

仓库中不得保存 API Token、密码、Cookie、SSH 私钥或其他实际凭据。运行时凭据由 WorkBuddy、本地凭据系统或环境变量提供；配置文件只能保存不构成秘密的信息或凭据引用。

## 当前状态

项目处于初始化与接口设计阶段。目前已经确定：

- WorkBuddy 是主要人机交互入口；
- deepFlow 是无状态 Skill；
- MyWorkflow/deepFlow 是工作流配置与研究计划来源；
- deepGraph 是有状态 LangGraph 研究后端；
- deepFlow 与 deepGraph 通过标准 API / Command Contract 解耦。

下一阶段将先确定 MyWorkflow/deepFlow 的 Manifest 与 Research Plan 数据模型，再据此冻结 deepFlow Skill 的 IO 和 deepGraph API 契约。
