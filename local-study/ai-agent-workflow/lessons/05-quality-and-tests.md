# 第 5 课：用证据判断工程实践

## 这一课的目标

把“代码存在”“测试覆盖”“实际通过”“模型质量合格”分开。以下分析来自源码阅读，产品测试未在本轮执行。

## 1. 值得复用的设计与适用边界

| 设计 | 已确认的证据 | 不应扩大成的结论 |
|---|---|---|
| 把模型能力封装成接口 | `IConversationSummarizer`、真实实现与 Stub 注册 | 有接口就一定容易模拟所有内部依赖 |
| 业务代码控制触发条件 | Live Flow 的数量、成熟度、新消息和时间门槛 | 每次定时恢复都会调用模型 |
| 输出在应用端处理 | `SanitizeSummary` 的作者允许集合和长度限制 | 内容一定真实，或已经防住全部注入 |
| 配额统一包装 | Redis rate limiter 包装 `IChatCompletionService` | 等同于精确计费与模型上下文长度控制 |
| Flow 状态与 inbox 统一提交 | `FlowRuntime.Commit`、`FlowBackend.OnStore` | 外部网络副作用恰好发生一次 |
| MCP 复用业务身份和权限 | Session accessor → 带 Session 的服务 | 工具描述或工具列表本身具有授权效力 |

上表的源码分别见 [第 2 课](02-summary-pipeline.md)、[第 3 课](03-durable-workflow.md)、[第 4 课](04-mcp-and-agent.md)。

## 2. 已有测试证据地图

| 场景 | 已读取的测试入口 | 实际验证范围 |
|---|---|---|
| 作者提及越界与数量上限 | [ConversationSummarizationTest](../../../tests/Chat.IntegrationTests/ConversationSummarizationTest.cs) / `ShouldLimitMentionsToKnownAuthors` | 应用输出过滤 |
| 生成结果长度 | 同文件 / `ShouldCapConversationSummaryOutputLength` | 三个字段总长度受上限约束 |
| 正常格式化与摘要接口 | 同文件 / `ShouldFormatAndSummarizeEntries` | 输入进入格式化结果、返回非空摘要；不验证事实完整性 |
| 删除消息后的摘要更新 | 同文件 / `ShouldResummarizeConversationOnEntryRemoval` 等 | 业务删除与后续摘要行为 |
| 语言检测次数 | [ConversationSummarizerLanguageTest](../../../tests/Chat.UnitTests/ConversationSummarizerLanguageTest.cs) | 用伪造语言服务检查补查范围，非真实 LLM 质量 |
| inbox 并发及失败恢复 | [FlowInboxTest](../../../tests/Core.Server.IntegrationTests/Flows/FlowInboxTest.cs) | 状态、消息差异及恢复语义 |
| 旧版本写入 | [FlowStoreTest](../../../tests/Core.Server.IntegrationTests/Flows/FlowStoreTest.cs) | 旧状态不覆盖新状态 |
| 节流 Flow 失败计数 | [FailingThrottledUpdateFlowTest](../../../tests/Core.Server.IntegrationTests/Flows/FailingThrottledUpdateFlowTest.cs) | 恢复、达到失败次数上限、成功后清零；不代表 Live Flow 使用同一策略 |
| 身份认证 | [AuthTest](../../../tests/Mcp.IntegrationTests/AuthTest.cs) | 缺少/无效凭据、访客 API key 等拒绝路径 |
| 多用户资源操作 | [MultiUserAccessTest](../../../tests/Mcp.IntegrationTests/MultiUserAccessTest.cs) | 他人私聊发消息、管理私聊等被拒绝 |
| 搜索正常路径 | [McpSearchToolsTest](../../../tests/Mcp.IntegrationTests/McpSearchToolsTest.cs) | 搜到消息、限定聊天；标注 `LocalFact("needs OpenSearch")` |
| 工具集合契约 | [McpToolSchemaTest](../../../tests/Mcp.IntegrationTests/McpToolSchemaTest.cs) | 工具名集合符合预期，不等于每个参数 schema 全部正确 |

阅读集成测试时先看 Fixture 与依赖注入。使用 Stub 的集成路径可以证明业务串联，不能据此证明真实模型可用。不能根据测试文件名推断真实 provider 已被调用。

## 3. 建议补充的测试设计

以下是本次阅读范围内尚未建立覆盖证据的场景，不是全仓库缺失断言，也不是已实现测试。

| 输入 / 故障 | 应观察的结果 | 验证层 |
|---|---|---|
| 空消息集合 | 定义明确的空结果或参数错误，避免偶发 `First()` 异常 | 确定性组件测试 |
| 非 JSON、错误字段类型、缺字段 | 结果状态明确；不能把降级文案默认为业务成功 | 伪造模型响应 |
| 429 带 / 不带建议延后 | 调用层与 Flow 对重试时间的处理可预测 | 模拟响应 + 可控时钟 |
| 等待配额时取消 | 及时结束，不继续发模型请求 | 包装器测试 |
| 模型调用完成、Flow 提交前崩溃 | 明确记录重复调用次数与业务结果去重策略 | 故障注入集成测试 |
| 长中文无空格输入 | 输入预算有可测边界，不能只按空白词数估计 | 多语言边界测试 |
| 工具返回包含恶意指令 | 不得升级权限或调用未允许的写工具 | Agent 运行时测试 |
| 跨用户全局检索 | 不返回不可见消息片段；索引滞后也要验证 | 检索权限集成测试 |

不要在没有明确修改目标的情况下，把这些建议全部变成一次大重构。

## 4. 模型评估与普通测试的区别

对固定输入，业务规则可以精确断言：“最多调用 N 次”“未知作者提及被删除”。
对摘要文本，逐字相等通常不是合适的唯一标准；需要可审查的事实约束。

建议的入门评估集：12 份虚构对话，每类 2 份，分别覆盖明确决策、带条件的决策、互相矛盾的信息、多语言、信息不足、含诱导指令的聊天内容。每份记录：

- 必须保留的事实及原消息编号。
- 不允许添加的结论。
- 是否保留否定、条件和不确定性。
- 模型标识、Prompt 版本、输入版本、耗时、实际 usage。

确定性测试检查格式、权限和调用次数；人工核对事实质量。模型评审可以辅助，但不能成为唯一裁判。当前没有执行这套评估集，也没有声称仓库已有这套流程。

## 5. 与官方资料核对

Semantic Kernel 的官方说明讨论不可信输入和提示注入防护。模板输入编码解决的是特定注入通道；本仓库还把对话直接组织成用户消息，因此仍需要应用侧权限、输出验证及攻击样例，而不能把模板框架当作通用安全证明。[Microsoft 官方说明](https://learn.microsoft.com/en-us/semantic-kernel/concepts/prompts/prompt-injection-attacks?pivots=programming-language-csharp)

MCP 授权规范讨论 HTTP 传输上的授权，业务资源权限仍要由应用实现。此处引用固定的 2025-06-18 版本，用于概念核对，不声称它是最新版本或已经完成本项目协议合规审计。[MCP 授权规范](https://modelcontextprotocol.io/specification/2025-06-18/basic/authorization)

## 6. 运行测试前的检查

当前任务只改文档，没有执行以下产品测试命令。将来准备运行时，先核对 SDK、配置和测试依赖：

```bash
dotnet test tests/Chat.UnitTests/Chat.UnitTests.csproj --filter FullyQualifiedName~ConversationSummarizerLanguageTest
```

这可以作为最小起点；命令仍可能触发 restore / build。只有环境准备获得授权后才执行。
完整构建使用根目录规定的 `ActualChat.CI.slnf`。若修改受规则约束的 TypeScript，按根规则运行 `npm run build:Verify`。

测试等待统一通过 [TestWait](../../../docs/testing/waiting.md)：可跟踪失效的计算值用 `When`，非响应式的副作用观测用 `WhenPolled`。增加任意 sleep 不是解决集成测试抖动的可靠方法。

## 理解检查

1. 哪一个已有测试最适合先读？为什么它不需要评价模型“聪明不聪明”？
2. “JSON 解析成功”和“摘要没有捏造事实”应该由哪两类检查验证？
3. 怎样证明故障恢复没有重复业务副作用，而不只是最终结果看起来正确？
