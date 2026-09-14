# Starlink demand-driven allocation × AQM × transport：从专有现象到通用慢反馈控制问题

> **Context only / not formal scientific state.**
>
> 本文件记录当前 ChatGPT 会话中关于 Starlink demand-driven bandwidth allocation、AQM、拥塞控制、SATPIPE / Confucius 论文方法，以及“无 Starlink 终端时如何继续”的增量讨论。它不是 `ccf-b-research202609` 或 `ccf-b-emerging-architecture202609` 的正式 candidate / KEEP / L4 / Phase / verdict。涉及正式科研状态时必须读取对应正式仓库最新文件。

## Background

讨论起点来自 SIGCOMM 2026 的 **Dissecting the StarLink**：生产测量显示，Starlink 宽带专用用户终端的可用带宽并非始终一次性给到峰值，而会随着持续需求逐步增加；与此同时，瓶颈队列受到主动队列管理（AQM）控制，可能在缓存满之前产生 loss。会话最初尝试把这一现象写成：

> queue pressure 一方面帮助 Starlink 判断用户仍有额外需求、提高 grant；另一方面又触发 AQM / 端到端拥塞控制退让，因此同一队列状态存在相反控制语义。

用户的硬资源约束是：**目前无法获得 Starlink 专用用户终端**。因此本轮持续审查：这个问题是否只是 Starlink 私有黑盒问题、能否泛化、能否用公开 artifact + 开源系统做成 B 会级工作。

## Question

核心问题逐步变成：

1. Starlink 实测约 400 ms 的额外带宽 ramp-up 到底意味着什么？是传播时延还是内部资源分配控制时间尺度？
2. AQM、端到端拥塞控制（CUBIC / BBR 等）和 demand-driven grant 是否会形成新的多控制环冲突？
3. 这一冲突是否只是 Starlink 私有实现，还是更一般的“慢按需无线资源分配 × transport feedback”问题？
4. 在没有 Starlink UT 的情况下，公开数据、交互式 emulator、SNS3 / DVB-RCS2、5G-LENA / NR、Linux transport 能支撑到什么程度？
5. 更重要的是，这种机制有没有**足够大的真实用户层代价**，而不只是 speedtest / peak throughput 差一点？

## What we learned

### 1. 已确认的 Starlink 生产现象

基于 `Dissecting the StarLink` 的讨论，当前已确认的公开现象包括：

- 上行瓶颈队列主要位于用户终端（UT），下行主要位于服务卫星；
- 资源分配是 **per-UT**，不是单个 TCP/QUIC flow 各自独占；
- 测试环境下存在大约 `DL ≈ 100 Mbps / UL ≈ 30 Mbps` 的立即可用基础分配；
- 持续更高需求后，额外容量在约 **400 ms** 内逐步出现；
- 停止持续需求后，额外 allocation 会在数百毫秒量级衰减；
- 系统存在 queue-aware AQM / early loss，以及 head-drop / shared-buffer 相关现象；
- 大约每 15 s 的重配置会重新触发容量恢复过程；
- CUBIC、BBR 等不同拥塞控制在该环境下表现明显不同；更积极持续 probe 的算法更容易接近 UDP 测得的可用容量。

这些是生产测量事实或论文支持的外部行为。**Starlink 内部精确 scheduler/AQM 算法仍未公开。**

### 2. 约 400 ms 不是简单的星地传播时延

一个重要修正是：

- Starlink RTT 只有几十毫秒量级；
- 无线 MAC frame / scheduling 本身可以在毫秒级工作；
- 固定速率 UDP 实验中，sender 并未慢慢加速，但 delivery capacity 仍花约 400 ms 上升。

因此当前理解是：

> **400 ms 更像网络内部 demand-to-allocation 控制环的响应时间，而不是“包飞到卫星再回来”的传播时间。**

但目前没有白盒证据解释为什么恰好是约 400 ms。诸如“为了过滤短 burst、维护公平性而做时间平均/积分”只能作为合理推测，不能写成已证实事实。

### 3. “无线资源有限”不等于一定要 400 ms 慢慢扩容

早期讨论曾接近“卫星资源有限，所以都会先少给、再慢慢给；地面基站则一般给足”。这已被修正。

**Current understanding:**

- 地面 LTE/5G 同样是多用户共享有限无线资源，也采用动态 MAC scheduling；
- NR 上行也有 Scheduling Request / Buffer Status Report / uplink grant；
- 因此“根据需求动态分资源”不是卫星独有；
- 真正可能产生新问题的是：**资源分配控制环慢到与端到端 transport / AQM 的反馈时间尺度相当甚至更慢**。

可以用一个待验证的抽象比值理解：

`rho = T_allocation / T_transport_feedback`

- 当 `rho << 1`：底层资源很快调整完，transport 看到的更像外生变化后的容量；
- 当 `rho ≈ 1` 或更大：MAC 还在调整，TCP/AQM 已经多轮反馈，两个控制环可能真正互相影响。

这只是研究抽象，不是已经证明的普适定律。

### 4. DVB-RCS / DAMA 是 scientific ancestry，不应成为 2026 主实验对象

会话补齐了基础概念：

- DVB 是数字视频广播标准体系，不是某颗卫星或星座；
- DVB-S/S2 主要负责卫星向用户的前向链路；
- DVB-RCS/RCS2 提供 satellite return channel；
- DAMA = Demand Assigned Multiple Access，即按需求动态分配卫星返回链路资源；
- RBDC 大致按所需速率请求，VBDC 大致按待发送数据量请求。

早在 2006 年前后，DVB-RCS 研究已经明确指出：

> TCP 发送控制是一条反馈环，DAMA 动态带宽分配又是一条反馈环；两者时间尺度接近时会相互作用。

因此：

- **不能**把“TCP 与卫星按需分配互相影响”当作 2026 novelty；
- **不能**把“TCP-MAC cross-layer coordination”本身当 novelty；
- DVB-RCS 现在更适合作为“老问题的 scientific ancestor / control-loop precedent”。

现代问题若成立，必须来自新的机制组合：低 RTT NGSO + demand-driven allocation + modern AQM + CUBIC/BBR + per-UT allocation + periodic reset / reconfiguration 等。

### 5. 对“把问题泛化到 DAMA”的一次重要反转

**Earlier view:** AQM 把物理队列清掉 → RBDC/VBDC 看到需求减少 → grant 过早下降。

**Later correction:** 这个故事过度简化。DVB-RCS2 / SNS3 中已有 pending request、RBDC/VBDC persistence、backlog persistence 等机制，并不一定因为瞬时物理队列下降就忘记需求。

**Current understanding:** 更合理但仍待证明的可能链条是：

`AQM loss/mark → end-host CC 退让 → 之后进入终端的 offered load 真的下降 → demand allocator 只能看到已经被 transport feedback 压低后的需求`

如果现有 persistence 已经完全保护这种 latent demand，则这一 generalized DAMA motivation 应 KILL；如果 persistence 过短导致 under-allocation、过长又导致 stale/ghost allocation，才可能形成更有价值的 trade-off。

本讨论中已明确：**该 generalized DAMA 线不应因已有投入而自动升级为主线。**

### 6. SATPIPE 的方法被纠正：它不是只靠模型验证

用户曾问 SATPIPE 是否也是“真实数据建模后在模型里跑新算法”。核对后修正为：

- SATPIPE 先在真实 Starlink 上测量周期性 handover；
- 将设计实现为 Linux TCP/BBR 的内核 patch；
- 最后继续在 **真实 Gen-2 Starlink UT + 多地 AWS server** 上做闭环 evaluation；
- 因此它能够直接回答“新 sender action → 真实 Starlink reaction → throughput / retransmission / application QoE”。

这解释了为什么没有 Starlink UT 时，不能简单照搬 SATPIPE 的 evaluation 路线。

### 7. Trace replay 对 handover 和对 demand-driven grant 的适用性不同

这里形成了一个重要方法论区分。

**SATPIPE / 一般 handover：** handover schedule 近似是外生事件。新 TCP 行为通常不会决定卫星是否在某一时刻 handover，因此真实 trace/replay 更容易成立。

**Demand-driven allocation：** capacity 是内生的。sender 改变 offered load 后，未来 grant / queue / AQM 也可能跟着改变。因此：

> 历史 trace 能证明 motivation、做 workload replay 和机制量化，但**不能单独证明一个会改变发送行为的新 design 在真实 Starlink 上会怎样**。

这就是“counterfactual / 反事实”问题。

### 8. 没有 Starlink UT 时，合理路线不是普通 trace replay，而是 interactive model + independent validation

如果继续这条线，较合理的研究方法是：

1. 用 SIGCOMM artifact 的一部分实验校准模型，例如：
   - send rate / sustained pressure → allocation ramp；
   - queue → loss / AQM response；
   - cooldown → allocation decay；
   - reconfiguration → reset；
2. 留出未参与拟合的 trace 做 out-of-sample validation，例如 TCP traces、Mouse/Elephant、多流、重配置；
3. 确认 emulator 能复现 CUBIC/BBR 相对行为、ramp、loss、reset 等；
4. 之后才把新 design 放进去做闭环评价；
5. 最好再在一个**可完全修改的独立 grant-based 系统**中验证 generalization，例如 SNS3 / DVB-RCS2 或 5G-LENA / NR。

仍需承认：如果最终 design 的关键收益只在“自己拟合的 Starlink emulator”里成立，而无第二机制系统或 live-network validation，B 会审稿风险很高。

### 9. Starlink 只是一个可能的现代生产实例，不能直接扩成“所有 LEO”

会话中从“Starlink 专有问题”逐步转向更一般的研究抽象，但也明确限制了外推：

> **不能说所有卫星网络 / 所有 LEO 都有相同 400 ms + AQM 问题。**

更可辩护的抽象是：

> **slow demand-driven wireless allocation × end-to-end congestion feedback**

即：当“观察需求→追加服务能力”的控制环足够慢时，AQM / CCA 可能在 allocation 完成之前先做出退让，形成 cross-layer feedback conflict。

DVB-RCS 历史研究支撑“慢 demand allocation 与 TCP 会互相影响”这一母问题；Starlink 2026 提供现代 NGSO 生产实例。是否存在新的 2026 residual，仍需 collision 和实验。

### 10. D2C 不可直接作为同机制证据

D2C/LTE/NR 同样采用动态无线调度，资源也非常有限，但当前没有公开证据表明 D2C 具有 Starlink broadband 相同的：

- 约 400 ms demand ramp；
- 相同自定义 AQM；
- 相同 head-drop / per-UT allocation；
- 相同 15 s reset。

因此：

> D2C 可以作为“动态 grant-based access”的相关架构背景，但**不能当作 Starlink 400 ms/AQM 冲突已经泛化的事实证据**。

### 11. 现实用户代价目前只证明到 transport 层，应用层因果链仍不足

当前一个关键 reviewer attack 是：

> Starlink 测试环境的基础下行约 100 Mbps，而普通 Zoom、普通 4K video、常见 cloud gaming 往往远低于 100 Mbps；即使 CUBIC 拿不到 300+ Mbps peak capacity，这是否真的影响多数用户？

所以不能直接写：

> “400 ms ramp causes severe cloud-gaming stalls.”

现有证据较强的是：

- 不同 CCA 在 Starlink 上实际 throughput / loss / recovery 不同；
- Starlink 网络动态会影响视频、Zoom、cloud gaming 等应用，但目前这些应用层研究并未把影响严格归因到“400 ms demand ramp × AQM”这一条机制。

### 12. 比“单流少几十 Mbps”更有潜力的用户层后果：跨流外部性

当前讨论认为，更值得验证的不是“Speedtest少跑一点”，而是：

`bulk / elephant flow 需要 > baseline capacity → 持续 pressure 申请额外 per-UT grant → shared buffer / AQM / head-drop 产生 loss → 同一 UT 中本来只需要低码率的 latency-sensitive flow 也受伤`

`Dissecting the StarLink` 已给出重要基础：allocation 是 per-UT，下行存在共享 buffer，aggressive queue-building flow 可以让另一条 flow 发生 loss；但这一 loss 是否足够造成 RTC frame miss / FEC / stall，仍未量化。

这可能比“CUBIC peak throughput不够高”更有 B 会 motivation，因为它涉及**一个 flow 获取额外 radio capacity 时对另一个无辜实时 flow 的外部性**。

### 13. 另一个可能 residual：per-flow transport control × per-UT allocation 粒度错配

由于 TCP/QUIC 拥塞控制通常 per-flow 独立运行，而 Starlink additional allocation 是 per-UT，可能产生：

- 多个 flow 重复 probe；
- probe synchronization；
- aggressive flow 为整个 UT 抬高 grant；
- 其他 flow free-ride；
- per-flow fairness 与 per-UT radio fairness 冲突。

这是当前认为比“普通 TCP-DAMA 两反馈环”更新的 residual family，但尚未完成 strongest-related-work collision。

### 14. Confucius 提供的不是具体算法，而是 scientific problem 模板

上传的 **Confucius: Adapting Home Routers to Congestion Control’s Reactions for Consistent Low Latency** 展示了一个重要论文思路：

- router scheduler 可以瞬时改变某 flow 的可用带宽；
- endpoint CCA 需要多个 RTT 才能收敛；
- 两个各自合理的控制器因 reaction timescale mismatch 产生 delay spike；
- 论文不是简单“让 TCP 更快”，而是重新设计 scheduler 的带宽变化速度来匹配 CCA reaction，并给出 theory + ns-3 + Linux qdisc + live workload evaluation。

Starlink 线值得学习的是这种问题抽象：

> **不是“某个协议在卫星上慢”，而是两个/三个控制环的时间尺度与控制粒度是否存在结构性错配。**

不能把 Confucius 本身当作 Starlink 问题已经发生的证据。

### 15. SATPIPE / Confucius / Dissecting StarLink 三篇的角色分工

当前会话形成的研究使用方式：

- **Dissecting the StarLink（SIGCOMM 2026）**：真实生产机制、异常、数据/artifact 的“问题矿山”；
- **SATPIPE（INFOCOM 2025）**：学习“measurement → root cause → system design → real implementation → end-to-end evaluation”的论文构造；
- **Confucius**：学习“两个控制器各自正确，但反应时间尺度不匹配 → 新系统问题”的 scientific-problem 抽象。

目标不是把 SATPIPE / Confucius 的设计拼到 Starlink 上，而是用它们检查 measurement paper 是否存在 **paper-external residual**。

## Important reasoning / corrections timeline

### Earlier view 1
“Starlink queue pressure 是申请 grant 的信号，因此 AQM 和 MAC 是两个相反动作。”

### Correction
生产测量支持 sustained offered load / queue occupancy 与 allocation growth 相关，但没有白盒代码证明 scheduler 具体用哪个变量、哪个阈值。应写成外部可观察关系，不应把内部 policy 细节当事实。

### Earlier view 2
“这应该是所有卫星都有的问题，因为卫星资源有限；地面基站一般会一次给足。”

### Correction
地面蜂窝同样动态共享无线资源。新问题如果存在，应来自**allocation feedback timescale / control-loop interaction**，不是“卫星动态、地面固定”。

### Earlier view 3
“AQM删掉队列 → DAMA看不到需求。”

### Correction
RBDC/VBDC / SNS3 已有 pending request 和 persistence；更合理的未知是 AQM 通过 end-host CC 压低未来 offered load 后，allocator 是否仍能保留 latent demand，以及 persistence 是否出现 under-allocation ↔ stale-allocation trade-off。

### Earlier view 4
“SATPIPE也是拿数据建模再在模型里验证。”

### Correction
SATPIPE 最终使用真实 Starlink UT 做新 TCP 的闭环 evaluation；这正是无 UT 时必须寻找替代证据链的原因。

### Earlier view 5
“D2C应该也有同样问题。”

### Correction
D2C也有动态 grant-based scheduler，但目前没有公开证据支持相同 400 ms/AQM/reset 行为。不得把“架构相似”写成“现象已泛化”。

## Decision / current conclusion

### Discussion-level conclusion only

- 这条线**不应仅作为 Starlink 私有 CCA 调参题**；没有 UT 时，live Starlink design validation 是明显短板。
- 也**不能简单升级成“所有卫星网络的共同问题”**。
- 目前更有研究价值的母题是：

> **当按需无线资源分配的响应时间慢到与 transport / AQM 的反馈时间尺度相当时，是否产生系统性的跨层反馈冲突？**

- Starlink 2026 可以作为现代生产系统落入这一参数区域的证据；DVB-RCS 是历史 scientific ancestry；地面 NR / 5G 提供“快速动态调度”的对照和第二类机制系统。
- 真正能否达到 B 会，取决于是否找到一个老 TCP-DAMA 文献未覆盖的新 residual，例如：
  - 三控制环（CCA × AQM × slow allocation）的新时间尺度区间；
  - per-flow congestion control × per-UT resource allocation 粒度错配；
  - bulk flow 获取额外 grant 时对低延迟 sibling flow 的跨流外部性。
- 若最后只是“Starlink 400 ms导致 peak throughput 少一点”，应降级/KILL。

用户此前已明确要求避免路径依赖；本文件不把这一讨论升级为正式 candidate，也不修改任何正式仓库状态。

## Experimental / evidence strategy discussed

### Motivation characterization（不改变网络）

公开 trace 可以用于量化“真实已发生网络行为对应用的代价”，例如：

- 从 ramp / reset trace 计算容量恢复税、flow completion time；
- 用 Mouse/Elephant trace 检查跨流 loss；
- 将真实 bandwidth / RTT / loss trace 输入真实或成熟 RTC/DASH workload，观察 frame deadline miss、FEC、p95/p99 delay、bitrate switch、rebuffer 等。

讨论中提到可复用的开源方向包括 `ns3-sparkrtc` 和已有 LEO DASH / live streaming artifact。

这类 replay 可以回答：

> “真实 Starlink 已经发生的网络轨迹，对应用会有什么后果？”

但不能回答：

> “如果新 sender algorithm 改变发送动作，Starlink scheduler 将怎样重新分配？”

### Design evaluation（会改变 sender/network interaction）

若进入 design，需：

- measurement-calibrated interactive emulator；
- 留出独立 trace 做 out-of-sample validation；
- 最好有第二种可控的 demand/grant 系统做 generalization；
- 不能只用自己拟合的模型证明自己的算法最好。

### Potential low-cost phase-diagram experiment

一个值得继续讨论的机制实验是扫 `T_allocation`：

`1 / 5 / 10 / 50 / 100 / 200 / 400 / 800 / 1000 ms ...`

同时改变 RTT、AQM、CCA、base/max capacity ratio、多流 workload，观察何时开始出现：

- 可用空闲容量拿不满；
- queue/loss/oscillation；
- CUBIC/BBR fairness 异常；
- latency-sensitive flow deadline miss；
- periodic reset 后恢复被放大。

如果存在清晰 phase boundary，而且 Starlink 实测参数落入“问题区”，这个 scientific story 会明显强于“Starlink专用TCP调参”。

## Open questions

1. `Dissecting the StarLink` 公开 artifact 具体有哪些 raw trace、字段和实验脚本已经可下载？是否足以分 training/calibration 与 out-of-sample validation？
2. 2006–2026 的 TCP×DAMA、AQM×satellite、modern CCA×LEO、per-flow×per-terminal allocation literature，是否已经覆盖上述三控制环/粒度错配 residual？
3. Starlink Mouse/Elephant 的 coupled loss 到底多大？喂给 RTC workload 后是否产生明显 FEC / frame miss / tail delay？
4. 普通应用很多低于约100 Mbps baseline；哪些现实 workload 真正进入 additional-allocation 区域？是 bulk transfer、多用户家庭、high-rate RTC/VR，还是影响总体很小？
5. `T_allocation / T_transport_feedback` 是否真的能形成稳定可解释的 problem region，而不是人为模型参数效应？
6. 地面 NR / 5G 在类似负载与 buffer 下为什么通常不表现出相同严重现象？真正差异是调度时间尺度、队列结构、显式 BSR、还是其他机制？
7. 没有 Starlink UT 时，什么样的第二独立系统验证足以让 B 会审稿人接受 generalization？
8. D2C 当前是否有任何公开测量能证明 demand-to-grant 慢反馈，而不是仅仅存在普通动态调度？目前答案仍是“没有直接证据”。

## Relation to formal research repositories

- `yanye-cmd/ccf-b-research202609`：成熟地面机制 × D2C / LEO / NTN 的正式状态权威来源。
- `yanye-cmd/ccf-b-emerging-architecture202609`：Emerging / 6G Architecture 的正式状态权威来源。

本文件只保存当前跨对话恢复所需的研究背景、修正链和未决问题。**不得根据本文件推断或修改正式 Phase / KEEP / L4 / active candidate / TESTABLE SEED / EXPERIMENT ENTRY。**

## Sources discussed in this conversation

- *Dissecting the StarLink*, SIGCOMM 2026 — production measurement / demand-driven allocation / AQM / queue / reconfiguration；讨论中上传并引用了论文全文。
- *SATPIPE: Deterministic TCP Adaptation for Highly Dynamic LEO Satellite Networks*, INFOCOM 2025 — real Starlink handover measurement + Linux TCP implementation + real-Starlink evaluation；讨论中上传论文全文。
- *Confucius: Adapting Home Routers to Congestion Control’s Reactions for Consistent Low Latency* — bandwidth reallocation timescale vs CCA reaction mismatch；讨论中上传论文全文。
- DVB-RCS / DVB-RCS2 / DAMA / RBDC / VBDC literature — 作为历史 control-loop ancestry 与 collision source，而非默认现代实验对象。
- 3GPP NR / NTN MAC scheduling / BSR / grant mechanisms — 用于纠正“地面固定、卫星动态”的错误二分。

## Conversation Continuation State

### Current discussion state

当前会话已经从“Starlink 400 ms + AQM 能不能直接做一个新 CCA”推进到更抽象、更严格的问题：

> **是否存在 slow demand-driven wireless allocation 与 end-to-end transport/AQM feedback 的结构性时间尺度冲突，以及它是否造成足够大的用户层代价。**

目前仍处于 evidence / collision / characterization 讨论，不应视为正式 design 阶段。

### Last meaningful conclusion

最有价值的当前判断不是“Starlink算法有bug”，而是：

> **约400 ms 看起来是内部资源分配控制时间尺度，而非传播物理必然；动态资源调度本身地面/卫星都有。真正可能新的地方，是当 allocation feedback 太慢时，MAC还没扩容完，AQM/CCA已经开始退让，从而产生多控制环反馈冲突。**

如果这个问题只导致少量 peak-throughput损失，价值不足；如果能证明跨流 loss、周期恢复放大、fairness 或高码率实时业务尾延迟等显著后果，才值得继续。

### Unresolved questions

- 用户层 consequence 是否真的显著；
- strongest related work 是否已覆盖三控制环 / 粒度错配；
- Starlink公开artifact能否支撑严格模型校准与独立验证；
- 第二个可控系统应选 SNS3/DVB-RCS2、5G-LENA/NR，还是 Linux 自建 demand allocator；
- 是否能形成清晰的 allocation-timescale phase diagram。

### Likely next discussion step

优先继续两个低成本动作中的一个或并行推进：

1. **artifact-first consequence audit**：读取 SIGCOMM 2026 公开 artifact，尤其 Mouse/Elephant、ramp、reconfiguration 数据，先量化真实用户层影响；
2. **collision-first residual audit**：系统梳理旧 TCP-DAMA 与现代 AQM/BBR/LEO work，确认“三控制环时间尺度冲突”和“per-flow × per-UT 粒度错配”是否仍是真 residual。

若两者都不给出明显突破，应继续按 B-Venue-Oriented Motivation Search 原则寻找其他候选，不因本轮投入产生路径依赖。

## Keywords

Starlink, LEO, NGSO, demand-driven bandwidth allocation, grant, MAC scheduler, AQM, CUBIC, BBR, per-UT allocation, per-flow congestion control, control-loop mismatch, timescale mismatch, DVB-RCS, DVB-RCS2, DAMA, RBDC, VBDC, SATPIPE, Confucius, interactive emulator, measurement-calibrated model, Mouse-Elephant, cross-flow externality, transport, user QoE, artifact, B-venue motivation
