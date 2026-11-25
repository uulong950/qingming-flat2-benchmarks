# qingming-flat2-benchmarks

性能 · 正确性 · 数值精度 · 可复现性证据

Performance · Correctness · Numerical Accuracy · Reproducibility Evidence

[公开基准页面 / Benchmark dashboard](https://uulong950.github.io/qingming-flat2-benchmarks/) · v0.6.0

本仓库公开同设备、同数据集和同协议下的性能、CPU 数值审计、独立确认轮次原始采样与环境证据。计算内核和内部调优仍保存在独立工程，不公开。

Public evidence for within-device comparisons under a shared search contract. Proprietary kernels and tuning traces are not included.

## 设备 / Devices

| Device | Execution environment | Baselines | Datasets |
|---|---|---|---:|
| MetaX C500 | 25% compute sGPU · 16,000 MiB quota | mcFaiss | 6 |
| AMD Radeon RX 7900 XTX 24GB | gfx1100 · Ubuntu desktop · ROCm 7.2.4 | hipVS, Faiss ROCm | 1 |

## MetaX C500 · 既有结果 / Existing results

既有 CSV、环境文件、数据集索引项原样保留。 / Existing evidence files and dataset entries are unchanged.

C500 GloVe 使用离线 FP32 归一化，CPU Oracle 依据原始向量；DEEP10M 使用抽样全库 CPU Oracle，不声称具备官方 10M GT。C500 的原始 mcFaiss 构建来源、非干净工作区状态、数据集与数值协议见其环境文件。 / GloVe preprocessing and DEEP10M oracle scope are retained in the original environment evidence, including the mcFaiss worktree qualification.

| Dataset | Qualified | GPU P50 GM speedup vs mcFaiss | Peak |
|---|---:|---:|---:|
| SIFT1M | 10/10 | 3.263× | 15.001× |
| GIST960 | 8/8 | 1.293× | 1.762× |
| Llama · Attention 128 | 8/8 | 2.547× | 8.781× |
| LAION · CLIP 512 | 8/8 | 1.696× | 4.393× |
| GloVe · 100 Angular | 10/10 | 3.173× | 16.073× |
| DEEP10M · 96D | 10/10 | 4.158× | 24.235× |

## AMD Radeon RX 7900 XTX · SIFT1M

N=1,000,000 · D=128 · squared L2 · Q=10,000 · B=1/10/100/1000/10000 · K=1/10.

| Batch | K | Qingming GPU P50 (ms) | hipVS (ms) | Faiss ROCm (ms) | vs hipVS | vs Faiss ROCm |
|---:|---:|---:|---:|---:|---:|---:|
| 1 | 1 | 0.194 | 1.730 | 3.479 | 8.926× | 17.949× |
| 1 | 10 | 0.306 | 1.200 | 4.645 | 3.915× | 15.157× |
| 10 | 1 | 0.376 | 1.271 | 4.289 | 3.376× | 11.395× |
| 10 | 10 | 0.852 | 1.596 | 4.619 | 1.872× | 5.420× |
| 100 | 1 | 2.707 | 7.538 | 6.751 | 2.784× | 2.494× |
| 100 | 10 | 2.150 | 7.732 | 5.047 | 3.597× | 2.348× |
| 1000 | 1 | 13.872 | 74.133 | 20.164 | 5.344× | 1.454× |
| 1000 | 10 | 11.148 | 78.903 | 21.393 | 7.078× | 1.919× |
| 10000 | 1 | 163.434 | 528.165 | 212.151 | 3.232× | 1.298× |
| 10000 | 10 | 86.772 | 583.193 | 210.517 | 6.721× | 2.426× |

| Baseline | Leading GPU P50 shapes | Geometric mean | Minimum | Maximum |
|---|---:|---:|---:|---:|
| hipVS | 10/10 | 4.226× | 1.872× | 8.926× |
| Faiss ROCm | 10/10 | 3.923× | 1.298× | 17.949× |

公开 30 次确认调用、390 条采样、120 条数值记录。每种 CPU 精度检查 366,663 个返回距离以及 78 条抽样查询的全库 Top-K；两种验证范围不同。完整私有实验 110 次调用审计通过，9 个常驻引擎块无中断。

Public R3 evidence: 30 invocations, 390 observations, 120 numerical rows; 366,663 returned-score checks and 78 sampled full-database Top-K query rows per precision, across all three engines. These are distinct coverage claims. Full private audit: 110 invocations, 9 resident blocks, uninterrupted.

## 协议与证据边界 / Protocol and limitations

- GPU Event: resident complete search, excluding preprocessing, indexing, H2D/D2H. API: synchronized search wall time, not end-to-end RPC. QPS is `1000 × B / GPU P50 ms`, not sustained service throughput.
- AMD 使用原始 FP32 SIFT 数据，经全量整数域检查后允许无损内部表示；不使用近似量化。 / Lossless representation of verified integer-valued FP32 SIFT inputs is permitted; no approximate quantization.
- CPU FP64/FP32 full-database ranking checks are sampled; exact distance boundary ties are allowed. Returned score checks cover all reported holdout IDs. Missing fields are not zero.
- AMD holdout repeats: B=1/10/100/1000/10000 → 21/21/11/7/5. Tail quantiles are descriptive. C500 keeps its original per-dataset protocol.
- AMD runs on a shared display/compute GPU; exclusive use and locked clocks are not claimed. C500 is partitioned. No cross-device ranking.
- AMD Faiss/hipVS source commits were not recorded at run time. Executable hashes are recorded; dependency hashes observed after the run are disclosed separately. Do not infer exact source revisions from these hashes.
- Public observations permit independent recomputation of latency statistics and inspection of audit summaries. Private kernels and full returned-ID arrays are not published; source-level reproduction and independent full numerical re-audit from this repository alone are not possible.

## 数据结构 / Layout

```text
results-devices/
  index.json
  metax-c500/
    environment.json
    <dataset>/{performance,numerical,samples}.csv
  amd-rx7900xtx/
    environment.json
    sift1m/{performance,numerical,samples}.csv
```

设备、数据集、测量口径和对照引擎均由页面动态选择。完整字段和 SHA256 参见各设备的公开环境文件。 / Select device, dataset, timing and baseline independently; see the environment files for hashes and full protocol details.
