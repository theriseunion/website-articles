[简体中文](./README.md) · [All articles](../../README.en.md) · [Read on the website](https://www.theriseunion.com/en/blog/hami-2-8-0.html)

# HAMi v2.8.0 Released: Dual Evolution in Standardization and Ecosystem Completeness

RiseUnion · February 04, 2026 at 08:00 (UTC+08:00)

HAMi v2.8.0 is officially released! It brings Kubernetes DRA standardized scheduling, enhanced support for Iluvatar, MetaX, and Ascend heterogeneous chips, introduces Scheduler HA via leader election and Mock Device Plugin, and significantly improves production stability.

![Article cover](./images/hami-logo.jpg)

**HAMi v2.8.0** is officially released! Since v2.7, the project has made significant progress in **architectural completeness, scheduling reliability, and ecosystem alignment**. v2.8 delivers systematic enhancements in **Kubernetes native standard alignment, heterogeneous device support, production readiness, and observability**, making HAMi better suited for long-running AI production clusters that demand stability and a clear evolution path.

- [View the full 2.8.0 Release notes](https://github.com/Project-HAMi/HAMi/releases/tag/v2.8.0)

## Highlights at a Glance

An overview of the key features in v2.8:

1. **Standardization**: Added support for Kubernetes DRA (Dynamic Resource Allocation) with a standalone implementation project `HAMi-DRA`, advancing HAMi from custom device scheduling logic toward Kubernetes native standard interfaces.
2. **Heterogeneous GPU Ecosystem Expansion**: Updated and enhanced support for Iluvatar, MetaX GPU, and Huawei Ascend domestic chips; fixed vLLM compatibility issues; improved Kueue integration.
3. **High Availability & Reliability**: Introduced leader election for Scheduler HA deployments; added CDI mode support for standardized device management; aligned with NVIDIA k8s-device-plugin v0.18.0 for ecosystem compatibility.
4. **HAMi Ecosystem Taking Shape**: HAMi has evolved from a single repo into a complete ecosystem including `HAMi-DRA`, `mock-device-plugin`, `ascend-device-plugin`, `HAMi-WebUI`, and more.

## Core Features: Standardization & High Availability

### 1. DRA (Dynamic Resource Allocation) — Toward Kubernetes Native Standards

DRA is the next-generation device resource declaration and allocation mechanism being developed by the Kubernetes community, designed to provide a **more standardized, composable, and extensible** resource management model for GPUs and AI accelerators.

#### Why DRA Matters

Traditional Kubernetes device management has the following limitations:

1. **Inflexible resource declarations**: Device resources are hard-coded via `limits[nvidia.com/gpu]`, unable to express complex requirements (e.g., separate memory and compute).
2. **Fragmented scheduling logic**: Each device plugin must implement its own scheduling logic, making unified management difficult.
3. **Difficult resource composition**: Cannot express complex needs such as "multiple GPUs with specific topology."

DRA introduces new APIs like `ResourceClaim` and `DeviceClass` to standardize device resource declaration, allocation, and management, making it more flexible and extensible.

#### HAMi-DRA Core Features

`HAMi-DRA` is a standalone DRA implementation provided by the HAMi community. It uses a Mutating Webhook architecture to automatically convert traditional GPU resource requests into DRA ResourceClaims.

1. **Automatic resource conversion**: Automatically converts `nvidia.com/gpu`, `nvidia.com/gpumem`, `nvidia.com/gpucores`, and other resource requests into DRA `ResourceClaim` objects.
2. **Device selection**: Supports selecting specific devices by UUID, device type, and more via Pod annotations.
3. **Metrics monitoring**: An optional Monitor component exposes GPU resource usage metrics via Prometheus.
4. **CDI support**: Integrates with Container Device Interface for standardized device injection.

**DRA Usage Example**

When a Pod is submitted, the HAMi-DRA Webhook automatically transforms it to use DRA ResourceClaims.

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

### 2. Leader Election — Scheduler High Availability

For large-scale clusters and high-availability deployments, HAMi v2.8.0 introduces **leader election for multiple Scheduler instances**. Using Kubernetes Lease mechanism, it ensures that only one Scheduler instance is Active at any given time for scheduling decisions.

**Key Benefits:**

1. **Avoiding scheduling conflicts**: Concurrent scheduling by multiple Scheduler instances can cause resource conflicts — leader election ensures only one instance schedules at a time.
2. **Automatic failover**: When the Leader instance fails, a Standby instance automatically takes over, improving system availability.
3. **Smooth upgrades**: During rolling upgrades, the new Pod automatically becomes Leader without manual intervention.

### 3. CDI (Container Device Interface) Mode Support

HAMi v2.8.0 adds support for NVIDIA CDI mode. CDI is a container device interface standard maintained by CNCF TAG, providing more standardized device injection. Users can enable it via `global.deviceListStrategy: cdi-annotations`.

### 4. Mock Device Plugin — A Developer's Best Friend

HAMi v2.8.0 introduces the `Mock Device Plugin` capability, providing a low-barrier device simulation for developers and CI/test environments.

**Core Features:**

- **Virtual device registration**: Registers virtual devices (e.g., gpu-memory, gpu-cores) to nodes.
- **Multi-vendor support**: Simulates NVIDIA GPU, Hygon DCU, Ascend, and other resource types.
- **Development convenience**: Enables functional verification and debugging without real GPU hardware.

### 5. Observability Enhancements

HAMi v2.8.0 includes systematic observability improvements, with new build info metrics and removal of deprecated metrics.

- **New metric**: `hami_build_info`, including version number, build time, Git commit, and more.
- **Optimized metrics**: Recommends using `vGPUMemoryAllocated` and `vGPUCoreAllocated` over legacy percentage-based metrics.

## Heterogeneous Ecosystem & Integration

### 1. Domestic Chip Support Updates

#### Iluvatar

HAMi v2.8 includes multiple enhancements for Iluvatar GPU support:

- **Multi-GPU scheduling optimization**: Fixed potential issues with vXPU features on P800 node slices.
- **Scheduling failure event improvements**: Enhanced scheduler event output for easier troubleshooting.
- **Device info enrichment**: Added podInfos to DeviceUsage for better scheduling decisions.

> **[Thanks to]** **[@qiangwei1983](https://github.com/qiangwei1983) [@Kyrie336](https://github.com/Kyrie336)** for their contributions to Iluvatar support!

#### MetaX

HAMi v2.8 continues to enhance MetaX GPU support:

- **sGPU compute/memory sharing**: Supports virtual GPU sharing to improve resource utilization.
- **Multiple QoS modes**: Supports BestEffort, FixedShare, and BurstShare modes.
- **Full WebUI support**: Heterogeneous metrics visualization.

> **[Thanks to]** **[@Kyrie336](https://github.com/Kyrie336)** for contributions to MetaX support!

#### Huawei Ascend

The HAMi community's `ascend-device-plugin` project now supports vNPU (virtual NPU) features, compatible with both HAMi and Volcano schedulers.

- **vNPU virtualization**: Supports virtual partitioning of Huawei Ascend 910 series chips.
- **Memory isolation**: Precise control over memory usage for each vNPU.

> **[Thanks to]** **[@DSFans2014](https://github.com/DSFans2014) [@archlitchi](https://github.com/archlitchi)** for their contributions to Ascend support!

### 2. Upstream/Downstream Ecosystem Integration

#### Kueue Integration Enhancements

Kueue is a batch job queue management project maintained by Kubernetes SIG Scheduling. The HAMi community contributed enhancements to Kueue, enabling native support for HAMi's device resource management and scheduling model. Kueue's `ResourceTransformation` can now automatically convert HAMi vGPU resource requests — for example, transforming `nvidia.com/gpu` and `nvidia.com/gpucores` into `nvidia.com/total-gpucores` for unified management.

#### vLLM Compatibility Fixes

HAMi v2.8 fixed multiple vLLM compatibility issues:

- Fixed crashes in multi-GPU scenarios.
- Fixed initialization failures when manually specifying `CUDA_VISIBLE_DEVICES`.

> **[Thanks to]** **[@archlitchi](https://github.com/archlitchi)** for vLLM compatibility fixes!

## Fixes & Optimizations

### Critical Bug Fixes & Stability Improvements

v2.8 concentrated on fixing issues from real production environments, improving system stability:

- **GPU/MIG instance allocation errors**: Fixed incorrect MIG instance allocation by the scheduler.
- **Concurrent map read/write crashes**: Fixed fatal errors from concurrent map iteration and writes.
- **Quota calculation errors**: Fixed ResourceQuota calculation bugs.
- **Device plugin uninstall residue**: Fixed residual node state after device plugin removal.
- **Heterogeneous device edge cases**: Fixed KunlunXin vXPU multi-GPU allocation Pending issues and MetaX P800 related problems.

> **[Thanks to]** **[@litaixun](https://github.com/litaixun) [@luohua13](https://github.com/luohua13) [@FouoF](https://github.com/FouoF) [@Shouren](https://github.com/Shouren)** for stability fixes!

### Engineering Improvements

- **Node registration logic refactored**: Improved stability and maintainability of node management.
- **Golang upgrade**: Upgraded to v1.25.5 for the latest language features and security fixes.
- **Certificate hot-reload support**: Watches and hot-reloads certificate changes without component restarts.
- **Repository size reduction**: Removed legacy binary files, significantly reducing repo size.

A heartfelt thanks to everyone who contributed to the community. HAMi continues to grow and break new ground because of you.

---

> This article references: [https://mp.weixin.qq.com/s/hvpMl4bRpMENZAbdWR2peg](https://mp.weixin.qq.com/s/hvpMl4bRpMENZAbdWR2peg), with edits.

---

[Read this article on the RiseUnion website](https://www.theriseunion.com/en/blog/hami-2-8-0.html) · [Explore more articles](https://www.theriseunion.com/en/blog.html)
