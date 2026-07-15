# Who is building zkAI — organizations & their top benchmark

A vendor/lab-centric companion to the paper-centric tables in [`README.md`](./README.md).
The README indexes *papers*; this file indexes *the teams shipping or integrating the work*,
and pins one **top benchmark** per team so the landscape is legible at a glance.

Two groups, because they answer different questions:

1. **Companies & integrators** — teams putting zkML into a product, a network, or a device.
   The question here is *"what is deployed, and for whom?"*
2. **Academic / open-source projects** — the research systems the companies build on. The
   question here is *"what is the fastest published number, and under what caveats?"*

> **Read the benchmark table as a map, not a leaderboard.** The rows are not comparable to
> each other. They differ in *guarantee* (integrity vs. model-privacy vs. input-privacy),
> in *what is proven* (one forward pass vs. a full multi-token generation), in *bit width*
> (a free accuracy-for-speed knob most papers don't report), and in *hardware*. Every number
> below traces to a row in [`README.md`](./README.md) / [`papers.yml`](./papers.yml); the
> caveats there apply here unchanged. Numbers current as of **July 2026**.

## Legend — the privacy column

The "ZK-privacy" column is the one most often flattened in vendor decks. It has **three
distinct meanings**, and conflating them is the single most common error in this space:

| Symbol | Meaning | What is hidden | What is public |
|---|---|---|---|
| **∅** | Integrity only | nothing | model + input + output |
| **🔒model** | Genuine zero-knowledge | model weights | input (or committed) |
| **🔒input** | Client-side inference | the user's input | the model weights (public) |

**🔒model and 🔒input are inverses, not degrees of the same thing.** A system that hides the
*model* (zkLLM, zkGPT) is protecting a model owner's IP against the person running inference.
A system that hides the *input* (Bionetta, World ID) is protecting a user's biometric against
a public model. They live in opposite threat models and their costs are not comparable — see
the [Bionetta caveat](./README.md#inference) in the README.

---

## 1. Companies & integrators

| Org | Product | What they're integrating it *for* | Stack (one line) | Privacy |
|---|---|---|---|---|
| **Lagrange Labs** | [DeepProve](https://github.com/Lagrange-Labs/deep-prove) | A proving **network** + enterprise AI attestation (partners incl. NVIDIA, Intel, IBM, Qualcomm). Open-sourced Jun 2026; claims 12M+ proofs, 3M+ inferences served. | GKR + LogUp-GKR lookups · Goldilocks · BaseFold / HyperKZG (Ceno, Rust) | ∅¹ |
| **ICME Labs** | [Jolt Atlas](https://github.com/ICME-Lab/jolt-atlas) | **On-device** verifiable inference — prove on a laptop/phone, verify on-chain. Authors of the widely-cited "Definitive Guide to ZKML". | Jolt zkVM → ONNX tensor ops · sumcheck DAG · HyperKZG · BlindFold ZK | 🔒model² |
| **Rarimo / Distributed Lab** | [Bionetta](https://docs.rarimo.com/zkml-bionetta/) | **Client-side biometrics** — prove a face/iris match on your phone without revealing the biometric. Powers privacy-preserving proof-of-personhood. | R1CS (Circom) · UltraGroth (Groth16 + in-circuit lookups) · BN254 · weights baked in as constants | 🔒input |
| **World** (Tools for Humanity) | World ID iris zkML | **Proof-of-personhood** — self-custody signed iris images, prove locally that the iris code came from the correct model, insert into the registry permissionlessly. Absorbed **Modulus Labs** as its applied-research team (2026). | On-device CNN proof (ezkl-lineage / custom) · public model, private image | 🔒input |
| **Zkonduit** | [EZKL](https://github.com/zkonduit/ezkl) | The **general-purpose** ONNX→SNARK toolchain everyone benchmarks against; Python/JS/CLI. Configurable visibility (hide model *or* input). | Halo2 (PLONKish) + ONNX · circuit-based arithmetic · lookup tables | 🔒model / 🔒input³ |
| **Polyhedra Network** | [zkPyTorch](https://blog.polyhedra.network/zkpytorch/) | A **compiler** (PyTorch/ONNX → circuits) marketed for model-IP protection + on-chain integrity; SDKs in Python/Rust over their Expander backend. | ONNX-DAG compiler · int-4 static quant (fits M61) · reuses zkCNN/zkLLM primitives · GKR/Expander | ∅ / 🔒model⁴ |
| **Giza** | [Giza Agents](https://github.com/gizatechxyz/giza-agents) | Started as on-chain zkML (Cairo/Orion); **pivoted to on-chain AI agents** ("xenocognitive finance", >$1B agentic volume). zkML now a trust-minimization option, not the headline. | Cairo / Orion (STARK-friendly) → agent framework | ∅ |

¹ **DeepProve's design center is integrity, not privacy.** It commits only model weights and
lookup witnesses (never intermediate activations), which is a *cost* optimization, not a
confidentiality guarantee. Lagrange's marketing says weights are "protected"; the repo does
**not** credit it as zero-knowledge the way it does zkLLM/zkGPT. Treat privacy as unproven
here until a hiding commitment is documented.
² Jolt Atlas offers ZK via the **BlindFold** technique — optional, and its LLM benchmarks
don't report whether it was enabled.
³ EZKL exposes per-tensor **visibility** flags, so it can hide the model, the input, or
neither — the guarantee is a config choice, not a fixed property.
⁴ Polyhedra advertises "protect model IP." The primary zkPyTorch paper is characterized in
the repo as an integrity/compiler contribution; genuine weight-hiding is claimed but not the
demonstrated result. Marked split.

**Adjacent trust models (not ZK — listed so the boundary is explicit).** ORA / Ora Protocol
and the opML lineage give *optimistic* verification (fraud proofs, no proof unless
challenged); Gensyn and TEE-based provers (Phala, and the Intel-SGX attestation path) give
*hardware* trust. These are cheaper by ~1000× and buy a **weaker** guarantee — see
[Alternatives to ZK](./README.md#alternatives-to-zero-knowledge).

---

## 2. Academic / open-source projects — the benchmark table

Top published number per system, LLM-first. **tok/min derived from a single forward-pass
time is marked `*` and is _not_ sustained-decode throughput** — the distinction the README
labors, preserved here.

| System / Team | Top model proven | Params | Throughput | Quant (bits) | Privacy | Stack | HW |
|---|---|---|---|---|---|---|---|
| **[DeepProve](https://eprint.iacr.org/2026/1112)** · Lagrange/HKUST/Yale/UIUC | GPT-2 (all tokens) | 124M | **174 tok/min** · 1855 distributed (17×) | **12** | ∅ | GKR + LogUp lookups · BaseFold/HyperKZG | Ryzen 9 7950X3D, 16c |
| ″ | Gemma-3 | 270M | **86 tok/min** | 12 | ∅ | GQA + RMSNorm + RoPE support | Ryzen 9 7950X3D |
| **[Jolt Atlas](https://arxiv.org/abs/2602.17452)** · ICME | GPT-2 | 125M | 38 s/proof (tok/min **not derivable**) | ? | 🔒model | Jolt→ONNX · sumcheck DAG · BlindFold | M3 MacBook, 16 GB |
| ″ | nanoGPT | 0.25M | 14 s/proof (≈4.3 tok/min\*) | ? | 🔒model | on-device; 17× faster than EZKL | M3 laptop |
| **[zkGPT](https://eprint.iacr.org/2025/1184)** · NUS/HKUST | GPT-2 (1 pass) | 124M | 21.8 s/pass (≈2.8 tok/min\*) · **101 KB proof** | **16** | 🔒model | GKR + Lasso · Hyrax/BN254 · non-interactive | 16-core CPU |
| **[zkPyTorch](https://eprint.iacr.org/2025/535)** · Polyhedra/Berkeley/NUS | Llama-3 | **8B** | 150 s/token (0.4 tok/min) | **4** (w/act) | ∅ | ONNX-DAG compiler · M61 · reuses zkCNN/zkLLM | 1 CPU core |
| **[zkLLM](https://arxiv.org/abs/2404.16109)** · Waterloo | LLaMA-2 | **13B** | <15 min full inference | ? (fixed-pt) | 🔒model | tlookup + zkAttn · Hyrax · **CUDA/GPU** | A100 |
| **[ZKML](https://ddkang.github.io/papers/2024/zkml-eurosys.pdf)** · Berkeley (ddkang) | GPT-2 | 124M | 3652 s/pass | ? | ∅ / config | Halo2 compiler (the standard baseline) | 128 vCPU |
| **[ZKTorch](https://arxiv.org/abs/2507.07031)** · ddkang | *(compiler)* | — | up to 6× faster than ZKML | ? | ∅ | basic-blocks + Mira accumulation | — |
| **[SpaGKR](https://eprint.iacr.org/2024/1018)** | ternary nets | — | 45× (sparsity) × ~5× (ternary) | **~1.58** | ∅ | GKR exploiting sparsity + ternary weights | — |
| **[Bionetta](https://arxiv.org/abs/2510.06784)** · Rarimo | ResNet-18 (CNN) 🔁 | — | 14.1 s (**13.6 s on iPhone**) · **0.88 KB** | ? | 🔒input | R1CS · UltraGroth · weights as constants | phone / laptop |
| **[zkCNN](https://eprint.iacr.org/2021/673)** · Liu et al. | VGG-16 | 15M | 88.3 s | ? | ∅ | GKR-for-CNN — the lineage root below zkLLM | — |
| **[SafetyNets](https://arxiv.org/abs/1706.10268)** · NYU | FcNN-3-Quad 🔓 | — | +5% over plain exec | quadratic acts | ∅ | interactive GKR (the ancestor of all above) | — |

🔁 **inverted threat model** (public model, private input) — not comparable to the LLM rows.
🔓 **integrity only, not ZK.**

**No LLM proven** — CNN/vision-only stacks, listed for completeness: **EZKL** (nanoGPT 237 s;
MobileNetV2), **World ID** (iris CNN, on-device), **Giza/Orion** (small on-chain models).
These do not run a transformer.

---

## Three things this table makes visible

1. **Exactly one team proves a full multi-token LLM generation end-to-end today: Lagrange
   (DeepProve).** Everyone else in the LLM rows proves a *single forward pass*; their tok/min
   figures are derived and marked `*`. Jolt Atlas's GPT-2 number can't even be derived — no
   sequence length is reported.
2. **The two privacy modes barely overlap in personnel.** The 🔒model line (zkLLM, zkGPT,
   Jolt Atlas) and the 🔒input line (Bionetta, World) are different companies solving inverse
   problems — mirroring the [citation-disconnect](./README.md) the README found between the
   proving and privacy literatures.
3. **Bit width is unreported for most systems**, so throughput comparisons are confounded.
   Only zkGPT (16), DeepProve (12), and zkPyTorch (4) state it *and* an accuracy cost. A high
   tok/min at an undisclosed bit width is not a like-for-like number — see
   [Quantization: the hidden variable](./README.md#quantization-the-hidden-variable).

---

*Sources: in-repo [`papers.yml`](./papers.yml) / [`operators.yml`](./operators.yml) /
[`README.md`](./README.md) for all benchmark numbers; vendor/company status from
[Rarimo docs](https://docs.rarimo.com/zkml-bionetta/),
[Lagrange](https://lagrange.dev/blog/deepprove-is-now-open-source),
[World](https://world.org/blog/engineering/zkml-ai-thats-verifiable-private-and-right-on-your-phone),
[Polyhedra](https://blog.polyhedra.network/zkpytorch/), and
[Giza](https://github.com/gizatechxyz/giza-agents), as of July 2026.*
