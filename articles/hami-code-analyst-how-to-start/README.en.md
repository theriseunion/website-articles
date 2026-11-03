[简体中文](./README.md) · [All articles](../../README.en.md) · [Read on the website](https://www.theriseunion.com/en/blog/hami-code-analyst-how-to-start.html)

# HAMi Configuration Guide: GPU Resource Pool Management

RiseUnion · January 01, 2025 at 08:00 (UTC+08:00)

Comprehensive guide to HAMi scheduler and device plugin parameters for GPU resource pooling and scheduling. Learn how to optimize GPU memory allocation, core utilization, MIG strategies, and cluster performance for both testing and production environments.

![Article cover](./images/cover.jpg)

[HAMi](https://www.theriseunion.com/en/blog/project-hami.html) is an intelligent platform designed for heterogeneous GPU resource pooling and scheduling. To support flexible configuration across different environments and requirements, HAMi's scheduler and device plugins offer comprehensive startup parameters. This guide provides a detailed overview of these parameters and their default behaviors to help you quickly get started and optimize your deployment.

> Note: This documentation is based on the current master branch and parameters may be updated.

### Scheduler Parameters

HAMi's scheduler component supports the following configuration options:

##### --http_bind
  Specifies the HTTP server binding address. Default: 127.0.0.1:8080.
  
##### --cert_file  
  Path to the TLS certificate file for HTTPS communication.

##### --key_file  
  Path to the TLS private key file, used in conjunction with cert_file.

##### --scheduler-name  
  Defines the scheduler name written to pod.spec.schedulerName. If empty, uses the default Kubernetes scheduler.

##### --default-mem  
  Default GPU memory allocation for pods when not explicitly specified.

##### --default-cores  
  Default GPU core utilization percentage for pods when not explicitly specified.

##### --default-gpu  
  Default number of GPUs allocated per pod when not specified. Default: 1.

##### --node-scheduler-policy  
  Node scheduling policy. Default: "binpack" to consolidate resource allocation.

##### --gpu-scheduler-policy  
  GPU scheduling policy. Default: "spread" to distribute workloads across GPUs.

##### --metrics-bind-address  
  Prometheus metrics endpoint binding address. Default: :9395.

##### --node-label-selector  
  Node selection based on labels, with multiple key-value pairs separated by commas.

##### --kube-qps  
  Queries per second (QPS) limit for kube-apiserver communication. Default: 5.0.

##### --kube-burst  
  Maximum burst request limit. Default: 10.

##### --kube-timeout  
  Timeout for communicating with the kube-apiserver (in seconds). Default is 30.

##### --profiling  
  Enables pprof performance profiling via HTTP server. Default: false.


### Device Plugin Parameters

HAMi's NVIDIA device plugin supports the following configuration options:

#### Basic Device Parameters

##### --node-name  
  Current node name, automatically read from environment variables by default.

##### --device-split-count  
  Number of virtual devices to create from a single GPU. Default: 2.

##### --device-memory-scaling  
  GPU memory scaling factor. Default: 1.0 (no scaling).

##### --device-cores-scaling  
  GPU core scaling factor. Default: 1.0.

##### --disable-core-limit  
  Disables GPU core utilization limits when set. Default: false.

##### --resource-name  
  Resource field name for GPU requests in containers. Default: "nvidia.com/gpu".

#### Advanced Features

##### --mig-strategy  
  Resource exposure strategy for MIG-capable (Multi-Instance GPU) devices. Options:
  - none (default): MIG disabled
  - single: Each MIG instance exposed as a separate GPU
  - mixed: Hybrid management mode

##### --fail-on-init-error  
  Terminates plugin execution on initialization errors. Default: true (strict mode).

##### --nvidia-driver-root  
  Root path for NVIDIA driver installation. Default: /.

##### --pass-device-specs  
  Controls whether DeviceSpecs list is passed to kubelet during Allocate(). Default: false.

##### --device-list-strategy  
  Method for passing device lists to runtime. Options:
  - envvar (default): Via environment variables
  - volume-mounts: Via mounted volumes
  - cdi-annotations: Via Container Device Interface annotations

##### --device-id-strategy  
  Device ID passing method. Options:
  - uuid (default): Using unique identifiers
  - index: Using index numbers

##### --gds-enabled  
  Ensures GPU Direct Storage (GDS) is enabled for containers at launch.

##### --mofed-enabled  
  Ensures Mellanox OpenFabrics (MOFED) is enabled for containers at launch.

##### --config-file  
  Path to configuration file for overriding command-line args or environment variables.

##### --cdi-annotation-prefix  
  Prefix for CDI annotation keys. Uses preset value by default.

##### --nvidia-ctk-path  
  Path to nvidia-ctk tool for CDI spec generation.

##### --container-driver-root  
  NVIDIA driver directory path mounted inside containers for CDI specs.

##### --v  
  Logging verbosity level. Default: 0, higher values increase detail.


## Summary

HAMi's startup parameters are designed with both **flexibility** and **security** in mind, enabling fine-grained tuning based on:
- Cluster scale
- Hardware capabilities
- Scheduling requirements
- Monitoring strategies

Whether you're testing on a single node or deploying in a large-scale production environment, properly configuring these parameters helps maximize GPU resource utilization while enhancing scheduling efficiency and system stability.

---

[Read this article on the RiseUnion website](https://www.theriseunion.com/en/blog/hami-code-analyst-how-to-start.html) · [Explore more articles](https://www.theriseunion.com/en/blog.html)
