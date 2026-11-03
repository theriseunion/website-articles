[简体中文](./README.md) · [All articles](../../README.en.md) · [Read on the website](https://www.theriseunion.com/en/blog/hami-2-9-0.html)

# HAMi v2.9.0 Released: Ascend HAMi-core, DRA, and Scheduler Ecosystem Advances

RiseUnion · May 19, 2026 at 17:18 (UTC+08:00)

HAMi v2.9.0 introduces userspace Ascend partitioning with HAMi-core, brings NVIDIA HAMi-DRA to a usable stage, aligns Volcano vGPU with v0.19 and CDI, and improves heterogeneous devices, observability, debugging tools, and production stability.

![Article cover](./images/hami-logo.jpg)

HAMi v2.9.0 was published on May 19, 2026. Rather than centering on one device or one scheduling algorithm, the release advances three tracks together: extending HAMi-core fine-grained sharing to Ascend devices, bringing the Kubernetes DRA-based lightweight path to a usable stage, and expanding the surrounding Volcano, CDI, heterogeneous-device, and operations ecosystem.

- [Official HAMi v2.9.0 Release Notes](https://github.com/Project-HAMi/HAMi/releases/tag/v2.9.0)
- [Full v2.8.0-to-v2.9.0 comparison](https://github.com/Project-HAMi/HAMi/compare/v2.8.0...v2.9.0)
- [Official Project-HAMi v2.9.0 announcement](https://github.com/Project-HAMi/website/blob/main/blog/hami-v2-9-0-release/index.md)

## Release at a Glance

| Area | What changed in v2.9.0 |
| --- | --- |
| Ascend virtualization | Ascend HAMi-core mode, 910C SuperPod module-pair allocation, vNPU core resources, and multi-device requests |
| Kubernetes-native interfaces | NVIDIA HAMi-DRA reached a usable stage and moved to v0.2.0 |
| Scheduler ecosystem | Volcano vGPU Device Plugin aligned with v0.19 and gained CDI device injection |
| Heterogeneous devices | Vastai support arrived, alongside fixes for MIG and several vendor-specific edge cases |
| Observability | ServiceMonitor resources, configurable metrics binding, device-type labels, and high-cardinality control |
| Development and operations | HAMi Skills, local-deploy support, and refreshed HAMi-core benchmarks |
| Production reliability | Stronger quota checks, NodeLock behavior, leader election, init container handling, and device health checks |

## Ascend HAMi-core: Moving Beyond Fixed Templates

One of the defining changes in v2.9.0 is HAMi-core mode for Ascend devices. Instead of relying only on fixed hardware partition templates, HAMi-core intercepts and governs device-runtime calls in userspace so that a workload can request memory and compute shares separately. This provides a more flexible model for mixed model sizes, denser inference services, and pooled accelerator environments.

### 910C SuperPod and vNPU Core Resources

The release cycle added module-pair-aware allocation for Ascend 910C SuperPod environments, helping multi-device jobs avoid unsuitable module combinations. The implementation is tracked in [PR #1610](https://github.com/Project-HAMi/HAMi/pull/1610).

The `hami-vnpu-core` path was also completed across resource naming, node filtering, and multi-device requests:

- [PR #1771](https://github.com/Project-HAMi/HAMi/pull/1771) added the Ascend core resource name required by `hami-vnpu-core` virtualization.
- [PR #1804](https://github.com/Project-HAMi/HAMi/pull/1804) added the device-specific Ascend `-core` resources.
- [PR #1812](https://github.com/Project-HAMi/HAMi/pull/1812) enabled scheduler filtering from the node's `hami-vnpu-core` annotation.
- [PR #1837](https://github.com/Project-HAMi/HAMi/pull/1837) added multi-device requests when `hami-vnpu-core` is enabled.

In v2.9.0, a Pod that needs soft partitioning must still declare `huawei.com/vnpu-mode: hami-core`. A Pod without that annotation continues down the template-based vNPU path. Operators should therefore validate that node mode, Pod annotations, and resource names agree before an upgrade; otherwise a workload can remain Pending because no compatible node is available.

### HAMi-core Performance and Reproducible Measurement

v2.9.0 updates the HAMi-core performance data and the tools used to reproduce it. The official repository's [`benchmarks`](https://github.com/Project-HAMi/HAMi/tree/v2.9.0/benchmarks) directory contains test images, deployment manifests, and report tooling. These assets matter more than a single published number because operators can repeat the comparison with their own driver, model, and accelerator combinations.

## HAMi-DRA and CDI: Aligning with Kubernetes Device Standards

### NVIDIA HAMi-DRA v0.2.0

The official Release Notes list NVIDIA HAMi-DRA as a core v2.9.0 capability. HAMi-DRA uses the Kubernetes Dynamic Resource Allocation model to place device declaration, allocation, and lifecycle management behind standardized APIs while retaining a familiar workload-facing request model where practical.

[PR #1845](https://github.com/Project-HAMi/HAMi/pull/1845) moved HAMi-DRA to v0.2.0. The implementation lives in the separate [Project-HAMi/HAMi-DRA](https://github.com/Project-HAMi/HAMi-DRA) repository. For clusters using a custom scheduler, DRA is more than a renamed resource: it offers a standard boundary between scheduling decisions and device drivers.

An evaluation should confirm four things:

1. Whether the Kubernetes version supports the target DRA APIs.
2. How existing Pod templates, ResourceClaims, and DeviceClasses will migrate.
3. Where responsibilities sit across the scheduler, admission webhook, and device driver.
4. Whether pre-existing Device Plugin workloads continue running through upgrade and rollback.

### Volcano vGPU v0.19 and CDI

v2.9.0 aligns the related Volcano vGPU Device Plugin with v0.19 and adds CDI (Container Device Interface) to its device-injection path. CDI supplies a normalized device description, reducing the coupling between a device plugin and runtime-specific container parameters.

The release also updated the NVIDIA device-plugin and container-runtime modules in [PR #1731](https://github.com/Project-HAMi/HAMi/pull/1731), while [PR #1826](https://github.com/Project-HAMi/HAMi/pull/1826) fixed MIG allocation in CDI mode. MIG users should still exercise their actual profiles, runtime, and CDI configuration before production rollout.

## Broader Heterogeneous-Device Support

### Vastai Devices

[PR #1645](https://github.com/Project-HAMi/HAMi/pull/1645) added scheduling support for Vastai devices, covering both full-card and Die-level allocation. The latter requires the scheduler to understand internal device structure rather than counting devices alone, so it can reduce unnecessary cross-Die communication.

This follows HAMi's shared-device abstraction: candidate-node and device scoring use a common interface, while each vendor implementation owns resource names, topology, and allocation rules. [PR #1715](https://github.com/Project-HAMi/HAMi/pull/1715) subsequently corrected the fit-loop direction in the Vastai integration.

### Ascend, MIG, and Device Edge Cases

Several fixes address conditions that surface during hardware faults, node restarts, or configuration drift:

- NVIDIA health-check handshakes on Linux Kernel 6.17, in [PR #1810](https://github.com/Project-HAMi/HAMi/pull/1810).
- MIG discovery and CDI allocation, in [PR #1819](https://github.com/Project-HAMi/HAMi/pull/1819) and [PR #1826](https://github.com/Project-HAMi/HAMi/pull/1826).
- Ascend node-device deserialization error handling, in [PR #1831](https://github.com/Project-HAMi/HAMi/pull/1831).
- The A100 80GB device template, in [PR #1847](https://github.com/Project-HAMi/HAMi/pull/1847).

These items are less visible than a new device integration, but they directly affect whether scheduling recovers cleanly after state changes.

## Observability: From Metrics Exposure to Operability

v2.9.0 groups several changes around Prometheus integration and metric quality:

- [PR #1613](https://github.com/Project-HAMi/HAMi/pull/1613) adds `--metrics-bind-address` to vGPUmonitor.
- [PR #1614](https://github.com/Project-HAMi/HAMi/pull/1614) and [PR #1633](https://github.com/Project-HAMi/HAMi/pull/1633) add ServiceMonitor resources for the Helm chart and device plugin.
- [PR #1612](https://github.com/Project-HAMi/HAMi/pull/1612) adds a device-type label to metrics.
- [PR #1644](https://github.com/Project-HAMi/HAMi/pull/1644) aligns Prometheus metric and label names with common conventions.
- [PR #1628](https://github.com/Project-HAMi/HAMi/pull/1628) addresses a cardinality explosion in the per-container device-memory description metric.

Clusters with existing Grafana dashboards and alert rules should compare metric names, labels, and value semantics before upgrading. A ServiceMonitor simplifies discovery, but a naming change can also invalidate an old query until the dashboard is updated.

## HAMi Skills: Reusable Troubleshooting Workflows

The release organizes debugging and metric analysis into HAMi Skills that developers and operators can reuse:

- [PR #1654](https://github.com/Project-HAMi/HAMi/pull/1654) adds `k8s-debug-gpu-pod` for inspecting GPU Pods, requests, and runtime state.
- [PR #1755](https://github.com/Project-HAMi/HAMi/pull/1755) adds a vGPU metrics summarizer skill.
- [PR #1842](https://github.com/Project-HAMi/HAMi/pull/1842) refreshes the skills and adds repository-level development guidance.

The repository's [`skill`](https://github.com/Project-HAMi/HAMi/tree/v2.9.0/skill) directory is a useful entry point for automated diagnosis. HAMi Skills do not replace logs or monitoring; they turn common inspection steps into a more repeatable operating contract.

## Quota, Security, and Reliability Work

### Earlier Quota Feedback

[PR #1605](https://github.com/Project-HAMi/HAMi/pull/1605) adds GPU resource-quota validation to the admission webhook. This moves obvious over-quota feedback earlier in the Pod lifecycle, rather than waiting for repeated scheduling attempts. Multi-GPU requests that combine device count, memory, and compute still deserve explicit validation against the cluster's ResourceQuota policy.

### NodeLock and Cache Behavior at Scale

[PR #1663](https://github.com/Project-HAMi/HAMi/pull/1663) uses exponential backoff and listers to improve NodeLock scalability and reduce API Server pressure during concurrent binds. The release also includes protections around leader-election nil values, zero-value score division, cache retention for terminating Pods, and recovery from stale `Deleted_` handshake state.

### Multi-Container Pods and Init Containers

[PR #1650](https://github.com/Project-HAMi/HAMi/pull/1650) fixes device allocation for Pods containing multiple application containers and init containers. This item is about delivering the chosen device allocation to the relevant containers. v2.10.0 later changes the accounting semantics for init container requests; the two changes should not be treated as the same feature.

### vLLM Compatibility

The official Release Notes call out a fix for tensor-parallel initialization with vLLM versions newer than 0.18. Multi-GPU inference users should still test their own vLLM, CUDA, driver, and model combination rather than assuming every upstream pairing is covered by one version-level fix.

## Upgrade Checklist

1. **Ascend**: align `huawei.com/vnpu-mode`, node mode, and the `-core` / `-memory` resource names.
2. **DRA**: confirm the Kubernetes DRA API level and HAMi-DRA v0.2.0 compatibility, with an independent rollback path.
3. **Volcano, CDI, and MIG**: validate discovery, injection, and restart recovery on the same runtime and profiles used in production.
4. **Monitoring**: compare ServiceMonitor resources, metrics binding, label names, and Grafana queries.
5. **Large clusters**: observe NodeLock, leader election, API Server QPS, and scheduler failure events.
6. **vLLM**: regress tensor parallelism, multi-container Pods, and explicitly selected devices.

## Summary

HAMi v2.9.0 advances device virtualization, Kubernetes-native device interfaces, and day-two operations at the same time. Ascend HAMi-core broadens software partitioning, HAMi-DRA v0.2.0 and CDI move device management toward standard interfaces, and Vastai support, observability, HAMi Skills, and reliability fixes make the project easier to operate across more real clusters.

Refer to the [HAMi v2.9.0 Release Notes](https://github.com/Project-HAMi/HAMi/releases/tag/v2.9.0) for the complete feature, fix, and contributor list.

## HAMi Release Series

- [HAMi v2.8.0: Standardization and Ecosystem Completeness](https://www.theriseunion.com/en/blog/hami-2-8-0.html)
- [HAMi v2.9.0: Ascend HAMi-core, DRA, and Scheduler Ecosystem Advances](https://www.theriseunion.com/en/blog/hami-2-9-0.html)
- [HAMi v2.10.0: Flexible MIG, Composable Scheduling, and Broader Accelerator Support](https://www.theriseunion.com/en/blog/hami-2-10-0.html)

---

[Read this article on the RiseUnion website](https://www.theriseunion.com/en/blog/hami-2-9-0.html) · [Explore more articles](https://www.theriseunion.com/en/blog.html)
