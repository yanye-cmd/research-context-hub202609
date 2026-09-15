# Context Hub 长期规则

本仓库是跨对话的**长记忆层**，不是正式科研状态机。详细分层见 `meta/research-memory-and-agent-workflow.md`。

## 1. Context ≠ Scientific State

不得根据这里的旧 idea、旧聊天或 ChatGPT Memory：
- 创建或升级正式 candidate；
- 修改 KEEP / L4 / TESTABLE SEED / EXPERIMENT ENTRY；
- 修改任何正式科研项目的 Phase；
- 推翻正式仓库 verdict；
- 重新激活已 KILL 的方向。

涉及当前科研状态时，直接读取对应正式科研仓库最新 `AGENTS.md`、`HANDOFF.md`、`tasks/NEXT_CODEX_TASK.md`，再按当前任务需要读取其指向的材料。不要为了恢复状态预读整个仓库。

当前正式科研仓库注册表见 `README.md` / `INDEX.md`。

## 2. Progressive disclosure

先按 `INDEX.md` 找到与当前问题最相关的主题文件，只读取本次问题需要的内容。不要机械加载全部 learning、sidequest、conversation 或 meta 文档。

若问题只是当前正式科研进度，优先去正式仓库，不必先读取本 Hub 的历史上下文。

## 3. Conversation compression

不要机械复制完整 ChatGPT 对话。长期价值内容优先压缩为：

```text
Background
Question
What we learned
Important reasoning
Decision / current conclusion
Open questions
Relation to formal research repositories
Keywords
```

重要的是保留“原先判断 -> 改变判断的证据 -> 新结论 -> 可复用教训”，而不是保存每句对话。

优先更新已有主题文件；只有真正独立、以后需要单独检索的主题才新增文件。

## 4. Source / history discipline

- 不能把 AI 推测伪装成事实；未验证内容、回忆性总结和正式状态必须清楚区分。
- 被否决方向可以保存，但必须标记 `HISTORICAL / KILLED / NOT CURRENT SCIENTIFIC STATE`。
- 若正式科研仓库随后更新或推翻这里的判断，保留历史并补充新状态链接，不反向修改正式结论。
- LEO / NTN / D2C / Starlink related-work 导航按需查看 `meta/satellite-related-work-navigation.md`；curated list 只能导航到原论文、标准、代码、专利、数据或正式 evidence 记录。

## 5. Beginner learning

`learning/` 面向仍在补计算机网络、5G 核心网和分布式系统基础的研究生。解释时：
- 先讲术语本意，再讲完整机制流程；
- 箭头标明从属、调用、承载、状态同步、消息传递或先后关系；
- 写明关键前提、隐藏步骤和边界；
- 最后再联系科研问题。

## 6. Public-repository privacy

本仓库是 public。不要写入密码、token、账号标识、私人联系方式、私人医疗信息、机密沟通、受限未公开材料或其他敏感个人数据。

## 7. Incremental sync

每次整理一个独立且有长期价值的主题时，更新对应 Markdown 和必要的 `INDEX.md`，提交清楚的 commit。不要为保存一句话制造碎文件。
