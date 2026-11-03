[English](./README.en.md) · [文章目录](../../README.md) · [官网阅读](https://www.theriseunion.com/zh/blog/hami-2-8-0.html)

# HAMi v2.8.0发布：标准化与生态完整性的双重演进

睿思智联 · 2026/02/04 08:00（北京时间）

HAMi v2.8.0正式发布！支持Kubernetes DRA标准化调度，增强天数、沐曦、昇腾等异构芯片支持，引入Scheduler高可用与Mock插件，全面提升生产环境稳定性。

![文章封面](./images/hami-logo.jpg)

**HAMi 的最新版本 v2.8.0** 正式发布！自 v2.7 发布以来，项目在**架构完整性、调度可靠性以及生态对齐层面**取得重要进展。v2.8 在 **Kubernetes 原生标准对齐、异构设备支持、生产可用性与可观测性**等方面进行了系统性增强，使 HAMi 更加适合在长期运行、对稳定性和演进路径敏感的 AI 生产集群中使用。

- [查看完整 2.8.0-Release 信息](https://github.com/Project-HAMi/HAMi/releases/tag/v2.8.0)

## 亮点速览

v2.8 版本的主要特性进展概览：

1. **标准化能力建设：** 新增对 Kubernetes DRA（Dynamic Resource Allocation）的支持，提供独立实现项目 `HAMi-DRA`，推动 HAMi 从“自定义设备调度逻辑”向 Kubernetes 原生标准接口演进。
2. **异构 GPU 生态扩展：** 天数智芯、沐曦 GPU、华为昇腾等国产芯片支持更新与增强，vLLM 兼容性问题修复，Kueue 集成完善。
3. **高可用与可靠性提升：** 引入 Leader 选举机制支持 Scheduler 高可用部署；新增 CDI 模式支持提升设备管理标准化；对齐 NVIDIA k8s-device-plugin v0.18.0 保持生态兼容。
4. **HAMi 生态体系成型：** HAMi 从单一 repo 发展为包含 `HAMi-DRA`、`mock-device-plugin`、`ascend-device-plugin`、`HAMi-WebUI` 等子项目的完整生态体系。

## 核心特性：标准化与高可用

### 1. DRA（Dynamic Resource Allocation）- 迈向 Kubernetes 原生标准

DRA 是 Kubernetes 社区正在推进的下一代设备资源声明与分配机制，旨在为 GPU/AI 加速器等设备提供 **更标准化、可组合、可扩展** 的资源管理模型。

#### 为什么 DRA 重要

传统的 Kubernetes 设备管理存在以下局限：

1. **资源声明不灵活：** 设备资源通过 `limits[nvidia.com/gpu]` 硬编码声明，无法表达复杂的资源需求（如显存、算力分离）。
2. **调度逻辑分散：** 每个设备插件需要实现自己的调度逻辑，难以统一管理。
3. **资源组合困难：** 无法表达“需要特定拓扑的多个 GPU”等复杂需求。

DRA 通过引入 `ResourceClaim` 和 `DeviceClass` 等新 API，将设备资源的声明、分配和管理标准化，使设备资源管理更加灵活和可扩展。

#### HAMi-DRA 核心特性

`HAMi-DRA` 是 HAMi 社区提供的 DRA 独立实现项目，采用 Mutating Webhook 架构，自动将传统 GPU 资源请求转换为 DRA ResourceClaim。

1. **自动资源转换：** 自动将 `nvidia.com/gpu`、`nvidia.com/gpumem`、`nvidia.com/gpucores` 等资源请求转换为 DRA `ResourceClaim`。
2. **设备选择支持：** 通过 Pod Annotation 支持按 UUID、设备类型等选择特定设备。
3. **指标监控：** 可选的 Monitor 组件，通过 Prometheus 暴露 GPU 资源使用指标。
4. **CDI 支持：** 与 Container Device Interface 集成，提供标准化的设备注入方式。

**DRA 使用示例**

提交 Pod 时，HAMi-DRA Webhook 会自动将 Pod 转换为使用 DRA ResourceClaim 的形式。

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: gpu-pod
spec:
  containers:
    - name: gpu-container
      image: nvidia/cuda:11.8.0-base-ubuntu22.04
      command: ["nvidia-smi"]
      resources:
        limits:
          nvidia.com/gpu: 2
          nvidia.com/gpumem: 4096
          nvidia.com/gpucores: 80
```

### 2. Leader 选举机制 - Scheduler 高可用能力

在大规模集群或高可用部署场景下，HAMi v2.8.0 引入了 **多 Scheduler 实例的 Leader 选举机制**，通过 Kubernetes 的 Lease 机制实现 Leader 选举，确保同一时刻只有一个 Scheduler 实例处于 Active 状态进行调度决策。

**核心优势：**

1. **避免调度冲突：** 多 Scheduler 实例并发调度可能导致资源冲突，Leader 选举确保只有一个实例进行调度。
2. **故障自动切换：** Leader 实例故障时，Standby 实例自动接管，提升系统可用性。
3. **平滑升级：** 滚动升级 Scheduler 时，新 Pod 自动成为 Leader，无需人工干预。

### 3. CDI（Container Device Interface）模式支持

HAMi v2.8.0 新增对 NVIDIA CDI 模式的支持。CDI 是 CNCF TAG 维护的容器设备接口标准，提供更标准化的设备注入方式。用户现可通过 `global.deviceListStrategy: cdi-annotations` 配置启用。

### 4. Mock Device Plugin - 开发测试利器

HAMi v2.8.0 新增 `Mock Device Plugin` 能力，为开发者和 CI/测试环境提供更低门槛的设备模拟方式。

**核心功能：**

- **虚拟设备注册：** 将虚拟设备（如 gpu-memory、gpu-cores）注册到节点。
- **多厂商支持：** 支持 NVIDIA GPU、Hygon DCU、Ascend 等多种资源类型的模拟。
- **开发测试便利：** 无需真实 GPU 硬件即可进行功能验证和开发调试。

### 5. 可观测性增强

HAMi v2.8.0 在可观测性方面进行了系统性增强，新增了构建信息指标，并移除了一些废弃指标。

- **新增指标：** `hami_build_info`，包含版本号、构建时间、Git 提交等信息。
- **优化指标：** 推荐使用 `vGPUMemoryAllocated` 和 `vGPUCoreAllocated` 替代旧的百分比指标。

## 异构生态与集成

### 1. 国产芯片支持更新

#### 天数智芯 (Iluvatar)

HAMi v2.8 对天数智芯 GPU 的支持进行了多项增强：

- **多卡调度优化：** 修复 P800 节片上 vXPU 特性的潜在问题。
- **调度失败事件优化：** 增强调度器事件输出，便于排查调度问题。
- **设备信息增强：** 在 DeviceUsage 中添加 podInfos，提升调度决策质量。

> **[致谢]** **[@qiangwei1983](https://github.com/qiangwei1983) [@Kyrie336](https://github.com/Kyrie336)** 对天数智芯支持相关特性的贡献！

#### 沐曦 (MetaX)

HAMi v2.8 对沐曦 GPU 的支持持续增强：

- **sGPU 算力/显存共享：** 支持虚拟 GPU 共享，提升资源利用率。
- **多种 QoS 模式：** 支持 BestEffort、FixedShare、BurstShare 三种模式。
- **WebUI 全面支持：** 异构指标可视化展示。

> **[致谢]** **[@Kyrie336](https://github.com/Kyrie336)** 对沐曦支持相关特性的贡献！

#### 华为昇腾 (Ascend)

HAMi 社区的 `ascend-device-plugin` 项目现已支持 vNPU（虚拟 NPU）特性，同时支持 HAMi 和 Volcano 调度器。

- **vNPU 虚拟化：** 支持华为昇腾 910 系列芯片的虚拟切分。
- **显存隔离：** 精确控制每个 vNPU 的显存使用。

> **[致谢]** **[@DSFans2014](https://github.com/DSFans2014) [@archlitchi](https://github.com/archlitchi)** 对昇腾支持及相关特性的贡献！

### 2. 上下游生态集成进展

#### Kueue 集成增强

Kueue 是 Kubernetes SIG Scheduling 维护的批量作业队列管理项目。HAMi 社区向 Kueue 贡献了增强能力，使其原生支持 HAMi 的设备资源管理与调度模型。Kueue 的 `ResourceTransformation` 现可自动转换 HAMi vGPU 资源请求，例如将 `nvidia.com/gpu` 和 `nvidia.com/gpucores` 转换为 `nvidia.com/total-gpucores` 进行统一管理。

#### vLLM 兼容性修复

HAMi v2.8 修复了多项 vLLM 相关的兼容性问题：

- 修复多卡场景下的崩溃问题。
- 修复手动指定 `CUDA_VISIBLE_DEVICES` 时的初始化失败问题。

> **[致谢]** **[@archlitchi](https://github.com/archlitchi)** 对 vLLM 兼容性修复的贡献！

## 优化与修复

### 关键问题修复与稳定性提升

v2.8 版本集中修复了一批来自真实生产环境的问题，提升了系统稳定性：

- **GPU / MIG 实例分配错误：** 修复调度器错误分配 MIG 实例的问题。
- **并发 Map 读写崩溃：** 修复并发场景下的 map 迭代和写入致命错误。
- **配额（Quota）计算错误：** 修复 ResourceQuota 计算错误。
- **设备插件卸载残留：** 修复设备插件卸载后节点仍有残留状态的问题。
- **异构设备边界异常：** 修复昆仑芯 vXPU 多卡分配 Pending 问题及沐曦 P800 相关问题。

> **[致谢]** **[@litaixun](https://github.com/litaixun) [@luohua13](https://github.com/luohua13) [@FouoF](https://github.com/FouoF) [@Shouren](https://github.com/Shouren)** 对稳定性修复的贡献！

### 版本优化与工程改进

- **节点注册逻辑优化：** 重构节点注册逻辑，提升节点管理的稳定性和可维护性。
- **Golang 升级：** 升级至 v1.25.5，获得最新的语言特性和安全修复。
- **证书热更新支持：** 支持监听并热加载证书变更，避免组件重启。
- **项目瘦身：** 删除了初期二进制文件，REPO 大小大幅缩减。

再次感谢积极参与社区贡献的每一个人，正是因为有你们，HAMi 才能不断突破与成长。

---

> 本文参考：[https://mp.weixin.qq.com/s/hvpMl4bRpMENZAbdWR2peg](https://mp.weixin.qq.com/s/hvpMl4bRpMENZAbdWR2peg), 有删减。

---

[在睿思智联官网阅读本文](https://www.theriseunion.com/zh/blog/hami-2-8-0.html) · [浏览更多文章](https://www.theriseunion.com/zh/blog.html)
