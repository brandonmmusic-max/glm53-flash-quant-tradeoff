# GLM-5.3-Flash quantization tradeoff: TrellisMX-MXFP8 vs TR3/EXL3 4bpw vs stock NVFP4

Speed and fidelity of three GLM-5.3-Flash quantizations served on the same four RTX PRO 6000 Blackwell GPUs, each at its published reference serving stack.

- **Interactive graph:** open `trellismx_tradeoff.html` (hover any mark for the value, conditions and source; "Table view" lists every number; "Dark" toggles the theme). Served at the GitHub Pages URL for this repository.
- **Share images:** `trellismx_tradeoff_light_charts.png` and `trellismx_tradeoff_dark_charts.png` (3200 px wide). Full pages with notes and sources: `trellismx_tradeoff_light.png`, `trellismx_tradeoff_dark.png`. The table view: `trellismx_tradeoff_table.png`.

All plotted values live in the `DATA` block at the top of the HTML `<script>`, with a source per row.

## What the graph shows

- **Fidelity (matched 32-window, true-decode, MTP off):** TR3/EXL3 4bpw 0.0282 (FP8 KV) / 0.0305 (NVFP4 KV); TrellisMX reference 0.0319 / 0.0355. TR3 is lower in 22 of 32 windows in each cache mode; the paired interval excludes zero for NVFP4 KV and crosses zero for FP8 KV.
- **Stock NVFP4 fidelity:** 0.0398 on the same 32 windows and teacher, but scored from a single 2,048-token prefill pass with eager execution, the Humming MoE backend and FP8 KV. Two other stock preparations on the same windows measured 0.0385 and 0.0389. Context, not a matched arm.
- **Speed (each family's published reference stack):** TrellisMX C1 decode 204.6 tok/s at 0K and 199 to 222 across 0K to 128K, prefill 8,457 tok/s at 32K. TR3 v84 (TP2, DFlash2-7, two GPUs at 600 W) 145.5 at 0K, prefill 6,225. Stock NVFP4 on Jovian r7 (TP4/DCP1, DFlash2 K7, FP8 KV) 252.9 at 0K (median of three samples spanning 125.8 to 261.4), 156.0 at 64K, 138.7 at 240K, prefill 9,421 at 32K.
- Reading: TrellisMX is faster than TR3/EXL3 at every measured cell and less accurate; it is slower than NVFP4 only at short context with the DFlash2 K7 draft and on prefill. From 64K upward TrellisMX C1 decode is above NVFP4 K7, and on the same B12X MTP3 stack NVFP4 measured 149 to 161 tok/s, below TrellisMX at every context.

## Sources

- TrellisMX model card and receipts: https://huggingface.co/brandonmusic/GLM-5.3-Flash-TrellisMX-MXFP8 (revision 7d37df2e): `results/speed-20260909`, `results/kld-reference-20260909/comparison.json`, `results/kld-tr3-20260909/comparison.json`, `results/quality-expanded-20260909/REPORT.md`.
- TR3 / EXL3 4bpw model card: https://huggingface.co/brandonmusic/GLM-5.3-Flash-tr3-4bpw (revision a5fee929): `runtime-results/v84/benchmarks/llm-decode-c1-clean-4096-600w.json`, `llm-decode-c1-prefill32k64k-600w.json`.
- Campaign repository: https://github.com/brandonmmusic-max/glm53-hadamard-shapleymcg-kld (main @03ff56b): `results/P8_FULL_COUPLED_CAMPAIGN_RESULTS_20260906.md` (stock NVFP4 0.039789 row), `results/P8_FC1_COLD_COMPARISON_V1.md` (older P8 image vs EXL3 TP4/EP4/DCP4, five cold starts), `results/rotation-v6-blocklocal-h16/run-stock-cf32-v3.json` (stock 0.0389264), `results/rotation-v7-selective-h16/run-stock-cf32-v1.json` (stock 0.0384505).
- Stock NVFP4 speed on the Jovian Judgement runtime: https://github.com/brandonmmusic-max/glm-5.3-flash-nvfp4-jovian-benchmarks (README tables; `evidence/r7/dcp1-bt4096-k7/decode/cc1-sample-{1,2,3}.json`, `evidence/r7/dcp1-bt4096-k7-high-context/decode/exact-high-context.json`, `evidence/r7/dcp1-bt4096-k7/prefill/exact-cold.json`).
- Stock NVFP4 checkpoint (carrier): https://huggingface.co/local-inference-lab/GLM-5.3-Flash-NVFP4/tree/520de24eabf507659eaef7c70f14fd584527facc. Later revisions replaced the checkpoint with a distilled version that is not measured here.
- Local run records on the measurement host, not yet published: the TR3 v93 TP2 benchmark (`bench-v93-dflash2-split-cache`), the TR3 r10 TP2 MTP3 qualification of 2026-09-01, the NVFP4 B12X MTP3 TP4/DCP4 run of 2026-08-31 and its target-only control, the Jovian r8 DCP4 matrices of 2026-08-31, and the stock NVFP4 receipt `run-trellismx-p8-rot3-cf32-stock-v1.json` from the rotation-l3-l20-l22 CF32 experiment (2026-09-05).

## Method notes

- KLD intervals for TrellisMX and TR3 are the published BCa 95% window intervals. The bias-corrected and accelerated (BCa) interval method is described by Bradley Efron, ["Better Bootstrap Confidence Intervals," *Journal of the American Statistical Association* 82(397), 1987](https://doi.org/10.1080/01621459.1987.10478410). The stock NVFP4 intervals were recomputed from the per-window means with the same estimator (BCa, 20,000 resamples, `numpy.random.RandomState(20260902)`); the same code reproduces the four published intervals to all printed digits.
- Speed values are copied from the raw `llm_decode_bench` JSON (`aggregate_tps`, client `tok_per_sec`), not from rolling server logs.
- Speed rows are each family's published reference stack, not an interleaved A/B: speculators, GPU counts and KV cache formats differ between families. The notes panel of the graph states every caveat.
- Rendering: headless Chromium at a 1600 px viewport and device scale factor 2.

