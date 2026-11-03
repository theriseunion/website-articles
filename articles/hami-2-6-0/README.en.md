[简体中文](./README.md) · [All articles](../../README.en.md) · [Read on the website](https://www.theriseunion.com/en/blog/hami-2-6-0.html)

# HAMi v2.6.0: Stronger Support for Chinese GPUs, Enhanced Stability and Observability

RiseUnion · January 01, 2025 at 08:00 (UTC+08:00)

In collaboration with the community, HAMi v2.6.0 has been officially released! This version brings significant improvements in stability, performance, and observability, particularly in support for Enflame, Metax, and Cambricon chips, helping more users easily implement GPU pooling and sharing to improve resource utilization.

![Article cover](./images/hami-logo.jpg)

## HAMi v2.6.0: Enhanced Stability, Performance, and Observability

Thanks to the collaborative efforts of community developers and users, **HAMi v2.6.0** has been successfully released! This update focuses on system-level optimizations for stability, performance, and observability, further strengthening support for domestic heterogeneous chips and significantly improving the reliability and flexibility of GPU pooling and shared scheduling.

### Release Highlights

#### Enhanced Domestic Chip Compatibility

* Support for Enflame GCU shared mode (`gcu-share`);
* New recognition and management capabilities for Metax GPU / sGPU;
* Fixed Cambricon chip scheduling allocation anomalies;

> [2.6.0-Release reference](https://github.com/Project-HAMi/HAMi/releases/tag/v2.6.0)

update for bugfix：
- [2.6.1-Release reference](https://github.com/Project-HAMi/HAMi/releases/tag/v2.6.1)

> HAMi continues to advance unified compatibility for domestic GPUs, working to break down barriers between vendors and establish cross-chip platform unified scheduling standards.

#### Scheduler and Device Plugin Optimizations

* Enhanced scheduling logs: Improved structure and readability for easier troubleshooting of scheduling anomalies and behavior tracking.
* New GPU topology-aware scoring mechanism: Currently supports NVIDIA topology affinity scoring, laying the foundation for future complex scheduling strategies.

#### Smarter Deployment and Operations Enhancements

* Support for ConfigMap annotation change auto-rollout restart: Automatically updates corresponding components through `checksum` annotation injection, ensuring configuration consistency.
* Support for NVIDIA RuntimeClass: More flexible container runtime environment configuration, meeting multi-driver version coexistence scenarios.
* Optimized device unload logic: Prevents residual device information in node managers from causing abnormal states.

#### Performance and Observability Improvements

* Introduced `net/http/pprof` runtime performance analysis: Developers can diagnose system performance bottlenecks in real-time.
* vGPUmonitor supports MIG mode display: Better support for MIG-based GPU resource partitioning solutions and monitoring requirements.

### Critical Issue Fixes

HAMi v2.6.0 addresses multiple important issues reported by the community:

* Fixed potential scheduling stuttering issues under NVIDIA driver 570+ versions;
* Fixed VRAM statistics anomalies in ComfyUI scenarios;
* Corrected `vgpu-devices-allocated` annotation data inconsistencies;
* Fixed VRAM statistics errors under `cuMallocAsync`;
* Prevented MIG tasks from incorrectly running on non-MIG nodes causing scheduler crashes;
* Optimized VRAM statistics accuracy for multi-process tasks;
* Addressed dynamic partitioning's lack of single-card granularity management.

> Thanks to every user and contributor in the community who provided detailed feedback and participated in fixes!

### Future Roadmap

HAMi's evolution doesn't stop here. We are actively preparing for **v2.7.0**, with key focus areas including:

* New support for Kunlun chip series GPUs, further expanding domestic ecosystem coverage;
* Comprehensive adaptation to Kubernetes DRA specifications, strengthening integration capabilities with native ecosystems;
* Deep optimization of HAMi-WebUI, delivering more intuitive and efficient heterogeneous resource visualization experience.

Experience HAMi v2.6.0 now! For any suggestions or feedback, [please engage with us in the community](https://github.com/Project-HAMi/HAMi/issues) and help HAMi deliver value in more scenarios.

---

[Read this article on the RiseUnion website](https://www.theriseunion.com/en/blog/hami-2-6-0.html) · [Explore more articles](https://www.theriseunion.com/en/blog.html)
