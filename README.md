# research-context-hub202609

这是一个跨 ChatGPT 对话的长期上下文知识库，用来沉淀学习笔记、支线讨论、已否决方向、人员交流和临时研究想法。它的价值在于让新会话可以迅速理解背景与历史，而不必把每段旧聊天当作当前结论。

## 正式科研仓库

以下两个仓库是正式科研状态的唯一权威来源：

1. [`yanye-cmd/ccf-b-research202609`](https://github.com/yanye-cmd/ccf-b-research202609)：成熟地面网络机制 × D2C / LEO / NTN 主线。
2. [`yanye-cmd/ccf-b-emerging-architecture202609`](https://github.com/yanye-cmd/ccf-b-emerging-architecture202609)：Emerging / 6G Architecture 主线。

各正式仓库中的 `HANDOFF.md`、`AGENTS.md`、`tasks/NEXT_CODEX_TASK.md` 及其最新记录，定义该主线的 KEEP / L4 / candidate / Phase / verdict。本仓库只能保存背景与历史，绝不替代或更新这些状态。

## 适合放进来的内容

- 经压缩整理、具有长期价值的跨对话讨论；
- 暂不构成正式 candidate 的支线 idea、观察和论文阅读笔记；
- 已被否决方向的历史记录与否决理由；
- 5G 核心网、网络、卫星、分布式系统的学习材料；
- 师兄、导师、组会与协作项目的交流记录；
- 科研方法、ChatGPT 工作流和跨对话恢复规则。

## 不适合放进来的内容

- 两条正式科研主线的现行状态、正式 candidate 或最终 verdict；
- 未经核实而补写的历史、猜测或 AI 推断；
- 为保存一句话而产生的大量碎片文件；
- 全量机械复制的聊天记录（除非少量原话本身不可替代）。

## 新 ChatGPT 会话的读取方式

先读本仓库的 `AGENTS.md`、`README.md`、`INDEX.md`，再按当前问题读取索引中的相关 context 文件。若问题涉及正式科研状态，必须额外读取对应正式科研仓库最新的 `HANDOFF.md` 等权威状态文件。

可直接使用下面的 bootstrap prompt：

> 读取 `yanye-cmd/research-context-hub202609` 最新 `AGENTS.md`、`README.md`、`INDEX.md`，然后根据我的当前问题读取相关 context 文件。这个仓库只提供跨对话背景；如果问题涉及正式科研状态，必须再读取对应正式科研仓库最新 HANDOFF，不得从 context hub 推断 KEEP/L4/candidate/Phase。

## 冲突处理

当本仓库中的旧聊天、旧 idea 或旧方向与任一正式科研仓库冲突时，永远以对应正式科研仓库的最新状态为准。Context hub 中的记录应保留其历史属性，并注明已被更新或推翻，而不是反向修改正式结论。
