[English](./README.en.md) · [文章目录](../../README.md) · [官网阅读](https://www.theriseunion.com/zh/blog/hami-support-mig.html)

# HAMi 正式支持 NVIDIA 动态 MIG 分区管理

睿思智联 · 2025/02/10 08:00（北京时间）

HAMi 现已支持动态 MIG 功能，通过 mig-parted 实现 GPU 资源的自动分区和管理。用户无需手动操作，即可实现 MIG 实例的动态创建、调整和监控，大幅简化了 Kubernetes 环境下的 GPU 资源管理。支持 NVIDIA Ampere、Hopper 和 Blackwell 系列设备，并可与 HAMi-core 无缝集成。

![文章封面](./images/hami-logo.jpg)

## 简介

**我们现已支持通过 mig-parted 动态调整 MIG 设备的功能**，主要包括：

- ***动态 MIG 实例管理***：用户无需在 GPU 节点上执行 'nvidia-smi -i 0 -mig 1' 等命令来管理 MIG 实例，所有操作都将由 HAMi 设备插件自动完成。
- ***动态 MIG 配置调整***：HAMi 管理的每个 MIG 设备都会根据提交的任务需求自动调整其 MIG 模板配置。
- ***设备 MIG 监控***：所有由 HAMi 创建的 MIG 实例都会在调度器监控界面中显示，包括任务信息，使用户能够清晰地查看 MIG 节点概况。
- ***兼容 HAMi-core 节点***：HAMi 可以统一管理 `HAMi-core 节点`和 `MIG 节点`的 GPU 资源池。如果不通过 `nvidia.com/vgpu-mode` 注解手动指定，任务可以被调度到这两种节点中的任意一种。
- ***与 HAMi-core 统一 API***：无需额外配置即可使作业兼容动态 MIG 特性。

## 设计原理
[查看 HAMi 支持 MIG 特性的设计原理](https://www.theriseunion.com/zh/blog/hami-dynamic-mig.html)

## 系统要求

* NVIDIA Blackwell、Hopper™ 和 Ampere 系列设备
* HAMi 版本 >= v2.5.0，[HAMi v2.5.0 release](https://www.theriseunion.com/zh/blog/hami-2-5-0.html)
* Nvidia-container-toolkit

## 启用动态 MIG 支持

步骤 1: 使用 helm 安装图表
参考[此处](https://github.com/Project-HAMi/HAMi#enabling-vgpu-support-in-kubernetes)的"在 Kubernetes 中启用 vGPU 支持"章节

步骤 2: 在设备插件的 ConfigMap 中为 MIG 节点将 `mode` 配置为 `mig`

```bash
kubectl describe cm  hami-device-plugin -n kube-system
```

```json
{
    "nodeconfig": [
        {
            "name": "MIG-NODE-A",
            "operatingmode": "mig",
            "filterdevices": {
              "uuid": [],
              "index": []
            }
        }
    ]
}
```

步骤 3: 重启以下 Pod 使配置生效：
  * hami-scheduler 
  * 'MIG-NODE-A' 节点上的 hami-device-plugin

## 自定义 MIG 配置（可选）
HAMi 目前包含一个[内置的 MIG 配置](https://github.com/Project-HAMi/HAMi/blob/master/charts/hami/templates/scheduler/device-configmap.yaml)。

您可以按照以下步骤自定义 MIG 配置：

### 修改 charts/hami/templates/scheduler 目录下的 'device-configmap.yaml' 文件内容如下

  ```yaml
    nvidia:
      resourceCountName: {{ .Values.resourceName }}
      resourceMemoryName: {{ .Values.resourceMem }}
      resourceMemoryPercentageName: {{ .Values.resourceMemPercentage }}
      resourceCoreName: {{ .Values.resourceCores }}
      resourcePriorityName: {{ .Values.resourcePriority }}
      overwriteEnv: false
      defaultMemory: 0
      defaultCores: 0
      defaultGPUNum: 1
      deviceSplitCount: {{ .Values.devicePlugin.deviceSplitCount }}
      deviceMemoryScaling: {{ .Values.devicePlugin.deviceMemoryScaling }}
      deviceCoreScaling: {{ .Values.devicePlugin.deviceCoreScaling }}
      knownMigGeometries:
      - models: [ "A30" ]
        allowedGeometries:
          - 
            - name: 1g.6gb
              memory: 6144
              count: 4
          - 
            - name: 2g.12gb
              memory: 12288
              count: 2
          - 
            - name: 4g.24gb
              memory: 24576
              count: 1
      - models: [ "A100-SXM4-40GB", "A100-40GB-PCIe", "A100-PCIE-40GB", "A100-SXM4-40GB" ]
        allowedGeometries:
          - 
            - name: 1g.5gb
              memory: 5120
              count: 7
          - 
            - name: 2g.10gb
              memory: 10240
              count: 3
            - name: 1g.5gb
              memory: 5120
              count: 1
          - 
            - name: 3g.20gb
              memory: 20480
              count: 2
          - 
            - name: 7g.40gb
              memory: 40960
              count: 1
      - models: [ "A100-SXM4-80GB", "A100-80GB-PCIe", "A100-PCIE-80GB"]
        allowedGeometries:
          - 
            - name: 1g.10gb
              memory: 10240
              count: 7
          - 
            - name: 2g.20gb
              memory: 20480
              count: 3
            - name: 1g.10gb
              memory: 10240
              count: 1
          - 
            - name: 3g.40gb
              memory: 40960
              count: 2
          - 
            - name: 7g.79gb
              memory: 80896
              count: 1
  ```

  > **注意** Helm 安装和更新将基于此文件中的配置，覆盖 Helm 的内置配置

  > **注意** 请注意，HAMi 将按照此配置文件中的顺序查找并使用第一个适合任务的 MIG 模板

## 运行 MIG 作业

现在可以像使用 `hami-core` 一样请求 MIG 实例，只需指定 `nvidia.com/gpu` 和 `nvidia.com/gpumem` 资源类型即可。

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: gpu-pod
  annotations:
    nvidia.com/vgpu-mode: "mig" #（可选），如果不设置，此 Pod 可以被分配到 MIG 实例或 hami-core 实例
spec:
  containers:
    - name: ubuntu-container
      image: ubuntu:18.04
      command: ["bash", "-c", "sleep 86400"]
      resources:
        limits:
          nvidia.com/gpu: 2 
          nvidia.com/gpumem: 8000
```

在上述示例中，任务分配了两个 MIG 实例，每个实例至少具有 8G 设备内存。

## 监控 MIG 实例

HAMi 管理的 MIG 实例将在调度器监控页面（调度器节点 ip:31993/metrics）中显示，如下所示：

```bash
# HELP nodeGPUMigInstance GPU 共享模式。0 表示 hami-core，1 表示 mig，2 表示 mps
# TYPE nodeGPUMigInstance gauge
nodeGPUMigInstance{deviceidx="0",deviceuuid="GPU-936619fc-f6a1-74a8-0bc6-ecf6b3269313",migname="3g.20gb-0",nodeid="aio-node15",zone="vGPU"} 1
nodeGPUMigInstance{deviceidx="0",deviceuuid="GPU-936619fc-f6a1-74a8-0bc6-ecf6b3269313",migname="3g.20gb-1",nodeid="aio-node15",zone="vGPU"} 0
nodeGPUMigInstance{deviceidx="1",deviceuuid="GPU-30f90f49-43ab-0a78-bf5c-93ed41ef2da2",migname="3g.20gb-0",nodeid="aio-node15",zone="vGPU"} 1
nodeGPUMigInstance{deviceidx="1",deviceuuid="GPU-30f90f49-43ab-0a78-bf5c-93ed41ef2da2",migname="3g.20gb-1",nodeid="aio-node15",zone="vGPU"} 1
```

## 注意事项

1. 您无需在 MIG 节点上进行任何操作，所有管理工作都由 hami-device-plugin 中的 mig-parted 完成。
2. Ampere 架构之前的 NVIDIA 设备不能使用 'mig' 模式。
3. 您不会在节点上看到任何 MIG 资源（如 `nvidia.com/mig-1g.10gb`），因为 HAMi 对 'mig' 和 'hami-core' 节点使用统一的资源名称。

> [参考文档](https://github.com/Project-HAMi/HAMi/blob/master/docs/dynamic-mig-support.md)

---

[在睿思智联官网阅读本文](https://www.theriseunion.com/zh/blog/hami-support-mig.html) · [浏览更多文章](https://www.theriseunion.com/zh/blog.html)
