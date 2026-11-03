[English](./README.en.md) · [文章目录](../../README.md) · [官网阅读](https://www.theriseunion.com/zh/blog/kubernetes-dynamic-resource-allocation.html)

# Kubernetes动态资源分配：GPU资源管理新突破

睿思智联 · 2025/09/16 08:00（北京时间）

深入解析Kubernetes DRA动态资源分配技术，学习如何通过ResourceClaim和DeviceClass实现GPU、FPGA等异构资源的智能调度与池化共享，提升集群资源利用率和运维效率。

**作者：** Simardeep Singh  
**发布日期：** 2024年12月16日  
**原文链接：** [点击这里](https://medium.com/@simardeep.oberoi/kubernetes-dynamic-resource-allocation-a-leap-in-resource-management-c39fdca6b99e)

## 引言

在当今快速发展的云原生环境中，Kubernetes 已成为容器编排的事实标准。随着人工智能、机器学习和高性能计算工作负载的兴起，对专门硬件资源（如 GPU、FPGA 和加速器）的需求急剧增长。传统的 Kubernetes 资源管理方法在处理这些复杂需求时遇到了挑战，这就是动态资源分配（Dynamic Resource Allocation，DRA）API 发挥作用的地方。

## 什么是动态资源分配？

动态资源分配（DRA）是 Kubernetes 中的一项革命性功能，旨在简化在 Pod 和容器之间请求和共享资源的过程。它将持久卷（Persistent Volumes）API 泛化，以支持 GPU 等各种通用资源。通过动态分配资源，DRA 提高了资源利用率，降低了运维复杂性，使 Kubernetes 能够有效处理现代工作负载。

![dra](./images/dra.gif)


### DRA 解决的核心问题

传统的 Kubernetes 资源管理主要依赖于 CPU 和内存的静态分配。然而，现代工作负载需要更复杂的资源类型：

- **GPU 资源**：用于机器学习和深度学习任务
- **FPGA**：用于特定计算加速
- **网络加速器**：用于高性能网络处理
- **存储加速器**：用于高速存储访问

DRA 通过引入结构化的资源管理方法，解决了这些挑战。

## DRA 的关键概念

### 1. ResourceClaim（资源声明）

ResourceClaim 表示工作负载所需的特定资源请求。它类似于持久卷声明（PVC），但适用于更广泛的资源类型。

```yaml
apiVersion: resource.k8s.io/v1alpha2
kind: ResourceClaim
metadata:
  name: gpu-claim
spec:
  resourceClassName: nvidia-gpu
  parameters:
    memory: "16Gi"
    driver-version: "470.57.02"
```

### 2. ResourceClaimTemplate（资源声明模板）

ResourceClaimTemplate 自动为每个 Pod 创建和管理 ResourceClaim，简化了大规模部署的资源管理。

```yaml
apiVersion: resource.k8s.io/v1alpha2
kind: ResourceClaimTemplate
metadata:
  name: gpu-template
spec:
  spec:
    resourceClassName: nvidia-gpu
    parameters:
      memory: "16Gi"
```

### 3. DeviceClass（设备类）

DeviceClass 定义特定资源的选择标准和配置，为资源分配提供标准化的方法。

```yaml
apiVersion: resource.k8s.io/v1alpha2
kind: DeviceClass
metadata:
  name: nvidia-gpu
spec:
  driverName: nvidia.com/gpu
  parameters:
    memory: "16Gi"
    compute-capability: "8.0"
```

### 4. ResourceSlice（资源切片）

ResourceSlice 发布可供分配的资源信息，为调度器提供资源可用性的实时视图。

```yaml
apiVersion: resource.k8s.io/v1alpha2
kind: ResourceSlice
metadata:
  name: gpu-slice-1
spec:
  nodeName: worker-node-1
  driverName: nvidia.com/gpu
  resourceModel:
    namedResources:
      instances:
        - name: gpu-0
          attributes:
            - name: memory
              quantity: "16Gi"
            - name: compute-capability
              string: "8.0"
```

## DRA 的优势

### 1. 提高效率

kube-scheduler 现在可以在不需要与外部驱动程序交互的情况下管理资源分配，从而：
- 减少调度延迟
- 加快决策过程
- 提高整体集群性能

### 2. 增强灵活性

用户可以指定详细的资源需求，如：
- GPU 内存大小
- 驱动程序版本
- 特定硬件属性
- 计算能力要求

这种细粒度的控制确保工作负载在最佳资源上运行。

### 3. 大规模可扩展性

对于大型集群，DRA 使管理复杂的资源需求变得更加简单：
- Kubernetes 可以在节点之间动态分配资源
- 保持高资源利用率
- 简化运维管理

## 实际应用示例

### 场景：金融机构的机器学习模型

假设某金融机构运行一个机器学习模型来预测股市趋势。该模型需要多个具有至少 16GB 内存的 GPU 来执行密集计算。

#### 传统方法的问题

```yaml
# 传统方法 - 使用 nodeSelector 和资源限制
apiVersion: v1
kind: Pod
spec:
  nodeSelector:
    nvidia.com/gpu: "true"
  containers:
  - name: ml-model
    resources:
      limits:
        nvidia.com/gpu: 2
```

这种方法的问题：
- 无法指定具体的 GPU 要求
- 调度器无法优化资源分配
- 资源利用率低

#### 使用 DRA 的解决方案

```yaml
# 1. 定义 DeviceClass
apiVersion: resource.k8s.io/v1alpha2
kind: DeviceClass
metadata:
  name: nvidia-gpu-16gb
spec:
  driverName: nvidia.com/gpu
  parameters:
    memory: "16Gi"
    compute-capability: "8.0"

---
# 2. 创建 ResourceClaimTemplate
apiVersion: resource.k8s.io/v1alpha2
kind: ResourceClaimTemplate
metadata:
  name: gpu-template
spec:
  spec:
    resourceClassName: nvidia-gpu-16gb
    parameters:
      memory: "16Gi"

---
# 3. 在 Pod 中使用
apiVersion: v1
kind: Pod
spec:
  resourceClaims:
  - name: gpu-claim
    source:
      resourceClaimTemplateName: gpu-template
  containers:
  - name: ml-model
    resources:
      claims:
      - name: gpu-claim
```

### DRA 带来的好处

1. **精确匹配**：调度器可以找到完全符合要求的 GPU
2. **动态分配**：资源可以根据需求动态分配和释放
3. **优化利用**：提高整体资源利用率
4. **简化管理**：减少手动配置和运维工作

## 实施 DRA 的最佳实践

### 1. 资源规划

- 评估现有工作负载的资源需求
- 识别需要 DRA 的资源类型
- 规划资源池的配置

### 2. 设备类设计

- 根据硬件特性创建合适的 DeviceClass
- 考虑资源属性和约束条件
- 确保命名规范的一致性

### 3. 监控和观察

- 实施资源使用监控
- 跟踪分配效率
- 优化资源配置

### 4. 安全考虑

- 实施适当的 RBAC 策略
- 确保资源隔离
- 监控资源访问

## 未来展望

DRA 代表了 Kubernetes 资源管理的重要进步，但仍有发展空间：

### 即将到来的功能

1. **多资源支持**：支持同时分配多种资源类型
2. **动态调整**：运行时调整资源分配
3. **成本优化**：基于成本的资源选择
4. **跨集群支持**：跨多个集群的资源管理

### 生态系统集成

- 与云提供商的深度集成
- 与监控和可观测性工具的集成
- 与 CI/CD 管道的集成

## 结论

Kubernetes 的动态资源分配（DRA）功能通过引入更灵活和高效的资源管理方式，显著提升了 Kubernetes 在处理现代复杂工作负载时的能力。通过消除对第三方驱动程序的依赖，DRA 使 Kubernetes 能够直接高效地管理和分配资源，为云原生应用提供了更强大的基础。

随着 AI 和机器学习工作负载的持续增长，DRA 将成为 Kubernetes 生态系统中不可或缺的一部分。它不仅能提高资源利用率，还能简化运维管理，为组织提供更好的云原生体验。

对于正在考虑实施 DRA 的组织，建议从评估现有工作负载开始，逐步引入 DRA 功能，并建立相应的监控和管理流程。通过合理规划和实施，DRA 将为企业带来显著的运营效益和技术优势。

## 扩展阅读
那么，作为开源的异构算力管理中间件 HAMi 来说，该如何结合并引入 DRA 相关特性？[继续阅读《HAMi 与 DRA 演示》](https://www.theriseunion.com/zh/blog/hami-with-dra-demo.html)


**相关资源**

- [Kubernetes 官方 DRA 文档](https://kubernetes.io/docs/concepts/scheduling-eviction/dynamic-resource-allocation/)
- [DRA 设计提案](https://github.com/kubernetes/enhancements/tree/master/keps/sig-node/3063-dynamic-resource-allocation)
- [Kubernetes 资源管理最佳实践](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/)

---

[在睿思智联官网阅读本文](https://www.theriseunion.com/zh/blog/kubernetes-dynamic-resource-allocation.html) · [浏览更多文章](https://www.theriseunion.com/zh/blog.html)
