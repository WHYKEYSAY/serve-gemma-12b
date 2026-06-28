# serve-gemma-12b — how to serve **Gemma-3-12B (dense)** efficiently (any GPU)

*A GPU-agnostic operations manual for one model. The reference tok/s is from one rig; the **recipe applies to any GPU**
— figure out your VRAM, then follow it.*

- **Architecture:** Dense 12B
- **Fits in VRAM?** YES — Q4 fits a 16 GB card fully

## Recipe
**Trivially fast:** dense + small → **single GPU, full residency, Q4_K_M, -fa on**. No offload, no split. The simplest 'fits → flies' case; great fast worker / tool model.

**Serving flags (llama.cpp):**
```
-ngl 99 -fa on   # single 16 GB card, full GPU
```

## Reference throughput
~87 tok/s on a single 16 GB card.

## Failures → fixes
- None notable when it fits; only slows if you needlessly offload/split.

## Verdict
Fast tool/worker model on any ≥16 GB card.

---
## The one decision: does it FIT in your VRAM?
Estimate size ≈ params × bytes/weight (Q4≈0.5, Q8≈1, FP16≈2 B/param) + KV + ~2–3 GB overhead.
- **Fits** → full GPU residency, **no offload**, single card if it fits on one → *bandwidth-bound, fast.*
- **Doesn't fit** → offload experts to RAM (use `-fit on`), keep the active path on GPU → *RAM-bandwidth-bound, slower.*

## Measure honestly
Use the server's **`/completion` decode timings** (`predicted_per_second`), greedy, cache off, multiple workloads —
NOT short OpenAI wall-time (it understates decode). See `bench_decode.py`.

## Files
- `REPORT.md` — the detailed benchmark (throughput · config · tuning-research+sources · analysis · failures), if present.
- `bench_decode.py` — honest decode-tok/s measurement (`/completion` timings).
- `launcher-entry.json` — a ready-to-paste config (name + serving flags) for whatever model server you run.

*Part of a per-model serving-playbook set.*
