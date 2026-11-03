[简体中文](./README.md) · [All articles](../../README.en.md) · [Read on the website](https://www.theriseunion.com/en/blog/hami-2-10-0.html)

# HAMi v2.10.0 Released: Flexible MIG, Composable Scheduling, and Broader Accelerator Support

RiseUnion · August 21, 2026 at 14:31 (UTC+08:00)

HAMi v2.10.0 adds Flexible MIG, mutex and composable scheduling policies, PodGroup and init container accounting, expands AMD MI300X, Biren, Ascend DRA, and heterogeneous Ascend support, and continues improving HAMi-core and production reliability.

![Article cover](./images/hami-logo.jpg)

HAMi v2.10.0 was published on August 21, 2026. The release focuses on finer device partitioning, more expressive scheduling, and broader heterogeneous accelerator support. NVIDIA MIG instances can be created and reclaimed dynamically, GPU policies can be composed, PodGroup and init container semantics are stronger, and the device matrix now includes AMD MI300X, Biren, and additional Ascend deployment modes.

- [Official HAMi v2.10.0 Release Notes](https://github.com/Project-HAMi/HAMi/releases/tag/v2.10.0)
- [Full v2.9.0-to-v2.10.0 comparison](https://github.com/Project-HAMi/HAMi/compare/v2.9.0...v2.10.0)
- [Official Project-HAMi v2.10.0 announcement](https://github.com/Project-HAMi/website/blob/main/blog/hami-v2-10-0-release/index.md)

This article follows the official release record and separates shipped behavior from design work. In particular, the Release Notes item labeled “Support autoscaling” points to a design document. It is therefore described below as a Cluster Autoscaler integration direction, not as a complete runtime capability delivered simply by installing v2.10.0.

## Release at a Glance

| Area | What changed in v2.10.0 |
| --- | --- |
| Installation and architecture | DRA components were removed from the main Helm chart so drivers can evolve independently |
| Scheduling semantics | Correct init container accounting, PodGroup behavior, mutex policy, policy composition, and optional NUMA alignment |
| NVIDIA MIG | Flexible MIG can dynamically allocate and deallocate instances |
| Heterogeneous devices | AMD MI300X and Biren support, Ascend DRA, and mixed Ascend scheduling modes |
| HAMi-core | Correct vNPU memory behavior in HAMi-core mode and a UBI8-based compilation path |
| Reliability | Cleaner registration handshakes, safer device checks, concurrency fixes, and plugin fault tolerance |

## Separating the Main Chart from DRA Drivers

[PR #2038](https://github.com/Project-HAMi/HAMi/pull/2038) removes DRA components from HAMi's main Helm chart. This is not a retreat from Dynamic Resource Allocation. It separates device-specific DRA drivers from the core HAMi release unit so the scheduler, the established Device Plugin path, and independent DRA implementations can move at appropriate cadences.

The same release highlights [Ascend DRA](https://github.com/Project-HAMi/ascend-dra-driver) as an official DRA direction. For clusters adopting Kubernetes Dynamic Resource Allocation, the installation boundary is now clearer: the main HAMi chart no longer implicitly installs every DRA component, and operators must select, install, version, and validate a driver that matches their accelerator.

An upgrade review should answer four questions:

1. Does the current values file still contain DRA options that no longer belong to the main chart?
2. Which independent component now owns ResourceClass, ResourceClaim, or DeviceClass objects?
3. Are the Kubernetes DRA API level, HAMi scheduler, and selected driver mutually compatible?
4. Can legacy Device Plugin workloads and DRA workloads be restored independently during rollback?

## Scheduling: From One Policy to Composable Decisions

### Correct init container Accounting

[PR #1773](https://github.com/Project-HAMi/HAMi/pull/1773) corrects accelerator resource accounting for an init container. Kubernetes gives init containers different semantics from ordinary application containers: they execute in sequence, and the Pod's effective requirement is not the simple sum of every init and application container. HAMi now evaluates device demand in a way that better reflects the Pod lifecycle, avoiding both unnecessary reservation and underestimation of startup-time GPU needs.

The accompanying [design PR #2064](https://github.com/Project-HAMi/HAMi/pull/2064) and [ResourceQuota documentation PR #2535](https://github.com/Project-HAMi/HAMi/pull/2535) explain the accounting boundary. Workloads that use accelerators to load a model, warm a cache, or transform data during startup should be tested again after upgrade. Operators should compare the requested resources, ResourceQuota results, scheduler events, and the devices actually delivered to application containers.

### PodGroup and Gang Scheduling

Distributed training and multi-replica parallel jobs often need a group of Pods to acquire resources together. Starting only part of the group can reserve expensive devices without producing useful work. [PR #2066](https://github.com/Project-HAMi/HAMi/pull/2066) retries NodeLock during Bind for PodGroup members, while [PR #2206](https://github.com/Project-HAMi/HAMi/pull/2206) adapts HAMi to the gangScheduling feature gate used by Kubernetes 1.36 and later.

Together, these changes reduce the chance that a transient lock conflict during a grouped bind causes the whole job to fail. A production deployment still needs consistent PodGroup CRDs, scheduler configuration, minimum-member counts, and timeout policies. It should also ensure that two schedulers are not both trying to control the same workload group.

### Flexible MIG

[PR #2378](https://github.com/Project-HAMi/HAMi/pull/2378) introduces Flexible MIG by allowing HAMi to dynamically allocate and deallocate MIG instances instead of consuming only a fixed set of instances created in advance. For inference fleets whose request shapes change over time, this can reduce fragmentation caused by permanently reserving the wrong MIG profiles.

Flexible MIG performs GPU-level reconfiguration, so maintenance state, supported profiles, driver versions, and active workloads still need explicit coordination. “Dynamic” does not mean every profile can be switched at any time without disruption. Before enabling the feature in production, validate instance creation, device discovery, CDI injection, reclamation after Pod termination, failure handling for an unavailable profile, and state reconstruction after a node reboot.

### The mutex Policy and Policy Composition

[PR #2011](https://github.com/Project-HAMi/HAMi/pull/2011) adds a `mutex` GPU scheduling policy for relationships that cannot safely share one device even when the numeric resource request appears to fit. This is useful when runtime constraints, isolation requirements, or a particular combination of workloads makes co-location invalid.

[PR #2621](https://github.com/Project-HAMi/HAMi/pull/2621) then permits comma-separated `gpu-scheduler-policy` combinations. A scheduling decision no longer has to choose exactly one of binpack, spread, or mutex; multiple constraints can be expressed together. Composition also raises the cost of a configuration mistake. Upgrade testing should cover ordering, device scoring, mutual-exclusion rules, per-Pod overrides, and the diagnostic response when no device satisfies the combined policy.

### Registration Handshake and NUMA Alignment

The device plugin registers with the scheduler through node annotations. [PR #2052](https://github.com/Project-HAMi/HAMi/pull/2052) removes stale `deleted` annotations during registration, preventing an obsolete handshake state from keeping a recovered node unavailable. This is especially relevant when a device plugin restarts frequently, a node returns after failure, or a driver reload changes the reported inventory.

[PR #2065](https://github.com/Project-HAMi/HAMi/pull/2065) adds opt-in NUMA topology information for vGPU replicas. The scheduler and kubelet can use that information to consider CPU-to-GPU locality and reduce cross-NUMA access. Because the capability is optional, operators should confirm that node topology data, CPU Manager policy, and device-plugin reporting agree before enabling it, then measure the workload rather than assuming a topology hint always improves performance.

### Cluster Autoscaler: A Design Direction in This Release

The Release Notes link “Support autoscaling” to [PR #2528](https://github.com/Project-HAMi/HAMi/pull/2528). That pull request is a design for Cluster Autoscaler scale-up simulation: it describes how an autoscaler might understand template nodes, HAMi device resources, and scheduler feasibility when deciding whether a new node would help a pending Pod.

Accordingly, the v2.10.0 artifact here is a design and integration direction. It should not be read as a complete, production-ready autoscaling runtime that appears automatically after a HAMi upgrade. Teams planning Cluster Autoscaler integration still need to track the implementation, cloud-provider node templates, the simulation interface, and a published compatibility matrix.

## Heterogeneous Device Support

### AMD MI300X

[PR #2290](https://github.com/Project-HAMi/HAMi/pull/2290) adds AMD vGPU support, with the release highlights naming the AMD MI300X series. [PR #2067](https://github.com/Project-HAMi/HAMi/pull/2067) updates the AMD design, while the earlier [AMD Instinct vGPU design PR #1985](https://github.com/Project-HAMi/HAMi/pull/1985) records the wider approach.

Supporting a new accelerator requires more than registering another Kubernetes resource name. It covers discovery, memory and compute accounting, container injection, health state, scheduler scoring, and metrics. An AMD deployment should therefore be tested with its actual ROCm, driver, node image, and container-runtime combination. Resource fields in workload manifests must also match the HAMi device configuration used by the cluster.

### Biren

[PR #1711](https://github.com/Project-HAMi/HAMi/pull/1711) adds Biren device support, and [PR #1979](https://github.com/Project-HAMi/HAMi/pull/1979) marks the roadmap item complete. The integration follows HAMi's shared device interface: vendor-specific discovery and resource semantics connect to a common scheduling framework.

Operators should still check the compatibility matrix for the exact Biren driver and device-plugin release they deploy, including full-card or partitioned use, health monitoring, and metrics. “Supported by HAMi” means the project has an implementation path; it does not replace the hardware vendor's compatibility guidance or production capacity testing.

### Ascend DRA and Heterogeneous Ascend Modes

Alongside the independent [Ascend DRA Driver](https://github.com/Project-HAMi/ascend-dra-driver), v2.10.0 lets an Ascend Pod without an explicit mode schedule across both vNPU-template and HAMi-core nodes. [PR #2035](https://github.com/Project-HAMi/HAMi/pull/2035) implements this compatibility path, reducing the hard binding between an application template and a node's partitioning mode.

The two modes do not become semantically identical. Operators still need to align memory, core, device count, node annotations, and runtime mounts. Workloads that must use one mode for performance, isolation, or compatibility should keep an explicit constraint instead of relying on mode-agnostic placement.

### New 910C Mock Templates

[PR #2005](https://github.com/Project-HAMi/HAMi/pull/2005) adds the `vir05_1c_16g` and `vir10_3c_32g` 910C vNPU templates to mock-device-plugin. Mock templates let developers validate resource names, template matching, and scheduling flow without physical hardware. They do not reproduce real drivers, throughput, memory behavior, or hardware failure recovery, so the final qualification step must still use 910C devices.

## HAMi-core and Build Compatibility

### HAMi-core vNPU Memory No Longer Follows a Fixed Template by Mistake

The Release Notes describe this change as “HAMi-core mode for vnpu doesn't need to align device memory to template.” [PR #2696](https://github.com/Project-HAMi/HAMi/pull/2696) fixes automatic memory trimming in vNPU HAMi-core mode, so a fine-grained software-partition request is no longer incorrectly aligned to a fixed vNPU template's memory size.

This matters because independent memory and compute requests are central to HAMi-core. Applying a template boundary anyway reduces packing density or delivers a result different from the workload's declaration. After upgrading, regress small-memory requests, multiple Pods sharing a device, resource release after exit, and the values shown by device monitoring.

### UBI8 Compilation Image

[PR #1958](https://github.com/Project-HAMi/HAMi/pull/1958) aligns HAMi with the HAMi-core compilation image and moves the build path to UBI8 to broaden compatibility across GLIBC environments. [PR #2102](https://github.com/Project-HAMi/HAMi/pull/2102) then removes a stale upper GLIBC bound from the prerequisites documentation.

This is primarily a build and binary portability change. Teams using custom base images, offline registries, or self-built HAMi-core artifacts should revalidate build arguments, dynamic libraries, image architectures, and software-supply-chain scans rather than assuming their old image pipeline remains identical.

## RiseUnion Contributions

RiseUnion's work in this release goes beyond a feature demonstration. It grew out of real production deployments of China-developed accelerators: adding vNPU templates and heterogeneous scheduling for Ascend 910C, then fixing resource-delivery errors, bypassed node-mode checks, missing `npu-smi` inside containers, health-check panics, node-cache races, and leader-update failures. The changes span the scheduler, device plugins, node configuration, and container runtime, showing end-to-end delivery experience from hardware enablement through production stability.

### @ouyangluwei163: Ascend Templates, Mode Compatibility, and Plugin Deployment

In the main HAMi repository, `ouyangluwei163` completed two directly linked Issue→PR records:

- [HAMi Issue #2004](https://github.com/Project-HAMi/HAMi/issues/2004) → [HAMi PR #2005](https://github.com/Project-HAMi/HAMi/pull/2005): add the `vir05_1c_16g` and `vir10_3c_32g` 910C vNPU mock templates.
- [HAMi Issue #2034](https://github.com/Project-HAMi/HAMi/issues/2034) → [HAMi PR #2035](https://github.com/Project-HAMi/HAMi/pull/2035): allow mode-agnostic Ascend Pods to match both vNPU-template and HAMi-core nodes.

Three more direct relationships are recorded in `ascend-device-plugin`:

- [ascend-device-plugin Issue #94](https://github.com/Project-HAMi/ascend-device-plugin/issues/94) → [PR #95](https://github.com/Project-HAMi/ascend-device-plugin/pull/95): automatically enable device-share configuration for relevant modes such as HAMi-core.
- [ascend-device-plugin Issue #103](https://github.com/Project-HAMi/ascend-device-plugin/issues/103) → [PR #104](https://github.com/Project-HAMi/ascend-device-plugin/pull/104): mount `/usr/local/sbin` so `npu-smi` can be found inside the container.
- [ascend-device-plugin Issue #105](https://github.com/Project-HAMi/ascend-device-plugin/issues/105) → [PR #106](https://github.com/Project-HAMi/ascend-device-plugin/pull/106): make a mode-agnostic Pod follow the node's current vNPU-template or HAMi-core mode.

These changes connect the HAMi scheduler, Ascend plugin, node mode, and container-runtime configuration. Together they directly support the 910C templates and heterogeneous Ascend mode highlighted in v2.10.

### @Wangmin362: Reliability, Configuration Boundaries, and Device Compatibility

`Wangmin362` has two verifiable Issue→PR relationships in the main HAMi repository:

- [HAMi Issue #2025](https://github.com/Project-HAMi/HAMi/issues/2025) → [HAMi PR #2026](https://github.com/Project-HAMi/HAMi/pull/2026): a hard-split vNPU `-core` request was accounted by the scheduler but could not be delivered to the container; the fix rejects that request in hard-split mode and corrects the related message.
- [HAMi Issue #2028](https://github.com/Project-HAMi/HAMi/issues/2028) → [HAMi PR #2029](https://github.com/Project-HAMi/HAMi/pull/2029): whole-card or memory-less HAMi-core requests could bypass the node's soft/hard mode gate; the fix applies the node-mode constraint to those requests as well.

- [HAMi Issue #2030](https://github.com/Project-HAMi/HAMi/issues/2030): a discussion record related to HAMi-core vNPU memory handling.

The v2.10.0 Release Notes also include the following reliability and configuration pull requests from `Wangmin362`:

- [HAMi PR #2043](https://github.com/Project-HAMi/HAMi/pull/2043): prevent a panic when health state is nil.
- [HAMi PR #2045](https://github.com/Project-HAMi/HAMi/pull/2045): handle empty `use` / `nouse` GPU UUID configuration.
- [HAMi PR #2050](https://github.com/Project-HAMi/HAMi/pull/2050): correct configuration when only Vastai or only Biren is enabled.
- [HAMi PR #2053](https://github.com/Project-HAMi/HAMi/pull/2053): prevent a panic when only Cambricon is enabled and the first container has no MLU request.
- [HAMi PR #2068](https://github.com/Project-HAMi/HAMi/pull/2068): fix a concurrent read race in the node cache.
- [HAMi PR #2075](https://github.com/Project-HAMi/HAMi/pull/2075): avoid exiting the process on a transient leader-label update failure.
- [HAMi PR #2156](https://github.com/Project-HAMi/HAMi/pull/2156): treat `gpumem-percentage: 0` as unset.
- [HAMi PR #2393](https://github.com/Project-HAMi/HAMi/pull/2393): skip event-based health checks for devices that do not support them.

Companion repository work includes:

- [ascend-device-plugin PR #100](https://github.com/Project-HAMi/ascend-device-plugin/pull/100): add handling around `VDeviceCount`.
- [ascend-device-plugin PR #127](https://github.com/Project-HAMi/ascend-device-plugin/pull/127): remain compatible with the pre-v2.9 device configuration layout.
- [mock-device-plugin PR #18](https://github.com/Project-HAMi/mock-device-plugin/pull/18): support both new and legacy vNPU configuration and add simulated Ascend and Hygon core resources.

This work concentrates on the boundaries most likely to surface in production: nil values, a single enabled vendor, concurrent cache access, leader updates, backward-compatible configuration, and devices without a uniform event-based health mechanism.

### From Ascend 910C Support to Productionizing China-Developed Accelerators

Adding a resource name is only the first step in supporting a domestic accelerator. The harder work connects device discovery, resource modeling, scheduling decisions, container injection, node modes, health checks, and upgrades. The new Ascend 910C templates make hardware profiles expressible; compatibility between vNPU-template and HAMi-core nodes makes placement practical; automatic device-share setup and the `npu-smi` mount make the device plugin operable in real clusters.

The release also closes a broad set of defects that tend to appear only under complex configuration and sustained operation: hard-split resources counted but not delivered, whole-card requests bypassing node-mode checks, single-vendor configurations failing to start, nil health state causing a panic, concurrent node-cache reads, a transient leader-label error terminating the process, and incompatible historical device configuration. Though individually small, these fixes determine whether a domestic-accelerator cluster can schedule consistently, recover from faults, and survive upgrades.

From 910C enablement to closing these production issues, the verified contribution record demonstrates RiseUnion's hands-on experience bringing China-developed accelerators into production. The team is not only making devices discoverable and requestable by Kubernetes; it is making them schedulable, stable, and operable inside heterogeneous resource pools. That is the practical difference between basic compatibility and production adoption.

## Production Reliability and Upgrade Checklist

Beyond these headline items, v2.10.0 includes a large set of changes in concurrency, quota handling, discovery, monitoring, and security. Because the generated changelog spans many components, an upgrade review should filter the [complete Release Notes](https://github.com/Project-HAMi/HAMi/releases/tag/v2.10.0) by the device types and cluster integrations actually in use.

At minimum, validate the following:

1. **Chart and DRA**: confirm that DRA is separated from the main chart and that the independent driver, RBAC, and resource objects install and roll back as one tested unit.
2. **init container behavior**: verify accounting across sequential initialization, ordinary application containers, and ResourceQuota.
3. **PodGroup**: exercise group scheduling and cleanup under lock contention, partial node failure, and timeout.
4. **Flexible MIG**: cover creation, reclamation, node reboot, CDI injection, and the failure path for an unsatisfied profile.
5. **Policy composition**: use representative Pods to test binpack, spread, mutex, and per-Pod weights instead of checking only that configuration parses.
6. **NUMA**: align topology reporting, CPU Manager, and device-plugin settings, then measure cross-NUMA effects.
7. **Heterogeneous accelerators**: qualify discovery, allocation, health, metrics, and recovery independently for AMD MI300X, Biren, and Ascend.
8. **HAMi-core**: confirm small memory requests are no longer trimmed to a fixed template and load the UBI8-built artifacts in every target GLIBC environment.
9. **Observability**: watch registration handshake state, scheduling failures, node cache behavior, leader election, and quota metrics.
10. **Autoscaling**: treat the Cluster Autoscaler item as a design direction until an implementation and compatibility statement are available.

## Summary

HAMi v2.10.0 moves the project beyond simply sharing accelerators toward scheduling them under device-specific and workload-specific constraints. Flexible MIG makes NVIDIA partitioning more dynamic; mutex and policy composition make placement rules more expressive; init container accounting, PodGroup, handshake, and NUMA work strengthen production semantics; and AMD MI300X, Biren, Ascend DRA, and heterogeneous Ascend modes broaden the common scheduling framework.

Refer to the [HAMi v2.10.0 Release Notes](https://github.com/Project-HAMi/HAMi/releases/tag/v2.10.0) for the complete feature, fix, and contributor list.

## HAMi Release Series

- [HAMi v2.8.0: Standardization and Ecosystem Completeness](https://www.theriseunion.com/en/blog/hami-2-8-0.html)
- [HAMi v2.9.0: Ascend HAMi-core, DRA, and Scheduler Ecosystem Advances](https://www.theriseunion.com/en/blog/hami-2-9-0.html)
- [HAMi v2.10.0: Flexible MIG, Composable Scheduling, and Broader Accelerator Support](https://www.theriseunion.com/en/blog/hami-2-10-0.html)

---

[Read this article on the RiseUnion website](https://www.theriseunion.com/en/blog/hami-2-10-0.html) · [Explore more articles](https://www.theriseunion.com/en/blog.html)
