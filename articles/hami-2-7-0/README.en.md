[简体中文](./README.md) · [All articles](../../README.en.md) · [Read on the website](https://www.theriseunion.com/en/blog/hami-2-7-0.html)

# HAMi v2.7.0: Comprehensive Heterogeneous Hardware Support

RiseUnion · January 01, 2025 at 08:00 (UTC+08:00)

HAMi v2.7.0 delivers enhanced heterogeneous computing support for Kunlunxin XPU, Enflame GCU, and AWS Neuron. Features improved GPU virtualization scheduling, topology-aware optimization, and seamless integration with vLLM and Xinference frameworks for enterprise AI workloads.

![Article cover](./images/hami-logo.jpg)

**HAMi v2.7.0** is officially released! This major update delivers significant enhancements across **hardware ecosystem support, core scheduler optimization, critical stability improvements, and developer community growth**, providing users with a more powerful, stable, and user-friendly GPU resource management and scheduling solution.

- [View complete 2.7.0 Release information](https://github.com/Project-HAMi/HAMi/releases/tag/v2.7.0)

---

## Key Highlights

HAMi v2.7.0 introduces substantial improvements across three critical dimensions:

1. **Comprehensive Hardware Ecosystem Expansion:**
   HAMi v2.7.0 significantly broadens support for heterogeneous computing hardware. The release adds complete support for **Kunlunxin XPU**, **Enflame GCU**, and **AWS Neuron**, covering full-card scheduling, virtualization partitioning, and topology-aware capabilities. For domestic chip support, **RiseUnion** contributed comprehensive scheduling solutions for Kunlunxin XPU and Enflame GCU. Additionally, sGPU compute and memory sharing with three QoS management modes have been implemented on Metax MetaX, while NVIDIA GPU topology-aware scheduling capabilities have been upgraded.

2. **Core Scheduler Optimization:**
   This release introduces multiple mechanisms in core scheduling capabilities, significantly enhancing system robustness and observability. Key highlights include **scheduling failure event aggregation** capabilities, primarily contributed by **RiseUnion**, which transforms ambiguous failure messages into intuitive root-cause visualization. Additionally, extended ResourceQuota support addresses the limitation where native quotas cannot understand resource correlation in multi-GPU requests, while enhancing NVIDIA anomaly card handling mechanisms.

3. **Application Layer Ecosystem Integration:**
   To better serve upper-layer AI applications, HAMi has enhanced compatibility with mainstream frameworks, fixed and optimized **vLLM** compatibility, completed **Xinference** platform integration, and supports **Volcano Dynamic MIG**, enabling users to deploy and manage large model services more flexibly and efficiently.

---

## Core Features: Heterogeneous Ecosystem and Scheduling Enhancements

### 1. Hardware Vendor Ecosystem Enhancement

This release provides deep optimization and expansion of support for mainstream heterogeneous computing hardware platforms, offering users broader choices and more efficient resource management capabilities.

#### New Features for Additional Domestic Chips

**> Kunlunxin XPU: Complete vXPU Support**

* Mixed Deployment and Virtualization: Enables simultaneous scheduling of Kunlunxin full cards and **vXPU (virtualized slices)** within the same cluster, supporting granular partitioning like 1/4 and 1/2 cards, improving resource utilization flexibility.
* Automatic Memory Alignment: When users request vXPU memory, the system automatically aligns upward to the nearest hardware-supported specification, simplifying resource requests.
* Topology-Aware Scheduling: The scheduler can perceive XPU card wing-side interconnect topology, prioritizing low-latency combinations for multi-slice tasks to enhance application performance.

> **[Acknowledgments]** **[@ouyangluwei163](https://github.com/ouyangluwei163)(RiseUnion) [@FouoF](https://github.com/FouoF) [@archlitchi](https://github.com/archlitchi)** for their contributions! This feature received strong support from **Baidu Intelligent Cloud** and **Kunlunxin team**.

[Learn more about Kunlunxin virtualization features](https://www.theriseunion.com/en/blog/hami-kunlunxin-p800-support.html)
[View Rise VAST's pioneering Kunlunxin adaptation](https://www.theriseunion.com/en/blog/kunlunxin-compatibility.html)

**> Enflame GCU: Complete Integration of gcushare Mechanism**

* GPU Sharing and Percentage Slicing: Allows multiple task containers to share the same physical GCU card, supporting users to request GCU compute and memory through **percentage** (e.g., 25%).
* Device UUID Selection: Supports precise specification of desired or excluded GCU devices through Pod annotations.

> **[Acknowledgments]** **[@archlitchi](https://github.com/archlitchi) [@zhaikangqi331](https://github.com/zhaikangqi331) (RiseUnion)** for their contributions!

**> Metax MetaX: sGPU Sharing, QoS, and MetaXLink Intelligent Scheduling**

* GPU Sharing (sGPU): Enables multiple container tasks to share the same physical GPU card.
* Quality of Service (QoS) Strategies: Supports three resource service levels: BestEffort, FixedShare, and BurstShare.
* Topology-Aware Scheduling: Enhanced topology awareness based on MetaXLink, improving multi-card task performance.

> **[Acknowledgments]** **[@Kyrie336](https://github.com/Kyrie336) [@darker-thanBlack](https://github.com/darker-thanBlack)** for their contributions, along with deep participation and support from the **Metax team**.

#### Support for Additional International Chips

**> AWS Neuron: Device-Level and Core-Level Resource Allocation with Topology Awareness**

* Core-Level Sharing: Allows users to request resources with single NeuronCore as the minimum unit, greatly improving utilization of Inferentia/Trainium accelerators.
* Topology-Aware Scheduling: Prioritizes scheduling multi-core tasks to NeuronCore combinations with the lowest network latency and highest communication efficiency.

**> NVIDIA GPU Topology Scheduling Upgrade**

* Added GPU topology-aware scheduling capabilities, ensuring multi-card tasks are prioritized for scheduling to GPU combinations connected via high-speed NVLink, maximizing HPC and AI large model training efficiency.

> **[Acknowledgments]** **[@lengrongfu](https://github.com/lengrongfu) [@fyp711](https://github.com/fyp711)** for their contributions!

### 2. Core Scheduler Optimization

#### Scheduling Failure Event Aggregation

* Enhanced Observability: Resolves the issue where Kubernetes Pods only return vague **"no available node"** messages. The scheduler now **aggregates rejection reasons by node type** during filtering failures, writing standardized labels like **"CardInsufficientMemory"** and **"NumaNotFit"** along with node counts into **FilteringFailed events**. This mechanism provides intuitive visualization of real bottlenecks like resource insufficiency and topology mismatches.
* Enhanced Event Chain: The event system strengthens both success and failure diagnostic chains. If the filtering phase fails to find candidate nodes, the system will **aggregate by reason** and write **Warning events**; if suitable nodes are found, both hit nodes and scores are listed in **Normal events**. This information, combined with v4/v5 hierarchical log formats, greatly helps users identify issues.

> **[Acknowledgments]** **[@Wangmin362](https://github.com/Wangmin362) (RiseUnion)** for their contribution!

#### Extended ResourceQuota Support

* Intelligent Correlation Calculation: Addresses the pain point where native ResourceQuota cannot understand **resource correlation** (such as the relationship between GPU count and memory) in multi-GPU requests, ensuring total memory/compute quota calculations accurately reflect real resource consumption.
* Dynamic Real-Time Calculation: For dynamic resource requests by percentage, HAMi can **dynamically calculate** exact usage based on actually allocated physical GPU specifications during scheduling decisions and include it in quotas.

> **[Acknowledgments]** **[@FouoF](https://github.com/FouoF)** for their contribution!

### 3. Application Layer Ecosystem Integration

* vLLM Compatibility Enhancement: Fixed multiple stability issues including asynchronous memory requests and context management. Additionally, the vLLM community now natively supports HAMi resource variables, further reducing integration costs.

> **[Acknowledgments]** **[@andresd95](https://github.com/andresd95)** for their contribution!

* Xinference Integration: Native support for HAMi vGPU in Helm Chart, enabling lightweight models to safely share GPUs, significantly improving overall utilization.

> **[Acknowledgments]** **[@calvin0327](https://github.com/calvin0327)** for their contribution!

* Volcano Dynamic MIG: Supports dynamic MIG partitioning and scheduling capabilities provided by Volcano v1.12, allowing real-time selection of appropriate MIG instance sizes based on user requests, improving resource utilization.

> **[Acknowledgments]** **[@sailorvii](https://github.com/sailorvii) [@archlitchi](https://github.com/archlitchi)** for their contributions!

---

## Optimizations and Fixes

* **HAMi Scheduler:** Added NVIDIA anomaly card handling, unified device interface refactoring, and Ascend 910 scheduling strategy updates.
* **HAMi-core (underlying engine):** Enhanced interface compatibility (added `cuMemcpy2D` hook), fixed NVML interface null pointer dereference, resolved multi-process GPU utilization statistics duplicate accumulation issues.
* **WebUI:** Core functionality enhancements, comprehensive support for Metax MetaX GPU monitoring metrics display with more intuitive visualization.

## Community Growth: New Members and Role Appointments

To further promote HAMi community development and governance, the community welcomes new contributors and role appointments:

* Member: [@fyp711](https://github.com/fyp711)
* Reviewer: [@Shouren](https://github.com/Shouren), [@ouyangluwei163](https://github.com/ouyangluwei163), [@lengrongfu](https://github.com/lengrongfu), [@chaunceyjiang](https://github.com/chaunceyjiang)
* volcano-vgpu-device-plugin Reviewer & Approver: [@SataQiu](https://github.com/SataQiu)
* HAMi Website Owner: [@windsonsea](https://github.com/windsonsea)

We thank the above members for their long-term contributions and dedication, and look forward to their continued efforts in their new roles to drive community growth and prosperity.

## Future Outlook

HAMi will continue to focus on enhancing the intelligence and automation of GPU resource management. Key future directions include:

1. **DRA (Dynamic Resource Allocation):** Complete support for Kubernetes DRA to achieve fine-grained, flexible dynamic allocation of heterogeneous resources.
2. **WebUI Continuous Optimization:** Adding more advanced features and rich visualization charts.
3. **Ecosystem Expansion:** Continued deep integration with more hardware vendors and AI frameworks.

Thank you to all community members and contributors for your tremendous support! We look forward to building an even more powerful HAMi together!

> This article references: https://mp.weixin.qq.com/s/557IU6blBBcV4_nIcFeAqQ, with edits.

---

[Read this article on the RiseUnion website](https://www.theriseunion.com/en/blog/hami-2-7-0.html) · [Explore more articles](https://www.theriseunion.com/en/blog.html)
