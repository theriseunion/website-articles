[简体中文](./README.md) · [All articles](../../README.en.md) · [Read on the website](https://www.theriseunion.com/en/blog/deepseek-v3-r1-671b-gpu-requirements.html)

# DeepSeek-V3/R1 671B Deployment Guide: GPU Requirements

RiseUnion · January 01, 2025 at 08:00 (UTC+08:00)

Comprehensive analysis of hardware requirements for DeepSeek-V3 and R1 671B models, covering NVIDIA and Ascend GPU memory, compute precision, and bandwidth specifications. Provides detailed configuration recommendations from full-size to distilled versions, helping enterprises select optimal hardware solutions for efficient model deployment.

![Article cover](./images/deepseek-models.jpg)

## Background

With the release of [DeepSeek V3 and R1 series models](https://www.theriseunion.com/en/blog/deepseek-r1-vs-v3.html), their powerful performance has quickly garnered widespread attention in the AI community both domestically and internationally. The [MoE (Mixture of Experts) architecture adopted by DeepSeek-V3](https://www.theriseunion.com/en/blog/deepseek-moe.html) has become a subject of study and reference in the industry, positioning DeepSeek at the forefront of large language model technology.

Recently, demand for on-premises deployment of DeepSeek models in enterprise environments has been steadily increasing, particularly for the **full-scale versions of DeepSeek-V3 and R1 671B**, as well as the **distilled models** optimized for efficient inference. However, [deploying the full-scale DeepSeek 671B models](https://www.theriseunion.com/en/blog/deepseek-v3-r1-671b-intro.html) imposes extremely high requirements on computing resources, involving multiple critical factors such as GPU compute power, memory capacity, and communication bandwidth.

This article will explore the computational requirements for both full-scale and distilled versions of DeepSeek models in enterprise server environments, analyze deployment solutions suitable for different GPU hardware, and help enterprises utilize AI computing resources more efficiently.


## Comparison of Mainstream Nvidia GPUs

### **GPU Specifications Comparison**

| **Parameter/Metric** | **A100 (80GB)** | **RTX 4090 (Standard)** | **RTX 4090 (48GB)** | **H20 (Standard)** | **H20 (141GB)** | **H200** |
|--------------|---------------|-----------------|----------------|--------------|--------------|------|
| **Memory Capacity**  | 80GB          | 24GB            | 48GB           | 96GB         | 141GB        | 141GB  |  
| **Memory Type**  | HBM2e         | GDDR6X          | GDDR6X         | HBM3         | HBM3e        | HBM3e  |  
| **Memory Bandwidth**  | 2TB/s         | 1TB/s           | 1TB/s          | 4TB/s        | 4TB/s      | 4.8TB/s  |  
| **FP32** | 19.5 TFLOPS   | 82.6 TFLOPS     | 82.6 TFLOPS    | 44 TFLOPS    | 44 TFLOPS    | 66.9 TFLOPS  |  
| **FP16** | 312 TFLOPS    | 165.3 TFLOPS    | 165.3 TFLOPS   | 148 TFLOPS   | 148 TFLOPS | 1,979 TFLOPS  |  
| **FP8**  | **Not Supported**    | **Not Supported**      | **Not Supported**     | 3,958 TFLOPS   | 3,958 TFLOPS | 3,958 TFLOPS  |  
| **INT8** | 624 TOPS      | 661 TOPS        | 661 TOPS       | 296 TOPS     | 296 TOPS   | 3,958 TOPS  |  
| **NVlink**  | NVLink 600GB/s | **Not Supported**      | **Not Supported**     | NVLink 900GB/s | NVLink 900GB/s | NVLink 900GB/s  |  
| **TDP** | 400W          | 450W            | 450W           | 400W         | 400W         | 700W  |  

---

## Three Key GPU Metrics

### 1. Memory Capacity (VRAM)
In deep learning and high-performance computing, memory capacity (VRAM) is one of the critical factors determining whether a model can run smoothly. Memory capacity directly affects the model's processable scale, inference efficiency, and training/inference stability.
Different versions of DeepSeek models range from billions (1.5B, 7B, 8B, 14B, 32B, and 70B) to hundreds of billions of parameters (671B full version). Insufficient memory can prevent models from loading or cause low operational efficiency.

### 2. Computation Precision (FP8 vs. INT8)

DeepSeek uses native FP8 for training and inference, so on GPUs with FP8 support, memory usage approximates the parameter count (671B ≈ 700GB).

However, not all GPUs have FP8 compute units (hardware support).

For GPUs that don't support FP8 but support BF16 (such as Nvidia A100, Ascend 910B, etc.), the model must be converted to a supported format before running. Due to the increased precision (BF16 > FP8), this results in doubled memory usage (approximately 1.4TB). [> Learn more about DeepSeek full version definitions and different forms](https://www.theriseunion.com/en/blog/deepseek-v3-r1-671b-intro.html)

[DeepSeek-R1 distilled model series](https://www.theriseunion.com/en/blog/deepseek-r1-models-intro.html) **natively support BF16** computation format for training and inference, requiring no additional precision conversion. Compared to traditional FP16 (Half-Precision Floating Point) format, BF16 offers a wider exponent range, making it more stable in deep learning training and inference processes, especially in large-scale model computing environments.

### 3. Memory Bandwidth

Memory bandwidth directly affects data transfer speed and inference efficiency, especially in deep learning inference and training processes where large amounts of model parameters, activation values, and KV Cache need to be transferred within or between GPUs. If memory bandwidth is insufficient, data transfer becomes a performance bottleneck, affecting computational efficiency.

---

## Deployment Requirements for DeepSeek Full-Scale and Distilled Models

### Support for Mainstream Nvidia Cards

| Model Version | Parameter Scale | Computation Precision | GPU | Minimum Deployment Requirements |
|----------|----------|----------|------------|------------|
| **DeepSeek-V3, R1 Full Version** | 671B | **FP8 (Native Support)** | H200, H20, H100, H800 | **8x** H200 (141GB) / **8x** H20 (141GB) <br/> **16x** H100 (80GB) / **16x** H800 (80GB) |
| **DeepSeek-V3, R1 Full Version** | 671B | **BF16 (Converted)** | A100, A800 | **16x** A100 (80GB) / **16x** A800 (80GB) |
| DeepSeek-R1-Distill-Llama-70B | 70B | **BF16 (Native Support)** | H20, H100, L20, RTX 4090 | **4x** H100 (80GB) / **4x** A100 (80GB) <br/> **8x** L20 (48GB) / **8x** RTX 4090 (48GB) |
| DeepSeek-R1-Distill-Qwen-32B | 32B | **BF16 (Native Support)** | RTX 4090 | **1x** A100 (80GB) <br/> **4x** RTX 4090 (24GB) |
| DeepSeek-R1-Distill-Qwen-14B | 14B | **BF16 (Native Support)** | RTX 4090 | **1x** RTX 4090 (24GB) |
| DeepSeek-R1-Distill-Qwen-7B | 7B | **BF16 (Native Support)** | RTX 4090 | **1x** RTX 4090 (24GB) |


### Support for Ascend Cards

- DeepSeek-V3 and R1 671B FP8 versions cannot run directly on Ascend 910B as the hardware doesn't support FP8. They need to be converted to BF16, doubling the model's memory footprint (approximately 1.4TB).
- Software adaptation is based on MindIE, with official support rapidly provided.
- Therefore, complete inference requires 32 Ascend 910B cards (each with 64GB memory). For hardware and deployment requirements, refer to: [Deploying DeepSeek-V3 in Ascend Environment](https://www.hiascend.com/software/modelzoo/models/detail/678bdeb4e1a64c9dae51d353d84ddd15) and [Deploying DeepSeek-R1 in Ascend Environment](https://www.hiascend.com/software/modelzoo/models/detail/68457b8a51324310aad9a0f55c3e56e3).


| Model Version | Parameter Scale | Computation Precision | Accelerator | Minimum Deployment Requirements |
|----------|----------|----------|------------|------------|
| **DeepSeek-V3, R1 Full Version** | 671B | **W8A8 (Quantized)** | Ascend 910B | **2** Atlas 800I A2 (8x64GB) servers |
| **DeepSeek-V3, R1 Full Version** | 671B | **BF16 (Converted)** | Ascend 910B | **4** Atlas 800I A2 (8x64GB) servers |
| DeepSeek-R1-Distill-Llama-70B | 70B | **BF16 (Native Support)** | Ascend 910B | **1** Atlas 800I A2 (8x64GB) server |
| DeepSeek-R1-Distill-Qwen-32B | 32B | **BF16 (Native Support)** | Ascend 910B, Ascend 310P | **1** Atlas 800I A2 (8x32GB) server <br/>**1** Atlas 300I DUO (1x96GB) server |
| DeepSeek-R1-Distill-Qwen-14B | 14B | **BF16 (Native Support)** | Ascend 310P | **1** Atlas 300I DUO (4x48G) server |
| DeepSeek-R1-Distill-Qwen-7B | 7B | **BF16 (Native Support)** | Ascend 310P | **1** Atlas 300I DUO (1x48G) server |
| DeepSeek-R1-Distill-Llama-8B | 8B | **BF16 (Native Support)** | Ascend 310P | **1** Atlas 300I DUO (1x48G) server |
| DeepSeek-R1-Distill-Qwen-1.5B | 1.5B | **BF16 (Native Support)** | Ascend 310P | **1** Atlas 300I DUO (1x48G) server |


## Rise MAX-DS Pooled Appliance: Optimal Solution for Enterprise Deployment

With the growing deployment demands for DeepSeek large models, traditional single-machine deployment solutions can no longer meet enterprise requirements. The [Rise MAX-DS Pooled Appliance](https://www.theriseunion.com/en/product/max.html) provides a more flexible and efficient deployment solution:

### Intelligent Compute Pooling and Scheduling
- Supports multiple DeepSeek models (from 1.5B to 671B) running collaboratively within the same compute pool
- Dynamically adjusts compute allocation, improving GPU utilization by 30%+
- Automatic load balancing, avoiding resource waste and inefficient occupation

### Elastic Scaling Capability
- Supports on-demand expansion without architecture reconstruction
- Breaks through single-machine physical limitations, achieving resource pooling and sharing
- Supports cloud-edge collaborative deployment, optimizing resource configuration

### Reduced Deployment Costs and Improved Efficiency
- Reduces initial hardware investment with on-demand expansion
- Improves resource utilization efficiency, lowering operational costs
- Simplifies operations management, reducing labor costs

For more detailed information about the Rise MAX-DS Pooled Appliance, please refer to [DeepSeek AI Computing Appliance Details](https://www.theriseunion.com/en/product/max.html).

## Conclusion

1. To leverage DeepSeek's native performance without quantization, FP8 compute units are the key factor determining DeepSeek deployment effectiveness.
2. H100, H200, H800, and H20 have clear advantages in DeepSeek inference due to native FP8 support. To avoid network overhead, single-machine deployments capable of running full models (such as H200, H20) are preferred.
3. A100/A800 require double the memory for deploying 671B versions due to lack of hardware FP8 support.
4. Ascend 910B requires BF16 conversion due to lack of hardware FP8 support, necessitating more memory and devices to complete full-version deployment. Alternatively, W8A8 quantization can be used to support 671B deployment with fewer resources.

As quantization techniques and distributed frameworks mature, DeepSeek's cost-effectiveness and excellent performance enable more enterprises to afford on-premises deployment of high-performance large models, directly promoting the democratization of computing power and accelerating the popularization and application of AI technology.

---

## Appendix

### GPU Memory Bandwidth
For example, H20 has **900GB/s** NVLink bandwidth, higher than A100's **600GB/s**, and **7 times** higher than **PCIe 5.0 (128GB/s)**. This allows H20 to effectively reduce communication latency in multi-GPU computing, improving overall computational efficiency.

#### Impact of Memory Bandwidth on Model Performance
**Single GPU Operation**:
   - Memory bandwidth primarily affects the loading speed of model data, including parameters, activation values, and KV Cache transfer.
   - Low-precision computation modes like FP8/BF16 have lower bandwidth requirements, but when models exceed the memory capacity of a single GPU, frequent data swapping becomes limited by bandwidth.

**Single-Node Multi-GPU Operation (NVLink Interconnect)**:
   - **High bandwidth (such as H20's 900GB/s)** can accelerate parameter synchronization and KV Cache sharing between multiple GPUs, reducing communication overhead and improving inference and training efficiency.
   - For example, when inferencing 671B-level models across multiple GPUs, insufficient bandwidth makes KV Cache synchronization a bottleneck, affecting inference speed.
   - A100 uses NVLink at **600GB/s**, performing adequately in single-node multi-GPU mode, but with lower communication efficiency compared to H20's **900GB/s**.

**Multi-Node Multi-GPU Operation (NVLink + InfiniBand)**:
   - When single-node GPU count is insufficient, **InfiniBand** (such as 200Gb/s, 400Gb/s) is used to connect multiple servers.
   - The combination of **NVLink (high-speed intra-node communication) + InfiniBand (inter-node communication)** can improve cross-machine communication efficiency, but InfiniBand bandwidth is typically much lower than NVLink, resulting in higher communication overhead for multi-node deployment.
   - If the model requires cross-node Tensor Parallel or Pipeline Parallel operations, low-bandwidth InfiniBand becomes a performance bottleneck.

#### Summary
- Higher memory bandwidth means faster data transfer, reducing communication overhead within and between GPUs, improving inference and training efficiency.
- In **single-node multi-GPU mode**, high-bandwidth NVLink (such as H20's 900GB/s) effectively reduces cross-GPU data transfer latency, improving multi-GPU computational efficiency.
- In **multi-node multi-GPU mode**, although InfiniBand provides inter-node connectivity, its bandwidth is typically lower than NVLink, potentially becoming a communication bottleneck for model training and inference, especially when deploying large models (such as 671B-level).

---

### What do W8A8 and W8A16 Quantization Mean for Ascend DeepSeek?

In AI computing, **W8A8** and **W8A16** represent **low-bit quantization** techniques used to reduce model computational overhead and memory usage while maintaining inference accuracy as much as possible. These quantization schemes are primarily used to adapt to Huawei Ascend series AI accelerators, enabling DeepSeek models to run efficiently.

#### 1. W8A8 Quantization
- **W8 (Weight Quantization 8-bit)**: Weights are quantized to **8-bit** (int8 or uint8).
- **A8 (Activation Quantization 8-bit)**: Activation values (intermediate computation results) are also quantized to **8-bit** (int8 or uint8).

**Characteristics**:
  - Significantly reduces computational requirements and storage usage, saving **50%** memory compared to FP16.
  - Suitable for **extreme performance optimization** scenarios, such as high-throughput AI server inference deployment.
  - Since activation values also use 8-bit quantization, compared to A16 quantization, it may introduce greater precision loss.

#### 2. W8A16 Quantization
- **W8 (Weight Quantization 8-bit)**: Weights are still quantized to **8-bit** (int8 or uint8).
- **A16 (Activation Quantization 16-bit)**: Activation values use **16-bit** (typically FP16 or BF16).

**Characteristics**:
  - Computational complexity between W8A8 and FP16, with improved computational performance while retaining some of FP16's precision advantages.
  - Memory usage **reduced by 25%-30%** compared to FP16, but **slightly higher** than W8A8.
  - Suitable for inference tasks requiring **higher precision**, such as NLP or CV tasks with higher numerical stability requirements.

#### 3. Significance for Ascend Adaptation
Huawei Ascend (such as **Ascend 910B, Ascend 310P**) AI compute units (Cube Units) are specially optimized for low-bit quantized computation, particularly efficient in INT8 and FP16 calculations.
DeepSeek's adaptation to **W8A8** and **W8A16** quantization means its models can run efficiently on the **Ascend** platform, reducing power consumption and computational resource usage while optimizing inference throughput.

#### 4. Choosing Between W8A8 vs. W8A16
| Scheme | Computation Speed | Memory Usage | Precision Loss | Suitable Scenarios |
|------|--------|--------|--------|--------|
| **W8A8** | Highest | Lowest | Relatively Larger | Ultra-high throughput inference: search, recommendations, real-time AI applications |
| **W8A16** | Medium | Medium | Higher Precision | AI tasks requiring higher precision, such as NLP large model inference |

#### 5. Summary
- **W8A8**: Maximizes computation and memory optimization, suitable for high-throughput scenarios, may affect inference precision.
- **W8A16**: Balances computation and precision, suitable for AI tasks requiring higher precision.

If your goal is **efficient deployment of DeepSeek on Ascend hardware**, then **W8A8** is suitable for extreme performance optimization, while **W8A16** is suitable for scenarios balancing precision and performance.

---

### Nvidia GPU Architecture Series

| Architecture | Pascal | Volta | Turing | Ampere | Ada | Hopper | Blackwell |
|------|--------|-------|--------|--------|-----|--------|----------|
| Release Year | 2016 | 2017 | 2018 | 2020 | 2022 | 2022 | 2024 |
| Typical GPUs | Tesla P40, GTX 1080 | Tesla V100 | T4, Quadro RTX 6000, RTX 2080 | A100, A40, RTX 3090 | RTX 6000 Ada, L40, RTX 4090 | H100, H200 | B200, RTX 5090 |
| China-specific | | V100S | T4G | A800, A800 80GB, A800 PCIe, A800 SXM4 | L20, L40S, H20 | H800 | B100 |


---

### Ascend 910B Models

| NPU Model | FP16 Performance | Memory | Corresponding Huawei Host |
|------|--------|-------|--------|--------|
| Ascend 910B4 | 280T | 32GB HBM2 | Atlas 800I A2 |
| Ascend 910B3 | 313T | 64GB HBM2 | Atlas 800T A2 |
| Ascend 910B2 | 376T | 64GB HBM2 | n/a |
| Ascend 910B1 | 414T | 64GB HBM2 | n/a |


---


### Comparison Between Atlas 300I DUO and Atlas 300I Pro

| **Comparison Item** | **Atlas 300I Duo** | **Atlas 300I Pro** |
|------------------|--------------------------------------|-----------------------------------|
| **Processor** | 2 **Ascend 310P** AI processors | 1 **Ascend 310P** AI processor |
| **AI Cores** | 8 AI cores | 8 AI cores |
| **CPU Cores** | 8 self-developed CPU cores | 8 self-developed CPU cores |
| **Compute Power (INT8)** | 280 TOPS | 140 TOPS |
| **Compute Power (FP16)** | 140 TFLOPS | 70 TFLOPS |
| **Memory** | 48GB or 96GB LPDDR4X | 24GB LPDDR4X |
| **Memory Bandwidth** | 408GB/s | 204.8GB/s |
| **Power Consumption** | 150W | 72W |
| **Form Factor** | Single-slot full-height, full-length PCIe card | Single-slot half-height, half-length PCIe card |
| **Application Scenarios** | High compute demand scenarios: search recommendations, content moderation, OCR recognition, video analysis, etc. | Medium compute demand scenarios: OCR recognition, speech analysis, search recommendations, etc. |

Positioning Difference:
- **Atlas 300I Duo**: Suitable for high compute demand tasks, with higher compute power and memory capacity, ideal for large-scale AI inference tasks.
- **Atlas 300I Pro**: Focuses on energy efficiency and compact design, suitable for medium-load AI inference tasks, with lower power consumption and more compact design.

---

[Read this article on the RiseUnion website](https://www.theriseunion.com/en/blog/deepseek-v3-r1-671b-gpu-requirements.html) · [Explore more articles](https://www.theriseunion.com/en/blog.html)
