# 卫星网络 related-work 导航经验

更新时间：2026-09-15

## 核心经验

以后凡是讨论 LEO / NTN / Direct-to-Cell / Starlink / 卫星移动性 / 卫星边缘计算 / 卫星传输等科研问题，优先把 **Awesome Satellite Networking** 当作 related-work、artifact、dataset、testbed 的导航入口之一：

- https://liuwei-network.github.io/awesome-satellite-network/

它适合帮助快速发现：

- closest papers；
- 公开代码、testbed、emulator、simulator；
- datasets；
- 同一问题在不同 topic/community 下的已有工作。

## 不能怎么用

这个 curated list 本身不是 primary evidence，也不是 novelty evidence。

不能因为某工作列在这里就直接把网页摘要当作论文事实；更不能因为没有列到某个 idea，就推断“没人做过”。

正确流程：

```text
Awesome Satellite Networking 等导航页
        -> 找到最相关论文 / 项目 / 数据集 / 工具
        -> 回到原论文全文 / 官方代码 / 官方标准 / 专利 / 原始数据文档
        -> 再进入正式科研仓库的 source / evidence / collision ledger
```

## 跨 topic 搜索经验

不要只按 idea 表面关键词搜一个分类。要按真正拥有该 abstraction 的多个 topic 交叉看。

例如：

- LEO mobility / handover / logical path vs physical path：`Mobility + Routing + Transport + Measurement`；必要时再看 `Architecture / Resource Management`。
- D2C：`Direct-to-Cell + Measurement + Mobility + Architecture`。
- Starlink queue / congestion control / bandwidth allocation：`Measurement + Transport + Resource Management`。
- satellite edge / space computing：`Space Computing + Resource Management + Mobility + Architecture`。
- failure / recovery：`Reliability` 加上真正拥有该状态的功能领域。

这样做的目的，是尽早发现“看起来是 mobility，其实 transport/routing 社区已经做过”的 collision，避免后期才发现撞车。

## 和正式科研仓库的关系

这条经验是方法/上下文，不改变任何正式仓库的 Phase、KEEP、L4、candidate、TESTABLE SEED、EXPERIMENT ENTRY 或 kill verdict。

正式科研状态仍必须读取对应仓库最新的 `AGENTS.md / HANDOFF.md / tasks/NEXT_CODEX_TASK.md`。

## 实验资源经验

Awesome Satellite Networking 的 Projects / Tools / Datasets 部分可以用来帮助没有商用 Starlink 终端或运营商内部数据时寻找公开替代实验入口，但每个工具是否真的支持目标机制，必须回到其官方仓库/论文/文档单独核实。
