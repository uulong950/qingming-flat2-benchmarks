# qingming-flat2-benchmarks

**性能 · 正确性 · 数值精度 · 可复现性证据**  
Performance · Correctness · Numerical Accuracy · Reproducibility Evidence

Qingming Flat2 的公开基准数据。计算、调优与 GPU 内核保存在独立的 `qingming-flat2` 工程。本仓库只展示可比较的性能结果、CPU 数值审计、确认轮次的原始采样和实验环境，不公开任何 GPU 内核、内部配置名称或消融参数。

Public benchmark evidence for Qingming Flat2. This repository contains benchmark results, CPU numerical audits, holdout observations, and environment metadata, **not** proprietary kernels, internal tuning configurations, or ablation traces.

## 目录 / Layout

```text
qingming-flat2-benchmarks/
├── README.md
├── index.html
└── results-devices/
    ├── index.json
    └── metax-c500/
        ├── environment.json
        ├── sift1m/
        │   ├── performance.csv
        │   ├── numerical.csv
        │   └── samples.csv
        └── gist960/
            ├── performance.csv
            ├── numerical.csv
            └── samples.csv
```

## 首批结果 / Initial measurements

| 数据集 / Dataset | Batch | K | 当前协议合格 / Qualified | 几何平均加速 / GM speedup | 最大加速 / Peak |
|---|---|---|---:|---:|---:|
| SIFT1M (1M × 128) | 1, 10, 100, 1000, 10000 | 1, 10 | 10 / 10 | 1.804× | 4.302× |
| GIST960 (1M × 960) | 1, 10, 100, 1000 | 1, 10 | 8 / 8 | 1.294× | 1.768× |

来自同一 MetaX C500 **25% 算力 sGPU 分区、16,000 MiB 显存配额**，并非整卡对照。测试用原始查询，无重复扩展；GIST960 最多 B=1000。

Measured on the same **25% compute MetaX C500 sGPU partition (16,000 MiB VRAM quota)**, **not** a full board. Original queries only; no synthetic query repetition.

## 公开文件含义 / Published data contract

- **`performance.csv`**：每个 Batch/K 的两引擎独立确认轮次延迟、QPS、加速比、正确性状态。**无内部配置名称**。One row per workload; final holdout performance and qualification without internal configuration details.
- **`numerical.csv`**：仅针对最终报告的 Qingming 和 mcFaiss，在独立确认轮次记录 FP64/FP32 的抽样查询全库 Top-K 验证及全返回 ID 距离误差。Separate sampled Top-K audits and all-returned-ID distance errors.
- **`samples.csv`**：只包含报告所用确认轮次的原始 Qingming/mcFaiss GPU Event 与 Host API 测量，足以复算公开的 P50 和加速比；**不包含私人调优过程**。Only the two reported backends' holdout observations; no private tuning traces.
- **`environment.json`**：实际硬件配额、软件版本、测量协议、数据集哈希及外部 mcFaiss 版本和构建库哈希。Device allocation, versions, protocol, dataset checksums, and linked mcFaiss hashes.

## 审查范围 / Audit scope

CPU 全数据库 Top-K 验证只对预先固定的查询执行，不是所有查询的完整精确性证明。返回 ID 的 FP64/FP32 距离误差覆盖报告结果中的全部返回 ID。严格相等距离的边界并列是合法情况。QPS 按 Batch/P50 计算，属于批吞吐代理指标，并非长期持续服务 QPS；每轮 21 个样本的 P99 仅供描述。

CPU Top-K oracle validation scans the full database for **preregistered query rows only**, not all queries. Distance error evaluation covers all IDs returned by both reported backends. Exact-distance ties are accepted. QPS is a batch/P50 proxy, not sustained service throughput; P99 from 21 observations is descriptive.

因 GPU 内核未公开，第三方可以审计公开测量样本和数值检查结果，但无法独立从源码完整复现 Qingming 实验。公开文件中的完整性能统计与原测量记录一致；出于知识产权保护，私有调优轨迹不可作为公开证据审计。

Source-level independent reproduction is not available without the private kernels. Published timing summaries can be recomputed from the final holdout samples; private tuning traces are not independently auditable from this repository.

## 发布 / GitHub Pages

仓库 Settings → Pages → Deploy from a branch → `main` / `(root)`。无需 Jekyll 主题。  
Repository Settings → Pages → Deploy from a branch → `main` / `(root)`.

本地预览 / Local preview:

```bash
python3 -m http.server 8000
# open http://localhost:8000/
```

## 版本 / Version

**v0.1.1** 仅修改公开数据边界和展示：中英双语极简界面、去除所有公开的内部配置/消融名称、确认轮次双引擎采样与数值记录；**未产生新的 GPU 实测数据**。Underlying source archive SHA-256 remains recorded in `environment.json`.


**v0.1.2 · UI refinement / 版式调整**：矩阵、数值表格与环境字段统一左对齐；优化视觉层级、间距与窄屏交互。**所有 CSV/JSON 证据文件与 v0.1.1 字节完全一致**，没有新增 GPU 测量；`environment.json` 保留其原始证据发布标识。

Matrix, numerical, and environment columns now share consistent left alignment. Responsive spacing and selection visibility are refined. All source evidence bytes remain unchanged from v0.1.1.
