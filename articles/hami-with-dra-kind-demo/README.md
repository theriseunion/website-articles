[English](./README.en.md) · [文章目录](../../README.md) · [官网阅读](https://www.theriseunion.com/zh/blog/hami-with-dra-kind-demo.html)

# HAMi-Core DRA Driver 在 kind 环境中的完整部署指南

睿思智联 · 2025/10/09 08:00（北京时间）

详细教程：如何在 kind 集群中部署和测试 HAMi-Core DRA Driver，实现 Kubernetes 动态资源分配和 GPU 虚拟化。包含完整的安装步骤、配置方法 and 实际演示。

![dra](./images/dra.gif)
> 本文转自：[第四范式开发者社区 - 《教程：在 kind 中体验 HAMi-Core DRA Driver》](https://zhuanlan.zhihu.com/p/1956405580425037002)


## 前言

在几周之前的 [HAMi 与 DRA 演示](https://www.theriseunion.com/zh/blog/hami-with-dra-demo.html) 中介绍了 HAMi-Core DRA Driver 的相关内容，在第四范式的开发者经过几周紧锣密鼓开发和测试之后，HAMi-Core DRA Driver 的代码已经发布到 [k8s-dra-driver](https://github.com/Project-HAMi/k8s-dra-driver) 这个仓库的主分支上了，欢迎大家前去体验

## 前置要求
在体检 HAMi-Core DRA Driver 之前，请确保 Linux Server 环境中已经安装配置了如下的软件：

- NVIDIA GPU Driver & CUDA
- Docker
- kind

### 构建镜像

```bash
# 下载代码
git clone https://github.com/Project-HAMi/k8s-dra-driver.git cd k8s-dra-driver

# 构建镜像
make image
```

当镜像构建完成之后，通过 docker image list 可以看到名为 projecthami/k8s-dra-driver:v0.0.1-dev 的镜像

### 创建集群

在执行创建的集群的脚本之前，可以通过 demo/clusters/kind/scripts/common.sh 中定义的环境变量来配置集群，比如可以通过 KIND_K8S_TAG 这个环境变量来设置希望部署的 Kubernetes 集群的版本

另外，可以查看 demo/clusters/kind/scripts/kind-cluster-config.yaml 这个文件确认 featureGates 中的 DRAConsumableCapacity 已经设置为 true；同时，如果想要配置多个 node，可以增加 role: worker 的配置；最后更多的 kind 相关的配置，可以参考 https://kind.sigs.k8s.io/docs/user/configuration

```bash
# 创建集群
./demo/clusters/kind/create-cluster.sh
```

创建集群之后，脚本会自动将 projecthami/k8s-dra-driver:v0.0.1-dev 镜像加载到集群中

### 安装 HAMi-Core DRA Driver

```bash
cd demo/yaml

# 配置 Namespace 和权限
kubectl apply -f rbac.yaml

# 创建 HAMi-Core DRA Driver 的 DaemonSet
kubectl apply -f ds.yaml
```

## 体验 HAMi-Core DRA

在体检 HAMi-Core DRA 的相关功能之前，我们需要将测试的 Pod 所需要的镜像加载到集群中去：

```bash
# 下载 Pod 运行所需的镜像
docker pull ubuntu:24.04

# 将镜像导入到 kind 集群中
kind load docker-image ubuntu:24.04 --name k8s-dra-driver-cluster
然后，我们需要配置体验所需要 DeviceClass 和 ResourceClaim 等 Kubernetes API Resource

# 配置体检 DRA 的 DeviceClass、ResourceClaim 和 ResourceClaimTemplate
kubectl apply -f setup.yaml

# 创建 Pod
kubectl create -f pod-0.yaml
在 demo/yaml 中还包含了基于 ResourceClaimTemplate 的配置方式，请参考 pod-tpl-0.yaml 来创建基于 ResourceClaimTemplate 的 Pod
```

清理环境
```bash
# 删除集群
./demo/clusters/kind/delete-cluster.sh
```

---

[在睿思智联官网阅读本文](https://www.theriseunion.com/zh/blog/hami-with-dra-kind-demo.html) · [浏览更多文章](https://www.theriseunion.com/zh/blog.html)
