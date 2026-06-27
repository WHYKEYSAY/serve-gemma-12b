# flash — Gemma-3-12B on RTX 5080

`google/gemma-3-12b-it` (**Q4_K_M** GGUF) on a single **RTX 5080 16 GB**, full GPU
residency, served by `llama.cpp` (`llama-server`). The cluster's small/fast model —
kept warm on **kechengy** as the Aquila failover and a quick general worker.

## Decode throughput
Measured honestly — **greedy** (temperature 0), **prompt cache disabled**, a **fresh
generation per request**; decode tok/s read from llama.cpp's own `timings`.

| Workload | Decode tok/s |
|---|---|
| Free-form prose | ~88 |
| Code generation | ~87 |
| JSON / structured | ~86 |
| Chat / dialogue | ~87 |
| Math / reasoning | ~86 |
| Translation (multilingual) | ~88 |
| Summarization | ~87 |

**Average decode ≈ 87 tok/s · Prefill ≈ 661 tok/s.** Flat across workloads — decode is
bandwidth-bound, not content-bound; a sign of a healthy full-residency config.

## Serving configuration
| Param | Value |
|---|---|
| Model | google/gemma-3-12b-it |
| Quant | Q4_K_M (GGUF) |
| Engine | llama.cpp `b9660-7dad2f1a1` |
| GPU | RTX 5080 16 GB (consumer — **no ECC**) |
| VRAM in use | ~13 GB / 16 GB |
| GPU layers | `-ngl 99` (all layers on GPU) |
| Memory mapping | `--no-mmap` (weights fully resident, RAM-safe) |
| Context window | 8192 (`-c 8192`) |
| KV cache | in VRAM (fp16) |
| Parallel slots | 4 |
| RAM offload | none — fully on GPU |

## Notes
- **Stable** through the full run (7 workloads × 2, zero errors).
- ECC is N/A — consumer GPUs have none, so the datacenter "disable ECC for +10 % bandwidth"
  trick doesn't apply to us; we're already ECC-free.
- Host **kechengy**; served as `gemma-3-12b` (alias `flash`) on `:8003`.

_Measured 2026-06-26 · [`_bench.py`](./_bench.py)._
