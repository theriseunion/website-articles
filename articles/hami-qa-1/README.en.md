[简体中文](./README.md) · [All articles](../../README.en.md) · [Read on the website](https://www.theriseunion.com/en/blog/hami-qa-1.html)

# [Q&A] HAMi Frequently Asked Questions - Series 1

RiseUnion · January 01, 2025 at 08:00 (UTC+08:00)

Deep dive into HAMi's vGPU technology, explaining how virtual GPU partitioning works, its limitations, and why workloads can't request multiple vGPUs from the same physical GPU. Learn about HAMi's unique approach to GPU virtualization and resource management in enterprise environments.

![Article cover](./images/qa.jpg)

Summary: Deep dive into HAMi's vGPU technology, explaining how virtual GPU partitioning works, its limitations, and why workloads can't request multiple vGPUs from the same physical GPU. Learn about HAMi's unique approach to GPU virtualization and resource management in enterprise environments.

## Background and Challenges

In enterprise environments, GPU resources are expensive and often distributed across different teams. Efficient GPU utilization becomes a critical challenge. vGPU technology enables resource pooling and flexible allocation by logically partitioning a single GPU to serve multiple workloads, significantly improving resource utilization. However, this mechanism comes with certain misconceptions and limitations:
- A single workload cannot request multiple logical views of the same physical GPU
- vGPU partitioning is purely logical and doesn't increase physical resources
- Workloads requiring substantial GPU resources still need multiple physical GPUs

To address user questions about the HAMi platform, we're launching a series of Q&As to help users better understand and utilize HAMi effectively.

## Understanding vGPU

vGPU (Virtual GPU) uses virtualization technology to partition a physical GPU into multiple logical instances, each representing a GPU view available for workloads. This approach doesn't add hardware resources but provides virtualized resource interfaces that appear as dedicated GPUs to workloads.

In HAMi's implementation, vGPU logical partitioning is configured through the `deviceSplitCount` parameter. For example, setting `deviceSplitCount: 10` partitions a physical GPU into up to 10 logical instances, each assignable to different workloads. This logical partitioning aims to improve resource utilization rather than provide additional physical resources.

### vGPU Characteristics

- **Memory Isolation**: Each vGPU instance has its dedicated memory space, preventing memory interference between workloads
- **Compute Allocation**: GPU compute cores are shared using time-slice rotation
- **Bandwidth Sharing**: All vGPU instances share the physical GPU's PCIe bandwidth
- **Granular Monitoring**: Supports resource usage monitoring at the individual vGPU instance level

## Why Can't You Request Multiple vGPUs on a Single Card?

### Understanding vGPU Implementation

- vGPU represents a logical view of a physical GPU, not a separate hardware partition. It's designed to enable multiple workloads to share GPU compute resources, not for a single workload to occupy multiple logical views of the same physical GPU.
- In containerized environments like Kubernetes, vGPU resource allocation is workload-based.
- When a workload requests GPU resources (e.g., `nvidia.com/gpu: 2`), it requires two separate GPUs (physical or logical), not two vGPUs from the same physical GPU.

### Resource Allocation Mechanism

- The vGPU resource allocation mechanism ensures that a physical GPU's resources can only be accessed by a single workload through one logical view. This implementation maintains workload isolation and stability while preventing resource contention.
- HAMi's `deviceSplitCount` parameter is designed to support multiple concurrent workloads on a single GPU, not multiple logical views for a single workload.

### Container and Node View Consistency

- In containerized deployments, each vGPU logical view corresponds to a unique UUID that maps directly to the physical GPU's UUID. Even when multiple vGPUs are visible, they represent logical partitions of the same GPU without increasing physical GPU count.
- When a workload requests multiple vGPUs, the system interprets this as a need for multiple independent GPUs, not multiple logical instances of the same GPU.

### Design Philosophy and Objectives

- vGPU technology aims to improve GPU resource utilization through virtualization, not to provide larger resource pools for individual workloads. By partitioning a GPU into multiple logical instances, vGPU supports concurrent lightweight workloads, enhancing overall GPU cluster efficiency.
- Individual workloads can only bind to one GPU logical view, preventing simultaneous requests for multiple vGPUs from the same physical GPU.

### HAMi's Target Use Cases

- In HAMi's architecture, vGPU primarily optimizes heterogeneous compute resource scheduling and utilization. In heterogeneous GPU clusters (NVIDIA, Ascend, Cambricon, etc.), vGPU enables unified scheduling across different GPU types based on workload requirements.
- HAMi follows these vGPU design principles, requiring workloads to use logical allocations across different GPUs rather than multiple views of the same GPU.

## HAMi Platform Differentiators

Compared to other GPU virtualization solutions, HAMi offers:

### Unified Scheduling Framework
   - Support for diverse heterogeneous compute devices (NVIDIA, Ascend, Cambricon, etc.)
   - Consistent resource allocation and scheduling policies

### Flexible Resource Partitioning
   - Configurable deviceSplitCount
   - Optimized allocation strategies based on workload characteristics

### Comprehensive Monitoring
   - vGPU-level resource usage monitoring
   - Detailed performance metrics and alerting mechanisms

## Next Steps

If you have questions about other vGPU use cases or [HAMi](https://www.theriseunion.com/en/blog/project-hami.html) platform features, please contact us. We'll continue updating our Q&A series to help you use [HAMi](https://www.theriseunion.com/en/blog/project-hami.html) effectively.

If you also want to become a contributor to HAMi, please refer to: [Contributor Guide](https://github.com/Project-HAMi/HAMi/blob/master/CONTRIBUTING.md).

---

[Read this article on the RiseUnion website](https://www.theriseunion.com/en/blog/hami-qa-1.html) · [Explore more articles](https://www.theriseunion.com/en/blog.html)
