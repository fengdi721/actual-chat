# 计划修订历史

## 初始方案：产品开发为主，结合 AI 编程协作

学习次序：架构地图 → 聊天摘要 → 持久化 Flow → MCP 与 Agent → 工程质量 → AI 编程协作。
交付包含中文教程、源码位置、调用链、设计取舍、理解检查和渐进练习。
初始方案未要求本地归档或 Git 同步。

## 版本 1：增加 Markdown 归档与独立分支

增加 `local-study/ai-agent-workflow/`、`PLAN.md`、`PROGRESS.md`、教程与记录目录。
所有实质交付均归档；实现代码与测试同时记录变更及验证证据。
分支采用 `study/ai-agent-workflow`，保留已有工作区内容。

当时指定目标为 `upstream`；除此之外，版本 1 与当前计划的目标、顺序、复用原则、验收和执行边界一致。
旧同步条款为：

> 用户指定的同步目标是 upstream。推送前核对其真实 URL 和写入权限，按学习里程碑提交，仅推送本任务分支到 upstream 的同名远程分支。不得自动改用 origin、推送默认分支、强制推送或合并。

此条款已废止，不可用于后续操作。

## 版本 2：同步目标更正为个人 fork

实施过程中已核对：

- `origin`：`git@github.com:fengdi721/actual-chat.git`。
- `upstream`：`git@github.com:Actual-Chat/actual-chat.git`。
- GitHub API 对上游仓库返回 `permissions.push = false`；未向上游执行推送。

当前生效规则：仅向 `origin` 的学习分支提交同步；禁止向 `upstream` 推送。
完整生效文本见 [PLAN.md](../PLAN.md)。

## 中断恢复

此前 issue 搜索请求被中断，没有创建、关联、分配 issue 或修改项目看板。
分支创建成功后发生会话中断；恢复时确认当前分支为 `study/ai-agent-workflow`。
本记录保留计划的实质变化，不包含私人消息或内部推理。
