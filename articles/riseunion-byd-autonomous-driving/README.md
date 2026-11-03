[English](./README.en.md) · [文章目录](../../README.md) · [官网阅读](https://www.theriseunion.com/zh/blog/riseunion-byd-autonomous-driving.html)

# 睿思智联驱动比亚迪智驾加速进化，让顶级算力跑出中国速度

睿思智联 · 2026/03/20 08:00（北京时间）

睿思智联通过 Rise CAMP 智算平台，为比亚迪构建高性能、高可靠的智驾大模型算力底座，通过协同调度与冷热数据分层策略，解决 AI 算力集群利用率低和存储成本高昂的痛点。

作为全球新能源汽车领域的领跑者，**比亚迪**凭借强大的垂直整合能力和技术研发实力，已连续多年产销量位居行业前列。在智能驾驶步入大模型时代的关键期，其对高性能算力的需求规模和利用效率有着极高的行业标准。


## 智驾训练的算力陷阱

随着自动驾驶任务进入端到端训练阶段，分布式任务中的"**死锁**"风险成了算力浪费的罪魁祸首。传统的裸金属模式下，一旦部分节点未就绪，全局任务就会陷入停滞，导致昂贵的 AI 算力集群产生大量算力闲置。

此外，海量的路采数据对存储性能要求极高，如果全量采用高性能全闪存，基础设施的整体成本（TCO）将变得难以承受。


## 协同调度：榨干每一分算力

针对这些挑战，睿思智联通过 [Rise CAMP](https://www.theriseunion.com/zh/product/camp.html) 智算平台来管理本次项目中的高性能 AI 算力节点，通过智能框架确保分布式训练任务高效调度。

平台基于[拓扑感知调度策略](https://www.theriseunion.com/zh/blog/brain-02-topology-aware.html)，自动感知不同硬件间的物理连接结构，将计算任务与通信路径进行最合理的匹配。同时，平台配置了任务超时自动重试功能，确保智驾大模型在长周期训练中即使偶发故障也能快速恢复，显著减少了算力损耗。

> 了解更多调度策略：[优先级感知](https://www.theriseunion.com/zh/blog/brain-01-priority-aware.html) · [负载感知](https://www.theriseunion.com/zh/blog/brain-03-load-aware.html) · [资源感知](https://www.theriseunion.com/zh/blog/brain-04-resource-aware.html)


## 冷热分层：破解存储成本难题

为了平衡性能与成本，Rise CAMP 智算平台通过**冷热数据分层管理**机制，让高并发读写的热数据运行在全闪阵列，而海量的温冷数据则自动流向成本更低的混闪阵列。这种策略在保障智驾模型训练 I/O 需求的前提下，有效控制了存储投入的整体规模。


## 业务价值与客户收益

通过构建这套稳定的智算底座，客户不仅大幅提升了 AI 算力集群的有效运行时间，更通过可视化监控工具降低了工程排错成本。这一系列技术手段缩短了端到端智驾模型的交付周期，为客户在激烈的智能驾驶市场博弈中抢占了先机。

---

[在睿思智联官网阅读本文](https://www.theriseunion.com/zh/blog/riseunion-byd-autonomous-driving.html) · [浏览更多文章](https://www.theriseunion.com/zh/blog.html)
