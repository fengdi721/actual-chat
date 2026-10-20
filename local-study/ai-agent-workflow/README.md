# 从 ActualChat 学习 AI 应用、Agent 与 Workflow

这是一份基于真实源码的中文入门教程。适合已有 Web / 后端经验、希望理解 AI 应用工程的人；不要求先掌握整个 .NET 生态。

**核心结论：这个仓库最适合学习“把模型能力接入一个可靠的业务系统”。** 已确认的重点是摘要、持久化工作流和 MCP 工具服务；不能仅凭 Bot、Semantic Kernel 或 Agent 包名，就认定产品实现了完整的自主 Agent。

分析基线：`92cc6a04a1227cbb5904a8650a3f88a541232dbe`，日期：2026-10-20。文中的源码相对链接指向当前工作树；日后源码变化时，可使用该提交核对旧行为。本文档是源码分析，没有宣称已运行产品或通过产品测试。

## 从哪里开始

第一次学习只读第 1、2 课，完成第 2 课的“跟踪一份输入”练习。暂时不安装整个应用，也不从几千行的 Backend 文件开头往下读。

| 顺序 | 内容 | 完成后应能回答 |
|---|---|---|
| 1 | [架构地图与阅读方法](lessons/01-architecture.md) | 身份、业务规则、模型调用、状态同步分别由谁负责？ |
| 2 | [一条聊天摘要调用链](lessons/02-summary-pipeline.md) | 输入怎样变成请求，输出怎样进入产品？ |
| 3 | [持久化工作流](lessons/03-durable-workflow.md) | 重启、失败和重复执行分别意味着什么？ |
| 4 | [MCP 与 Agent 的分工](lessons/04-mcp-and-agent.md) | 工具服务和自主决策循环有什么区别？ |
| 5 | [工程实践与测试证据](lessons/05-quality-and-tests.md) | 哪些行为有确定性测试，哪些仍需要模型评估？ |
| 6 | [AI 编程协作流程](lessons/06-ai-development-workflow.md) | 如何把约定落实为可验证的修改？ |
| 7 | [渐进练习设计](lessons/07-exercises.md) | 怎样从摘要逐步走到有工具的 Agent？ |
| 8 | [理解检查参考答案](lessons/08-answer-notes.md) | 能否用自己的话解释关键边界？ |

每课阅读建议 30–60 分钟，具体速度自行调整。这是顺序建议，不代表已经诊断你的能力或安排了固定课程。

## 三个需要分开的概念

| 概念 | 本仓库的对应入口 | 决定下一步的是谁 |
|---|---|---|
| LLM 功能 | `ConversationSummarizer` | 应用代码决定何时调用，模型生成内容 |
| Workflow | `LiveConversationSummaryFlow` / `FlowBackend` | 代码中的条件与持久化状态 |
| Agent | 可由外部 Agent 调用本仓库 MCP | 外部模型 / Agent 运行时决定工具使用；本次未确认产品内有通用自主循环 |

这个区分采用 Anthropic 的架构定义：预设代码控制流程的是 Workflow，模型动态控制步骤和工具使用的是 Agent。它是理解设计的框架，而非宣称该仓库使用 Anthropic 的 Agent 产品。[官方说明](https://www.anthropic.com/engineering/building-effective-agents)

## 计划、证据与同步

- [当前计划与归档提示词](PLAN.md)
- [完成状态](PROGRESS.md)
- [计划修订历史](records/2026-10-20-plan-revisions.md)
- [执行、验证与提交记录](records/2026-10-20-execution.md)
- [官方资料及适用范围](records/2026-10-20-sources.md)

所有后续学习成果继续放在本目录，唯一推送目标是个人 fork 的 `origin`。产品代码与练习实现尚未修改。
