# Context Hub 索引

本索引按主题定位已整理材料。新增内容时优先更新已有主题文件；只有真正独立、以后需要单独检索的主题才新增文件。

## Formal Research Repositories

| 仓库 | 用途 | 状态权威性 |
| --- | --- | --- |
| [`yanye-cmd/ccf-b-research202609`](https://github.com/yanye-cmd/ccf-b-research202609) | 成熟地面网络机制 × D2C / LEO / NTN 主线 | 该主线正式状态唯一权威来源 |
| [`yanye-cmd/ccf-b-emerging-architecture202609`](https://github.com/yanye-cmd/ccf-b-emerging-architecture202609) | Emerging / 6G Architecture 主线 | 该主线正式状态唯一权威来源 |

## Conversations

未来记录跨对话压缩整理内容。参见 `conversations/`。

## Research Sidequests

- 临时 idea：`research-sidequests/ideas/`
  - [`2026-09-14-closed-loop-control-x-leo-matrix.md`](research-sidequests/ideas/2026-09-14-closed-loop-control-x-leo-matrix.md)：无人机/无人车/机器人闭环控制业务机制 × LEO/NTN 属性矩阵；记录从“泛 QoS/低时延猜题”修正为“业务机制左轴 × LEO 特性逐格 collision”的方法，以及多 flow 闭环事务、C2 mode transition、redundant C2 三个待审 seed family。
  - [`2026-09-14-gateway-multipath-d2c-context.md`](research-sidequests/ideas/2026-09-14-gateway-multipath-d2c-context.md)：从 Gateway/feeder 动态多卫星容量池，扩展到 MPTCP/MPQUIC、INFOCOM 2026 PMPS、普通 D2C 终端约束，以及 Rel-20 TN→NTN connected-mode mobility × SCS 频谱共存的持续讨论；含当前 continuation state，不代表正式科研状态。
  - [`2026-09-14-starlink-demand-allocation-control-loop-context.md`](research-sidequests/ideas/2026-09-14-starlink-demand-allocation-control-loop-context.md)：Starlink demand-driven allocation × AQM × transport 的长期讨论上下文；记录从“Starlink 专用 400 ms/AQM 冲突”修正为“slow demand-driven allocation × transport feedback”候选母题，SATPIPE/Confucius 方法启发、无 Starlink UT 时的 interactive-emulator / second-system 证据路线、DVB-RCS/D2C 泛化边界、用户层代价与当前 continuation state；不代表正式科研状态。
- 被 KILL 方向：`research-sidequests/killed-directions/`
- 论文笔记：`research-sidequests/paper-notes/`
- 观察记录：`research-sidequests/observations/`

所有条目仅为历史或背景；不可覆盖正式科研仓库的状态。

## Learning

- 5G 核心网：`learning/5g-core/`
- 网络：`learning/networking/`
- 卫星：`learning/satellite/`
- 分布式系统：`learning/distributed-systems/`

## People & Communication

- 师兄交流：`people-and-communication/senior-brother/`
- 导师交流：`people-and-communication/advisor/`
- 组会与汇报准备：`people-and-communication/meeting-notes/`

## Group Projects

- MicroVM / 半实物网络平台：`group-projects/microvm-platform/`

## Meta

`meta/` 用于科研方法、ChatGPT 工作流、explanation style 与跨对话恢复规则；`inbox/` 用作尚待整理的材料入口。
