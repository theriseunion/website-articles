[English](./README.en.md) · [文章目录](../../README.md) · [官网阅读](https://www.theriseunion.com/zh/blog/hami-2-10-0.html)

# HAMi v2.10.0 发布：Flexible MIG、可组合调度与异构生态扩展

睿思智联 · 2026/08/21 14:31（北京时间）

HAMi v2.10.0 正式发布：带来 Flexible MIG、mutex 与可组合调度策略、PodGroup 和 init container 资源计量，扩展 AMD MI300X、Biren、Ascend DRA 与异构昇腾模式，并持续改进 HAMi-core 和生产可靠性。

![文章封面](./images/hami-logo.jpg)

HAMi v2.10.0 已于 2026 年 8 月 21 日发布。这个版本把重点放在更细致的设备切分、更灵活的调度策略和更广泛的异构设备支持上：NVIDIA MIG 可以按工作负载动态创建和回收实例，GPU 调度策略可以组合，PodGroup 与 init container 的资源语义得到补强，AMD MI300X、壁仞（Biren）和昇腾设备的接入范围也继续扩大。

- [HAMi v2.10.0 官方 Release Notes](https://github.com/Project-HAMi/HAMi/releases/tag/v2.10.0)
- [v2.9.0 到 v2.10.0 完整变更](https://github.com/Project-HAMi/HAMi/compare/v2.9.0...v2.10.0)
- [Project-HAMi 官方 v2.10.0 中文发布公告](https://github.com/Project-HAMi/website/blob/main/i18n/zh/docusaurus-plugin-content-blog/hami-v2-10-0-release/index.md)

本文按官方发布记录梳理主要能力、使用边界和升级注意事项。Release Notes 中“Support autoscaling”所链接的是设计文档，不是已经交付的运行时实现，本文会把它作为后续集成方向说明。

## 版本重点

| 方向         | v2.10.0 的主要变化                                       |
| ---------- | --------------------------------------------------- |
| 安装与架构      | 主 Helm Chart 移除 DRA 组件，DRA 驱动改为独立演进                 |
| 调度语义       | 补齐 init container 资源计量、PodGroup、mutex、策略组合和 NUMA 对齐 |
| NVIDIA MIG | Flexible MIG 支持实例的动态创建与回收                           |
| 异构设备       | 增加 AMD MI300X、Biren，扩展 Ascend DRA 与异构昇腾调度           |
| HAMi-core  | 修正 vNPU HAMi-core 模式下的显存对齐行为，构建镜像切换到 UBI8           |
| 稳定性        | 优化设备注册 handshake、异常设备检查、并发缓存和插件容错                   |

## 主 Chart 与 DRA 解耦

[PR #2038](https://github.com/Project-HAMi/HAMi/pull/2038) 从 HAMi 主 Helm Chart 中移除了 DRA 组件。这个调整不是放弃 DRA，而是把不同设备的 DRA 驱动从 HAMi 主发布单元中拆开，让核心调度组件、传统 Device Plugin 路径和独立 DRA 驱动能够按各自节奏演进。

v2.10.0 同时把 [Ascend DRA](https://github.com/Project-HAMi/ascend-dra-driver) 列为正式的 DRA 方向。对于准备采用 Kubernetes Dynamic Resource Allocation 的集群，安装边界因此变得更清楚：HAMi 主 Chart 不再隐式承担所有 DRA 部署工作，管理员需要单独选择、安装并验证与设备类型相匹配的驱动。

升级前应特别核对以下项目：

1. 现有 values 文件是否仍包含已经移除的 DRA 配置。
2. ResourceClass、ResourceClaim 或 DeviceClass 是否由独立驱动正确管理。
3. 调度器、驱动和 Kubernetes DRA API 版本是否互相兼容。
4. 回滚时，传统 Device Plugin 工作负载和 DRA 工作负载是否有各自独立的恢复路径。

## 调度能力：从单一策略走向可组合决策

### init container 资源计量

[PR #1773](https://github.com/Project-HAMi/HAMi/pull/1773) 修正了 init container 的设备资源计量。Kubernetes 对 init container 的资源语义与普通业务容器不同：init container 按阶段顺序运行，Pod 的有效资源需求不能简单地把所有 init container 与所有应用容器相加。HAMi 调度器现在按照更符合 Pod 生命周期的方式评估设备需求，避免过度预留，也避免低估启动阶段所需的 GPU 资源。

配套的 [设计说明 PR #2064](https://github.com/Project-HAMi/HAMi/pull/2064) 和 [ResourceQuota 交互文档 PR #2535](https://github.com/Project-HAMi/HAMi/pull/2535) 进一步解释了资源计量边界。使用 GPU 初始化模型、预热缓存或执行启动期转换的工作负载，升级后应重新检查 ResourceQuota、调度事件和最终分配结果。

### PodGroup 与 Gang Scheduling

分布式训练和多副本并行任务常常要求一组 Pod 同时获得资源。若只启动其中一部分，先运行的 Pod 会占住设备，却无法完成有效计算。[PR #2066](https://github.com/Project-HAMi/HAMi/pull/2066) 为 PodGroup 成员在 Bind 阶段增加 NodeLock 重试，[PR #2206](https://github.com/Project-HAMi/HAMi/pull/2206) 则适配 Kubernetes 1.36 及以后版本的 gangScheduling feature gate。

这些变化降低了批量绑定期间短暂锁冲突造成整体任务失败的概率。实际部署仍需确认所使用的 PodGroup CRD、调度器配置、最小成员数和超时策略一致，尤其要避免不同调度器同时接管同一组工作负载。

### Flexible MIG

[PR #2378](https://github.com/Project-HAMi/HAMi/pull/2378) 引入 Flexible MIG，使 HAMi 能够根据工作负载请求动态创建和回收 MIG 实例，而不只是消费节点上预先固定好的实例。对于请求形态随业务变化的推理集群，这可以降低长期保留不合适 MIG profile 带来的碎片。

Flexible MIG 涉及 GPU 级别的重配置，因此应在维护窗口、驱动版本、MIG profile 兼容性和正在运行的工作负载之间建立清晰约束。动态能力并不意味着所有 profile 可以在任意时刻无中断切换；生产启用前应验证实例创建、设备发现、CDI 注入、Pod 终止后的回收，以及节点重启后的状态重建。

### mutex 策略与策略组合

[PR #2011](https://github.com/Project-HAMi/HAMi/pull/2011) 增加新的 `mutex` GPU 调度策略，用于表达不能同时落在同一设备上的互斥关系。它适合处理资源数值看似满足，但运行时、隔离要求或任务组合不允许共置的场景。

[PR #2621](https://github.com/Project-HAMi/HAMi/pull/2621) 进一步允许通过逗号组合多种 `gpu-scheduler-policy`。这使调度决策不再只能在 binpack、spread 或 mutex 之间三选一，而可以把多个约束按配置组合。策略可组合也意味着配置语义更复杂：升级时必须用代表性 Pod 验证组合顺序、设备评分、互斥条件和无可用设备时的错误信息。

### Handshake 与 NUMA 对齐

设备插件通过节点注解与调度器进行注册 handshake。[PR #2052](https://github.com/Project-HAMi/HAMi/pull/2052) 在设备重新注册时移除旧的 `deleted` 注解，减少失效状态残留导致的调度阻塞。对于频繁重启设备插件、节点恢复或驱动重载的环境，这类状态清理会直接影响节点重新进入可调度状态的速度。

[PR #2065](https://github.com/Project-HAMi/HAMi/pull/2065) 为 vGPU 副本增加可选的 NUMA topology 信息。调度器和 kubelet 可以据此把 CPU 与 GPU 的拓扑关系纳入决策，降低跨 NUMA 访问。该能力是 opt-in，启用前需要确认节点拓扑数据、CPU Manager 策略和设备插件暴露的信息一致。

### Cluster Autoscaler：当前是设计方向

 [PR #2528](https://github.com/Project-HAMi/HAMi/pull/2528) 是 Cluster Autoscaler scale-up simulation 设计，讨论如何让自动扩容模拟理解 HAMi 的模板节点、设备资源和调度判断。

因此，v2.10.0 在这里交付的是设计和集成方向，不应被理解为仅安装 HAMi v2.10.0 就已经获得完整可用的自动扩容运行时能力。计划接入 Cluster Autoscaler 的团队仍需要跟踪后续实现、云厂商节点模板、扩容模拟接口和版本兼容性。

## 异构设备支持

### AMD MI300X

[PR #2290](https://github.com/Project-HAMi/HAMi/pull/2290) 增加 AMD vGPU 支持，官方发布重点指向 AMD MI300X 系列。相关设计由 [PR #2067](https://github.com/Project-HAMi/HAMi/pull/2067) 更新，并在更早的 [AMD Instinct vGPU 设计 PR #1985](https://github.com/Project-HAMi/HAMi/pull/1985) 中记录总体方案。

引入新设备类型不仅是增加一个资源名，还涉及设备发现、显存和算力计量、容器注入、健康状态、调度评分以及监控。AMD 集群应以实际 ROCm、驱动、节点镜像和容器运行时组合做验证，并确认工作负载请求使用的资源字段与 HAMi 配置一致。

### 壁仞（Biren）

[PR #1711](https://github.com/Project-HAMi/HAMi/pull/1711) 为 Biren 设备增加支持，[PR #1979](https://github.com/Project-HAMi/HAMi/pull/1979) 在路线图中将其标记为完成。该集成沿用 HAMi 的统一设备接口，把厂商设备发现和资源语义接入同一调度框架。

部署时仍应按所使用的 Biren 驱动和设备插件版本确认整卡、切分、健康检查和监控能力。HAMi 的“支持”表示已有项目实现，并不替代厂商兼容矩阵或生产容量测试。

### Ascend DRA 与异构昇腾模式

除独立的 [Ascend DRA Driver](https://github.com/Project-HAMi/ascend-dra-driver) 外，v2.10.0 还允许未指定模式的昇腾 Pod 在 vNPU-template 节点和 HAMi-core 节点之间调度。[PR #2035](https://github.com/Project-HAMi/HAMi/pull/2035) 实现了这一兼容路径，减少应用模板与节点切分模式之间的硬绑定。

这并不代表两种模式的所有资源语义完全相同。集群管理员仍需确认显存、core、设备数量、节点注解和运行时挂载方式，并为必须固定到某一模式的工作负载保留显式约束。

### 910C Mock 模板

[PR #2005](https://github.com/Project-HAMi/HAMi/pull/2005) 为 mock-device-plugin 增加 910C vNPU 模板 `vir05_1c_16g` 与 `vir10_3c_32g`。Mock 模板使开发者可以在没有真实设备的环境里验证资源名称、模板匹配和调度流程，但不能替代真机上的驱动、性能和故障恢复测试。

## HAMi-core 与构建兼容性

### vNPU HAMi-core 不再被模板显存错误裁剪

官方 Release Notes 将“HAMi-core mode for vnpu doesn't need to align device memory to template”列为设备支持改进。[PR #2696](https://github.com/Project-HAMi/HAMi/pull/2696) 修正了 HAMi-core 模式下自动显存裁剪的问题，使软切分请求不再被错误对齐到固定 vNPU 模板的显存规格。

这一修复很重要，因为 HAMi-core 的目标就是允许显存与算力按请求细粒度分配。如果仍套用模板边界，就会降低可用密度或让请求得到与声明不一致的资源。升级后应回归验证小显存请求、多个 Pod 共卡、任务退出后的资源释放和设备监控数值。

### UBI8 编译镜像

[PR #1958](https://github.com/Project-HAMi/HAMi/pull/1958) 让 HAMi 构建与 HAMi-core 编译镜像保持一致，并切换到 UBI8 路径，以扩大对不同 GLIBC 环境的兼容范围。[PR #2102](https://github.com/Project-HAMi/HAMi/pull/2102) 随后移除了文档中过时的 GLIBC 上限说明。

这类构建变化主要影响二进制在不同基础镜像和宿主环境中的可运行性。使用自定义镜像、离线镜像仓库或自行编译 HAMi-core 的团队，应重新验证构建参数、动态链接库、镜像架构和供应链扫描结果。

## 睿思智联贡献

本次版本中，睿思智联团队的贡献直接来自国产卡实际部署与持续运行：既补齐了昇腾 910C 的 vNPU 模板和异构模式调度，也修复了资源请求无法正确交付、节点模式检查被绕过、容器内找不到 `npu-smi`、插件健康检查 panic、并发缓存竞态、Leader 更新失败等一批真实运行问题。这些工作横跨调度器、设备插件、节点配置与容器运行时，形成了从硬件接入到生产稳定性的完整落地链路。

### @ouyangluwei163：昇腾模板、模式兼容与插件部署

- [HAMi Issue #2004](https://github.com/Project-HAMi/HAMi/issues/2004) → [HAMi PR #2005](https://github.com/Project-HAMi/HAMi/pull/2005)：为 910C 补充 `vir05_1c_16g` 和 `vir10_3c_32g` vNPU mock 模板。

- [HAMi Issue #2034](https://github.com/Project-HAMi/HAMi/issues/2034) → [HAMi PR #2035](https://github.com/Project-HAMi/HAMi/pull/2035)：允许未指定模式的昇腾 Pod 同时匹配 vNPU-template 与 HAMi-core 节点。

- [ascend-device-plugin Issue #94](https://github.com/Project-HAMi/ascend-device-plugin/issues/94) → [PR #95](https://github.com/Project-HAMi/ascend-device-plugin/pull/95)：在 HAMi-core 等模式下自动开启 device-share 配置。

- [ascend-device-plugin Issue #103](https://github.com/Project-HAMi/ascend-device-plugin/issues/103) → [PR #104](https://github.com/Project-HAMi/ascend-device-plugin/pull/104)：挂载 `/usr/local/sbin`，确保容器内能够找到 `npu-smi`。

- [ascend-device-plugin Issue #105](https://github.com/Project-HAMi/ascend-device-plugin/issues/105) → [PR #106](https://github.com/Project-HAMi/ascend-device-plugin/pull/106)：让未指定模式的 Pod 跟随节点当前的 vNPU-template 或 HAMi-core 模式。

这些修改连接了 HAMi 调度器、Ascend 插件、节点模式和容器运行时配置，直接支撑 v2.10 的 910C 新模板与异构昇腾模式。

### @Wangmin362：稳定性、配置边界与设备兼容

- [HAMi Issue #2025](https://github.com/Project-HAMi/HAMi/issues/2025) → [HAMi PR #2026](https://github.com/Project-HAMi/HAMi/pull/2026)：硬切分 vNPU 请求的 `-core` 会被调度器计量、却无法交付给容器；修复方案是在硬切分模式拒绝该请求，并修正文案拼写。

- [HAMi Issue #2028](https://github.com/Project-HAMi/HAMi/issues/2028) → [HAMi PR #2029](https://github.com/Project-HAMi/HAMi/pull/2029)：整卡或不申请显存的 HAMi-core 请求会绕过节点软切分/硬切分模式检查；修复后这些请求同样受节点模式约束。

- [HAMi Issue #2030](https://github.com/Project-HAMi/HAMi/issues/2030)：记录 HAMi-core vNPU 显存处理相关讨论。

- [HAMi PR #2043](https://github.com/Project-HAMi/HAMi/pull/2043)：避免健康状态为空时触发 panic。

- [HAMi PR #2045](https://github.com/Project-HAMi/HAMi/pull/2045)：处理 `use` / `nouse` GPU UUID 为空的配置。

- [HAMi PR #2050](https://github.com/Project-HAMi/HAMi/pull/2050)：修正仅启用 Vastai 或 Biren 时的设备配置。

- [HAMi PR #2053](https://github.com/Project-HAMi/HAMi/pull/2053)：避免只启用寒武纪设备时首个容器不含 MLU 请求导致 panic。

- [HAMi PR #2068](https://github.com/Project-HAMi/HAMi/pull/2068)：修正并发读取 node cache 的竞态。

- [HAMi PR #2075](https://github.com/Project-HAMi/HAMi/pull/2075)：Leader 标签更新短暂失败时不再直接退出进程。

- [HAMi PR #2156](https://github.com/Project-HAMi/HAMi/pull/2156)：把 `gpumem-percentage: 0` 按未设置处理。

- [HAMi PR #2393](https://github.com/Project-HAMi/HAMi/pull/2393)：对不支持基于事件健康检查的设备跳过相应检查。

- [ascend-device-plugin PR #100](https://github.com/Project-HAMi/ascend-device-plugin/pull/100)：补充 `VDeviceCount` 相关处理。

- [ascend-device-plugin PR #127](https://github.com/Project-HAMi/ascend-device-plugin/pull/127)：兼容 v2.9 之前的设备配置布局。

- [mock-device-plugin PR #18](https://github.com/Project-HAMi/mock-device-plugin/pull/18)：同时支持新旧 vNPU 配置，并补充 Ascend、Hygon core 资源模拟。

这些贡献主要集中在生产环境最容易暴露的边界：空值、只启用单一厂商、并发缓存、Leader 更新、配置向后兼容和不具备统一健康检查能力的设备。

### 从昇腾 910C 支持到国产算力生产化

对国产卡的支持，真正的难点通常不在于增加一个资源名称，而在于打通设备发现、资源建模、调度决策、容器注入、节点模式、健康检查和版本升级。昇腾 910C 新模板解决的是资源规格能否被准确表达；vNPU-template 与 HAMi-core 的兼容调度解决的是任务能否匹配正确节点；device-share 自动配置和 `npu-smi` 挂载解决的是设备插件能否在真实环境中完成初始化和管理。

同样重要的是，这一版本修复了大量只有在复杂集群和长期运行中才容易出现的 Bug：硬切分资源被计量却无法交付、整卡请求绕过节点模式、单一国产设备厂商配置无法启动、空健康状态导致 panic、node cache 并发读取竞态、Leader 标签短暂更新失败导致进程退出，以及不同版本设备配置不兼容。这些问题看似分散，实际共同决定了国产卡集群能否稳定调度、故障恢复和持续升级。

从 910C 能力接入到上述生产问题闭环，这批可验证的社区贡献集中体现了睿思智联丰富的国产算力落地经验：不仅让国产卡能够被 Kubernetes 发现和申请，也让它能够在异构资源池中被正确调度、稳定运行并平滑演进。对客户而言，这意味着睿思智联的工程能力覆盖国产算力从“可用”走向“好用、稳定可运营”的关键环节。

## 生产稳定性与升级关注点

除上述重点外，v2.10.0 还包含大量并发、配额、设备发现、监控和安全修复。由于官方自动生成的完整变更范围较大，升级评估不应只看功能标题，还应以当前使用的设备类型和集群组件筛选 [完整 Release Notes](https://github.com/Project-HAMi/HAMi/releases/tag/v2.10.0)。

建议至少执行以下检查：

1. **Chart 与 DRA**：确认 DRA 已从主 Chart 拆出，独立驱动、RBAC 和资源对象可以完整安装及回滚。
2. **init container**：验证顺序型初始化任务、普通容器和 ResourceQuota 的资源计量符合预期。
3. **PodGroup**：在锁冲突、部分节点不可用和超时条件下测试整组任务的调度与清理。
4. **Flexible MIG**：覆盖实例创建、回收、节点重启、CDI 注入和 profile 不可满足的失败路径。
5. **策略组合**：用真实 Pod 验证 binpack、spread、mutex 组合和每 Pod 权重，不只检查配置是否能被解析。
6. **NUMA**：确认拓扑信息、CPU Manager 和设备插件设置一致，并对比跨 NUMA 性能。
7. **异构设备**：分别对 AMD MI300X、Biren 和 Ascend 执行设备发现、资源申请、健康检查、监控和故障恢复测试。
8. **HAMi-core**：验证小显存请求不会再被模板错误裁剪，并检查 UBI8 构建产物在目标 GLIBC 环境中的加载。
9. **可观测性**：观察设备注册 handshake、调度失败事件、节点缓存、Leader 选举与配额指标。
10. **自动扩容**：把 Cluster Autoscaler 条目视为设计方向，待具体实现和兼容矩阵确认后再制定上线方案。

## 总结

HAMi v2.10.0 把项目从“支持共享设备”继续推向“可按不同硬件和业务约束组合调度”。Flexible MIG 改善 NVIDIA 资源切分的动态性，mutex 与策略组合增强调度表达力，init container、PodGroup、handshake 和 NUMA 工作补足生产语义，AMD MI300X、Biren、Ascend DRA 与异构昇腾模式则扩大了统一调度框架的覆盖范围。

完整功能、修复和全体贡献者清单请参考：[HAMi v2.10.0 Release Notes](https://github.com/Project-HAMi/HAMi/releases/tag/v2.10.0)。

## HAMi 版本文章

- [HAMi v2.8.0：标准化与生态完整性的双重演进](https://www.theriseunion.com/zh/blog/hami-2-8-0.html)
- [HAMi v2.9.0：昇腾 HAMi-core、DRA 与调度生态同步演进](https://www.theriseunion.com/zh/blog/hami-2-9-0.html)
- [HAMi v2.10.0：Flexible MIG、可组合调度与异构生态扩展](https://www.theriseunion.com/zh/blog/hami-2-10-0.html)

---

[在睿思智联官网阅读本文](https://www.theriseunion.com/zh/blog/hami-2-10-0.html) · [浏览更多文章](https://www.theriseunion.com/zh/blog.html)
