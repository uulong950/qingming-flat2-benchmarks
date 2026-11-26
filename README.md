# Qingming Flat2 Benchmarks

**跨实现性能对比与数值审计证据平台**  
Cross-implementation performance comparisons and numerical audit evidence.

[查看基准 / Explore benchmarks](https://uulong950.github.io/qingming-flat2-benchmarks/) · [核心引擎 / Core engine](https://github.com/uulong950/qingming-flat2)

统一数学问题。在同一设备、同一数据集和同一协议下比较不同精确向量检索实现。这里公开结果与证据，核心引擎在独立仓库维护。

Compare exact vector search implementations on the same device, dataset and measurement protocol. This repository publishes results and evidence; the core engine is maintained separately.

## 两个仓库 / Two repositories

| Repository | Purpose |
|---|---|
| **qingming-flat2-benchmarks** | 性能、正确性、数值审计与公开证据 / Benchmarking and audit evidence |
| [**qingming-flat2**](https://github.com/uulong950/qingming-flat2) | 核心算法、原生 GPU 算子与硬件适配 / Core algorithms, native GPU kernels and hardware mapping |

## 当前覆盖 / Coverage

**2 devices · 6 datasets · 12 device–dataset combinations** · v0.11.0

| Device | Baselines |
|---|---|
| MetaX C500 (25% compute sGPU) | mcFaiss |
| AMD Radeon RX 7900 XTX | hipVS, Faiss ROCm |

SIFT1M · GIST960 · Llama-128-IP · LAION-CLIP-512 · GloVe-100-Angular · DEEP10M

## 证据 / Evidence

[查看完整索引 / Dataset index](results-devices/index.json)

- `performance.csv` — 延迟、吞吐量、加速比 / Latency, throughput, speedup
- `numerical.csv` — Top-K 正确性与 CPU FP64 / FP32 误差审计 / Correctness and numerical audits
- `samples.csv` — 原始计时采样 / Raw timing observations
- `environment.json` — 环境、协议、来源与限制 / Environment, protocol, provenance and limitations

## 测量边界 / What the results mean

只做同设备内对照，不对设备排名。GPU Event 与同步 Host API 计时不等于端到端服务延迟；批量 QPS 不等于持续服务吞吐量。全库排名审计采用抽样，返回分数审计覆盖确认轮次的返回结果。公开采样支持统计复算；核心内核及完整返回数组未公开，不能仅凭此仓库进行源码级复现或独立完整数值复审。各数据集的具体协议与限制见环境证据。

Within-device comparisons only, not hardware rankings. GPU and synchronized host timings are not end-to-end service latency; batch QPS is not sustained throughput. Full-database ranking checks are sampled; returned-score audits cover holdout outputs. Public samples support statistical recomputation, not source-level reproduction or a complete independent numerical re-audit. See the environment evidence for dataset-specific protocols and limitations.
