# 闭环无人系统业务 × LEO/NTN 特性矩阵：选题支线讨论

> Context only — NOT CURRENT SCIENTIFIC STATE
>
> 本文件记录 ChatGPT 对话中形成的研究支线思路与 related-work 线索，不代表正式 candidate / KEEP / L4 / Phase / verdict。任何正式科研状态都必须回到 `yanye-cmd/ccf-b-research202609` 或 `yanye-cmd/ccf-b-emerging-architecture202609` 的最新权威文件核对。

Latest incremental sync: 2026-09-14

## Background

讨论起点是一个业务观察：无人机、无人车、灾害救援机器人等在无地面网络覆盖区域，可能需要依靠 LEO 卫星把感知信息上传给远端控制/计算节点，再把新的控制决策返回设备。最初的问题是：这类业务的特殊需求与 LEO 网络特性结合后，是否可能产生值得 CCF-B / strong Q2 深挖的网络系统问题。

## Question

不直接从“卫星时延高”“算力卸载”“QoS 优化”猜题，而是先系统拆解：

1. 这类无人系统业务有哪些真正特殊的流程、状态机、流量关系和控制语义？
2. LEO/NTN 有哪些会改变这些机制的特殊属性？
3. 将“业务机制 × LEO 属性”逐格交叉后，哪些格子出现潜在机制冲突、组合缺口或可观察 consequence？
4. 哪些交叉已经被 related work 做掉，哪些仍值得 bounded deep audit？

## What we learned

### 1. 业务不能只概括成“低时延”

当前讨论识别出至少四类有长期价值的业务机制：

- **遥控车辆 / teleoperation 多流闭环**：高带宽 perception/video 上行、小流量但严格 deadline 的 command 下行、状态/ACK 等流共同完成一个控制任务；它们不是彼此独立的普通 flow。
- **UAV C2 多控制模式**：3GPP TS 22.125 区分 waypoint、direct stick steering、UTM automatic flight、approaching autonomous navigation infrastructure 等控制模式；不同模式的消息周期、大小、时延、可靠性和视频反馈需求不同。3GPP Portal 的相关材料给出了 direct stick steering 约 40 ms 周期、24 B 级控制消息和 ACK 需求，并给出 Non-VLOS 视频反馈约 4 Mbps / 140 ms 的示例 KPI。
- **SC3 因果闭环**：sensor 上传 sensing data → computing center 计算 → actuator 接收控制命令 → 物理世界变化 → 新一轮 sensing。2024/2025 的 *Structured Connectivity for 6G Reflex Arc* 已明确把 sensor→compute 与 compute→actuator 两段作为同一任务的 UL/DL，并提出 task-oriented virtual user，因此“闭环耦合”本身不能直接当创新。
- **冗余 / 主备 C2**：讨论注意到 3GPP UAS Stage-2 规范包含 redundant C2 connectivity；“应用知道业务主备关系、网络掌握链路/拓扑状态”可能形成信息分裂，但当前具体 primary/secondary 语义仍需要后续逐字核验 TS 23.256，不能把聊天中的表述直接当已证实事实。

### 2. LEO 侧需要按机制属性拆，而不是只写“高时延”

当前讨论形成的 LEO 属性轴：

- S1：较长且时变的传播/端到端时延；
- S2：卫星、波束高速移动，即使 UE 静止也存在频繁 mobility / handover；
- S3：可见窗口、遮挡与 intermittent connectivity；
- S4：gateway / feeder-link / 后端路径切换，UE 不一定移动但网络路径会变；
- S5：链路容量有限且随几何、资源分配、拥塞而变化；
- S6：若使用星上/边缘计算，compute endpoint 本身也可能随拓扑移动或需要迁移；
- S7：multi-connectivity / multi-link 可提高可用性，但引入同步、冗余、资源与状态协调问题；
- S8：轨道运动高度可预测，因此“未来可见窗口/链路寿命”可能提前获知，这一点不同于很多随机地面移动场景。

LEONE（Low-Latency Command and Control via LEO Satellites, 2023–2026）直接研究无地面覆盖区域中经 LEO command/control ground and aerial autonomous vehicles，说明业务场景本身是真实研究问题，但也意味着“无人系统 + LEO”这个大标题已经不新。

WONS 2026 的 *Impact of Geometry and Satellite Mobility on Handover Strategies for Remote Driving* 已研究 remote driving 场景中的 V2S 可见性/卫星选择/handover；其公开全文说明目前只分析 vehicle→satellite 的数据 offload / V2S links，使用周期 Vehicle Status Messages 上报 GCC。这给出了一个值得核查的 negative space：完整 perception→decision→command 事务与 NTN path transition 的交叉，是否仍有未覆盖机制问题。

## Important reasoning

### Earlier view

对话一开始曾较快跳到“5G QoS/SLO 与闭环控制目标失配”“network KPI 与 control outcome ranking reversal”这一类假设。

### Later correction

用户明确纠正：不能先拍脑袋想一个 QoS mismatch，再去找论文；必须先把业务的特殊流程/机制拆清楚，再与卫星网络属性做矩阵运算，并逐格做 related-work collision。

### Current understanding

当前更合适的 problem-generation 方法是：

```text
业务状态机 / flow dependency / redundancy semantics / control mode transition
                         ×
LEO handover / feeder switch / intermittent connectivity / capacity variation /
multi-connectivity / predictable topology / moving compute
                         ↓
寻找两个各自正确的机制组合后出现的 observable consequence
                         ↓
related-work collision + standards audit + minimal falsification
```

重点不是“把 LEO 加到机器人上”，而是寻找 **composition gap**：成熟业务机制与成熟网络机制单独都合理，但在 LEO 的高速拓扑变化、路径切换或断续覆盖下组合后，原先依赖的假设不再成立。

## Mechanism × LEO matrix (discussion scaffold)

下表是用于生成和筛选问题的讨论脚手架，不是 coverage-complete 正式矩阵。

| 业务机制 | 时延变化 | 卫星/波束 HO | 断续/遮挡 | GW/路径切换 | 时变容量 | 移动算力 | 多连接 | 可预测拓扑 |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 感知→计算→控制因果链 | 中 | 高 | 高 | 高 | 中 | 高 | 高 | 中 |
| 大 UL perception + 小 DL command | 中 | 高 | 高 | 高 | 高 | 中 | 高 | 中 |
| 周期流 + event-triggered 紧急流 | 中 | 中 | 高 | 中 | 高 | 中 | 中 | 高 |
| 控制模式 / KPI 动态切换 | 中 | 高 | 高 | 高 | 中 | 中 | 高 | 高 |
| 冗余 / 主备 C2 | 中 | 高 | 高 | 中 | 中 | 中 | 高 | 高 |
| compute / offloading 在环 | 中 | 高 | 中 | 高 | 中 | 高 | 中 | 高 |
| 信息与 command 会过期 | 高 | 高 | 高 | 中 | 中 | 高 | 中 | 中 |
| 多机器人共享通信+算力 | 中 | 中 | 中 | 中 | 高 | 高 | 中 | 中 |

“高/中”只表示当前讨论优先级，不表示已有证据强度。

## Current seed families worth auditing

### Seed A — 多 flow 闭环事务 × NTN path transition

一个控制任务同时依赖 video/sensor/status/command/ACK 等多条 flow；若 satellite handover、gateway switch、multi-link transition 恰好发生在 perception→decision→command 中间，需要核查这些 task-correlated flows 在真实 5GC/NTN 中是否始终同步/原子地迁移，以及若不同步是否产生可观察的 task-level consequence。

**关键 kill-test**：如果这些流在对应 3GPP PDU session / QoS-flow / mobility procedure 中天然作为一个一致整体切换，且没有跨 PDU session / access / path / processing 的时序差，那么该故事应大幅降级或 KILL。

### Seed B — C2 mode/KPI transition × NTN topology transition

UAV/机器人可能从自动/waypoint 模式切到 direct/manual/emergency mode，同时网络可能发生 satellite HO、beam switch 或 feeder/gateway switch。需要核查两个独立状态机 transition 重叠时，是否存在短暂但真实的 QoS/path/state coordination gap。

**关键 kill-test**：先读 3GPP SA1/SA2 规范确认 C2 mode/QoS change 与 mobility/path switch 是否已有完整协调语义；若已覆盖，不应重复造题。

### Seed C — redundant C2 semantics × predictable LEO multi-connectivity

潜在问题是业务侧主备/冗余语义与网络侧未来链路寿命、satellite visibility、gateway transition 信息分散在不同层。LEO 的轨道可预测性可能放大或暴露这种 cross-layer information split。

**当前证据状态**：仅是待核验 seed；关于 3GPP 是否明确“网络不知道哪条是 primary/secondary”的具体规范表述，必须回到 TS 23.256 原文逐字确认后才能继续。

### Low-confidence Seed D — correlated emergency burst × limited LEO capacity

灾害/危险事件可能让多个机器人同时从常态周期上报进入高码率感知上传、人工接管或紧急控制，产生相关突发，而非独立 Poisson 流量。再与共享 LEO 容量、MEC/compute queue 交叉，可能产生 tail collapse / deadline miss。

**当前状态**：想象力较强、现实证据不足，只保留为 L0 观察，不应当作课题结论。

## Related-work collision already identified

以下方向当前应默认视为拥挤或需要非常强 residual，不能直接包装成新题：

- “无人机/无人车/机器人 + LEO + 低时延控制”本身：LEONE 已直接覆盖该场景。
- “闭环 UL + compute + DL 联合优化”：SC3 / Reflex Arc 已把 sensor 与 actuator 作为 task-oriented virtual user 联合考虑。
- “仅优化 handover”：WONS 2026 已直接研究 remote-driving LEO handover；更广泛 NTN handover 文献也很密集。
- “仅做 task offloading / service migration”：LEO edge-computing / task migration 工作已很多，不能只因 compute endpoint 移动就造题。
- “发明一个 loop-latency / freshness metric”：Age-of-Loop、VoI/timeliness、Motion-to-Motion latency 等邻近工作意味着单纯换指标很危险。

## Decision / current conclusion

- 当前对话的长期价值不是确定了一个候选课题，而是**纠正了研究问题生成方法**：必须从业务特殊机制左轴出发，再和 LEO 属性做矩阵交叉，之后逐格 collision。
- 当前讨论优先继续核查 Seed A / B / C；它们仍只是 context-level seed families。
- 不能把任何一个 seed 写成正式 KEEP、L4 或 candidate，也不能根据本文件修改两条正式科研主线。

## Open questions

1. TS 23.256 对 redundant C2 connection、primary/secondary、应用层与 5GS 所知状态的原文到底如何规定？
2. 3GPP 5GC/NTN mobility 时，多 QoS flow / 多 PDU session / redundant paths 的切换原子性和时序语义是什么？
3. C2 control-mode / KPI transition 与 handover、path switch、PDU session modification 同时发生时，标准是否已有 coordination procedure？
4. WONS 2026 / LEONE 后续工作是否已覆盖双向完整 teleoperation loop，而不是单向 VSM offload？
5. 是否存在公开 trace、开源 simulator 或 standards procedure，可以最小化验证“task-related flows across transition”是否真的产生 consequence？
6. 多机器人 correlated emergency burst 是否有真实 workload / trace / field evidence，而不是纯假设？

## Conversation Continuation State

### Current discussion state

已从宽泛的“无人系统 + LEO”想法推进到：业务机制轴、LEO 属性轴、初版交叉矩阵以及三个优先 seed family。尚未进入正式候选建立或 solution 设计。

### Last meaningful conclusion

最值得继续的方法不是泛化低时延/QoS，而是寻找：

> **业务状态机 / 多流因果关系 / 冗余语义的 transition，与 LEO 网络 topology/path transition 重叠时，是否出现现有机制组合未覆盖且可测的 consequence。**

### Unresolved questions

优先 unresolved 是 standards semantics（TS 22.125 / TS 23.256 / 5GC mobility procedures）与 closest-work collision，而不是算法设计。

### Likely next discussion step

从 Seed A、B、C 中选一个 bounded cell 深审：逐字读标准机制 → 画完整业务/网络状态机 → 找 closest related work → 做 kill-test。若均被现有工作覆盖，再回到矩阵扫描其他格子。

## Relation to formal research repositories

- 本文件仅供跨对话恢复背景。
- 若后续要把某一 seed 正式纳入 `ccf-b-research202609` 或 `ccf-b-emerging-architecture202609`，必须由对应正式仓库当前 workflow 自己完成 evidence/collision gate，并在那里记录正式状态。
- 本文件中的“Seed A/B/C/D”“优先级”“KILL test”均不是正式科研 verdict。

## Source anchors used in this discussion

- 3GPP TS 22.125 — *Unmanned Aerial System (UAS) support in 3GPP*；3GPP Portal / CR materials 可核对 UAV C2 control modes 与 KPI。
- 3GPP TS 23.256 — *Support of Uncrewed Aerial Systems (UAS) connectivity, identification and tracking; Stage 2*；redundant C2 具体语义待逐字核验。
- Xinran Fang et al., *Structured Connectivity for 6G Reflex Arc: Task-Oriented Virtual User and New Uplink-Downlink Tradeoff*, arXiv:2410.18370.
- LEONE — *Low-Latency Command and Control via LEO Satellites*, TU Dresden / LIST, 2023–2026.
- Giuseppe Avino et al., *Impact of Geometry and Satellite Mobility on Handover Strategies for Remote Driving*, IEEE/IFIP WONS 2026.

## Keywords

UAV; autonomous vehicle; robot; teleoperation; C2; remote driving; SC3; sensing-communication-computing-control; LEO; NTN; handover; feeder link; gateway switch; multi-connectivity; redundancy; QoS flow; PDU session; closed-loop control; task semantics; transition overlap; composition gap; related-work matrix
