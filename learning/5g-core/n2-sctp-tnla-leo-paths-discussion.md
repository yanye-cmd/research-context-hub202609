# N2 / SCTP / multiple TNLA × 再生式 LEO：分层入门与讨论修正

> **Context / beginner explainer / ongoing conversation (2026-09-16).** 仅保存这段对话的长期学习成果和历史判断，**不是正式科研状态、实测报告或新 candidate**。本主题与 [`research-sidequests/ideas/2026-09-14-gateway-multipath-d2c-context.md`](../../research-sidequests/ideas/2026-09-14-gateway-multipath-d2c-context.md) 的 MPTCP/MPQUIC、Gateway capacity-pool 讨论相关；此文件只单独讲 **N2 控制面与 SCTP/TNLA**，不复制那份多主题笔记。

## Background

讨论从 MPTCP/MPQUIC 的 logical subflow 与实际 LEO route 可能漂移开始，逐步追问：相同思想能否迁移到 D2C/NTN 核心网；N2 的 SCTP/multiple TNLA 与 N3/N9 的 GTP-U 有何不同；为何 SCTP 有多个地址却无法保证中途物理路径独立；在星上 gNB 场景下是否真的需要多路径以及是否真正影响 NGAP。用户希望从零理解，并为面向师兄的朴素、证据充分的 PPT 做准备，要求箭头标明关系类型、解释隐藏步骤及分层边界。

## Question

1. 地面 N2 为何支持 SCTP multihoming、multiple TNL associations（TNLAs），怎样建立、选择、使用、恢复？
2. SCTP primary path、TNLA、SCTP association、IP route、卫星/ISL/Gateway 物理路径是什么关系？AMF 给 TNLA 的 weight 如何决定、到底用于什么？
3. full-gNB-onboard regenerative NTN 中，一颗卫星到同一地面 AMF 究竟有多少条**独立可用**路径？不同 SCTP 地址/TNLA 是否真的映射不同路径？底层拓扑和路由变化是否给 N2 造成额外可观测后果？
4. SkyOctopus 的“多锚点”与 N2 多 TNLA 是否是同一问题？有哪些相邻已有工作及可公开复现限制？

## What we learned

### 1. 先固定系统位置，避免所有东西都叫 path

- N2：gNB ↔ AMF 的**控制面参考点/接口**；NGAP (NG Application Protocol) 是具体控制消息协议；NGAP / SCTP / IP 是其传输栈。UE 的 NAS 经 gNB 转交，不是 UE 与 AMF 自己建立 SCTP association。
- TNLA (Transport Network Layer Association)：N2 管理的一条传输关联，在本讨论的 NG-C/SCTP 场景中由一个 SCTP association 承载；**多个 TNLA**通常指多个 associations，不等于一个 association 内的多个 IP 地址。
- SCTP association：两个 SCTP endpoints 握手建立的可靠传输协议状态；一个 association 可有多个 SCTP streams；stream 不是物理路径。
- SCTP multihoming：**同一 association** 有多个可用的传输地址。Primary path 是该 association 默认发送 DATA 所用的对端目的传输地址（还可指定本端源地址）；**不等于** TNLA-A，也不是中途经过的卫星清单。多个 associations 可各有自己的 primary。
- IP route / forwarding path：底层路由与转发表决定下一跳；physical path 才涉及具体卫星、ISL（Inter-Satellite Link，星间链路）、feeder link（馈电链路）、NTN Gateway 及地面线路。
- 防歧义关系链：`UE 的 NGAP 消息 --[UE-TNLA 绑定/选择]--> TNLA-A --[由协议关联实现]--> SCTP association 1 --[默认目的地址选择]--> primary 地址 IP-A --[IP 逐跳转发]--> 卫星/ISL/Gateway/地面 AMF`。前四个箭头是**选择/承载关系**，最后一个才是网络转发；这不是五个顺序经过的服务器。

### 2. 地面机制的初衷和完整状态边界

- SCTP 起源与电信信令有关；multihoming 主要为了网络地址/接口/路径出现故障时保持 association 的可用性，原生**不以多路径同时聚合带宽为目标**。默认普通 DATA 偏向 primary，但备用地址可用于探测、重传与 failover；不是必须等到 primary 正式判 INACTIVE 才能向备用地址发任何包。
- Multiple TNLA 提供同一 gNB↔AMF 的多关联管理、新 UE 初始 NGAP 信令负载分配、关联新增/移除及 UE-TNLA 重绑定。不同 UE 可以通过不同 associations 同时传信令；不能拿“单 association 备用地址不聚合带宽”否定多 TNLA 的负载分配。
- 状态边界依次是：程序/接口准备；**IP reachability**；SCTP INIT→INIT ACK→COOKIE ECHO→COOKIE ACK；NGAP NG SETUP REQUEST/RESPONSE；按配置新增其他 TNLAs；UE initial NGAP message 选关联并形成绑定；正常 UE procedure。`IP reachable ≠ SCTP established ≠ NG Setup complete ≠ UE binding established`。不同部署的新增 TNLA 时序可以不同。
- AMF 给出各 TNLA 的 **TNL Address Weight Factor**，gNB 对初始 N2 消息考虑关联 availability 和权重；weight 主要用于**初始信令分流**，不是 RTT、带宽、卫星路由代价或 primary-path 评分。标准规定字段/语义，**没有统一规定 AMF 如何计算具体 weight 或星座最优评分公式**。避免把“运营商必然按某个容量/RTT 公式设置 weight”写成事实。
- CM-CONNECTED UE 的 NGAP UE-TNLA binding 通常保持，AMF 能通过现有标准程序更新/释放绑定；“绑定以后永远不能换”是错的。需要分开：选择哪个 AMF 的权重、同一 AMF 下选哪个 TNLA 的权重。

### 3. multihoming 原生局限 ≠ 已证实的 LEO/N2 故障

- 不同 IP 地址**不保证**不同物理路径；单张网卡也可以有多个 IP，多张网卡也可能共享网关或线路。部分重合时，可绕开独有故障；共同 gateway/ISL 坏了则多地址也可能一起失效。真实故障独立性须靠底层物理冗余、路由规划、共享故障风险识别等。若 IP 网络能重路由，**同一个 primary 地址不变也可能已经走上新的物理路**。
- 原生 primary/backup 不默认聚合带宽；故障检测和切换有非零代价，但 RFC 7829 SCTP-PF 已专门改善慢 failover。SCTP 并非只有 alive/dead 两个观测值：还维护 RTT/RTO、拥塞及重传状态；它只是**不保证默认持续选择最低 RTT 的地址**，上层能显式修改 primary。
- 多地址本身不能恢复整个 AMF 实例和其 UE 上下文；那属于不同的核心网功能/状态恢复层。
- “ACTIVE 但另一地址更快”、共享瓶颈、路径 RT T/稳定性变化、跨馈电链路切换时的控制过程延迟：均须核实实际候选路线、已有下层路由/TE、SCTP 和 AMF 重绑定后是否仍存在**可观测 NGAP/UE 后果**。不能因 LEO 拓扑变化而直接判定 N2 失效。

### 4. 再生式 LEO 的部署条件与路由形成

- **Transparent payload** 下完整 gNB 仍可在地面，不能把 service-link 动态直接归因于 N2 的卫星 underlay；**full onboard regenerative gNB** 才使 gNB↔地面 AMF 的 N2 可能经过 feeder / NTN Gateway，按具体部署也可有 ISL。
- 不把商业 Starlink Direct to Cell 已公开的 LTE eNodeB modem 直接等同于 Rel-19 full NR gNB / N2 商用部署。
- 3GPP 支持多网关/ISL 和再生式 NTN 的相关管理，但**没有规定一个卫星 gNB 到 AMF 固定有两条物理独立路线**，也没有统一规定卫星逐跳选路算法。实际可能单 feeder 无绕行、双出口、多个候选 route 但同一时刻只用一条。`多个 TNLA ≠ 多馈电链路 ≠ 多条独立物理路线`。
- 一种公开路由工作流：轨道/位置 → [可见性及物理链路判定] → 拓扑图与代价 → [Dijkstra/OSPF/其他控制策略] → 转发表/下一跳部署 → 实际 IP 逐跳传输 → 轨道运动后更新。LeoEM、StarryNet 可作为公开路由实现的学习入口，**不是 Starlink 商业内部路由的证据**。
- 网关切换可伴随或不伴随 gNB IP 变化；IP 变化可能要更新 SCTP/TNLA 或依赖地址保持方案；服务 AMF 不一定变。要区分链路可用、IP 可达、SCTP association、NG Setup、UE-TNLA binding、NG Removal/AMF 替换等不同边界。若旧/新 feeder 之间无重叠且无其他出口，多 TNLA 不能创造物理连通性。
- 路由 churn 可能改变同一目的 IP 的逐跳路线，但这**不等于** SCTP primary 或 TNLA 已换身份、不等于 association 必断、更不等于 NGAP 出错。反之若两条关联实际共用唯一 feeder，重选 TNLA 也绕不开共同故障。

### 5. N2、N3/N9、SkyOctopus 不应混淆

- N3（gNB↔UPF）和 N9（UPF↔UPF）是 **GTP-U / UDP / IP** 用户面；N2 则 NGAP / SCTP / IP。不能因 UPF 搬运大量用户流量便推导其 N9 必须使用 MPSCTP。用户面可通过 GTP-U 隧道冗余和底层 IP/ECMP/SR 等实现路径工程；UE↔UPF 还另有 ATSSS 的 MPTCP/MPQUIC 特定功能，但不是普通 N9 替换。
- SkyOctopus（INFOCOM 2025）的 multi-anchor 指**用户面 PSA-UPF/出口锚点**，主要针对固定锚点造成绕路与锚点移动影响会话连续性的取舍，**不是 N2 多 TNLA 或 SCTP primary**。5G 已有 UL CL / Branching Point / 多 PSA，不能把“首次发明多锚点”当其 novelty。

## Important reasoning — Earlier view → Later correction → Current understanding

| Earlier view（对话早期简化） | Later correction（后来核对） | Current understanding（以后复用） |
| --- | --- | --- |
| “两个 SCTP 地址就是两张网卡、两条独立的路” | 地址数量、网卡数量、物理故障域是三个不同量；RFC 9260 明确不保证路径分离。 | 独立性须画真实路由及共享资源，不能从 IP/TNLA 计数推导。 |
| “primary 不坏就不换，所以一直走原来卫星路线” | SCTP primary 是默认目的地址；同一地址的底层 IP 路由可以无须更改 primary 就发生变化。 | 先检查实际 IP/卫星路由有没有自行恢复，再问 SCTP 是否造成额外代价。 |
| “SCTP 只判断 alive/dead” | SCTP 也估计 RTT/RTO、维护拥塞/错误状态，允许显式 primary 更改；RFC 7829 改进 quick failover。 | 默认并不保证实时最优，但不能写成只知道是否存活或不会切。 |
| “multiple TNLA 只为两条备用物理路” | 它还做初始信令分配、关联管理及 UE 绑定；可只共享一条 feeder。 | 多 TNLA 的控制面用途≠物理冗余保证。 |
| “LEO 运动 → route change → N2 故障；需要 RTT+稳定性新评分” | StarryNet/LeoEM/StableRoute 等底层系统会计算/更新路由；TR 38.821、Rel-19 与 Pizzi 已研究 feeder continuity；通用 SCTP quality-aware selection 也已有文献。 | 必须先有公开部署/测量证据、定位到 NGAP 外部后果、排除现有标准/路由/传输解决方案，L4 前不设计新评分。 |
| “Starlink 已按 Rel-19 NR full-gNB/N2 部署” | 官方 D2C 公开说的是 onboard LTE eNodeB modem；标准的 NR full-gNB regenerative 架构是另一限定对象。 | 不拿商业 Starlink 未公开的核心网内部机制充当 5G N2 事实。 |

## Decision / current conclusion

**本对话当前认识：** 再生式 NTN 的 N2 连通性是实际需求，SCTP multihoming / multiple TNLA 提供可用的多地址/多关联工程能力；卫星动态不自动使这些机制失效。是否需要物理独立双路径、动态改 primary 或 UE-TNLA 绑定，必须先确证有多条可分别利用的真实路径，然后观察底层路由、SCTP、N2 共同作用后是否留下显著、可复现的 NGAP/UE 后果。没有证据证明每个星上 gNB 到 AMF 都有两条独立 path，也没有证据证明“仍可达但非最低 RTT”的绑定造成新型 N2 事故。此结论是讨论层总结，不是新科研 verdict。

## Open questions

1. 找到**具体且公开可信**的 full-gNB-onboard regenerative 部署/论文，核实星上 gNB ↔ AMF 当前有几条独立可用的 feeder/ISL/Gateway/地面 route；不同 SCTP 源/目的地址、associations、TNLAs 是否可实际分别使用这些路线？
2. 若已有卫星 IP 路由/TE 在正常运动下完成重路由，同一 SCTP association 的 RTT、loss、failover 与 NGAP procedure 表现如何；有没有真正由 SCTP/TNLA 管理造成的*额外*可测后果？
3. 是否存在已公开的 N2 实测或真实部署 trace，能明确定位到 degraded-but-alive UE-TNLA binding，而非一般 feeder 断联、普通路由收敛或旧 SCTP 问题？
4. 公开 Open5GS/free5GC + RAN/NTN 仿真是否具有完整的 multi-TNLA、动态关联管理与 UE rebinding 路径？不把“能模拟星座路由”与“可以直接复现 3GPP 完整 N2”混为一谈。
5. PPT 应循序介绍地面成熟机制→再生式架构变化→三层 path 身份→已有解决方案及缺失证据，避免先假设缺陷或宣称达到投稿级别。

## Relation to formal research repositories

- **当前科研状态只以** [`ccf-b-research202609`](https://github.com/yanye-cmd/ccf-b-research202609) 最新 `AGENTS.md`、`HANDOFF.md`、`tasks/NEXT_CODEX_TASK.md` **为准**。本次同步时核查：正式审查已将泛化的 N2/TNLA shared-fate story 作为成熟故障域工程排除；degraded-but-alive UE-TNLA stickiness 为 `REVISE / DO NOT SELECT`（缺公开 LEO 主证据、现成完整可复现实验路径）；慢 SCTP failover 已被 RFC 7829 和成熟文献覆盖。此次 Hub 更新不创建候选、不改变正式科研状态。若后续正式仓库更新，以其新记录为准。
- Emerging Architecture 和 D2C/Starlink 的正式现状须分别读取各自仓库，不从本文件推断。
- 关联旧 context：[`Gateway / Multipath / PMPS / D2C`](../../research-sidequests/ideas/2026-09-14-gateway-multipath-d2c-context.md)。

## Conversation Continuation State

- **Current discussion state:** 在本对话中已逐步完成地面 N2 入门、星上 gNB 架构、三层路径映射、SCTP 五类原生局限、LEO 工程反例和 related-work 初筛；尚未建立真实 N2 现象。
- **Last meaningful conclusion:** LEO route churn 不自动推翻 SCTP/multiple-TNLA 的可用性/信令分配初衷；可能由下层路由消化，或实际断联时无任何备用物理路可用。不能把“默认不选实时最低 RTT”当成 N2 协议故障。
- **Unresolved questions:** 真实多路径部署与地址/TNLA 映射、现有路由/TE 如何持续管理独立性、是否有直接 NGAP 外部后果、开源是否支持完整动态 N2 试验。
- **Likely next discussion step:** 用户继续当前会话时，优先沿**公开部署与现象证据**逐一核实上述问题；如果先做 PPT，则依据本文件分层机制和标准/论文原文讲清“已确认／假设／尚不确定”，不主动推进正式科研 Phase 或设计解法。

## Source navigation (non-evidentiary bibliography)

**标准及协议原文：**
- [RFC 9260 — SCTP](https://www.rfc-editor.org/rfc/rfc9260.html)（§1.3 path/primary；§5.1.2 association startup；§6.4 multihoming；§6.3 RTT/RTO）。
- [RFC 3286 — SCTP introduction](https://www.rfc-editor.org/rfc/rfc3286.html)（§4 多归属目标及不同物理路径的部署条件）。
- [RFC 7829 — SCTP-PF quick failover](https://www.rfc-editor.org/rfc/rfc7829.html)（已有慢 failover collision）。
- [3GPP TS 23.501](https://www.etsi.org/deliver/etsi_ts/123500_123599/123501/19.08.00_60/ts_123501v190800p.pdf)（§5.21.1 多 TNLA、权重、UE binding；§5.4.11.9 再生式 NTN 的 N2 管理）。
- [3GPP TS 23.502](https://www.etsi.org/deliver/etsi_ts/123500_123599/123502/19.08.00_60/ts_123502v190800p.pdf)（§4.2.7 关联新增/移除/UE binding 程序）。
- [3GPP TS 38.412](https://www.etsi.org/deliver/etsi_ts/138400_138499/138412/19.00.00_60/ts_138412v190000p.pdf)（§4、§7 NG-C SCTP/IP）。
- [3GPP TR 38.821](https://www.etsi.org/deliver/etsi_tr/138800_138899/138821/16.00.00_60/tr_138821v160000p.pdf)（§5.2 regenerative 架构；§8.7 feeder switchover；研究报告非最终强制方案）。

**相邻论文/公开工具（阅读导航，非“完整审稿已证”）：**
- [Pizzi et al., *Tackling Satellite Mobility in LEO-Based Non-Terrestrial Networks*, IEEE VTM 2024](https://vtmagazine.ieee.org/2024/12/16/tackling-satellite-mobility-in-leo-based-non-terrestrial-networks-principles-and-enhancements-of-feeder-link-switch/)：feeder switch / virtual visibility。
- [StableRoute, INFOCOM 2025](https://ieeexplore.ieee.org/document/11044485/)：LEO underlay route churn，非直接 N2 证据。
- [SkyOctopus, INFOCOM 2025](https://doi.org/10.1109/INFOCOM55648.2025.11044594)：用户面多 PSA/绕路，非 N2 多 TNLA。
- [LeoEM](https://github.com/XuyangCaoUCSD/LeoEM)、[StarryNet](https://github.com/SpaceNetLab/StarryNet)：开放卫星路由实现入口，非商用 Starlink 内部实现证明。

## Keywords

N2, NG-C, NGAP, SCTP, SCTP multihoming, SCTP association, primary path, SCTP stream, multiple TNLA, TNL address weight factor, NGAP UE-TNLA binding, regenerative NTN, onboard gNB, feeder-link switchover, IP reachability, ISL, Gateway, routing convergence, logical path, physical path, shared fate, NGAP procedure, related-work collision, beginner explainer, incremental context sync
