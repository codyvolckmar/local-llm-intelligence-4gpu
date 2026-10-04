# Local LLM Intelligence Report — 5 GPU Tiers

Hunting the best local-model intelligence (reasoning + coding/agentic + long-context document work) attainable on an **RTX 3090, RTX 4090, RTX 5090, RTX 6000 Ada (48 GB), and RTX PRO 6000 Blackwell (96 GB)**.

Speeds are memory-bandwidth-scaled estimates from measured 3090 figures — decode is bandwidth-bound, so bandwidth ratio is the honest scaling law. Marked ° throughout.

> **Revised 2026-10-04.** Five hardware claims, five pricing errors and one context-length error were corrected, and the **RTX PRO 6000 Blackwell** was added as a full tier. See [Fact-check log](#fact-check-log).
> Previous revision covered four cards and predates several model releases.

---

## Fact-check log

Every claim below was re-checked against NVIDIA datasheets, HuggingFace `config.json`/model cards, and the OpenRouter models API on 2026-10-04.

### Errors corrected

| # | Old claim | Reality | Impact |
|---|---|---|---|
| 1 | Kimi K3 — *"No — **weights not yet posted**"* | **Weights are public.** `moonshotai/Kimi-K3` (1.23M downloads), plus `unsloth/Kimi-K3-GGUF` and `nvidia/Kimi-K3-NVFP4`. | **Material.** It is a local option at ≥192 GB, not API-only-by-necessity. |
| 2 | GLM-5.2 API price *"$1.40 / $4.40"* | That is **GLM-5.3's** price. GLM-5.2 is **$0.104 / $8.00**. | **Material.** GLM-5.2 is ~13× cheaper on input and ~1.8× *more* expensive on output than stated. |
| 3 | GLM-5.2 is the current GLM | **GLM-5.3** shipped 2026-08-18. | Stale reference frame. |
| 4 | DeepSeek V4-Flash *"$0.14 / $0.28"* | **$0.0224 / $1.28** (0731: $0.0152 / $1.28). | **Material.** Input is 6× cheaper than claimed, **output 4.6× more expensive**. The "cheapest per token, period" claim inverts: it is cheapest on *input* only. |
| 5 | Kimi K3 *"$2.85 / $14.25"* | **$0.72 / $13.00**. | Input 4× cheaper than claimed. |
| 6 | Nemotron 3.5 Lightning context *"1M"* | **256K served.** NVIDIA's card says "up to 1M tokens (for single H100 deployment, we use 256K)". `config.json` `max_position_embeddings: 262144`; OpenRouter reports 262144. | **Material.** The 1M figure is a theoretical ceiling requiring a large deployment. |
| 7 | *"Qwen3.6-27B hits ~77.2 SWE-bench V"* | **Unverifiable.** Qwen publishes no SWE-bench Verified for the 3.6/3.8 27B line; the nearest published figure is Qwen3.6-35B-A3B at **73.4**. | Removed from headline claims and flagged. |
| 8 | Readme file named `readme` (no extension) | GitHub rendered it as **plain text**, so every table collapsed into one unreadable line. | Renamed to `README.md`; all tables are now real Markdown. |
| 9 | `local-llm-intelligence-4gpu (1).html` | Leftover alternate rendering with a junk filename. | Renamed `report.html`; README.md is canonical. |

### Claims confirmed correct

RTX 3090 24 GB/936 GB/s · RTX 4090 24 GB/1008 GB/s + Ada FP8 · RTX 5090 32 GB/1792 GB/s + Blackwell NVFP4 · RTX 6000 Ada 48 GB/960 GB/s, NVLink-pairable · L40S 48 GB/864 GB/s · **Qwen3-Coder-Next = 80B total / 3B active, 262K native, SWE-bench Verified 70.6** (70.6/71.1/71.3 across SWE-Agent/miniSWE-Agent/OpenHands) · **Ornith-1.5-35B-A3B SWE-bench Verified 79** · Qwen3.8-27B = 27B dense, 262K native, GPQA Diamond 89.2 · Qwen3.5-35B-A3B 262K · Laguna XS 2.1 = 33B-A3B · Nemotron 3.5 Lightning = 30B/3B · Qwen3.6-27B SWE-bench Pro 61.7 · the "4090 does not raise the intelligence ceiling" thesis · the "6000 Ada is 3090-class decode" thesis.

### Licenses now tracked (previously absent)

| Model | License |
|---|---|
| Qwen3.8-27B, Qwen3.5-35B-A3B, Qwen3-Coder-Next | Apache-2.0 |
| Ornith-1.5 (all sizes) | MIT |
| Kolibri-1 | Apache-2.0 |
| GLM-5.2, GLM-4.7-Flash | MIT |
| Laguna XS 2.1 | OpenMDW-1.1 |
| Kimi K3 | Kimi (custom) |
| Nemotron 3.5 Lightning | NVIDIA Open Model License ("other") |

⚠️ **GLM-5.3 is *not* open weights** (`zai-org/GLM-5.3` → license "other"). GLM-5.2 and GLM-5.3-Flash are the MIT-licensed members. Don't plan a local deployment around 5.3.

---

## New models since the last revision

### Kolibri-1 — the important addition (Aleph Alpha, 2026-10-03)

| Property | Value |
|---|---|
| Params | **78.1B total / 3.46B active** (384 routed experts, top-6, +1 shared) |
| Layers / hidden | 50 / 2,560 · 4:1 sliding-window GQA → full causal GQA |
| Context | 262,144 native; 1,048,576 extrapolated |
| License | **Apache-2.0** |
| Specialisation | Bilingual **German + English**, strong OCR'd-document training (33.3% of long-context data) |

| Benchmark | Kolibri-1 |
|---|---|
| LiveCodeBench v6 | **85.9** |
| HumanEval+ | 92.7 |
| SWE-Bench Verified | 66.4 |
| TerminalBench 2.1 | 27.7 |

This is the **best new fit for the 48 GB and 96 GB tiers**: at 78B it lands squarely in the gap between the 35B MoEs (cheap, fast) and the 70B-dense class (slow, heavy). It fits on a 48 GB 6000 Ada at Q4, and on the PRO 6000 at **Q8 or BF16**. The German-English bilinguality and document/OCR weighting are a genuine differentiator for European enterprise document work — nobody else in this size class advertises it.

### Also new, but out of reach locally

| Model | Size | Context | Notes |
|---|---|---|---|
| Naive-N0.5-Flash | 309B / 15.5B | **1M native** | Built on MiMo-V2.5; hybrid SWA-DSA. ~160 GB at NVFP4 — needs 2× PRO 6000. |
| IQuest-Q1 | 320B / 15B | 512K | Agentic coding and tool use, 256 experts. ~171 GB at NVFP4 — needs 2× PRO 6000. |
| Ornith-1.5-397B | 397B | — | SWE-bench Verified **86**, DeepSWE 56. ~212 GB at NVFP4 — needs 2× PRO 6000. |
| Heimr 570M | 0.57B | — | Experimental edge model, 2–8× faster decode on long contexts than comparables. Runs on anything, incl. laptops. |

**Pattern worth noting:** the frontier is moving to ~300B-total / ~15B-active MoEs with native 1M context. At ~0.56 B/param for NVFP4 that is **~170 GB — just past one PRO 6000.** Two cards bridged over NVLink is now the practical frontier-local entry point.

### Correctly absent
`Qwen3.8-27B` (Aug 2026) and `Laguna XS 2.1` (Jul 2026) were already in the previous revision and are now widely regarded as the best-overall and best-small-agentic local picks respectively.

---

## The five cards at a glance

| Card | Arch | VRAM | Bandwidth | vs 3090 | Run cost | FP4 | Notes |
|---|---|---|---|---|---|---|---|
| **3090** | Ampere | 24 GB | 936 GB/s | 1.00× | 0.18–0.22 $/hr | ❌ | Value baseline; no FP8 tensors |
| **4090** | Ada | 24 GB | 1008 GB/s | 1.08× | 0.22–0.27 $/hr | ❌ | FP8 tensors; speed-only bump |
| **5090** | Blackwell | 32 GB | 1792 GB/s | 1.92× | 0.30–0.40 $/hr | ✅ NVFP4 | Speed + native FP4 |
| **6000 Ada** | Ada | 48 GB | 960 GB/s | **1.03×** | ~0.70 $/hr | ❌ | Capacity card, NVLink→96 GB, 3090-class decode |
| **PRO 6000 Blackwell** | Blackwell | **96 GB** | **1792 GB/s** | **1.92×** | **1.29 $/hr** (rental) | ✅ NVFP4 | **Capacity *and* speed** — new top tier |

### What each card actually changes

**The 4090 is not an intelligence upgrade.** Same 24 GB as the 3090, so it unlocks nothing. It buys ~8% bandwidth plus Ada FP8 tensor cores → roughly 10–25% faster decode, mostly on dense models.

**The 5090 is bandwidth, not capacity.** 1.92× the 3090's bandwidth at 32 GB. That buys full 262K context at Q5–Q8 on the 30–35B MoEs plus ~2× decode. It does **not** unlock an 80B brain — a 70B at Q4 is ~41 GB and still doesn't fit.

**The 6000 Ada is capacity without speed.** At 960 GB/s it decodes at *exactly* 3090-class speed for a same-size model. Its entire value is 2× memory, NVLink pairing to 96 GB, and ECC. It is the cheap way into an 80B MoE — not the fast way.

**The PRO 6000 Blackwell is the only card that does both.** This is the key correction to the previous revision's framing: it carries the **same 1792 GB/s as the 5090** but with **3× the VRAM**. So it is a 5090's decode speed *and* 6000 Ada's capacity class, in one card, with native NVFP4 and ECC. For anyone choosing one card, this is the one.

---

## RTX PRO 6000 Blackwell (96 GB) — the new top tier

**Verified specs (NVIDIA datasheet):** Blackwell (GB202) · **96 GB GDDR7 with ECC** · 512-bit · **1792 GB/s** · 5th-gen Tensor Cores with FP4 · 4th-gen RT Cores · 24,064 CUDA cores · 125 TFLOPS SP · 4,000 AI TOPS · PCIe 5.0 x16. Two editions — Workstation (active cooling, single-GPU boxes) and Server (passive, multi-GPU). The Max-Q Blackwell edition is a lower-power variant, not this card.

### VRAM budget — what 96 GB actually buys

Effective bytes-per-parameter including scales and metadata:

| Quant | B/param | Max params in 96 GB | Max params in 48 GB | Max in 32 GB | Max in 24 GB |
|---|---|---|---|---|---|
| Q4_K_M | 0.58 | **166B** | 83B | 55B | 41B |
| NVFP4 | 0.56 | **171B** | 86B | 57B | 43B |
| FP8 | 1.02 | 94B | 47B | 31B | 24B |
| Q8_0 | 1.05 | 91B | 46B | 30B | 23B |
| BF16 | 2.05 | **47B** | 23B | 16B | 12B |

The headline: **at NVFP4 a 96 GB card holds a ~170B model, but at BF16 only a ~47B model.** Any comparison that says "PRO 6000 runs X" without naming the quant is meaningless — the same card swings from Kolibri-1-at-Q8 to Qwen3.8-27B-at-BF16.

### What runs on it

| Model | Params | Q4_K_M | NVFP4 | Q8_0 | BF16 | Role |
|---|---|---|---|---|---|---|
| **Kolibri-1** | 78.1B / 3.46B | 45 GB | 44 GB | 82 GB ✅ | — | New best 78B MoE; DE/EN, document-heavy |
| **Qwen3-Coder-Next** | 80B / 3B | 46 GB | 45 GB | 84 GB ✅ | — | Best coding-agent MoE that fits one box |
| **Nemotron 3 Super** | 120B / 12B | 70 GB | 67 GB | — | — | Largest single-card brain; RULER 96%+ @512k |
| **Qwen3.8-27B** | 27B dense | 16 GB | 15 GB | 28 GB | **55 GB ✅** | **Only card here that runs it at full BF16** |
| Qwen3.5/3.6-35B-A3B | 35B / 3B | 20 GB | 20 GB | 37 GB | 72 GB | Two copies fit at Q8 → parallel agents |
| Ornith-1.5-35B-A3B | 35B / 3B | 20 GB | 20 GB | 37 GB | 72 GB | Agentic coding, MIT |
| Laguna XS 2.1 | 33B / 3B | 19 GB | 18 GB | 35 GB | 68 GB | Fast small agentic |
| Gemma-4 31B | 30.7B dense | 18 GB | 17 GB | 32 GB | 63 GB | Vision + Codeforces 2150 |
| Inkling-Small | 276B / 12B | 160 GB ❌ | 155 GB ❌ | — | — | Needs 2 cards |
| DeepSeek V4 Flash | 284B / 13B | 165 GB ❌ | 159 GB ❌ | — | — | Needs 2 cards |

✅ = fits with headroom · ⚠️ leaves <4 GB for KV cache at long context

### Estimated decode (bandwidth-scaled from the measured 3090 baselines, ×1.92)

| Model | At the quant you'd actually pick here | Est t/s° |
|---|---|---|
| Qwen3.5/3.6-35B-A3B @ Q8 | 37 GB | ~120° |
| Ornith-1.5-35B-A3B @ Q8 | 37 GB | ~120° |
| Laguna XS 2.1 @ NVFP4 | 18 GB | ~210° |
| Qwen3-Coder-Next 80B @ NVFP4 | 45 GB | ~200° |
| Kolibri-1 78B @ NVFP4 | 44 GB | ~200° |
| Qwen3.8-27B @ BF16 | 55 GB | ~30° |
| Nemotron 3 Super 120B @ NVFP4 | 67 GB | ~55° |

**Best single-card pick:** **Kolibri-1 at NVFP4** (or Qwen3-Coder-Next if coding is the whole job). Both are ~200 t/s with ~50 GB spare — enough for a second model or a long KV cache simultaneously.
**Best quality pick:** **Qwen3.8-27B at BF16**, the only card in this report that runs a strong dense reasoning model at full precision.
**Best throughput pick:** two Laguna XS 2.1 or 35B-A3B instances in parallel — the 96 GB budget supports genuinely concurrent multi-agent work that no other card here can do.

⚠️ **NVLink caveat:** RTX PRO 6000 Blackwell cards support NVLink bridging, but the 96 GB is a *single* card. Two cards = 192 GB, which is where the 280–400B MoEs (Kolibri's larger cousins, Inkling-Small, DeepSeek V4 Flash, Naive-N0.5-Flash) finally become local. That is the real frontier-local tier, and it costs two cards' worth of power.

---

## RTX 6000 Ada (48 GB) — the capacity tier

The only real IQ jump on 48 GB. Fits the strongest single-card models: **Kolibri-1 at Q4 (45 GB)** and **Qwen3-Coder-Next at Q4 (46 GB)**, plus the 70B-dense class at Q4 and the 30–35B MoEs at Q8 with a huge window.

| Model | Params | Fit on 48 GB | Est t/s° | $/1M @ 0.70 $/hr |
|---|---|---|---|---|
| **Kolibri-1** | 78.1B / 3.46B | Q4 ≈45 GB | ~110° | ~3.4° |
| **Qwen3-Coder-Next 80B** | 80B / 3B | Q4 ≈46 GB | ~110° | ~3.5° |
| Qwen3.6-35B-A3B @ Q8 | 35B / 3B | ✅ +20 GB spare | ~105° | ~1.85° |
| Ornith-1.5-35B-A3B @ Q8 | 35B / 3B | ✅ | ~105° | ~1.85° |
| Gemma-4 31B @ Q8 | 30.7B dense | ✅ ~32 GB | ~45° | ~4.3° |
| 70B dense @ Q4 | 70B | ✅ ~40 GB | ~30–45° | ~4–5.5° |
| DeepSeek V4 Flash | 284B | ❌ (~165 GB) | — | — |

**Update to the previous revision:** this tier's headline model was Qwen3-Coder-Next at "~70.6 SWE-bench". That is correct — but **Kolibri-1 now offers 78B at the same footprint with a 262K native window and Apache-2.0**, and is the better general pick if you also handle documents or German.

---

## RTX 3090 / 4090 (24 GB) — the 3B-active MoE tier

Ceiling is 27–35B class. Full 262K only with q4_0/q8_0 KV-cache quant; the 35B MoEs top out ~224–262K tight. The 4090 changes nothing about which models fit.

| Model | Type | Ctx | 262K fit | 3090 t/s° | 4090 t/s° | Verdict |
|---|---|---|---|---|---|---|
| **Qwen3.8-27B** | 27B dense · hybrid | 262K | q4_0 KV only | 55° | 65° | **Reasoning.** GPQA 89.2, SWE-bench Pro 61.7, TB2.1 73.0. Best all-round for finance/doc synthesis. |
| **Qwen3.5-35B-A3B** | 35B MoE · 3B act | 262K | ✅ | 110° | ~120° | **Balanced.** Best documented full-262K on 24 GB. |
| **Ornith-1.5-35B-A3B** | 35B MoE · 3B act | 262K | ~224–256K | 110° | ~128° | **Agentic.** SWE-bench Verified **79**, Pro 59.6, TB2.1 67.8. Highest coding IQ per GB. |
| **Laguna XS 2.1** | 33B MoE · 3B act | 262K | ✅ ~19 GB Q4 | ~105° | ~120° | **Agentic.** SWE-bench Verified 70.9 at 33B; OpenMDW licence. |
| **Ornith-1.5-9B** | 9B | — | ✅ | ~200° | — | Cheap sub-agent / router. SWE-bench Verified 70.6. |
| Nemotron 3.5 Lightning | 30B MoE · Mamba-2 | **256K** | ✅ | 120° | 140° | Speed. ⚠️ Mamba weakens exact-number recall — poor for precise finance. |
| GLM-4.7-Flash | 30B MoE | 200K | below 262K | 45° | — | Agentic-per-GB, but misses the long-window brief. |
| DeepSeek-R1:32b | 32B dense | 128K | ❌ | 35° | 42° | Reasoning specialist for hard math on a bounded doc. |

> ⚠️ **Corrected:** Nemotron 3.5 Lightning's context is **256K as served**, not 1M. The 1M figure is NVIDIA's stated ceiling for a large deployment.

**Smartest pick on 24 GB:** Qwen3.8-27B for reasoning/doc work; **Ornith-1.5-35B-A3B** when the task is tool- or agent-driven; add **Laguna XS 2.1** if you want a second agentic option for routing. Keep DeepSeek-R1:32b for math-heavy sub-problems.

**4090 over 3090 is rarely an intelligence upgrade.** It buys the 65+ t/s margin and Ada FP8, not capability.

---

## RTX 5090 (32 GB) — bandwidth without enough capacity

~1.92× bandwidth → near-2× decode. 32 GB means the 30–35B MoEs run at Q5–Q8 with full context and no compromises. NVFP4 adds up to ~3× more *via vLLM/TensorRT-LLM only* — not Ollama/GGUF.

| Model | Params | Fit on 32 GB | GGUF t/s° | NVFP4 t/s° | Verdict |
|---|---|---|---|---|---|
| **Qwen3.8-27B** | 27B dense | ✅ Q8 full 262K | 105° | — | Reasoning + full context at Q8. |
| **Laguna XS 2.1** | 33B / 3B | ✅ Q5–Q6 full | ~210° | ~3×° | Fastest agentic MoE at full context. |
| Qwen3.6-35B-A3B | 35B / 3B | ✅ Q5–Q6 | 220° | ~3×° | Speed + context pair. |
| Gemma-4 26B-A4B | 26B / 4B | ✅ | 230° | — | 3.8B active; near-31B quality, ~⅛ the compute. |
| Nemotron 3.5 Lightning | 30B / 3B | ✅ 256K | 120°→230° | ~3×° | Fastest long-context buffer. |
| Kolibri-1 | 78.1B / 3.46B | ❌ (~44 GB NVFP4) | — | — | **Needs 48 GB+.** |
| DeepSeek V4 class | 1.6T | ❌ (≥400 GB) | — | — | Datacenter scale only. |

**Smartest pick on 5090:** Qwen3.8-27B at Q8/full-262K for layered reasoning, or Laguna XS 2.1 for maximum agentic throughput at full context. Only adopt vLLM/TensorRT if you want NVFP4 — on Ollama it's just a fast 32 GB card.

---

## Local vs the open-weight frontier — corrected

These are the models you can't fit on these cards. **Prices verified against the OpenRouter API on 2026-10-04**; the previous revision's price column contained four errors (see fact-check log).

| Model | Params | Ctx | Fits a PRO 6000? | Licence | SWE-bench V | API in / out per 1M |
|---|---|---|---|---|---|---|
| **Kimi K3** | 2.8T MoE | 1M | ❌ (~1.5 TB) | Kimi | ~93.4 | **$0.72 / $13.00** |
| DeepSeek V4 Pro | 1.6T / 49B | 1M | ❌ (~870 GB) | MIT | 96.4 (0813) | $0.85 / $5.00 |
| GLM-5.2 | ~744B / 40B | 1M | ❌ (~420 GB) | MIT | — | **$0.104 / $8.00** |
| GLM-5.3 | — | 1M | ❌ | ⚠️ **not open** | 95.4 | $1.40 / $4.40 |
| Naive-N0.5-Flash | 309B / 15.5B | 1M native | ❌ (~160 GB) | open | — | — |
| IQuest-Q1 | 320B / 15B | 512K | ❌ (~171 GB) | open | — | — |
| DeepSeek V4 Flash | 284B / 13B | 1M | ❌ (~159 GB) | MIT | 88.8 (0731) | **$0.0224 / $1.28** |
| **Kolibri-1** | **78.1B / 3.46B** | **262K native** | ✅ **NVFP4 44 GB** | **Apache-2.0** | **66.4** | — |
| **Qwen3-Coder-Next** | 80B / 3B | 262K | ✅ NVFP4 45 GB | Apache-2.0 | 70.6 | $0.12 / $0.80 |

**The important structural point:** the previous revision framed Kimi K3 as unreachable because "weights not yet posted." They *are* posted. The real reason it doesn't fit is **1.5 TB of weights at NVFP4** — physics, not availability. Meanwhile **Kolibri-1 and Qwen3-Coder-Next at 78–80B are the new bridge**: they deliver 66–71% SWE-bench Verified, Apache-2.0, and 262K native context on **one** card.

**On the corrected V4-Flash pricing:** $0.0224 input is the cheapest input in the entire market by an order of magnitude. But output at **$1.28** is 4.6× the previously-stated $0.28. For agentic and reasoning workloads — which are output-heavy — V4-Flash is *not* the price floor the old table implied. **GLM-5.3 Flash ($0.15 / $0.50, MIT, 1M context)** and **GLM-5.2 ($0.104 input)** are now the better cheap-inference options.

---

## Cost: local is no longer automatically cheaper

| Path | $/1M output | $/100M output | When it wins |
|---|---|---|---|
| DeepSeek V4 Flash (API) | **$1.28** | ~$128 | Cheapest *input*; output is not cheap — **corrected up from $0.28** |
| GLM-5.2 (API) | $8.00 | ~$800 | MIT-licensed 744B, 1M context, absurdly cheap input |
| Local 3090/4090 (24 GB) | $0.20–0.60 | $20–60 | Cheap and private — but 27–35B ceiling |
| Local 6000 Ada (48 GB) | $1.8–5.5 | $180–550 | Biggest local IQ at 48 GB, fully private |
| Local PRO 6000 (96 GB) @25% util | ~$7–12 | $700–1,200 | **Best local IQ per card**; 1.29 $/hr to run |
| Local PRO 6000 (96 GB) @100% util | ~$1.7–3.0 | $170–300 | Only competitive with cheap APIs if you saturate the card |

### At $1.29/hr, utilisation is the whole game

At the real rental rate, idle VRAM is pure burn. **Utilisation — not which card or model you pick — decides whether this is cheaper than an API:**

| Model | 5% util | 25% util | 100% util |
|---|---|---|---|
| Laguna XS 2.1 @NVFP4 (~210 t/s) | $34.13 | **$6.83** | $1.71 |
| Kolibri-1 @NVFP4 (~200 t/s) | $35.83 | **$7.17** | $1.79 |
| Qwen3-Coder-Next @NVFP4 (~200 t/s) | $35.83 | $7.17 | $1.79 |
| 35B-A3B @Q8 (~120 t/s) | $59.72 | $11.94 | $2.99 |
| Nemotron 3 Super @NVFP4 (~55 t/s) | $130.30 | $26.06 | $6.52 |
| Qwen3.8-27B @BF16 (~30 t/s) | $238.89 | $47.78 | $11.94 |

At 25% utilisation, **only the ~200 t/s NVFP4 MoEs beat any API** (GLM-5.2's $8.00 output). Nothing beats GLM-5.3 Flash ($0.50) or DeepSeek V4 Flash ($1.28) unless you are **saturated**, where Laguna XS at $1.71 is roughly at parity.

**The honest conclusion at $1.29/hr:** rent **on demand, not 24/7**. A PRO 6000 idle for 20 hours a day burns ~$775/month for nothing. If you can batch into a few hours a day, local wins on cost. If your workload is continuous, the cheap APIs win and the PRO 6000 is a sovereignty purchase. Monthly burn: 24/7 = **$942**, 12h/day = **$471**, 8h/day business = **$322**.

**The verdict that changes minds:** at **$1.29/hr**, a PRO 6000 burns **~$942/month** running 24/7, and at realistic 10–30% utilisation still costs more per output token than GLM-5.3 Flash or DeepSeek V4 Flash. Local's real wins are **privacy, data residency, unlimited volume, no rate limits, no lock-in** — not raw $/token. Treat it as a sovereignty purchase, and switch it on when you need it rather than leaving it idle.

---

## Reasoning effort, context, and signal loss

The other half of running a local model: what `reasoning_effort` actually does, how it mutates your context, and where signal gets lost.

### There are three different mechanisms, and they are not interchangeable

| Mechanism | How it works | Reliability |
|---|---|---|
| `chat_template_kwargs.reasoning_effort` | A **prompt instruction** — the template tells the model "think briefly" or "think hard". No hard cap. | Soft. Model may ignore it. |
| `thinking_token_budget` | vLLM **logits processor forcibly injects `reasoning_end_str`** once the count is hit. | Hard cap, but has bugs (see below). |
| `reasoning_level` | Soft limiting, used by gpt-oss. | Soft. |

⚠️ **They conflict.** Qwen's own client code has repeated fixes for this: DashScope *rejects* `reasoning_effort` combined with `thinking_budget`, and `qwen-code` now drops `enable_thinking`/`thinking_budget` when an effort tier is set. **Pick one knob and set it explicitly** — the ambiguity between `enable_thinking`, `thinking_budget` and `reasoning_effort` is itself a source of silent signal loss.

### The finding that matters most: highest effort is the *least* reliable

Open bug [QwenLM/Qwen3.8#216](https://github.com/QwenLM/Qwen3.8/issues/216) (opened 2026-08-19, ~1,000 test calls). Qwen3.8-27B returns an **empty `content` field with `finish_reason: "stop"`** — it thinks and then emits nothing:

| Setting | Failure rate |
|---|---|
| ⛔ `reasoning_effort: xhigh` (**the default**) | **38%** |
| ✅ `reasoning_effort: low` / `medium` | **8%** |
| ⛔ `temperature: 0` | **12/12 — 100%** |
| ⛔ `repetition_penalty` outside 1.05–1.10 | below → **100%**; 1.20 → 67% |
| ✅ **`frequency_penalty: 0.3`** | **0 failures in 12 calls** |

**Never leave Qwen3.8-27B on its default effort.** And `frequency_penalty: 0.3` is the reported mitigation — chosen over `repetition_penalty` precisely because it is an OpenAI-standard field, so it survives stacks that silently drop `repetition_penalty`.

This inverts the usual intuition. Combined with the overthinking literature — accuracy follows an **inverted-U** against chain-of-thought length, not a monotonic curve ([LESS, ICLR 2026](https://proceedings.iclr.cc/paper_files/paper/2026/file/ce916251b4fe04f54f99c8d304d68877-Paper-Conference.pdf); [When More Thinking Hurts](https://aclanthology.org/2026.findings-acl.1199.pdf)) — the practical rule is: **more reasoning effort is not more signal. It is often less.**

### How reasoning mutates your context

Three ways, in ascending order of damage:

1. **Thinking tokens live inside the context window.** Reasoning is emitted between `reasoning_start_str` and `reasoning_end_str` and occupies real KV cache. A 15K-token think at `high` is 15K tokens of your window.
2. **`preserve_thinking` re-injects prior reasoning into later turns.** Qwen3.8 exposes this flag; when on, historical `reasoning_content` is carried forward. Across a 20-turn document session that compounds badly — you are re-feeding scratchpad as if it were evidence.
3. **Hard truncation.** If `thinking_token_budget` fires mid-thought, the model is forced to emit its end-token and answer *before it has concluded*. The result is a fluent, confident, unverified answer. This is the worst failure mode and it is silent.

**Countermeasure for (2) and (3):** when you replay history to the model, send **only `content`, never past `reasoning_content`**. Reasoning is a scratchpad, not state. If your framework round-trips assistant messages verbatim, it is silently inflating your context and your cost.

### Good news: context is cheap on a 96 GB card

KV bytes/token = `2 × full-attention layers × kv_heads × head_dim × bytes_per_elem`. Hybrid-attention models only cache KV for their **full-attention** layers — the rest hold fixed-size recurrent state.

| Model | Full-attn layers | KB/token fp16 | KB/token fp8 | Full 262K window (fp8 KV) |
|---|---|---|---|---|
| **Kolibri-1** | 10 of 50 (4 sliding : 1 full) | 20 | 10 | **2.6 GB** |
| **Qwen3-Coder-Next** | 12 of 48 (3 linear : 1 full) | 24 | 12 | **3.1 GB** |
| **Qwen3.8-27B** | 16 of 64 (3 linear : 1 full) | 64 | 32 | **8.4 GB** |
| Nemotron 3 Super | 88 of 88 (all full) | 88 | 44 | 11.5 GB |

Weights + a **complete 262,144-token window**:

| Config | Weights | KV (fp8) | Total | Spare | Full-length sessions |
|---|---|---|---|---|---|
| Laguna XS 2.1 @NVFP4 | 18.5 GB | 21.0 GB | 39.5 GB | 56.5 GB | 2 |
| Kolibri-1 @NVFP4 | 45.0 GB | **2.6 GB** | 47.6 GB | 48.4 GB | **~18** |
| Qwen3-Coder-Next @NVFP4 | 45.5 GB | 3.1 GB | 48.6 GB | 47.4 GB | ~15 |
| Qwen3.8-27B @Q4 | 16.0 GB | 8.4 GB | 24.4 GB | 71.6 GB | ~8 |
| Qwen3.8-27B @BF16 | 55.0 GB | 8.4 GB | 63.4 GB | 32.6 GB | ~3 |
| Nemotron 3 Super @NVFP4 | 67.0 GB | 11.5 GB | 78.5 GB | 17.5 GB | 1 |

**Context is not your bottleneck.** On a 96 GB card a full-length 262K session costs single-digit GB of KV, and you can hold many concurrently. The binding constraints are `max_model_len` policy and decode throughput — not VRAM.

*(Laguna XS / 35B-A3B show 21 GB because they are 40-layer full-attention GQA models, not hybrid — priced at 8 KV heads × 128 head_dim.)*

### Minimum-signal-loss playbook

1. **Never run at the default effort.** Set `reasoning_effort` explicitly on every request. On Qwen3.8-27B, `low` and `medium` are ~4.7× more reliable than the `xhigh` default.
2. **Route by task class, not by one global setting.** `low`/off for extraction, classification, and lookup. `medium` for synthesis. Reserve `high` for genuinely multi-step reasoning — and on Qwen3.8-27B treat `xhigh` as unusable.
3. **Set `frequency_penalty: 0.3`.** Best reported mitigation for empty completions, and it's an OpenAI-standard field so it survives most stacks.
4. **Never use `temperature: 0`** on Qwen3.8-27B — 100% empty-completion rate. If you need determinism, pin a seed at low temperature instead.
5. **Keep `repetition_penalty` in 1.05–1.10** or leave it at 1.0; outside that band failure rates go to 100%/67%.
6. **Strip `reasoning_content` from replayed history.** Send only `content`. This is the single biggest lever on context mutation.
7. **Set `preserve_thinking: false`** for document work, where carried-forward scratchpad actively corrupts evidence.
8. **Budget the window explicitly:** `document + reserved_think_budget + max_answer + safety`. Pass `max_model_len` to the server rather than accepting the 262K default, so runaway thinking hits a predictable error instead of a truncated answer.
9. **Prefer a hard `thinking_token_budget` over template-level `effort`** when you need a guaranteed ceiling — but pin a vLLM version without the known re-entry bug ([#43757](https://github.com/vllm-project/vllm/pull/43757)), and note `thinking_token_budget` is still rejected on the V2 model runner unless you set `VLLM_USE_V2_MODEL_RUNNER=0`.
10. **Use fp8 KV cache, not q4_0, when fidelity matters.** Quantising KV throws away exactly the long-range attention signal that long documents depend on. You have the VRAM for fp8.
11. **Cache the document prefix.** Put the stable document + instructions first so the prefix cache hit rate is high; thinking tokens are not cacheable, the prefix is.
12. **Measure the inverted-U on your own workload.** The peak is task-specific. Run 50 real prompts at `low`/`medium`/`high` and pick the knee — don't inherit a default.

---

## Bottom line

**Intelligence scales with VRAM; speed scales with bandwidth. Until now, no card gave you both — the PRO 6000 Blackwell does.**

1. **Best single card, full stop: RTX PRO 6000 Blackwell (96 GB).** 1792 GB/s *and* 96 GB. Run **Kolibri-1 at NVFP4** (~200 t/s) or **Qwen3-Coder-Next** for coding; run **Qwen3.8-27B at BF16** if you want full-precision dense reasoning. Two cards over NVLink → 192 GB → the 280–400B MoE frontier goes local.
2. **Best value capacity card: RTX 6000 Ada (48 GB)** at ~half the power draw. Same models at Q4, ~110 t/s. Accept the 3090-class bandwidth.
3. **Best speed: RTX 5090 (32 GB)** — but you pay for bandwidth you can't fill with 80B models. Stay on Ollama and it's just a fast 32 GB card; adopt vLLM/TensorRT if NVFP4 matters.
4. **4090 over 3090 is almost never an intelligence upgrade.** Buy for the speed margin only.
5. **Be honest about the frontier.** Kimi K3 / DeepSeek V4 Pro / GLM-5.2 are API-only at 10–30× your size and lead on hard reasoning and 1M context. Local buys you sovereignty, not frontier scores.
6. **Model selection dominates hardware.** Qwen3.8-27B (reasoning), Ornith-1.5-35B-A3B or Laguna XS 2.1 (agentic coding), Kolibri-1 (78B documents/bilingual), DeepSeek-R1:32b (hard math). The card decides how high a quant and how long a context you can afford — and on 96 GB, whether you reach the 80B class at all.

---

## Assumptions & caveats

- **t/s are bandwidth-scaled estimates** from measured 3090 baselines (×1.92 for 5090/PRO 6000, ×1.08 for 4090, ×1.03 for 6000 Ada), assuming the runtime streams only routed + shared experts for MoE and full weights for dense. Real numbers will be lower where KV cache or attention becomes significant at long context.
- **Quant changes the answer, not just the size.** Every figure above states its quant. The same card swings from ~200 t/s at NVFP4 to ~30 t/s at BF16.
- **NVFP4 requires vLLM or TensorRT-LLM.** The GGUF/Ollama path does not use it.
- **Run costs are user-specified**, not vendor pricing: 0.18–0.22 (3090), 0.22–0.27 (4090), 0.30–0.40 (5090), 0.70 (6000 Ada), **1.29 (PRO 6000 rental rate)**. All $/1M figures scale linearly with these. PRO 6000 figures assume 24/7 billing — if you can switch the card off when idle, utilisation climbs and cost per token falls proportionally.
- **KV-cache figures are computed, not measured**, from each model's `config.json` (`num_hidden_layers`, `num_key_value_heads`, `head_dim`, `layer_types`) assuming 2 tensors × fp16/fp8. Models with `linear_attention` or bounded `sliding_window` layers only cache KV for their full-attention layers; verify against your engine's actual reported KV usage.
- **Benchmarks are vendor-reported** and harness-dependent. SWE-bench Verified varies by agent scaffold and turn budget; **differences under ~2 points are not meaningful.** Terminal-Bench **2.0 and 2.1 are different benchmarks** and appear in the wild interleaved.
- **Verify before you build.** Check fit and speed with `ollama ps` (confirm 100% GPU offload) or your engine's own metrics on the actual card. `nvidia-smi --query-gpu=memory.total` for real VRAM.
- **Prices** are OpenRouter list rates as of 2026-10-04 and move frequently. DeepSeek uses off-peak/peak time-of-day billing.

---

## Sources

- NVIDIA datasheets: [RTX PRO 6000 Blackwell Workstation Edition](https://www.nvidia.com/content/dam/en-zz/Solutions/data-center/rtx-pro-6000-blackwell-workstation-edition/workstation-blackwell-rtx-pro-6000-workstation-edition-nvidia-us-3519208-web.pdf) · [RTX PRO 6000 Blackwell Server Edition](https://www.nvidia.com/en-us/data-center/rtx-pro-6000-blackwell-server-edition/) · [RTX 5090](https://www.nvidia.com/en-us/geforce/graphics-cards/50-series/rtx-5090/)
- [OpenRouter models API](https://openrouter.ai/api/v1/models) — pricing, context windows, retrieved 2026-10-04
- Reasoning effort: [QwenLM/Qwen3.8 issue #216 — empty completions at xhigh](https://github.com/QwenLM/Qwen3.8/issues/216) · [vLLM Reasoning Outputs](https://docs.vllm.ai/en/stable/features/reasoning_outputs/) · [vLLM #43757 thinking_token_budget re-entry fix](https://github.com/vllm-project/vllm/pull/43757) · [QwenLM/qwen-code #8488 effort/knob conflict](https://github.com/QwenLM/qwen-code/pull/8488)
- Overthinking: [LESS, ICLR 2026 — inverted-U accuracy vs CoT length](https://proceedings.iclr.cc/paper_files/paper/2026/file/ce916251b4fe04f54f99c8d304d68877-Paper-Conference.pdf) · [When More Thinking Hurts (ACL Findings 2026)](https://aclanthology.org/2026.findings-acl.1199.pdf)
- Model cards: [Kolibri-1](https://huggingface.co/Aleph-Alpha/Kolibri-1) · [Qwen3-Coder-Next](https://huggingface.co/Qwen/Qwen3-Coder-Next) + [tech report](https://arxiv.org/html/2603.00729) · [Ornith-1.5-35B-A3B](https://huggingface.co/ornith-ai/Ornith-1.5-35B-A3B) · [Qwen3.8-27B](https://huggingface.co/Qwen/Qwen3.8-27B) · [Qwen3.5-35B-A3B](https://huggingface.co/Qwen/Qwen3.5-35B-A3B) · [Nemotron 3.5 Lightning](https://huggingface.co/nvidia/NVIDIA-Nemotron-3.5-Lightning-30B-A3B-BF16) · [Nemotron 3 Super](https://huggingface.co/nvidia/NVIDIA-Nemotron-3-Super-120B-A12B-FP8) · [Laguna XS 2.1](https://huggingface.co/poolside/Laguna-XS-2.1) · [Laguna S 2.1](https://huggingface.co/poolside/Laguna-S-2.1) · [Gemma 4 31B](https://huggingface.co/google/gemma-4-31B-it) · [Gemma 4 26B A4B](https://huggingface.co/google/gemma-4-26B-A4B-it) · [GLM-5.2](https://huggingface.co/zai-org/GLM-5.2) · [GLM-4.7-Flash](https://huggingface.co/zai-org/GLM-4.7-Flash) · [Kimi K3](https://huggingface.co/moonshotai/Kimi-K3)
- SWE-bench Verified cross-check: [vals.ai](https://www.vals.ai/benchmarks/swebench) · [Benchmark Atlas](https://atlas.kevinhu.io/benchmarks/swe-bench-verified) · [BenchLeader](https://www.benchleader.com/benchmarks/vals_swebench)
