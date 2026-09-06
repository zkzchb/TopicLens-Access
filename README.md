# deepFlow

> 统一研究入口 · Skill / MCP / Agent 接入层 · 开发中

## 项目定位

deepFlow 是整个研究体系的**统一入口层（Research Entry / Interface Layer）**。

它面向用户、Agent 和外部软件，负责理解研究意图、加载必要配置与 Research Plan、整理和校验参数，并将自然语言或上游请求转换为后端可以稳定执行的标准化研究请求。

deepFlow 可以表现为：

- Skill；
- MCP 入口；
- API；
- Web / Chat 入口；
- 其他 Agent 的研究能力入口。

WorkBuddy 是当前的重要使用场景之一，但 deepFlow 的长期定位不限定于 WorkBuddy。

它主要回答一个问题：

> **用户或上游 Agent 想研究什么？**

图书馆类比：**总服务台 / 窗口服务人员**。

## 与 deepGraph / deepField 的关系

| 项目 | 定位 | 图书馆类比 | 核心职责 |
| --- | --- | --- | --- |
| **deepFlow** | Research Entry / Interface Layer | 总服务台 / 窗口服务 | 接待请求、理解意图、形成标准化研究任务 |
| [deepGraph](https://github.com/zkzchb/deepGraph) | Research Orchestrator | 检索馆员 / 参考咨询馆员 | 拆解任务、编排工具、维护状态、交叉验证与综合 |
| [deepField](https://github.com/zkzchb/deepField) | Industry-native Research Service | 本馆馆藏 + 特色资源 + 专题编研 | 长期积累行业资料、证据和知识状态，并独立提供专业研究服务 |
| External Research Providers | 外部研究能力 | 馆际互借 / 文献传递 | 提供通用 Deep Research、论文检索、网页与专业数据能力 |

完整概念定义以 [deepField Canonical Glossary](https://github.com/zkzchb/deepField/blob/main/docs/glossary.md) 为当前统一术语来源。

## 当前总体链路

```text
用户 / Agent / 软件
        │
        ▼
     deepFlow
  ├─ 理解研究意图
  ├─ 加载配置 / Research Plan
  ├─ 参数补全、规范化与校验
  └─ 生成标准化 Research Request / Command
        │
        ▼
     deepGraph
  Research Orchestrator
        │
        ├─ Public Research Providers
        ├─ Academic / Search / Data Tools
        └─ deepField Provider 接口（当前预留，暂不依赖）
        │
        ▼
任务状态 / 阶段进展 / 最终结果
        │
        ▼
     deepFlow
        │
        ▼
用户 / Agent / 软件
```

## 核心职责

### 1. 理解研究意图

识别用户或上游 Agent 当前希望执行的研究动作，例如：

- 发起一次研究；
- 运行既有 Research Plan；
- 查询已有任务进展；
- 继续或补充既有研究；
- 指定约束、地区、时间范围或输出形式。

### 2. 加载配置与 Research Plan

deepFlow 自身不承担个人研究计划的权威存储。

当前工作流配置、Research Plan 和主题定义仍可由 [MyWorkflow/deepFlow](https://github.com/zkzchb/MyWorkflow/tree/main/deepFlow) 提供。

### 3. 将请求编译为标准化任务

deepFlow 将用户输入、上游 Agent 上下文、工作流配置、Research Plan 和本次临时要求合并为后端可验证的标准请求。

这一层重点处理：

- 任务和计划识别；
- 必要参数补全；
- 默认值与临时覆盖值合并；
- 参数类型和取值校验；
- 输出要求规范化；
- 后端请求构造。

### 4. 调用研究后端

当前主要后端是 deepGraph。

deepFlow 只通过稳定接口与 deepGraph 交互，不复制 deepGraph 的 Graph、路由、工具执行和状态维护逻辑。

未来如有必要，也可以对接其他兼容的研究后端，但不在当前阶段展开。

### 5. 返回状态与结果

deepGraph 返回的任务 ID、状态、阶段进展、最终结果和错误信息，由 deepFlow 转换为适合当前调用方继续处理的响应。

调用方可以是 WorkBuddy，也可以是其他 Agent、MCP Client、API Client 或未来的 UI。

## 无状态原则

deepFlow 保持尽可能无状态：

- 不保存 LangGraph checkpoint；
- 不成为任务历史的事实源；
- 不承担长期研究成果存储；
- 不把研究知识沉淀在入口 Skill / MCP 自身；
- 一次调用所需的信息应来自当前上下文、外部配置和后端返回状态。

这使 deepFlow 可以独立安装、升级和替换。

## 不属于 deepFlow 的职责

以下能力不应进入本项目：

- 复杂研究流程编排；
- checkpoint、队列和运行时持久化；
- 搜索、抓取、RAG、分析模型等研究工具的具体执行；
- deepField 的行业数据、Research Catalog、Evidence 或 Knowledge；
- 长期研究成果与个人知识库；
- 用户秘密和第三方凭据的版本控制存储。

## IO / Contract

deepFlow 与 deepGraph 之间需要建立稳定的 Command / Result Contract。

当前概念上至少需要表达：

输入：

- 用户意图或上游请求；
- Research Plan 标识或临时研究目标；
- 任务上下文和运行参数；
- 时间、地区、来源等约束；
- 期望输出形式。

输出：

- 任务标识；
- 当前状态；
- 阶段进展；
- 最终结果；
- 可恢复错误信息。

具体 Schema 在 deepGraph API 设计阶段冻结。

## 安全边界

仓库中不得保存 API Token、密码、Cookie、SSH 私钥或其他实际凭据。运行时凭据由调用环境、凭据系统或环境变量提供。

## 当前状态

当前已经确定：

- deepFlow 是统一 Research Entry / Interface Layer；
- WorkBuddy 是当前重要入口，但不是长期唯一入口；
- deepFlow 可以演化为 Skill、MCP、API 或其他 Agent 接入方式；
- deepGraph 是有状态研究流程编排与综合后端；
- deepField 是独立的行业原生研究服务，不属于 deepFlow 或 deepGraph 内部；
- 当前 deepGraph 暂不依赖 deepField，只保留 Provider 接口；
- deepFlow 与 deepGraph 通过标准化 Contract 解耦。

下一阶段继续与 deepGraph 对齐 Research Request / Command Contract，并保持入口层本身简单、无状态、可替换。
