[English](./README.en.md) · [文章目录](../../README.md) · [官网阅读](https://www.theriseunion.com/zh/blog/hami-code-analyst-1.html)

# HAMi源码解析：设备管理与调度系统的实现原理

睿思智联 · 2025/03/26 08:00（北京时间）

深入剖析HAMi的核心源码实现，详细讲解设备管理接口、设备插件系统、调度策略和监控系统的技术实现。通过源码级别的分析，帮助开发者理解HAMi如何实现异构设备管理和智能调度，以及如何与Kubernetes生态系统进行深度集成。

![文章封面](./images/cover.jpg)

## 项目简介

HAMi (Heterogeneous AI Computing Virtualization Middleware) 是一个用于管理Kubernetes集群中异构设备的中间件。它提供了一套完整的解决方案，用于在Kubernetes环境中管理和调度GPU、FPGA等异构加速设备。[详细介绍 Project-HAMi](https://www.theriseunion.com/zh/blog/project-hami.html)

## 项目架构

HAMi 的代码主要构成：

### 1. 命令行组件 (cmd/)

```bash
cmd/
├── device-plugin/     # 设备插件组件
│   ├── main.go       # 入口文件
│   └── nvidia/       # NVIDIA设备插件实现
│
├── scheduler/        # 调度器组件
│   ├── main.go      # 调度器入口
│   └── metrics.go   # 监控指标实现 
│
└── vGPUmonitor/     # GPU监控组件
    ├── main.go      # 监控服务入口
    ├── metrics.go   # 指标定义和收集
    ├── feedback.go  # 反馈机制实现
    ├── validation.go # 验证逻辑
    ├── build.sh     # 构建脚本
    ├── noderpc/     # 节点RPC通信
    └── testcollector/ # 测试收集器
```

### 2. 核心库 (pkg/)

```bash
pkg/
├── device/          # 设备管理核心
│   ├── devices.go        # 统一设备接口定义
│   ├── devices_test.go   # 设备接口测试
│   ├── nvidia/          # NVIDIA GPU实现
│   ├── cambricon/       # 寒武纪设备实现
│   ├── ascend/         # 华为昇腾设备实现
│   ├── hygon/          # 海光设备实现
│   ├── iluvatar/       # 天数智芯设备实现
│   ├── metax/          # 沐曦智能设备实现
│   └── mthreads/       # 摩尔线程设备实现
│
├── scheduler/       # 调度系统实现
│   ├── config/         # 配置管理
│   ├── policy/         # 调度策略
│   ├── routes/         # 路由规则
│   ├── scheduler.go    # 调度器核心
│   ├── nodes.go        # 节点管理
│   ├── pods.go         # Pod管理
│   ├── score.go        # 评分系统
│   ├── event.go        # 事件处理
│   ├── webhook.go      # Webhook集成
│   └── *_test.go      # 对应的测试文件
│
├── device-plugin/   # 设备插件实现
├── k8sutil/        # Kubernetes工具集
├── monitor/        # 监控系统
├── oci/            # OCI运行时集成
├── util/           # 通用工具库
└── version/        # 版本信息管理
```

## 核心功能模块

### 1. 设备管理 (`pkg/device`)

#### 1.1 统一设备接口
```go
// pkg/device/devices.go
type Devices interface {
    CommonWord() string
    GetNodeDevices(n corev1.Node) ([]*util.DeviceInfo, error)
    CheckHealth(devType string, n *corev1.Node) (bool, bool)
    GenerateResourceRequests(ctr *corev1.Container) util.ContainerDeviceRequest
    LockNode(n *corev1.Node, p *corev1.Pod) error
    ReleaseNodeLock(n *corev1.Node, p *corev1.Pod) error
    ScoreNode(node *corev1.Node, podDevices util.PodSingleDevice, policy string) float32
    CustomFilterRule(...) bool
}
```

#### 1.2 设备类型实现

##### NVIDIA GPU
```go
// pkg/device/nvidia/device.go
type NvidiaDevices struct {
    resourceName string
}

func (dev *NvidiaDevices) GetNodeDevices(n corev1.Node) ([]*util.DeviceInfo, error) {
    // 实现NVIDIA设备发现
}
```

##### Metax设备
```go
// pkg/device/metax/device.go
type MetaxDevices struct {}

const (
    MetaxGPUDevice  = "Metax"
    MetaxSGPUDevice = "Metax-SGPU"
)

func (dev *MetaxDevices) GetNodeDevices(n corev1.Node) ([]*util.DeviceInfo, error) {
    // 实现Metax设备发现
}
```



### 2. 设备插件 (`pkg/device-plugin`)

#### 2.1 插件接口实现
```go
// pkg/device-plugin/server/server.go
type DevicePluginServer struct {
    devicePlugin v1beta1.DevicePluginServer
}

func (s *DevicePluginServer) ListAndWatch(empty *pluginapi.Empty, stream pluginapi.DevicePlugin_ListAndWatchServer) error {
    // 实现设备监控和更新
}
```

#### 2.2 NVIDIA插件示例
```go
// pkg/device-plugin/nvidia/plugin.go
type NvidiaDevicePlugin struct {
    devices []*pluginapi.Device
    server  *grpc.Server
}

func (p *NvidiaDevicePlugin) Allocate(ctx context.Context, reqs *pluginapi.AllocateRequest) (*pluginapi.AllocateResponse, error) {
    // 实现设备分配逻辑
}
```

### 3. 调度器 (`pkg/scheduler`)

#### 3.1 调度策略
```go
// pkg/scheduler/policy/policy.go
type SchedulerPolicy interface {
    Filter(pod *corev1.Pod, node *corev1.Node) bool
    Score(pod *corev1.Pod, node *corev1.Node) int64
}

// pkg/scheduler/policy/binpack.go
type BinpackPolicy struct{}

func (p *BinpackPolicy) Score(pod *corev1.Pod, node *corev1.Node) int64 {
    // 实现Binpack策略评分
}
```

#### 3.2 节点管理
```go
// pkg/scheduler/nodes.go
type NodeManager struct {
    nodes map[string]*NodeInfo
}

func (nm *NodeManager) UpdateNode(node *corev1.Node) {
    // 实现节点状态更新
}
```

### 4. 监控系统 (`pkg/monitor`)

#### 4.1 设备监控接口
```go
// pkg/monitor/monitor.go
type DeviceMonitor interface {
    GetMetrics() (*DeviceMetrics, error)
    WatchDevices() (<-chan *DeviceEvent, error)
}
```

#### 4.2 NVIDIA监控实现
```go
// pkg/monitor/nvidia/monitor.go
type NvidiaMonitor struct {
    devices []*nvml.Device
}

func (m *NvidiaMonitor) GetMetrics() (*DeviceMetrics, error) {
    // 实现NVIDIA设备指标收集
}
```

### 5. OCI运行时集成 (`pkg/oci`)

#### 5.1 运行时接口
```go
// pkg/oci/runtime.go
type Runtime interface {
    CreateContainer(config *RuntimeConfig) error
    StartContainer(id string) error
    StopContainer(id string) error
}
```

#### 5.2 设备挂载
```go
// pkg/oci/device.go
func ConfigureDeviceMounts(spec *specs.Spec, devices []string) error {
    // 实现设备挂载配置
}
```

### 6. Kubernetes工具 (`pkg/k8sutil`)

#### 6.1 Pod操作
```go
// pkg/k8sutil/pod.go
func UpdatePodStatus(client kubernetes.Interface, pod *corev1.Pod) error {
    // 实现Pod状态更新
}
```

#### 6.2 资源管理
```go
// pkg/k8sutil/resource.go
func ValidateResourceRequirements(pod *corev1.Pod) error {
    // 实现资源验证
}
```

### 7. 通用工具 (`pkg/util`)

#### 7.1 资源锁定
```go
// pkg/util/nodelock/lock.go
type NodeLock struct {
    name      string
    namespace string
}

func (l *NodeLock) Lock() error {
    // 实现分布式锁
}
```

#### 7.2 配置管理
```go
// pkg/util/flag/flag.go
func ParseFlags() (*Config, error) {
    // 实现配置解析
}
```

## 测试覆盖率

项目的主要测试文件分布：

```bash
pkg/
├── device/
│   └── devices_test.go      # 设备管理测试
├── scheduler/
│   └── scheduler_test.go    # 调度器测试
├── monitor/
│   └── monitor_test.go      # 监控系统测试
└── util/
    └── util_test.go         # 工具函数测试
```

## 项目依赖

核心依赖配置：

```go
// go.mod
module github.com/Project-HAMi/HAMi

require (
    k8s.io/api v0.24.0
    k8s.io/apimachinery v0.24.0
    k8s.io/client-go v0.24.0
    k8s.io/klog/v2 v2.60.1
)
```

## 当前开发状态

Next Version: v2.6.0

关于我们最新的发布和功能，请查看：
- [HAMi 2.5.0 发布说明](https://www.theriseunion.com/zh/blog/hami-2-5-0.html)
- [HAMi 2.4.0 发布说明](https://www.theriseunion.com/zh/blog/hami-2-4-0.html)
- [HAMi 动态 MIG 支持](https://www.theriseunion.com/zh/blog/hami-dynamic-mig.html)
- [HAMi vGPU 介绍](https://www.theriseunion.com/zh/blog/hami-vgpu-intro.html)

Tasks:
- 支持昆仑芯 sGPU /assign @ouyangluwei163
- 支持沐曦智能 sGPU
- 支持燧原科技 sGPU
- 优化调度器事件
- 网站优化 /assign @ouyangluwei163 @Nimbus318
- 添加沐曦智能到网站
- 支持驱动 570+CUDA 12.8 /assign @archlitchi
- DRA 设计 /assign @Shouren
- 支持多种寒武纪型号 (370,590等)

详细参考：[HAMi RoadMap](https://github.com/Project-HAMi/HAMi/issues/923)

---

[在睿思智联官网阅读本文](https://www.theriseunion.com/zh/blog/hami-code-analyst-1.html) · [浏览更多文章](https://www.theriseunion.com/zh/blog.html)
