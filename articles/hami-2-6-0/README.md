[English](./README.en.md) · [文章目录](../../README.md) · [官网阅读](https://www.theriseunion.com/zh/blog/hami-2-6-0.html)

# HAMi v2.6.0 正式发布！国产 GPU 更强支持、稳定性全面升级

睿思智联 · 2025/06/18 08:00（北京时间）

在社区的共同努力下，HAMi v2.6.0 已正式发布！本版本在稳定性、性能与可观测性方面均带来了显著提升，尤其对燧原 Enflame、沐曦 Metax、寒武纪 Cambricon 等国产芯片的支持更加完善，将帮助更多使用国产芯片的用户轻松实现 GPU 的池化与共享，提升资源利用率。

![文章封面](./images/hami-logo.jpg)

在社区开发者和用户的共同努力下，**HAMi v2.6.0** 如期发布！本次更新聚焦稳定性、性能表现及可观测性的系统级优化，进一步强化对国产异构芯片的支持能力，显著提升 GPU 池化与共享调度的可靠性和灵活性。


## 版本亮点速览

### 更完善的国产芯片适配能力

* 支持燧原 Enflame GCU 共享模式（`gcu-share`）；
* 支持沐曦 Metax GPU / sGPU 的识别与管理；
* 修复寒武纪 Cambricon 芯片调度分配异常；

> [查看完整 2.6.0-Release 信息](https://github.com/Project-HAMi/HAMi/releases/tag/v2.6.0)

update for bugfix：
- [查看完整 2.6.1-Release 信息](https://github.com/Project-HAMi/HAMi/releases/tag/v2.6.1)

> HAMi 持续推进国产 GPU 的统一适配，努力打破厂商之间的壁垒，构建跨芯片平台的统一调度标准。


### 调度器与设备插件优化

* 调度日志增强：改进结构与可读性，便于排查调度异常及行为追踪。
* 新增 GPU 拓扑感知评分机制：当前已支持 NVIDIA 拓扑亲和性打分，为未来更复杂调度策略预埋基础。


### 更智能的部署与运维增强

* 支持 ConfigMap 注解变更自动滚动重启：通过注入 `checksum` 注解，自动更新对应组件，保障配置一致性。
* 支持 NVIDIA RuntimeClass：容器运行环境配置更灵活，满足多驱动版本并存的使用场景。
* 设备卸载逻辑优化：避免节点管理器残留设备信息引发异常状态。


### 性能与可观测性提升

* 引入 `net/http/pprof` 运行时性能分析：开发者可实时诊断系统性能瓶颈。
* vGPUmonitor 支持 MIG 模式展示：更好支持基于 MIG 的显卡资源切分方案与监控需求。


## 关键问题修复

HAMi v2.6.0 集中解决了多个来自社区反馈的重要问题：

* 修复 NVIDIA 驱动 570+ 版本下可能的调度卡顿问题；
* 修复 ComfyUI 场景中显存统计异常；
* 修正 `vgpu-devices-allocated` 注解数据不一致；
* 修复 `cuMallocAsync` 下显存统计错误；
* 避免 MIG 任务错误运行在非 MIG 节点导致调度器崩溃；
* 优化多进程任务的显存统计精度；
* 补足动态分区缺乏单卡粒度管理的问题。

> 感谢社区中每一位认真反馈问题、参与修复的用户与贡献者！


## 未来规划

HAMi 的演进不会止步于此，我们正在积极筹备 **v2.7.0** 版本，重点方向包括：

* 新增支持昆仑芯系列 GPU，进一步拓展国产生态覆盖；
* 全面适配 Kubernetes DRA 规范，强化与原生生态的集成能力；
* HAMi-WebUI 深度优化，带来更直观、高效的异构资源可视化体验。



欢迎立即体验 HAMi v2.6.0！如有任何建议或反馈，[欢迎在社区中与我们交流](https://github.com/Project-HAMi/HAMi/issues)，让 HAMi 在更多场景下发挥价值。

---

[在睿思智联官网阅读本文](https://www.theriseunion.com/zh/blog/hami-2-6-0.html) · [浏览更多文章](https://www.theriseunion.com/zh/blog.html)
