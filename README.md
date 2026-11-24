# qingming-flat2-benchmarks

**Results-only public benchmark for Qingming Flat2.** Performance, correctness, numerical accuracy, and raw timings from recorded experiments. This repository contains **no GPU kernels, build artifacts, or dataset payloads**.

第一版：**v0.1.0** · 设备 MetaX C500 **sGPU 25% compute / 16,000 MiB VRAM quota** · SIFT1M / GIST960 · L2 squared · K=1/10。使用真实原始查询，GIST 不重复扩展至 B=10000。

## Structure

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

The device **model** is the first grouping level; dataset is the second. The `index.json` catalog provides file paths so additional devices/datasets/metrics require no changes to the page's data logic. Metric is recorded as data, not as another directory level.

## First dataset release

| Dataset | Batches | K | Workloads qualified | Geometric mean acceleration* | Peak acceleration* |
|---|---|---|---:|---:|---:|
| SIFT1M (1M × 128) | 1, 10, 100, 1,000, 10,000 | 1, 10 | 10/10 | 1.804× | 4.302× |
| GIST960 (1M × 960) | 1, 10, 100, 1,000 | 1, 10 | 8/8 | 1.294× | 1.768× |

* Same-session GPU Event P50 ratios, Qingming best qualifying Streaming variant versus mcFaiss on the **same sGPU allocation**; tuning rounds 1–2 and independent confirmation round 3. The qualification is under the specified **sampled-query full-database CPU oracle protocol**, not an all-query mathematical proof.

## How to publish

1. Create a **public** GitHub repository named `qingming-flat2-benchmarks` and upload the content of this folder to its root.
2. Repository **Settings → Pages → Build and deployment → Deploy from a branch**. Select `main` and `/ (root)` and save.
3. The site will be served at `https://<github-user>.github.io/qingming-flat2-benchmarks/` once Pages deployment succeeds.

To preview locally (the browser cannot `fetch()` CSV with `file://`):

```bash
cd qingming-flat2-benchmarks
python3 -m http.server 8000
# Open http://localhost:8000
```

## Files and semantics

- `environment.json`: actual device allocation, software versions, dataset hashes, benchmark scope and release provenance; no local absolute paths or hostnames.
- `performance.csv`: **one row per** `(dataset, Batch, K)`; measured winner Qingming vs same-session mcFaiss, holdout P50/P90/P95/P99 GPU Event, Host API, QPS and acceleration. `QPS=1000×Batch/GPU_P50_ms` is a **batch throughput proxy**, **not** sustained multi-client throughput.
- `numerical.csv`: all original CPU FP64/FP32 sampled-query full-database oracle audits for every case, plus all-returned-ID distance error audits for the selected holdout Qingming and mcFaiss runs. `scope` distinguishes these two fundamentally different claims. Only *exact* CPU distance ties are accepted for Top-K boundary replacement; numerical output tolerances **do not** relax rank membership.
- `samples.csv`: **all** individual GPU Event and Host API timings: 3 rounds × 21 repeats × 9 backend/configuration combinations × the dataset's 8/10 workloads. This includes 8 Qingming variants and the mcFaiss baseline, without dropping slower variants.
- `results-devices/index.json`: a simple site data catalog to add GPUs and datasets without editing the website logic.

### Audit limitations

CPU oracle full-database Top-K validation covers **pre-selected query IDs within each Batch**, not all database × query pairs. For holdout selected Qingming and mcFaiss, CPU distance error is computed for **all returned IDs over every query row**. Both vendors are judged independently against the CPU FP64 and FP32 references. All datasets are FP32 storage and L2 squared; *internal instruction precision is not verified*.

The benchmark scope is **GPU device-resident search only**; index construction, transfers, service concurrency, peak device memory and stage-level timing are not in scope. Each P99 comes from only 21 samples in one round and is **descriptive**. The C500 was running as a **25% compute sGPU partition with a 16,000 MiB VRAM quota**, not exclusive full C500. Reported CPU identity from the virtualized host was inconsistent and was not promoted to a physical-host claim. mcFaiss had local worktree modifications; its Git commit and prebuilt library SHA256 are in `environment.json`.

This repo intentionally provides **auditable outcomes, not independently runnable Qingming source code**. The source test archive's SHA256 and per-dataset CSV SHA256 are in the environment record. Future results should use a new immutable run ID rather than silently overwrite prior published evidence.

## Data integrity

The published CSVs were copied/reformatted from the original experiment archive `qingming_c500_matrix_20261124T004646Z.tar.gz`; no additional GPU measurements were created. SHA256 for the source archive, build sources and libraries is recorded in `results-devices/metax-c500/environment.json`.
