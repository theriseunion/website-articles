[English](./README.en.md) · [文章目录](../../README.md) · [官网阅读](https://www.theriseunion.com/zh/blog/hami-2-9-0.html)

# HAMi v2.9.0 发布：昇腾 HAMi-core、DRA 与调度生态同步演进

睿思智联 · 2026/05/19 17:18（北京时间）

HAMi v2.9.0 正式发布：新增昇腾 HAMi-core 用户态切分，推动 NVIDIA HAMi-DRA 进入可用阶段，同步 Volcano vGPU v0.19 与 CDI，并增强异构设备、可观测性、调试工具和生产稳定性。

![文章封面](./images/hami-logo.jpg)

HAMi v2.9.0 已于 2026 年 5 月 19 日发布。这个版本同时推进了三条主线：把 HAMi-core 的细粒度共享能力带到昇腾设备，让基于 Kubernetes DRA 的轻量方案真正进入可使用阶段，并继续扩展 Volcano、CDI、异构芯片与运维工具生态。

- [HAMi v2.9.0 官方 Release Notes](https://github.com/Project-HAMi/HAMi/releases/tag/v2.9.0)
- [v2.8.0 到 v2.9.0 完整变更](https://github.com/Project-HAMi/HAMi/compare/v2.8.0...v2.9.0)
- [Project-HAMi 官方 v2.9.0 发布公告](https://github.com/Project-HAMi/website/blob/main/i18n/zh/docusaurus-plugin-content-blog/hami-v2-9-0-release/index.md)

## 版本重点

| 方向              | v2.9.0 的主要变化                                                              |
| --------------- | ------------------------------------------------------------------------- |
| 昇腾虚拟化           | 新增 Ascend HAMi-core 模式，并补充 910C SuperPod module-pair、vNPU core 资源和多设备请求能力 |
| Kubernetes 原生接口 | NVIDIA HAMi-DRA 进入可用阶段，版本提升到 v0.2.0                                       |
| 调度生态            | Volcano vGPU Device Plugin 对齐 v0.19，并支持 CDI 设备注入                          |
| 异构设备            | 新增 Vastai 设备管理，并持续修正 MIG、昆仑芯、天数智芯等边界问题                                    |
| 可观测性            | 增加 ServiceMonitor、指标监听地址与设备类型标签，治理高基数指标                                   |
| 开发与运维           | 提供 HAMi Skills、local-deploy 和更新后的 HAMi-core benchmark                     |
| 生产可靠性           | 加强配额校验、NodeLock、Leader 选举、init container、多容器与设备健康检查                       |

## 昇腾 HAMi-core：从固定模板走向用户态细粒度共享

v2.9.0 的核心变化之一，是为昇腾设备增加 HAMi-core 模式。与固定规格的硬件模板切分不同，HAMi-core 在用户态对设备运行时调用进行拦截和管控，使工作负载可以分别申请显存和算力比例。对于大小模型混部、推理服务密度提升和资源池化场景，这种方式提供了更灵活的切分粒度。

### 910C SuperPod 与 vNPU core 资源

版本周期内，HAMi 增加了昇腾 910C SuperPod 环境的 module-pair 分配能力，避免多卡任务跨越不合适的模块组合。相关实现见 [PR #1610](https://github.com/Project-HAMi/HAMi/pull/1610)。

围绕 `hami-vnpu-core`，项目还逐步补齐了资源名称、节点筛选和多设备请求：

- [PR #1771](https://github.com/Project-HAMi/HAMi/pull/1771) 增加 Ascend core 资源名称，用于支持 `hami-vnpu-core` 虚拟化。
- [PR #1804](https://github.com/Project-HAMi/HAMi/pull/1804) 增加不同 Ascend 设备对应的 `-core` 资源。
- [PR #1812](https://github.com/Project-HAMi/HAMi/pull/1812) 让调度器能够按节点的 `hami-vnpu-core` 注解过滤候选节点。
- [PR #1837](https://github.com/Project-HAMi/HAMi/pull/1837) 补充启用 `hami-vnpu-core` 后的多设备请求支持。

在 v2.9.0 中，使用软切分模式的 Pod 仍需要显式声明 `huawei.com/vnpu-mode: hami-core`。没有声明时，工作负载继续按模板化 vNPU 模式处理。升级时应确认节点模式、Pod 注解和资源名称保持一致，避免任务因为找不到匹配节点而 Pending。

### HAMi-core 性能与可复现实验

v2.9.0 同步更新了 HAMi-core 基准数据和可执行的 benchmark 工具。官方仓库的 [`benchmarks`](https://github.com/Project-HAMi/HAMi/tree/v2.9.0/benchmarks) 目录包含测试镜像、部署清单和报告生成工具。与只展示单一结果相比，这些材料更适合用于升级前复测：用户可以在自己的驱动、模型和卡型组合上对比原生设备插件与 HAMi-core 的影响。

## HAMi-DRA 与 CDI：对齐 Kubernetes 设备管理标准

### NVIDIA HAMi-DRA v0.2.0

HAMi-DRA 采用 Kubernetes Dynamic Resource Allocation 模型，把设备声明、分配和生命周期管理放进标准接口中，同时尽量保留用户熟悉的资源申请方式。

[PR #1845](https://github.com/Project-HAMi/HAMi/pull/1845) 将 HAMi-DRA 版本提升至 v0.2.0。对应实现位于独立的 [Project-HAMi/HAMi-DRA](https://github.com/Project-HAMi/HAMi-DRA) 仓库。对采用自定义调度器的集群来说，DRA 的意义不是简单替换一个资源名，而是为调度器与设备驱动之间提供更标准的连接面。

在评估 HAMi-DRA 时，建议重点确认：

1. Kubernetes 版本是否满足目标 DRA API 的要求。
2. 现有 Pod 模板、ResourceClaim 与设备类如何迁移。
3. 自定义调度器、Admission Webhook 和设备驱动的职责边界。
4. 升级与回滚时，旧 Device Plugin 工作负载能否继续运行。

### Volcano vGPU v0.19 与 CDI

v2.9.0 将关联的 Volcano vGPU Device Plugin 同步到 v0.19，并把 CDI（Container Device Interface）纳入设备注入路径。CDI 以规范化的设备描述完成容器注入，减少设备插件与具体容器运行时参数之间的耦合。

项目同时更新了 NVIDIA device plugin 与 container runtime 模块，见 [PR #1731](https://github.com/Project-HAMi/HAMi/pull/1731)，并修复了 MIG 在 CDI 模式下的分配失败，见 [PR #1826](https://github.com/Project-HAMi/HAMi/pull/1826)。使用 MIG 的集群升级前仍应按实际 profile、容器运行时和 CDI 配置做回归测试。

## 异构设备支持继续扩展

### Vastai 设备

[PR #1645](https://github.com/Project-HAMi/HAMi/pull/1645) 为 Vastai 设备增加调度支持，覆盖整卡与按 Die 分配的场景。与只按设备数量分配相比，调度器还需要识别设备内部结构，才能降低不必要的跨 Die 通信。

这项支持延续了 HAMi 的统一设备抽象思路：调度器使用同一套设备接口处理候选节点和设备评分，而具体资源名称、设备拓扑及分配规则由厂商实现负责。版本中也修复了 Vastai 适配循环方向错误，见 [PR #1715](https://github.com/Project-HAMi/HAMi/pull/1715)。

### 昇腾、MIG 与其他设备边界

除新增设备外，v2.9.0 还处理了多种真实集群中的边界情况：

- Linux Kernel 6.17 下 NVIDIA 健康检查握手边界，见 [PR #1810](https://github.com/Project-HAMi/HAMi/pull/1810)。
- MIG 设备发现和 CDI 分配，见 [PR #1819](https://github.com/Project-HAMi/HAMi/pull/1819) 与 [PR #1826](https://github.com/Project-HAMi/HAMi/pull/1826)。
- Ascend 节点设备信息反序列化错误检查，见 [PR #1831](https://github.com/Project-HAMi/HAMi/pull/1831)。
- A100 80GB 设备模板修正，见 [PR #1847](https://github.com/Project-HAMi/HAMi/pull/1847)。

这些修复不一定在功能列表中最醒目，却直接影响设备异常、节点重启或配置漂移时能否稳定恢复调度。

## 可观测性：从“能看到”走向可运营

v2.9.0 对 Prometheus 接入和指标质量做了成组更新：

- [PR #1613](https://github.com/Project-HAMi/HAMi/pull/1613) 为 vGPUmonitor 增加 `--metrics-bind-address`，允许明确控制监听地址。
- [PR #1614](https://github.com/Project-HAMi/HAMi/pull/1614) 和 [PR #1633](https://github.com/Project-HAMi/HAMi/pull/1633) 分别为 Helm Chart 与 device plugin 增加 ServiceMonitor。
- [PR #1612](https://github.com/Project-HAMi/HAMi/pull/1612) 在指标中增加设备类型标签。
- [PR #1644](https://github.com/Project-HAMi/HAMi/pull/1644) 统一 Prometheus 指标和标签命名。
- [PR #1628](https://github.com/Project-HAMi/HAMi/pull/1628) 处理容器显存描述指标的基数膨胀问题，避免监控系统因为标签组合过多而承压。

对于已经建立 Grafana 仪表盘和告警规则的集群，升级前需要比对指标名称、标签和值域。ServiceMonitor 能简化抓取配置，但指标命名调整也可能让旧查询暂时失效。

## HAMi Skills：把排障知识变成可复用工具

v2.9.0 把调试和指标分析能力整理为 HAMi Skills，面向开发者和运维人员提供可复用的排障流程：

- [PR #1654](https://github.com/Project-HAMi/HAMi/pull/1654) 增加 `k8s-debug-gpu-pod`，用于检查 GPU Pod、资源请求与运行状态。
- [PR #1755](https://github.com/Project-HAMi/HAMi/pull/1755) 增加 vGPU 指标汇总 Skill。
- [PR #1842](https://github.com/Project-HAMi/HAMi/pull/1842) 更新 Skills，并为代码库补充开发指导。

仓库中当前的 [`skill`](https://github.com/Project-HAMi/HAMi/tree/v2.9.0/skill) 目录可以作为后续自动化排障的入口。它并不替代监控与日志，而是把常见检查步骤组织成更稳定的执行约定。

## 配额、安全与稳定性修复

### 更早发现资源配额问题

[PR #1605](https://github.com/Project-HAMi/HAMi/pull/1605) 把 GPU 资源配额检查加入 Webhook。这样可以在 Pod 进入调度链路前识别明显超额的请求，减少反复调度后才失败的情况。对多 GPU、显存和算力同时申请的场景，应结合现有 ResourceQuota 策略验证计算方式。

### 大规模集群中的锁与缓存

[PR #1663](https://github.com/Project-HAMi/HAMi/pull/1663) 使用指数退避和 lister 优化 NodeLock，降低大量并发绑定时对 API Server 的压力。其他修复包括 Leader 选举空指针保护、调度评分除零保护、终止中 Pod 的缓存保留，以及 stale `Deleted_` 握手状态下的调度恢复。

### 多容器与 init container

[PR #1650](https://github.com/Project-HAMi/HAMi/pull/1650) 修复包含多个业务容器与 init container 时的设备分配。该修复关注的是设备结果能否正确传递到每个相关容器；v2.10.0 又进一步调整了 init container 的资源计量语义，两者不应混为同一项能力。

### vLLM 兼容性

官方 Release Notes 明确列出 vLLM 0.18 之后 tensor parallelism 初始化失败的修复。运行多卡推理的用户应使用自己的 vLLM、CUDA、驱动和模型组合复测，不应仅凭版本号推断所有上游组合都已覆盖。

## 升级前检查

1. **昇腾用户**：确认 `huawei.com/vnpu-mode`、节点模式和 `-core` / `-memory` 资源配置一致。
2. **DRA 用户**：先核对 Kubernetes DRA API 与 HAMi-DRA v0.2.0 的兼容范围，并准备独立回滚路径。
3. **Volcano/CDI/MIG 用户**：在与生产相同的运行时和 MIG profile 上验证设备发现、注入和重启恢复。
4. **监控用户**：检查 ServiceMonitor、指标监听地址、标签名称和 Grafana 查询。
5. **大规模集群**：观察 NodeLock、Leader 选举、API Server QPS 以及调度失败事件。
6. **vLLM 用户**：对 tensor parallelism、多容器和显式设备环境变量场景执行回归。

## 总结

HAMi v2.9.0 的价值，在于把设备虚拟化、Kubernetes 原生接口和日常运维向前推进了一步。Ascend HAMi-core 扩展了软切分覆盖面，HAMi-DRA v0.2.0 和 CDI 推动设备管理接口标准化，Vastai、监控、HAMi Skills 与稳定性修复则让项目在更多真实集群中更容易部署和维护。

完整功能、修复和贡献者清单请参考：[HAMi v2.9.0 Release Notes](https://github.com/Project-HAMi/HAMi/releases/tag/v2.9.0)。

## HAMi 版本文章

- [HAMi v2.8.0：标准化与生态完整性的双重演进](https://www.theriseunion.com/zh/blog/hami-2-8-0.html)
- [HAMi v2.9.0：昇腾 HAMi-core、DRA 与调度生态同步演进](https://www.theriseunion.com/zh/blog/hami-2-9-0.html)
- [HAMi v2.10.0：Flexible MIG、可组合调度与异构生态扩展](https://www.theriseunion.com/zh/blog/hami-2-10-0.html)

---

[在睿思智联官网阅读本文](https://www.theriseunion.com/zh/blog/hami-2-9-0.html) · [浏览更多文章](https://www.theriseunion.com/zh/blog.html)
