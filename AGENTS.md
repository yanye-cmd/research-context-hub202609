# Context Hub 长期规则

## 1. Context ≠ Scientific State

本仓库内容只能作为上下文。不得根据这里的旧 idea 或旧聊天：

- 创建正式 candidate；
- 修改 KEEP / L4；
- 修改两个正式科研项目的 Phase；
- 推翻正式仓库 verdict；
- 重新激活已 KILL 的方向。

涉及正式科研状态时，必须读取对应正式科研仓库的最新 `HANDOFF.md`、`AGENTS.md`、`tasks/NEXT_CODEX_TASK.md` 等权威文件；它们优先于本仓库的一切记录。

## 2. Source discipline

从聊天整理记录时，尽量保留：日期、主题、原始讨论背景、当时结论、尚未解决的问题，以及后续是否被正式科研仓库更新或推翻。

不能把 AI 推测伪装成当时已经发生的事实。未验证内容、回忆性总结和正式状态必须清楚区分。

## 3. Conversation compression

不要机械复制完整 ChatGPT 对话。一段有长期价值的聊天应压缩为以下结构：

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

只有原话本身非常重要时，才保留少量原文。

## 4. Historical state

被否决的方向可以保存，但必须醒目标记：

`HISTORICAL / KILLED / NOT CURRENT SCIENTIFIC STATE`

不得让未来读者把旧 idea 误认为当前研究方向；若正式科研仓库随后更新或推翻其结论，应在这里补充链接或状态说明。

## 5. Beginner learning

`learning/` 面向一位计算机网络、5G 核心网和分布式系统基础仍在补全的研究生。解释时应：

- 先讲术语本意，再讲机制流程；
- 标清组件间是从属、调用、承载、状态同步、消息传递还是先后关系；
- 图示避免裸箭头，说明箭头含义；
- 写明前提与隐藏步骤；
- 最后才联系科研问题。

## 6. Incremental sync

每次整理一个独立且有长期价值的聊天主题时：新增或更新对应 Markdown、更新 `INDEX.md`、提交 commit，并用清楚的 commit message 说明加入了什么。

不要为保存一句话制造大量碎文件。这里是 context hub，不是正式科研状态机。
