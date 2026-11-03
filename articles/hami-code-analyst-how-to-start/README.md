[English](./README.en.md) · [文章目录](../../README.md) · [官网阅读](https://www.theriseunion.com/zh/blog/hami-code-analyst-how-to-start.html)

# HAMi 配置指南：GPU 资源池化调度参数详解

睿思智联 · 2025/04/29 08:00（北京时间）

详细介绍 HAMi 调度器和设备插件的启动参数配置，包括 GPU 内存分配、核心利用率控制、MIG 策略等关键特性。助您快速掌握异构 GPU 资源池化管理，提升集群运行效率。适用于从单节点测试到大规模生产环境部署的各类场景。

![文章封面](./images/cover.jpg)

[HAMi](https://www.theriseunion.com/zh/blog/project-hami.html) 是一款面向异构 GPU 资源池化和调度的智能平台。为了帮助用户根据不同环境和需求灵活配置，HAMi 的调度器与设备插件均支持丰富的启动参数。本文将系统介绍各项参数及其默认行为，帮助用户快速上手并优化部署。

> 注意：本篇文章是基于当前 master 分支上的，参数可能会更新变化

### 调度器启动参数

HAMi 调度器组件支持以下启动配置：

##### --http_bind
  指定 HTTP 服务器绑定地址，默认为 127.0.0.1:8080。
  
##### --cert_file  
  配置 TLS 证书文件路径，开启 HTTPS 通信时使用。

##### --key_file  
  配置 TLS 密钥文件路径，与 cert_file 配合使用。

##### --scheduler-name  
  指定调度器名称，会写入 pod.spec.schedulerName 字段。若为空，使用默认 Kubernetes 调度器。

##### --default-mem  
  Pod 未指定时，默认为其分配的 GPU 设备内存大小。

##### --default-cores  
  Pod 未指定时，默认为其分配的 GPU 核心利用率（百分比）。

##### --default-gpu  
  Pod 未指定时，默认分配的 GPU 数量，默认为 1。

##### --node-scheduler-policy  
  节点调度策略，默认为 "binpack"，即尽量将资源集中部署。

##### --gpu-scheduler-policy  
  GPU 调度策略，默认为 "spread"，即尽量将负载在 GPU 间分散。

##### --metrics-bind-address  
  Prometheus 指标采集接口的绑定地址，默认为 :9395。

##### --node-label-selector  
  基于节点标签进行选择，多个键值对用逗号分隔。

##### --kube-qps  
  与 kube-apiserver 通信的每秒请求数 (QPS)，默认为 5.0。

##### --kube-burst  
  突发模式下允许的最大请求数量，默认为 10。

##### --kube-timeout  
  与 kube-apiserver 通信的超时时间（秒），默认为 30。

##### --profiling  
  是否通过 HTTP 服务器启用 pprof 性能分析，默认为 false（关闭）。


### 设备插件启动参数

HAMi 的 NVIDIA 设备插件支持以下启动配置：

#### 基本设备参数

##### --node-name  
  当前节点名称，默认从环境变量自动读取。

##### --device-split-count  
  单块 GPU 分割出的虚拟设备数量，默认为 2。

##### --device-memory-scaling  
  GPU 内存缩放比例，默认为 1.0，即无缩放。

##### --device-cores-scaling  
  GPU 核心缩放比例，默认为 1.0。

##### --disable-core-limit  
  设置后将忽略 GPU 核心利用率限制，默认为 false。

##### --resource-name  
  容器中声明 GPU 资源使用的字段名，默认为 "nvidia.com/gpu"。

#### 高级功能参数

##### --mig-strategy  
  针对支持 MIG（Multi-Instance GPU）技术的设备，公开资源的策略。选项包括：
  - none（默认）：不使用 MIG；
  - single：每个 MIG 实例作为一个单独 GPU 设备；
  - mixed：混合管理模式。

##### --fail-on-init-error  
  初始化出错时是否终止插件运行，默认为 true（严格模式）。

##### --nvidia-driver-root  
  NVIDIA 驱动程序的根路径，默认为 /。

##### --pass-device-specs  
  Allocate() 时是否将 DeviceSpecs 列表传递给 kubelet，默认为 false。

##### --device-list-strategy  
  向运行时传递设备列表的方式，支持：
  - envvar（默认）：通过环境变量；
  - volume-mounts：通过挂载卷；
  - cdi-annotations：通过容器描述注释 (CDI)。

##### --device-id-strategy  
  设备 ID 传递方式，支持：
  - uuid（默认）：使用唯一 ID；
  - index：使用索引编号。

##### --gds-enabled  
  确保容器启动时启用 GDS（GPU Direct Storage）。

##### --mofed-enabled  
  确保容器启动时启用 MOFED（Mellanox OpenFabrics）。

##### --config-file  
  指定配置文件路径，可替代命令行参数或环境变量配置。

##### --cdi-annotation-prefix  
  CDI 注释键的前缀，默认为预设值。

##### --nvidia-ctk-path  
  生成 CDI 规范时使用的 nvidia-ctk 工具路径。

##### --container-driver-root  
  容器内部挂载的 NVIDIA 驱动程序目录路径，用于 CDI 规范。

##### --v  
  日志详细级别，默认为 0，数值越大，日志越详细。



## 小结

HAMi 的启动参数设计兼顾了**灵活性**与**安全性**，允许用户根据集群规模、硬件能力、调度需求以及监控策略，进行细粒度调优。  
无论是单节点测试还是大规模生产环境部署，合理配置上述参数，都能帮助用户最大化利用 GPU 资源，提升整体调度效率和系统稳定性。

---

[在睿思智联官网阅读本文](https://www.theriseunion.com/zh/blog/hami-code-analyst-how-to-start.html) · [浏览更多文章](https://www.theriseunion.com/zh/blog.html)
