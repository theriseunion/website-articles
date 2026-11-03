[English](./README.en.md) · [文章目录](../../README.md) · [官网阅读](https://www.theriseunion.com/zh/blog/hami-2-7-0.html)

# HAMi v2.7.0发布：异构硬件生态全面升级

睿思智联 · 2025/10/09 08:00（北京时间）

HAMi v2.7.0正式发布，新增昆仑芯XPU、燧原GCU、AWS Neuron支持，实现GPU虚拟化调度与拓扑感知优化。强化调度器核心能力，支持vLLM、Xinference等AI框架，提供更稳定的GPU资源管理解决方案。

![文章封面](./images/hami-logo.jpg)

**HAMi 的最新版本 v2.7.0** 正式发布！本次更新在**硬件生态支持、核心调度器优化、关键稳定性修复以及开发者社区建设**方面取得了显著进展，旨在为用户提供更强大、更稳定、更易用的 GPU 资源管理和调度解决方案。

- [查看完整 2.7.0-Release 信息](https://github.com/Project-HAMi/HAMi/releases/tag/v2.7.0)

---

## 亮点速览

本次 HAMi v2.7.0 的发布带来了三大维度的重要升级：

1.  **硬件生态全面拓展：**
    HAMi v2.7.0 大幅扩展了异构计算硬件的支持范围。新增对 **昆仑芯 XPU**、**燧原 GCU**、以及 AWS Neuron 的完整支持，涵盖了从整卡调度、虚拟化切分到拓扑感知的全部功能。在国产芯片支持方面，**睿思智联**贡献了昆仑芯 XPU 和燧原 GCU 的完整调度方案。此外，在沐曦 MetaX 上实现了 sGPU 算力显存共享及三种 QoS 管理模式，并同步升级了 NVIDIA GPU 的拓扑感知调度能力。
2.  **调度器核心优化：**
    本次升级在核心调度能力上引入了多项机制，显著提升了系统的健壮性与可观测性。核心亮点包括引入了**调度失败事件聚合**能力，这一特性由**睿思智联**主要贡献，能将模糊的故障提示转化为直观的根因可视化。同时，扩展了 ResourceQuota 支持，解决了原生配额无法理解多 GPU 请求资源关联性的问题，并增强了对 NVIDIA 异常卡的处理机制。
3.  **应用层生态整合：**
    为了更好地服务上层 AI 应用，HAMi 增强了与主流框架的兼容性，修复并优化了 **vLLM** 兼容性，完成了 **Xinference** 平台的集成，并支持 **Volcano Dynamic MIG**，使用户能够更灵活、高效地部署和管理大模型服务。

---

## 核心特性：异构生态与调度增强

### 1. 硬件厂商生态完善

本次版本对主流异构计算硬件平台的支持进行了深度优化和扩展，为用户提供更广泛的选择和更高效的资源管理能力。

#### 支持更多国产芯片的新特性

**> 昆仑芯 XPU：新增完整 vXPU 支持**

* 混合部署与虚拟化： 实现了在同一集群中同时调度昆仑芯的整卡和 **vXPU（虚拟化切片）**，支持 1/4 卡和 1/2 卡等粒度切分，提高了资源利用率的灵活性。
* 显存自动对齐： 用户请求 vXPU 显存时，系统能自动向上对齐到最接近的硬件支持规格，简化了资源申请。
* 拓扑感知调度： 调度器能感知 XPU 卡的翼侧互联拓扑结构，为多切片任务优先选择低延迟组合，提升应用性能。
    > **[致谢]** **[@ouyangluwei163](https://github.com/ouyangluwei163)(睿思智联) [@FouoF](https://github.com/FouoF) [@archlitchi](https://github.com/archlitchi)** 对该特性的贡献！该特性得到了 **百度智能云** 和 **昆仑芯团队** 的大力支持。

[查看更多昆仑芯虚拟化的新特性](https://www.theriseunion.com/zh/blog/hami-kunlunxin-p800-support.html)
[查看 Rise VAST 率先适配昆仑芯的说明](https://www.theriseunion.com/zh/blog/kunlunxin-compatibility.html)

**> 燧原 GCU：实现 gcushare 机制的完整集成**

* GPU 共享与百分比切片： 允许多个任务容器共享同一张物理 GCU 卡，支持用户通过**百分比**（例如 25%）来申请 GCU 的算力和显存。
* 设备 UUID 选择： 支持通过 Pod 的 Annotation 来精确指定希望使用或排除的 GCU 设备。
    > **[致谢]** **[@archlitchi](https://github.com/archlitchi) [@zhaikangqi331](https://github.com/zhaikangqi331) (睿思智联)** 对该特性的贡献！

**> 沐曦 MetaX：sGPU 共享、QoS 与 MetaXLink 智能调度**

* GPU 共享（sGPU）：允许多个容器任务共享同一张物理 GPU 卡。
* 服务质量（QoS）策略：支持 BestEffort、FixedShare 和 BurstShare 三种资源服务等级。
* 拓扑感知调度：基于 MetaXLink 增强拓扑感知，提升多卡任务性能。

> **[致谢]** **[@Kyrie336](https://github.com/Kyrie336) [@darker-thanBlack](https://github.com/darker-thanBlack)** 对以上特性的贡献，以及 **沐曦团队** 的深度参与支持。

#### 支持更多国外芯片

**> AWS Neuron：支持设备级和核心级资源分配与拓扑感知**

* 核心级共享： 允许用户以单个 NeuronCore 为最小单位来申请资源，极大提高了 Inferentia/Trainium 加速器的利用率。
* 拓扑感知调度： 尽量将多核心任务调度到网络延迟最低、通信效率最高的 NeuronCore 组合上。

**> NVIDIA GPU 拓扑调度升级**

* 新增 GPU 拓扑感知调度能力，确保多卡任务优先调度到通过 NVLink 高速互联的 GPU 组合上，最大化提升 HPC 和 AI 大模型训练效率。

> **[致谢]** **[@lengrongfu](https://github.com/lengrongfu) [@fyp711](https://github.com/fyp711)** 对该特性的贡献！

### 2. 调度器核心优化

#### 调度失败事件聚合

* 提升可观测性：解决了 Kubernetes 中 Pod 仅返回**“no available node”**的模糊提示问题。调度器现在会在筛选失败时，**统计每类节点拒绝原因**，并将 **“CardInsufficientMemory”、“NumaNotFit”** 等标准化标签连同节点数量写入 **FilteringFailed 事件**。这一机制能直观呈现资源不足、拓扑不匹配等真实瓶颈。
* 事件链路增强：事件系统增强了成功和失败两条诊断链路。若筛选阶段未能找到候选节点，系统将**按原因聚合**写入 **Warning 事件**；若找到合适节点，则在 **Normal 事件**里同时列出命中的节点与得分。这些信息配合 v4/v5 分级日志格式，能极大地帮助用户定位问题。

> **[致谢]** **[@Wangmin362](https://github.com/Wangmin362) (睿思智联)** 对该特性的贡献！

#### 扩展的 ResourceQuota 支持

* 智能关联计算： 解决了原生 ResourceQuota 无法理解多 GPU 请求中**资源关联性**（如 GPU 数量与显存的关系）的痛点，确保总显存/算力配额计算精确反映真实资源消耗。
* 动态实时计算： 对于按百分比申请的动态资源请求，HAMi 能够在调度决策时，根据实际分配到的物理 GPU 规格**动态计算**出确切的占用量并计入配额。

> **[致谢]** **[@FouoF](https://github.com/FouoF)** 对该特性的贡献！

### 3. 应用层生态整合

* vLLM 兼容性增强： 修复了显存异步申请、上下文管理等多个稳定性问题。同时，vLLM 社区已原生支持 HAMi 资源变量，进一步降低集成成本。
 > **[致谢]** **[@andresd95](https://github.com/andresd95)** 对该特性的贡献！
* Xinference 集成： 在 Helm Chart 中原生支持 HAMi vGPU，使轻量模型可安全共享 GPU，显著提升整体利用率。
> **[致谢]** **[@calvin0327](https://github.com/calvin0327)** 对该特性的贡献！
* Volcano Dynamic MIG： 支持 Volcano v1.12 提供的动态 MIG 切分与调度能力，可根据用户申请实时选择合适的 MIG 实例大小，提升资源利用率。
> **[致谢]** **[@sailorvii](https://github.com/sailorvii) [@archlitchi](https://github.com/archlitchi)** 对该特性的贡献！

---

## 优化与修复

* **HAMi 调度器：** 新增 NVIDIA 异常卡处理、统一设备接口重构、以及昇腾 910 调度策略更新等。
* **HAMi-core（底层引擎）：** 增强接口兼容性（新增 `cuMemcpy2D` hook）、修复 NVML 接口空指针解引用、解决多进程 GPU 利用率统计重复累加问题等。
* **WebUI：** 核心功能增强，全面支持沐曦 MetaX GPU 的监控指标展示，可视化效果更直观。


## 社区发展：新成员与角色任命

为进一步推动 HAMi 社区的发展与治理，社区迎来了新的贡献者与角色任命：
* Member：[@fyp711](https://github.com/fyp711)；
* Reviewer:  [@Shouren](https://github.com/Shouren)、[@ouyangluwei163](https://github.com/ouyangluwei163)、[@lengrongfu](https://github.com/lengrongfu)、[@chaunceyjiang](https://github.com/chaunceyjiang)；
* volcano-vgpu-device-plugin Reviewer & Approver: [@SataQiu](https://github.com/SataQiu)；
* HAMi Website Owner: [@windsonsea](https://github.com/windsonsea)

感谢上述成员的长期贡献与付出，期待他们在新角色中继续推动社区的成长与繁荣。


## 未来展望

HAMi 将继续致力于提升 GPU 资源管理的智能化和自动化水平，未来的重点方向包括：

1.  **DRA（Dynamic Resource Allocation）：** 完善对 Kubernetes DRA 的支持，以实现细粒度、灵活的异构资源动态分配。
2.  **WebUI 持续优化：** 增加更多高级功能和丰富的可视化图表。
3.  **生态系统扩展：** 持续与更多硬件厂商和 AI 框架进行深度集成。

感谢所有社区成员和贡献者的大力支持！期待与大家共同建设更强大的 HAMi！

> 本文参考：https://mp.weixin.qq.com/s/557IU6blBBcV4_nIcFeAqQ, 有删减。

---

[在睿思智联官网阅读本文](https://www.theriseunion.com/zh/blog/hami-2-7-0.html) · [浏览更多文章](https://www.theriseunion.com/zh/blog.html)
