# qingming-flat2-benchmarks

性能 · 正确性 · 数值精度 · 可复现性证据

Performance · Correctness · Numerical Accuracy · Reproducibility Evidence

[公开基准页面 / Benchmark dashboard](https://uulong950.github.io/qingming-flat2-benchmarks/) · v0.11.0

本仓库公开同设备、同数据集和同协议下的性能、CPU 数值审计、独立确认轮次原始采样与环境证据。计算内核和内部调优仍保存在独立工程，不公开。

Public evidence for within-device comparisons under a shared search contract. Proprietary kernels and tuning traces are not included.

## 设备 / Devices

| Device | Execution environment | Baselines | Datasets |
|---|---|---|---:|
| MetaX C500 | 25% compute sGPU · 16,000 MiB quota | mcFaiss | 6 |
| AMD Radeon RX 7900 XTX 24GB | gfx1100 · Ubuntu desktop · ROCm 7.2.4 | hipVS, Faiss ROCm | 6 |

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

- GPU Event: resident complete search, excluding offline preprocessing, indexing and final output transfer. GIST960 Qingming includes internal query projection and count transfers. API: synchronized search wall time, not end-to-end RPC. QPS is `1000 × B / GPU P50 ms`, not sustained service throughput.
- AMD 使用原始 FP32 SIFT 数据，经全量整数域检查后允许无损内部表示；不使用近似量化。 / Lossless representation of verified integer-valued FP32 SIFT inputs is permitted; no approximate quantization.
- CPU FP64/FP32 full-database ranking checks are sampled; exact distance boundary ties are allowed. Returned score checks cover all reported holdout IDs. Missing fields are not zero.
- AMD SIFT1M holdout repeats: B=1/10/100/1000/10000 → 21/21/11/7/5. Tail quantiles are descriptive. C500 keeps its original per-dataset protocol.
- AMD runs on a shared display/compute GPU; exclusive use and locked clocks are not claimed. C500 is partitioned. No cross-device ranking.
- AMD SIFT1M Faiss/hipVS source commits were not recorded at run time. Executable hashes are recorded; dependency hashes observed after the run are disclosed separately. Do not infer exact source revisions from these hashes.
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
    gist960/{environment.json,performance.csv,numerical.csv,samples.csv}
    llama-128-ip/{environment.json,performance.csv,numerical.csv,samples.csv}
    laionclip512/{environment.json,performance.csv,numerical.csv,samples.csv}
    glove100/{environment.json,performance.csv,numerical.csv,samples.csv}
    deep10m/{environment.json,performance.csv,numerical.csv,samples.csv}
```

设备、数据集、测量口径和对照引擎均由页面动态选择。完整字段和 SHA256 参见各设备的公开环境文件。 / Select device, dataset, timing and baseline independently; see the environment files for hashes and full protocol details.

## GIST960 · AMD RX 7900 XTX / v0.7.0

N=1,000,000 · D=960 · squared L2 · Q=1,000 · B=1/10/100/1000 · K=1/10.

| Baseline | GPU P50 leading | Geometric mean | Min | Max |
|---|---:|---:|---:|---:|
| hipVS | 8/8 | 4.802× | 2.683× | 7.595× |
| Faiss ROCm | 8/8 | 4.403× | 1.103× | 18.359× |

| B | K | Qingming GPU P50 ms | hipVS ms | Faiss ROCm ms | vs hipVS | vs Faiss ROCm |
|---:|---:|---:|---:|---:|---:|---:|
| 1 | 1 | 0.977 | 6.552 | 17.930 | 6.708× | 18.359× |
| 1 | 10 | 1.205 | 6.451 | 19.633 | 5.354× | 16.295× |
| 10 | 1 | 1.414 | 6.519 | 19.492 | 4.609× | 13.782× |
| 10 | 10 | 2.200 | 6.628 | 19.679 | 3.013× | 8.947× |
| 100 | 1 | 10.274 | 42.741 | 22.177 | 4.160× | 2.158× |
| 100 | 10 | 16.282 | 43.683 | 20.690 | 2.683× | 1.271× |
| 1000 | 1 | 63.525 | 482.478 | 80.375 | 7.595× | 1.265× |
| 1000 | 10 | 72.269 | 482.805 | 79.743 | 6.681× | 1.103× |

公开确认轮包含 24 次调用、312 对 GPU/API 计时样本（每组每引擎 13 对）、96 条数值记录。每种 CPU 精度核对 36,663 个返回分数及 168 次全库抽样查询；88 次完整私有实验每种精度分别为 134,431 与 328。排名检查不是全部查询的严格证明。

Public R3: 24 invocations, 312 GPU/API timing pairs (13 per engine and shape), 96 audit rows. Per precision: 36,663 returned scores and 168 sampled full-database query checks across engines/shapes. Full private audit: 88 invocations; 134,431 scores and 328 sampled query checks per precision. Ranking validation is not exhaustive.

GIST960 独立环境：[environment.json](results-devices/amd-rx7900xtx/gist960/environment.json)。原始 FP32 输入、辅助投影摘要、最终 FP64 累积与 FP32 返回分数。辅助索引 1,538,949,192 bytes；基础索引构建约 3.730 s，扩展约 3.265 s（后者不含基础构建与后续模型审计）。离线索引构建不计入搜索；查询投影和内部计数传输计入 Qingming 搜索。文件大小不是峰值显存；未测量外部基线的建库成本。

GIST960 uses original FP32 inputs and database-derived auxiliary summaries, with FP64 final accumulation and FP32 returned scores. The auxiliary index is 1,538,949,192 bytes; base build ~3.730 s, extension ~3.265 s (excluding base build and later model audit). Offline setup is excluded; query projection and internal count transfer are included. File size is not peak VRAM; vendor setup costs were not measured.

ULP：统一正负零后计算，原始位模式最大值已与24次私有审计核对。大 ULP 不等于违反绝对/相对误差容差；Faiss ROCm 最大 FP32 ULP 为 922,746,880，原样保留。

ULP canonicalizes signed zeros; raw-bit maxima were reconciled against 24 private audits. Large ULP counts are not necessarily tolerance failures; Faiss ROCm maximum FP32 ULP is 922,746,880, retained without alteration.

GIST960 基线源码 commit 未记录；运行前采集并复核部分依赖库及二进制哈希，不构成完整构建来源证明。有限精度 GPU 证书尚无形式化证明。公开样本可复算统计，但私有内核及完整返回数组不公开。桌面 GPU 未声称独占使用或锁定频率。

GIST960 baseline source commits are unknown. Selected runtime library and executable hashes were captured at run start and rechecked, not complete build provenance. No formal GPU floating-point certificate or source-level independent reproduction is claimed. Exclusive GPU use and locked clocks are not claimed.

[Performance](results-devices/amd-rx7900xtx/gist960/performance.csv) · [Numerical](results-devices/amd-rx7900xtx/gist960/numerical.csv) · [Samples](results-devices/amd-rx7900xtx/gist960/samples.csv) · [Environment](results-devices/amd-rx7900xtx/gist960/environment.json)

## Llama-128-IP · AMD RX 7900 XTX / v0.8.0

N=256,921 · D=128 · IP max · Q=1,000 · B=1/10/100/1000 · K=1/10.

| B | K | Qingming GPU P50 ms | hipVS ms | Faiss ROCm ms | vs hipVS | vs Faiss ROCm |
|---:|---:|---:|---:|---:|---:|---:|
| 1 | 1 | 0.1996 | 0.6380 | 1.2028 | 3.197× | 6.027× |
| 1 | 10 | 0.3285 | 0.6658 | 1.2599 | 2.027× | 3.835× |
| 10 | 1 | 0.3662 | 0.7098 | 1.1941 | 1.938× | 3.261× |
| 10 | 10 | 0.4650 | 0.7177 | 1.0467 | 1.544× | 2.251× |
| 100 | 1 | 0.6592 | 3.2708 | 1.1975 | 4.962× | 1.817× |
| 100 | 10 | 0.6411 | 3.2739 | 1.2690 | 5.107× | 1.979× |
| 1000 | 1 | 3.6555 | 17.3801 | 4.2203 | 4.754× | 1.154× |
| 1000 | 10 | 3.7772 | 17.4472 | 4.3681 | 4.619× | 1.156× |

GPU P50：两套基线各 8/8 领先；几何平均加速 3.192× / 2.311×，最低 1.544× / 1.154×（hipVS / Faiss ROCm）。API P50 同样各 8/8 领先。
GPU P50: 8/8 leading against each baseline; geometric mean 3.192× / 2.311×, minimum 1.544× / 1.154× (hipVS / Faiss ROCm). API P50 also leads in 8/8 shapes against each.

公开 R3：24 次调用、312 对计时样本、96 条审计记录；每种精度共 36,663 个返回分数和 162 次抽样全库查询检查。私有三轮环境摘要报告 72 次调用、48/48 性能门槛通过。公开文件一致性已复核，未独立复算私有轮次或原始 ID/分数数组。
Public R3: 24 invocations, 312 timing pairs, 96 audit records; 36,663 returned scores and 162 sampled full-database query checks per precision across engines/shapes. Private summary reports 72 invocations and 48/48 performance gates. Public file consistency reviewed; private rounds and raw ID/score arrays were not independently re-audited.

原始 FP32 输入；较大 Batch 使用条件误差区间与 FP64 终端累积，返回 FP32 分数。内部证书与同步计入搜索；索引建立和最终结果传输不计入。基线源码 commit 未由公开文件提供；二进制与部分依赖哈希不能替代完整构建来源。桌面 GPU 未声称独占或锁频，13 次采样的尾分位数仅描述性。
Original FP32 inputs; larger batches use conditional intervals and FP64 terminal accumulation with FP32 returned scores. Internal certificates and synchronization are included; setup and final transfer excluded. Baseline source commits not supplied in public files; executable/dependency hashes are not full build provenance. Desktop GPU, no exclusive use or locked clocks claimed; tail quantiles from 13 samples are descriptive. No formal hardware proof or source-level reproduction claimed.

[Performance](results-devices/amd-rx7900xtx/llama-128-ip/performance.csv) · [Numerical](results-devices/amd-rx7900xtx/llama-128-ip/numerical.csv) · [Samples](results-devices/amd-rx7900xtx/llama-128-ip/samples.csv) · [Environment](results-devices/amd-rx7900xtx/llama-128-ip/environment.json)

## LAION-CLIP-512 · AMD RX 7900 XTX / v0.9.0

N=1,000,448 · D=512 · IP max · Q=1,000 · B=1/10/100/1000 · K=1/10.

| B | K | Qingming GPU P50 ms | hipVS ms | Faiss ROCm ms | vs hipVS | vs Faiss ROCm |
|---:|---:|---:|---:|---:|---:|---:|
| 1 | 1 | 2.9487 | 4.1427 | 11.0174 | 1.405× | 3.736× |
| 1 | 10 | 3.2889 | 4.1294 | 11.2046 | 1.256× | 3.407× |
| 10 | 1 | 3.0048 | 4.3933 | 11.0348 | 1.462× | 3.672× |
| 10 | 10 | 2.8969 | 4.4443 | 11.2469 | 1.534× | 3.882× |
| 100 | 1 | 5.7192 | 43.1511 | 11.7275 | 7.545× | 2.051× |
| 100 | 10 | 5.9019 | 43.0152 | 11.8487 | 7.288× | 2.008× |
| 1000 | 1 | 39.8832 | 235.2121 | 46.0044 | 5.898× | 1.153× |
| 1000 | 10 | 40.9011 | 235.4848 | 46.4921 | 5.757× | 1.137× |

GPU P50：两套基线各 8/8 领先；几何平均加速 3.045× / 2.365×，最低 1.256× / 1.137×（hipVS / Faiss ROCm）。
GPU P50: 8/8 leading against each baseline; geometric mean 3.045× / 2.365×, minimum 1.256× / 1.137× (hipVS / Faiss ROCm).

三轮 72 次调用，48/48 性能门槛通过；公开 R3 24 次调用、312 对计时、96 条审计记录。每种精度 36,663 返回分数和 174 次抽样全库查询检查。hipVS B1000 K10 有一条合法边界并列替换；按并列感知规则全部通过。
Three rounds, 72 invocations, all 48 performance gates passed. Public R3: 24 invocations, 312 timing pairs, 96 audit rows; 36,663 scores and 174 sampled full-database query checks per precision. One legitimate boundary-tie replacement for hipVS B1000 K10 per precision; all tie-aware checks pass.

原始 FP32 输入，不重新归一化；大 Batch 使用条件误差区间及 FP64 终端。索引构建和最终结果传输不计入。桌面运行，未锁频或声称独占；13 次采样的尾分位数仅描述性。上传证据的哈希、计时与原始结果结构已复核；缺少原始数据库，未独立重算 CPU oracle。未提供基线源码 commit。
Original FP32 inputs, no renormalization; larger batches use conditional intervals and FP64 terminal. Index setup and final result transfer excluded. Desktop GPU, no locked clocks or exclusive use claimed. Tail quantiles from 13 samples are descriptive. Uploaded evidence hashes, timings and raw result structures reviewed; CPU oracle scores/rankings not independently recomputed without the dataset. Baseline source commits unavailable. No formal hardware proof or source-level reproduction claimed.

[Performance](results-devices/amd-rx7900xtx/laionclip512/performance.csv) · [Numerical](results-devices/amd-rx7900xtx/laionclip512/numerical.csv) · [Samples](results-devices/amd-rx7900xtx/laionclip512/samples.csv) · [Environment](results-devices/amd-rx7900xtx/laionclip512/environment.json)

## GloVe100 · AMD RX 7900 XTX / v0.10.0

N=1,183,514 · D=100 · Cosine max · Q=10,000 · B=1/10/100/1000/10000 · K=1/10.

| B | K | Qingming GPU P50 ms | hipVS ms | Faiss ROCm ms | vs hipVS | vs Faiss ROCm |
|---:|---:|---:|---:|---:|---:|---:|
| 1 | 1 | 0.6084 | 1.2141 | 3.8981 | 1.995× | 6.407× |
| 1 | 10 | 0.8119 | 1.5336 | 4.1819 | 1.889× | 5.151× |
| 10 | 1 | 0.6126 | 1.5177 | 4.0472 | 2.477× | 6.606× |
| 10 | 10 | 1.1772 | 1.2849 | 4.2300 | 1.091× | 3.593× |
| 100 | 1 | 1.5831 | 11.1177 | 5.0225 | 7.023× | 3.173× |
| 100 | 10 | 1.7982 | 11.1759 | 5.2892 | 6.215× | 2.941× |
| 1000 | 1 | 12.0550 | 65.3349 | 20.7632 | 5.420× | 1.722× |
| 1000 | 10 | 12.9011 | 65.9373 | 21.3328 | 5.111× | 1.654× |
| 10000 | 1 | 122.7055 | 650.3803 | 202.8034 | 5.300× | 1.653× |
| 10000 | 10 | 127.8949 | 652.3517 | 205.9153 | 5.101× | 1.610× |

GPU P50：两套基线各 10/10 领先；几何平均加速 3.567× / 2.981×，最低 1.091× / 1.610×（hipVS / Faiss ROCm）。
GPU P50: 10/10 leading against each baseline; geometric mean 3.567× / 2.981×, minimum 1.091× / 1.610× (hipVS / Faiss ROCm).

三轮 90 次调用，60/60 性能门槛通过；公开 R3 30 次调用、390 对计时、120 条审计记录。每种精度 366,663 返回分数和 252 次抽样全库查询检查，无边界并列替换。
Three rounds, 90 invocations, all 60 performance gates passed. Public R3: 30 invocations, 390 timing pairs, 120 audit rows; 366,663 returned scores and 252 sampled full-database query checks per precision. No boundary-tie replacements in R3.

GPU 使用 FP32 归一化向量的内积；CPU oracle 使用原始向量余弦。条件证书的 FP64 终端针对归一化操作数。索引构建、预处理及最终传输不计入。桌面运行，未声称独占或锁频；13 次采样的尾分位数仅描述性。上传证据的哈希、计时及原始结果结构已复核，缺少原始数据库，未独立重算 CPU oracle。
GPU uses FP32-normalized vectors and inner product; CPU oracles use original-vector cosine. Conditional certificate terminal uses FP64 on normalized operands. Index setup, preprocessing and final transfer excluded. Desktop GPU, no exclusive use or locked clocks claimed. Tail quantiles from 13 samples are descriptive. Archive hashes, timing and raw output structure reviewed; CPU oracles not independently recomputed without the dataset. No formal hardware proof or source-level reproduction claimed.

[Performance](results-devices/amd-rx7900xtx/glove100/performance.csv) · [Numerical](results-devices/amd-rx7900xtx/glove100/numerical.csv) · [Samples](results-devices/amd-rx7900xtx/glove100/samples.csv) · [Environment](results-devices/amd-rx7900xtx/glove100/environment.json)

## DEEP10M · AMD RX 7900 XTX / v0.11.0

N=10,000,000 · D=96 · Squared L2 min · Q=10,000 · B=1/10/100/1000/10000 · K=1/10.

| B | K | Qingming GPU P50 ms | hipVS ms | Faiss ROCm ms | vs hipVS | vs Faiss ROCm |
|---:|---:|---:|---:|---:|---:|---:|
| 1 | 1 | 5.0355 | 8.2811 | 28.3065 | 1.645× | 5.621× |
| 1 | 10 | 6.1192 | 8.3581 | 39.1997 | 1.366× | 6.406× |
| 10 | 1 | 5.2166 | 8.4468 | 34.6894 | 1.619× | 6.650× |
| 10 | 10 | 6.1606 | 8.8349 | 38.6717 | 1.434× | 6.277× |
| 100 | 1 | 10.7401 | 62.5449 | 62.1247 | 5.823× | 5.784× |
| 100 | 10 | 10.8302 | 63.6526 | 44.4639 | 5.877× | 4.106× |
| 1000 | 1 | 94.7610 | 661.8525 | 173.3235 | 6.984× | 1.829× |
| 1000 | 10 | 95.5289 | 658.9425 | 180.6742 | 6.898× | 1.891× |
| 10000 | 1 | 934.3087 | 4707.6968 | 1734.3373 | 5.039× | 1.856× |
| 10000 | 10 | 935.7151 | 4702.0220 | 1798.2534 | 5.025× | 1.922× |

GPU P50：两套基线各 10/10 领先；几何平均加速 3.418× / 3.668×，最低 1.366× / 1.829×（hipVS / Faiss ROCm）。
GPU P50: 10/10 leading against each baseline; geometric mean 3.418× / 3.668×, minimum 1.366× / 1.829× (hipVS / Faiss ROCm).

三轮 90 次调用，60/60 性能门槛通过；公开 R3 30 次调用、390 对计时、120 条审计记录。每种精度 366,663 返回距离和 204 次抽样全库查询检查，无边界并列替换。
Three rounds, 90 invocations, all 60 performance gates passed. Public R3: 30 invocations, 390 timing pairs, 120 audit rows; 366,663 returned distances and 204 sampled full-database query checks per precision. No boundary-tie replacements in R3.

原始 FP32 向量，无归一化；条件证书使用仿射 L2 筛选及原始向量 FP64 终端。索引构建、预处理及最终传输不计入。桌面运行，未声称独占或锁频；13 次采样的尾分位数仅描述性。上传证据的哈希、计时及原始结果结构已复核，缺少原始数据库，未独立重算 CPU oracle。C500 历史数据哈希未独立对齐。
Original FP32 vectors without normalization. Conditional affine-L2 certificate and FP64 terminal on original vectors. Index setup, preprocessing and final transfer excluded. Desktop GPU, no exclusive use or locked clocks claimed. Tail quantiles from 13 samples are descriptive. Archive hashes, timing and raw output structure reviewed; CPU oracles not independently recomputed without the dataset. Historical C500 dataset hashes not independently matched. No formal hardware proof or source-level reproduction claimed.

[Performance](results-devices/amd-rx7900xtx/deep10m/performance.csv) · [Numerical](results-devices/amd-rx7900xtx/deep10m/numerical.csv) · [Samples](results-devices/amd-rx7900xtx/deep10m/samples.csv) · [Environment](results-devices/amd-rx7900xtx/deep10m/environment.json)
