# Gateway / Multipath / PMPS / D2C 讨论上下文（2026-09-14）

> Context only. This file records a continuing ChatGPT discussion and is **not** a source of formal Phase / KEEP / L4 / candidate status. For formal scientific state, read the latest authoritative files in `yanye-cmd/ccf-b-research202609` or `yanye-cmd/ccf-b-emerging-architecture202609`.

## Background

本轮讨论最初从一个对称性问题开始：此前较多关注 UE 一侧的 LEO service-link handover，那么卫星另一侧的 feeder link / NTN Gateway 如何连接、切换、恢复？随后讨论逐渐扩展到：

1. Gateway 侧是否应该维护多个 feeder links，而不是把 mobility 总是抽象成离散的 `A -> B` handover；
2. 多路径/多队列思想与 MPTCP / MPQUIC 的关系；
3. INFOCOM 2026 PMPS 为什么能把一个看似简单的“多卫星池”做成顶会论文；
4. PMPS 能否迁移到普通 D2C 手机；
5. D2C / SCS 下 TN -> NTN connected-mode handover 与共享频谱、干扰保护、target-preparation overlap 的关系。

讨论方法逐渐从“想到一个方案就搜论文”转成：

> **成熟机制脚手架 × 新趋势属性矩阵 -> broken assumption -> observable consequence -> related-work collision**。

## Question

当前长期问题族包括：

- Gateway / feeder mobility 的真实生命周期是否应理解为一个动态 capacity pool，而不是一次次二元 handover？
- 现有 feeder-handover / satellite traffic-engineering 工作如何处理 new-link warm-up、old-link draining、traffic-ready 与 safe-to-remove？
- MPTCP / MPQUIC 在 LEO/5G NTN 中，逻辑 path 与底层真实物理/路由 path 是否会发生系统性错位？
- PMPS 的 continuous multi-satellite pool 为什么到 2026 仍能发 INFOCOM，它与旧 multipath / soft handover 工作差在哪里？
- 普通 D2C UE 无法默认拥有多个并发高增益卫星 beam 时，PMPS 的 architecture assumption 如何改变？
- Rel-20 LTE TN -> NR NTN connected-mode HO 在 SCS / 共享频谱 / terrestrial-priority coexistence 下，是否仍有足够的 target-preparation overlap window？

## What we learned

### 1. Gateway 不是“互联网最终出口”，而是 satellite network 落到 terrestrial transport 的基础设施节点

需要区分：

- UE <-> satellite：service link；
- satellite <-> NTN Gateway：feeder link；
- Gateway 后面到 gNB / AMF / UPF：terrestrial transport / TNL；
- PSA-UPF / external data network 才更接近业务锚点或最终数据出口。

因此 UE 换 satellite 与 satellite 换 Gateway 并非一一绑定。存在：

- UE 换星，Gateway 不换；
- satellite 不换，Gateway 换；
- UE / Gateway 都换；
- 有 ISL 时，serving satellite 与 gateway-facing satellite 可以不同。

### 2. 3GPP 已支持 soft / hard feeder-link switchover；soft FLSO 本质上已经有 make-before-break 思想

讨论中确认：

- hard FLSO 可能有 feeder radio interruption；
- soft FLSO 可以让 payload 在一段时间内同时连接多个 Gateway；
- FLSO 是 TNL procedure；必要时可结合现有 Path Switch / PDU Session Resource Modify 等机制；
- 因而“Gateway side 完全没有 completion / fallback 保护”这一宽泛怀疑被明显削弱。

### 3. Gateway 同时连接多颗卫星是成熟 baseline，不是新想法

代表性 related work / baseline：

- Zhou et al., *A Novel Feeder Link Handover Strategy for Backhaul in LEO Satellite Networks*, Sensors 2023：允许一个 Gateway / GS 同时建立多条 feeder links，同时考虑 feeder quality、ISL available capacity、remaining service time 与 handover frequency；
- BMSB 2023：通过 forward/backward ISL 辅助 feeder-link switch；
- IEEE VTM 2024：用 ISL 创造 virtual visibility，减少 hard/soft FLSO interruption；
- 2025 Multi-Antenna Gateway Station (MAGS) 工作：把多天线 Gateway + massive feeder links 当作高容量 baseline，并研究 time-continuous frequency allocation / interference；
- 2025 grouping-based FLSO：利用 TLE / orbital prediction，把多个 FLSO 成组执行以减少重复的控制过程。

因此 broad idea：

> “Gateway 同时连多颗卫星、根据星历知道谁要飞走、提前准备下一颗”

已经不是 novelty。

### 4. 但现有 Gateway 文献常把 association 仍然抽象成离散状态；`warm-up -> active -> drain -> retire` 的完整 packet/flow lifecycle 仍是值得审计的边界

当前较精确的 lifecycle 抽象：

```text
PREDICTED
  -> VISIBLE
  -> FEEDER-READY
  -> ROUTE/TNL-READY
  -> TRAFFIC-READY
  -> ACTIVE
  -> STOP-NEW-TRAFFIC
  -> DRAINING
  -> SAFE-TO-REMOVE
  -> RELEASED
```

重要区分：

- `link exists` != `traffic-ready`；
- `new feeder up` != `old traffic drained`；
- `soft FLSO` != 已经定义了 PMPS 式 packet-weight draining policy。

Zhou 2023 主要用 time-slot association / maximum-flow abstraction；VTM 2024 更认真处理 new-path establishment / old-feeder release，但没有把整个 Gateway aggregate traffic 的 continuous packet/flow migration 作为核心；MAGS 2025 的“time-continuous”主要针对 frequency allocation / interference，而不是 flow draining。

**当前理解：** Gateway 方向不是 KILL，只是“dynamic feeder capacity-pool lifecycle”目前仍缺少足够强的公开 observable consequence，因而只能作为 HOLD / audit question，而不能直接当课题。

### 5. MPTCP / MPQUIC 基础概念已经补齐

核心层级：

```text
Application
  -> end-to-end transport (TCP / MPTCP / QUIC / MPQUIC)
  -> IP routing
  -> 5G user plane (PDU Session / N3 / UPF)
  -> LEO physical / logical network (satellite / ISL / Gateway)
```

关键概念：

- MPTCP = 一个应用 connection 下多个 TCP subflows；
- Path Manager 决定有哪些 subflows / 地址；
- Scheduler 决定下一批数据用哪个 subflow；
- Congestion Control 决定每条路发多快；
- DSN / DSS 解决多 subflow 数据在 connection-level 的映射与重组；
- MPQUIC = 一个 QUIC connection 同时管理多个 Path ID；当前标准化定义 path creation / validation / management，但不统一规定 scheduler；
- PDU Session != MPTCP / QUIC connection；一条 PDU Session 里可以承载很多 transport connections；
- transport 层看到的 logical path 不等于实际 satellite / ISL / Gateway 物理路由。

### 6. LEO x multipath 最近的 research evolution

近期代表性工作/方向：

- MAMS (ToN 2024)：mobility-aware MPQUIC scheduling，利用未来 path condition；
- MACO (TMC 2024)：LEO mobility-aware multipath congestion control；
- HotNets 2024 *Mind the Misleading Effects of LEO Mobility on End-to-End Congestion Control*：LEO reconfiguration 改变 transport signal semantics；
- StableRoute (INFOCOM 2025)：LEO route churn / shortest-path updates 本身会造成 packet reordering 与 TCP state mismatch；
- 2025 3D mobility MPQUIC scheduler evaluation：现有 schedulers 在高动态场景排序会变化；
- 2026 Linux MPTCP + FRRouting OSPF/ECMP + LEO emulation：真实 full-stack 地看到 routing/ECMP 影响 subflow diversity 与 throughput；
- PRISM 2026 / INFOCOM 2026 PMPS 等说明“轨迹预测 + proactive multipath scheduler”已经非常拥挤。

因此简单做：

> “知道 LEO 轨迹 -> 提前调度 path”

已高度碰撞。

目前更值得审的两个 cross-layer residual：

1. **logical multipath diversity vs physical path diversity drift**：subflow identity 不变，但 LEO routing churn 可能让原本独立的 subflows 反复 `独立 -> 重合 -> 独立`；
2. **transport path liveness vs 5G/NTN traffic-readiness**：transport 看见 path alive / validated，不代表 feeder / TNL / N3 / UPF 状态已真正 ready。

这两个目前仍只是 testable questions，不应在 Context Hub 中写成正式 candidate。

### 7. PMPS 2026 的真正贡献不是“第一次想到多卫星池”

用户上传并精读了 INFOCOM 2026：

> *PMPS: Predictive Multi-Path Scheduling for Handover-Free LEO Communications*

关键理解：

- 不是 MPTCP / MPQUIC 论文；它自己做 packet scheduling、sequence/reassembly、selective redundancy；
- 假设 phased-array terminal 能形成 `B_u >= 2` 个 concurrent beams；
- 将多个当前可用 satellite 当作连续可调度资源；
- 根据 remaining visibility、link quality、capacity/queue 等逐渐降低 departing satellite 的流量权重，而不是产生离散 handover moment；
- 对 vulnerable satellite 上少量尾部流量做 selective redundancy；
- 还补了 optimization formulation、complexity / approximation algorithm、reordering / deduplication / simulation evaluation。

**为什么一个简单想法还能发 INFOCOM：**

核心 idea 简单并不等于贡献弱；作者把“离散 handover -> continuous resource-weight transition”重新问题化，并补齐 theory + algorithm + reliability + evaluation。顶会 novelty 不要求核心点子复杂，而要求问题新、证据强、机制完整。

### 8. PMPS broad idea 不是第一次出现

Earlier / adjacent work 已经有：

- Demand Island Routing (Computer Networks 2023)：terminal 可同时连接多颗 satellite，并在 frame granularity 动态平衡 demand / routing；
- handheld soft handover / multi-satellite cooperation；
- multi-satellite MIMO / coordinated transmission 等。

因此不能把 PMPS 理解为“第一次有人让终端同时连多颗卫星”。其差异更接近：departure-aware continuous traffic migration + selective redundancy + handover-free framing。

### 9. PMPS 原样迁移到普通 D2C phone 不现实

PMPS 的隐藏关键假设：

> terminal 自己可以保持多颗卫星 active data links / concurrent beams。

普通 D2C smartphone 的现实目标恰恰是：不要求专用卫星 phased-array / high-gain terminal，功率、天线、RF/baseband resources 受限。

因此要区分：

- `measure / track multiple satellites`；
- `simultaneously carry user data over multiple satellite links`。

二者不是一回事。

更现实的 D2C 迁移方式可能是：

- network / satellite side 维护 multi-satellite candidate pool；
- UE 一次正常 uplink，可被多个 satellite 接收，再通过 ISL / network cooperation 处理；
- 下行使用 network-side coordinated / multi-satellite transmission；
- 尽量把复杂度放在 satellite / network，而不是普通 handset。

### 10. D2C / SCS 频谱共存纠正了一个早期过度简化

Earlier view：

> “Starlink 在有 terrestrial deployment 的地方因为同频干扰就必须关掉 satellite transmission，因此 TN -> NTN overlap 不存在。”

Later correction / current understanding：

- FCC SCS 框架把 satellite supplemental coverage 置于 terrestrial-primary coexistence 约束下；
- 运营商 / satellite provider 可通过频率规划、beam / power / geographic coordination、keep-out 等方式满足 harmful-interference limits；
- 不能简化成“只要有 terrestrial BS，satellite 就必须 OFF”；
- 当前 SCS commercial positioning 确实主要是 terrestrial coverage gaps / edge，但 regulatory coexistence 不是简单二值开关；
- spectrum strategy 还可分 dedicated spectrum 与 flexible/shared reuse，后者的 overlap / interference tension 更强。

### 11. Rel-20 TN -> NTN connected-mode handover 与 SCS coexistence 的交叉值得继续审

Rel-19 已先做 LTE TN -> NR NTN idle-mode mobility / redirection；Rel-20 正在补 connected-mode LTE TN -> NR NTN handover。

近期 related-work collision：

- shared-spectrum TN + LEO user association / power / seamless handover 已有 IEEE TCOM 2025 一类工作；
- interference-aware offloading / handover、dual connectivity、D2C cell/beam spatial isolation、co-channel interference measurement 等也已形成研究线；
- 因而“TN/NTN handover 要考虑 interference”这个 broad story 已经不新。

但更窄的 residual 仍值得审：

> **Rel-20 connected-mode HO protocol 需要的 target-measurement / preparation / execution window，与现实 SCS / shared-spectrum coexistence 能安全提供的 target-availability window 是否匹配？**

也就是：

```text
standard state machine needs:
  target measurable
  -> target prepared
  -> HO executable

vs.

radio / spectrum / regulatory reality provides:
  sufficient SINR
  + resource availability
  + terrestrial protection constraints
```

如果天然吻合则应 KILL；只有出现可重复的 HOF / interruption / capacity cliff / late-preparation consequence 才可能有研究价值。

## Important reasoning

### A. 不再把“新趋势 + 老协议”当作直接 novelty

统一方法：

```text
成熟机制的默认假设
    x
新趋势的具体属性
    ->
默认假设失效？
    ->
observable consequence？
    ->
related-work / standards collision？
```

LEO 属性必须具体拆成：轨道可预测、高频 route churn、有限 path lifetime、多星可见、underlay path identity 变化、Gateway/ISL/core path hidden from transport 等。

AI / Network-for-AI 也不能机械和 MPTCP/MPQUIC 相乘；训练集群主流通常是 RDMA/RoCE/NCCL，而 LLM token streaming / multimodal / cross-DC 等才与 QUIC / multipath 更自然相关。

### B. `Path` 在不同层含义不同

必须持续区分：

- transport logical path / MPTCP subflow / MPQUIC Path ID；
- IP route；
- 5G PDU Session / N3 path；
- physical satellite / ISL / Gateway path。

未来任何“多路径”课题都必须先声明研究的是哪一层 path。

### C. `link overlap` 与 `traffic continuity` 不是同一件事

Soft FLSO / dual connectivity / make-before-break 只能说明新旧 link 可以重叠；不能自动证明：

- old queue 已 drain；
- in-flight packet 已处理；
- new route / TNL / N3 已 traffic-ready；
- traffic weights 已连续迁移；
- aggregate capacity 没有瞬态 cliff。

反过来，也不能因为论文没显式写 draining 就自动宣称它有 correctness gap。

## Decision / current conclusion

1. **Gateway dynamic multi-satellite capacity pool**：broad idea 已是成熟 baseline；继续保留的是 member lifecycle / readiness / draining / retirement audit，不是“多连几颗星”本身。
2. **MPTCP/MPQUIC x LEO**：简单 predictive scheduler 已高度拥挤；优先审 logical-vs-physical path identity drift 与 transport-liveness-vs-network-readiness 两类 cross-layer mismatch。
3. **PMPS**：很重要的参考论文，但其 `multi-satellite pool` 不是首次出现；它的价值在 problem framing + continuous weighting + selective redundancy + theory/evaluation。它不是标准 MPTCP/MPQUIC，也没有建模完整 Gateway/ISL/5GC underlay。
4. **PMPS -> ordinary D2C**：不能原样迁移；应把 multi-satellite complexity 更多下沉到 satellite/network side。
5. **D2C TN->NTN + SCS coexistence**：broad “interference-aware handover” 已有强 related work；更窄的 protocol-preparation-window vs interference-safe target-window 值得继续标准/论文深审。

以上均为当前讨论结论，不是正式科研仓库 verdict。

## Open questions

1. feeder-handover / satellite TE 的公开系统里，是否真的存在显式的 `STOP-NEW-TRAFFIC -> DRAIN -> SAFE-TO-REMOVE` 状态，还是主要依赖 routing/TE 自然收敛？
2. new feeder association 的 `VISIBLE -> RF ready -> route/TNL ready -> traffic-ready` 各阶段在真实/开源实现中分别由谁宣布完成？
3. 是否有 packet-level full-stack evidence 表明离散 association / late warm-up 会造成可观测 capacity cliff、queue burst、loss、tail-latency spike？
4. Linux MPTCP / MPQUIC 在 LEO routing churn 下，logical subflows 的 physical path overlap 是否会随时间反复变化，而 transport 无法感知？
5. PMPS 与 Demand Island Routing / soft-handover / multi-satellite MIMO 的最准确 novelty boundary 是什么？
6. 在 ordinary D2C handset capability 下，network-side multi-satellite pool 能恢复 PMPS 的哪些收益，哪些保证必然失效？
7. Rel-20 LTE TN -> NR NTN connected-mode handover 的 measurement / preparation / execution state machine 到底要求多长的 source-target overlap？
8. 2025 IEEE TCOM shared-spectrum TN/LEO seamless-handover work 到底做到 protocol-level handover 还是主要做 time-slot association + power optimization？这是下一篇最需要精读的强 collision。

## Relation to formal research repositories

- 本文件只保存跨对话 context、learning、sidequest reasoning 与 paper-reading history。
- 任何正式 Phase / KEEP / L4 / candidate / KILL 状态必须重新读取 `yanye-cmd/ccf-b-research202609` / `yanye-cmd/ccf-b-emerging-architecture202609` 最新权威文件。
- 不应因为本文件使用“residual / 值得审”等措辞，就把对应想法自动升级为正式 candidate。

## Keywords

`Gateway` `feeder link` `FLSO` `soft FLSO` `hard FLSO` `dynamic capacity pool` `warm-up` `draining` `traffic-ready` `safe-to-remove` `MPTCP` `MPQUIC` `subflow` `logical path` `physical path` `path identity drift` `PMPS` `INFOCOM 2026` `multi-satellite` `D2C` `Direct-to-Cell` `SCS` `FCC` `TN-to-NTN` `Rel-20` `shared spectrum` `handover overlap` `3GPP` `ISL` `Gateway-facing satellite`

## Conversation Continuation State

### Current discussion state

本会话刚从 Gateway / feeder capacity-pool 与 PMPS 的比较，转向 D2C TN->NTN connected-mode mobility 及 SCS shared-spectrum coexistence。用户倾向先做 related-work 深审，而不是马上设计方案。

### Last meaningful conclusion

当前最窄、仍值得继续审的问题不是“D2C handover 要考虑干扰”，而是：

> **Rel-20 connected-mode TN->NTN HO 所需的 target preparation / overlap window，与现实 SCS coexistence 能安全提供的 target availability window 是否匹配。**

同时 Gateway 方向未被否定，只是当前证据不足，保留 `dynamic feeder capacity-pool lifecycle` 作为后续可回访的 audit question。

### Unresolved questions

- 2025 IEEE TCOM `shared-spectrum TN/LEO + seamless handover` 工作是否已经覆盖 protocol preparation window，而不只是 association / power optimization？
- Rel-20 LTE TN -> NR NTN connected HO 的准确 RRC/measurement/preparation timeline 是什么？
- SCS dedicated-spectrum 与 flexible/shared-spectrum 两种部署下，overlap tension 是否本质不同？
- 普通 D2C UE 的多星 tracking / active-data capability 到底有哪些现行 3GPP 明确能力边界？

### Likely next discussion step

优先精读最强 collision：2025 IEEE TCOM 关于 shared-spectrum TN/LEO seamless handover 的论文，按 `problem -> model -> handover definition -> interference model -> evaluation -> what it does not cover` 拆解；随后再与 Rel-20 connected-mode LTE TN -> NR NTN handover 标准时序逐状态对齐。
