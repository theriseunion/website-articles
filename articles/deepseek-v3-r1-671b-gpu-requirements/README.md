[English](./README.en.md) · [文章目录](../../README.md) · [官网阅读](https://www.theriseunion.com/zh/blog/deepseek-v3-r1-671b-gpu-requirements.html)

# DeepSeek-V3/R1 671B 模型部署指南：GPU 资源需求详解

睿思智联 · 2025/03/11 08:00（北京时间）

全面解析 DeepSeek-V3 和 R1 671B 模型的硬件部署要求，包括 NVIDIA 和昇腾 GPU 的显存、计算精度和带宽需求。提供从满血版到蒸馏版的详细配置建议，帮助企业选择最适合的硬件方案，实现高效模型部署。

![文章封面](./images/deepseek-models.jpg)

## 背景  

随着 [DeepSeek V3 与 R1 系列模型](https://www.theriseunion.com/zh/blog/deepseek-r1-vs-v3.html)的发布，其强大性能迅速引发国内外 AI 领域的广泛关注。其中，[DeepSeek-V3 采用的 MoE（混合专家模型，Mixture of Experts）架构](https://www.theriseunion.com/zh/blog/deepseek-moe.html)更是成为业界学习和借鉴的对象，使 DeepSeek 站上了 AI 大模型技术的前沿。  

近期，DeepSeek 系列模型在企业环境中的本地化部署需求持续攀升，尤其是 **DeepSeek-V3 和 R1 671B 的满血版**，以及面向高效推理优化的 **蒸馏版**（Distilled Model）。然而，[DeepSeek 671B满血版超大规模模型的部署](https://www.theriseunion.com/zh/blog/deepseek-v3-r1-671b-intro.html)对计算资源提出了极高要求，涉及 GPU 计算力、显存、通信带宽等多个关键因素。

本篇文章将围绕企业级服务器环境，深入探讨 DeepSeek 满血版与蒸馏版的计算需求，并分析适用于不同 GPU 硬件的部署方案，帮助企业更高效地利用 AI 计算资源。


## 主流 Nvidia GPU 配置差异

### **GPU 规格对比表**  

| **参数/指标**  | **A100 (80GB)** | **RTX 4090 (标准版)** | **RTX 4090 (48GB)** | **H20（标准版）** | **H20（141GB）** | **H200** |  
|--------------|---------------|-----------------|----------------|--------------|--------------|------|  
| **显存容量**  | 80GB          | 24GB            | 48GB           | 96GB         | 141GB        | 141GB  |  
| **显存类型**  | HBM2e         | GDDR6X          | GDDR6X         | HBM3         | HBM3      | HBM3e  |  
| **显存带宽**  | 2TB/s         | 1TB/s           | 1TB/s          | 4TB/s        | 4TB/s      | 4.8TB/s  |  
| **FP32算力** | 19.5 TFLOPS   | 82.6 TFLOPS     | 82.6 TFLOPS    | 44 TFLOPS    | 44 TFLOPS    | 66.9 TFLOPS  |  
| **FP16算力** | 312 TFLOPS    | 165.3 TFLOPS    | 165.3 TFLOPS   | 148 TFLOPS   | 148 TFLOPS | 1,979 TFLOPS  |  
| **FP8算力**  | **不支持**    | **不支持**      | **不支持**     | 296 TFLOPS   | 296 TFLOPS | 3,958 TFLOPS  |  
| **INT8算力** | 624 TOPS      | 661 TOPS        | 661 TOPS       | 296 TOPS     | 296 TOPS   | 3,958 TOPS  |  
| **NPU 互联**  | NVLink 600GB/s | **不支持**      | **不支持**     | NVLink 900GB/s | NVLink 900GB/s | NVLink 900GB/s  |  
| **TDP 功耗** | 400W          | 450W            | 450W           | 400W         | 400W         | 700W  |  

---

## 显卡的三个关键指标

### 1. 显存容量（VRAM）
在深度学习和高性能计算中，显存容量（VRAM） 是决定模型能否顺利运行的关键因素之一。显存容量直接影响模型的可处理规模、推理效率以及训练/推理的稳定性。
DeepSeek 不同版本的模型，参数规模从数十亿（1.5B、7B、8B、14B、32B 和 70B）到上千亿参数（671B 满血版），显存不足会导致模型无法加载或运行效率低下。

### 2. 计算精度（FP8 vs. INT8）

DeepSeek 采用原生 FP8 进行训练和推理，因此在 FP8 支持的 GPU 上，显存占用与参数量近似（671B ≈ 700GB）。

然而，并非所有 GPU 都具备 FP8 计算单元（硬件不支持）。

对于硬件不支持 FP8 但支持 BF16 的 GPU（如 Nvidia A100，昇腾 910B 等），需要转换成支持的类型后才能运行模型，但由于精度提升（BF16 > FP8），导致显存占用将翻倍（约 1.4T）。[> 进一步了解 DeepSeek 满血版定义与不同形态的差异](https://www.theriseunion.com/zh/blog/deepseek-v3-r1-671b-intro.html)

 [DeepSeek-R1 蒸馏版系列模型](https://www.theriseunion.com/zh/blog/deepseek-r1-models-intro.html)在训练和推理时，**原生支持 BF16** 计算格式，无需额外的精度转换。相比传统的 FP16（Half-Precision Floating Point）格式，BF16 具有更宽的指数范围，使其在深度学习训练和推理过程中更加稳定，尤其是在大规模模型的计算环境下。

### 3. 显存带宽

显存带宽直接影响数据传输速度和推理效率，尤其在深度学习推理和训练过程中，大量的模型参数、激活值和 KV Cache 需要在 GPU 内部或 GPU 之间传输。如果显存带宽不足，数据传输将成为性能瓶颈，影响计算效率。  

---

## DeepSeek 满血版及蒸馏版模型的部署需求

### Nvidia 主流卡的支持情况

| 模型版本 | 参数规模 | 计算精度 | 显卡 | 最小部署需求 |  
|----------|----------|----------|------------|------------|  
| **DeepSeek-V3、R1 满血版** | 671B | **FP8（原生支持）** | H200、H20、H100、H800 | **8x** H200 (141GB) / **8x** H20 (141GB) <br/> **16x** H100 (80GB) / **16x** H800 (80GB) |  
| **DeepSeek-V3、R1 满血版** | 671B | **BF16（转换）** | A100、A800 | **16x** A100 (80GB) / **16x** A800 (80GB)|  
| DeepSeek-R1-Distill-Llama-70B | 70B | **BF16（原生支持）** | H20、H100、L20、RTX 4090 | **4x** H100 (80GB) / **4x** A100 (80GB) <br/>  **8x** L20 (48GB) / **8x** RTX 4090 (48GB) |  
| DeepSeek-R1-Distill-Qwen-32B | 32B | **BF16（原生支持）** | RTX 4090 | **1x** A100 (80GB)  <br/>  **4x** RTX 4090 (24GB) |  
| DeepSeek-R1-Distill-Qwen-14B | 14B | **BF16（原生支持）** | RTX 4090 | **1x** RTX 4090 (24GB) |  
| DeepSeek-R1-Distill-Qwen-7B | 7B | **BF16（原生支持）** | RTX 4090 | **1x** RTX 4090 (24GB)|


### 昇腾卡的支持情况

- DeepSeek-V3 和 R1 671B 的 FP8 版本无法直接运行在昇腾 910B，因为其硬件不支持 FP8，需要转换为 BF16，导致模型占用显存翻倍（约 1.4T）。
- 软件适配基于 MindIE，官方已快速提供支持。
- 因此，完整推理需要 32 张昇腾 910B（每张 64GB 显存），硬件及部署要求，参考：[昇腾环境部署DeepSeek-V3](https://www.hiascend.com/software/modelzoo/models/detail/678bdeb4e1a64c9dae51d353d84ddd15) 和 [昇腾环境部署DeepSeek-R1](https://www.hiascend.com/software/modelzoo/models/detail/68457b8a51324310aad9a0f55c3e56e3)。




| 模型版本                  | 参数规模   | 计算精度 | 加速芯片    | 最小部署需求|
|----------|----------|----------|------------|------------|  
| **DeepSeek-V3、R1 满血版** | 671B| **W8A8（量化转换）** | Ascend 910B | **2台** Atlas 800I A2(8x64GB)服务器 |
| **DeepSeek-V3、R1 满血版**| 671B | **BF16（转换）**     | Ascend 910B | **4台** Atlas 800I A2(8x64GB)服务器 |
| DeepSeek-R1-Distill-Llama-70B | 70B       | **BF16（原生支持）**     | Ascend 910B | **1台** Atlas 800I A2(8x64GB)服务器 |
| DeepSeek-R1-Distill-Qwen-32B | 32B       | **BF16（原生支持）**     | Ascend 910B、Ascend 310P | **1台** Atlas 800I A2(8x32GB)服务器 <br/>**1台** Atlas 300I DUO卡(1x96GB)的服务器                          |
| DeepSeek-R1-Distill-Qwen-14B | 14B       | **BF16（原生支持）**     | Ascend 310P | **1台** Atlas 300I DUO卡(4x48G)的服务器                               |
| DeepSeek-R1-Distill-Qwen-7B  | 7B        | **BF16（原生支持）**     | Ascend 310P | **1台** Atlas 300I DUO卡(1x48G)的服务器                               |
| DeepSeek-R1-Distill-Llama-8B  | 8B        | **BF16（原生支持）**     | Ascend 310P | **1台** Atlas 300I DUO卡(1x48G)的服务器 |
| DeepSeek-R1-Distill-Qwen-1.5B | 1.5B       | **BF16（原生支持）**     | Ascend 310P | **1台** Atlas 300I DUO卡(1x48G)的服务器                               |


## Rise MAX-DS 池化一体机：企业级部署的最优解

面对 DeepSeek 大模型日益增长的部署需求，传统的单机部署方案已难以满足企业的实际需求。[Rise MAX-DS 池化一体机](https://www.theriseunion.com/zh/product/max.html)提供了更灵活、更高效的部署方案：

### 智能算力池化与调度
- 支持多个 DeepSeek 模型（从 1.5B 到 671B）在同一算力池内协同运行
- 动态调整算力分配，GPU 利用率提升 30%+
- 自动负载均衡，避免资源浪费和低效占用

### 弹性扩展能力
- 支持按需扩容，无需重构架构
- 突破单机物理限制，实现资源池化共享
- 支持云边协同部署，优化资源配置

### 降低部署成本与提高效率
- 减少初始硬件投入，按需扩展
- 提高资源利用效率，降低运营成本
- 简化运维管理，降低人力成本

更多关于 Rise MAX-DS 池化一体机的详细信息，请参阅[DeepSeek AI 计算一体机详解](https://www.theriseunion.com/zh/product/max.html)。

## 结论

1. 为了发挥 DeepSeek 的原生水平，在不量化的前提下，FP8 计算单元是决定 DeepSeek 部署效果的关键因素。
2. H100、H200、H800、H20 由于原生支持 FP8，在 DeepSeek 推理中优势明显。为了避免网络开销，优先选择可以部署满血的单机版（如：H200、H20）。
3. A100/A800 由于硬件不支持 FP8，部署 671B 版本显存需求翻倍。
4. 昇腾 910B 由于硬件不支持 FP8，需要进行 BF16 转换，且需要更多显存和设备才能完成满血版的部署。或者进行 W8A8 量化，以支持 671B 的更小资源部署。

随着量化技术和分布式框架的成熟，DeepSeek 的高性价比和卓越性能使得更多企业能够负担得起高性能大模型的本地部署，直接推动了算力平民化，加速 AI 技术的普及和应用。

---

## 附录

### 显卡显存带宽
例如，H20 具备 **900GB/s** 的 NVLink 带宽，高于 A100 的 **600GB/s**，并且远超 **PCIe 5.0（128GB/s）** 的 **7 倍**，这使得 H20 在多 GPU 计算时能有效减少通信延迟，提高整体计算效率。  

#### 显存带宽对模型运行的影响
**单卡运行**：
   - 显存带宽主要影响模型数据的加载速度，包括参数、激活值、KV Cache 的传输。  
   - FP8/BF16 等低精度计算模式对带宽的要求较低，但当模型超过单张 GPU 的显存容量时，需要频繁交换数据，这会受到带宽的限制。  

**单机多卡运行（NVLink 互联）**：
   - **高带宽（如 H20 的 900GB/s）** 可以加速多 GPU 之间的参数同步和 KV Cache 共享，减少通信开销，提高推理和训练效率。  
   - 例如，在多卡推理 671B 级别模型时，如果带宽不足，KV Cache 的同步会成为瓶颈，影响推理速度。  
   - A100 采用 NVLink **600GB/s**，在单机多卡模式下表现尚可，但相比 H20 的 **900GB/s**，通信效率较低。  

**多机多卡运行（NVLink + InfiniBand）**：
   - 当单机 GPU 数量不足时，需要使用 **InfiniBand**（如 200Gb/s、400Gb/s）连接多台服务器。  
   - **NVLink（机内高速通信）+ InfiniBand（机间通信）** 的组合可提升跨机器通信效率，但 InfiniBand 带宽通常远低于 NVLink，因此多机部署时通信开销更高。  
   - 如果模型需要跨节点进行张量并行（Tensor Parallel）或流水线并行（Pipeline Parallel），低带宽的 InfiniBand 会成为性能瓶颈。  

#### 小结
- 显存带宽越高，数据传输速度越快，能够减少 GPU 内部和 GPU 之间的通信开销，提高推理和训练效率。  
- **单机多卡模式下**，高带宽 NVLink（如 H20 的 900GB/s）能有效降低跨 GPU 数据传输延迟，提高多 GPU 计算效率。  
- **多机多卡模式下**，虽然 InfiniBand 提供机间互联，但带宽通常低于 NVLink，可能成为模型训练和推理的通信瓶颈，特别是在大型模型（如 671B 级）部署时。

---

### 昇腾 DeepSeek 的 W8A8 和 W8A16 量化是什么意思？

在 AI 计算中，**W8A8** 和 **W8A16** 代表了一种**低比特量化（Low-bit Quantization）**技术，用于减少模型的计算开销和显存占用，同时尽可能保持推理精度。这些量化方案主要用于适配华为昇腾（Ascend）系列 AI 加速器，使 DeepSeek 模型能够高效运行。  

#### 1. W8A8 量化
- **W8（权重量化 8-bit）**：权重（Weights）被量化为 **8-bit**（int8 或 uint8）。
- **A8（激活量化 8-bit）**：激活值（Activations，即中间计算结果）也被量化为 **8-bit**（int8 或 uint8）。

**特点**：
  - 显著减少计算需求和存储占用，相比 FP16 可节省 **50%** 显存。
  - 适用于**极致性能优化**的场景，如高吞吐率 AI 服务器推理部署。
  - 由于激活值也采用 8-bit 量化，相比 A16 量化，可能会引入更大的精度损失。

#### 2. W8A16 量化
- **W8（权重量化 8-bit）**：权重（Weights）仍然量化为 **8-bit**（int8 或 uint8）。
- **A16（激活量化 16-bit）**：激活值（Activations）采用 **16-bit**（通常是 FP16 或 BF16）。

**特点**：
  - 计算复杂度介于 W8A8 和 FP16 之间，计算性能有所提升，同时保留部分 FP16 的精度优势。
  - 显存占用比 FP16 **减少 25%-30%**，但比 W8A8 **略高**。
  - 适用于**需要更高精度**的推理任务，如对数值稳定性要求较高的 NLP 或 CV 任务。

#### 3. 适配昇腾（Ascend）的意义
华为昇腾（如 **Ascend 910B、Ascend 310P**）的 AI 计算单元（Cube Unit）对低比特量化计算有专门优化，尤其是在 INT8 和 FP16 计算上提供高效加速。  
DeepSeek 适配 **W8A8** 和 **W8A16** 量化，意味着其模型可以在 **昇腾** 平台上高效运行，减少功耗和计算资源消耗，同时优化推理吞吐量。  


#### 4. 选择 W8A8 vs. W8A16 的场景
| 方案  | 计算速度 | 显存占用 | 精度损失 | 适用场景 |
|------|--------|--------|--------|--------|
| **W8A8** | 最高 | 最低 | 相对较大 | 超高吞吐率推理，如搜索、推荐、实时 AI 应用 |
| **W8A16** | 中等 | 中等 | 精度较高 | 需要更高精度的 AI 任务，如 NLP 大模型推理 |

#### 5. 总结
- **W8A8**：计算和显存优化最大化，适用于高吞吐场景，可能影响推理精度。  
- **W8A16**：在计算和精度之间取得平衡，适用于需要较高精度的 AI 任务。  

如果你的目标是**在 Ascend 硬件上高效部署 DeepSeek**，那么 **W8A8** 适用于极致性能优化，而 **W8A16** 适用于平衡精度与性能的场景。

---

### Nvidia 显卡系列架构

| 架构 | Pascal | Volta | Turing | Ampere | Ada | Hopper | Blackwell |  
|------|--------|-------|--------|--------|-----|--------|----------|  
| 发布时间 | 2016 | 2017 | 2018 | 2020 | 2022 | 2022 | 2024 |  
| 典型显卡 | Tesla P40、GTX 1080 | Tesla V100 | T4、Quadro RTX 6000、RTX 2080 | A100、A40、RTX 3090 | RTX 6000 Ada、L40、RTX 4090 | H100、H200 | B200、RTX 5090 |  
| 中国特供 |  | V100S | T4G | A800、A800 80GB、A800 PCIe、A800 SXM4 | L20、L40S、H20 | H800 | B100 |  

---

### 昇腾 Ascend 910B 各型号及对应整机

| NPU 型号 | FP16 算力 | 显存 | 对应华为整机 |
|:---|:---|:---|:---|
| 昇腾 Ascend 910B4 | 280T | 32GB HBM2 | Atlas 800I A2 |
| 昇腾 Ascend 910B3 | 313T | 64GB HBM2 | Atlas 800T A2 |
| 昇腾 Ascend 910B2 | 376T | 64GB HBM2 | n/a |
| 昇腾 Ascend 910B1 | 414T | 64GB HBM2 | n/a |


---

### Atlas 300I DUO 和 Atlas 300I Pro 的对比

| **对比项**         | **Atlas 300I Duo**                     | **Atlas 300I Pro**                   |
|------------------|--------------------------------------|------------------------------------|
| **处理器**       | 2 颗 **Ascend 310P** AI 处理器        | 1 颗 **Ascend 310P** AI 处理器      |
| **AI 核心数**     | 8 个 AI 核                | 8 个 AI 核                          |
| **CPU 核心数**    | 8 个自研 CPU 核           | 8 个自研 CPU 核                    |
| **算力 (INT8)**   | 280 TOPS                             | 140 TOPS                            |
| **算力 (FP16)**   | 140 TFLOPS                           | 70 TFLOPS                           |
| **内存**         | 48GB 或 96GB LPDDR4X                  | 24GB LPDDR4X                        |
| **内存带宽**     | 408GB/s                               | 204.8GB/s                           |
| **功耗**         | 150W                                  | 72W                                  |
| **形态**         | 单槽位全高全长 PCIe 卡               | 单槽位半高半长 PCIe 卡              |
| **应用场景**     | 高算力需求场景：搜索推荐、内容审核、OCR 识别、视频分析等 | 中等算力需求场景：OCR 识别、语音分析、搜索推荐等 |

定位差异：
- **Atlas 300I Duo**：适用于高算力需求的任务，具有更高的算力和内存容量，适合大规模 AI 推理任务。
- **Atlas 300I Pro**：注重能效和紧凑设计，适用于中等负载的 AI 推理任务，功耗较低，设计较为紧凑。

---

[在睿思智联官网阅读本文](https://www.theriseunion.com/zh/blog/deepseek-v3-r1-671b-gpu-requirements.html) · [浏览更多文章](https://www.theriseunion.com/zh/blog.html)
