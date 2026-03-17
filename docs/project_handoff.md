# Project Handoff: Verifiable Reproducible AI ("Can't Be Evil")

*This document transfers context from a prior research conversation. Paste this into a new Claude Code / terminal session to continue the project.*

---

## What This Project Is

A research and engineering initiative to build a fully verifiable, reproducible LLM training pipeline — one where any technically capable third party can independently reproduce the exact model weights (verified by sha256 fingerprint) from source, using public tools, a public dataset, and a public build environment.

The philosophical framing: the same "rules without rulers" ethos as Linux and Bitcoin. Not a model that *promises* to be trustworthy, but one where trustworthiness is *structurally enforced* and independently verifiable. The difference between "don't be evil" and "can't be evil."

This is intentionally technology-neutral and politically agnostic — the transparency argument applies equally regardless of who builds the model.

---

## The Core Problem Being Solved

Most "open source" AI models today only release weights (the finished artifact). They do not release:
- Training data (what it learned from)
- Training code and exact hyperparameters (how it was built)
- The research decision log (why specific choices were made)
- A reproducibility guarantee (can you rebuild it and get the same result?)

This means users must trust the builder's intentions. The goal of this project is to eliminate that trust requirement entirely through verifiable reproducibility — the same way you can verify a Linux kernel build matches published source.

---

## Key Research Findings from Prior Conversation

### Open Weight vs. Truly Open Source
- "Open weight" = weights released, nothing else guaranteed
- "Truly open source" = weights + training data + training code + methodology + reproducibility
- Only OLMo (Allen Institute for AI) comes close to full openness at present
- OLMo releases: weights, Dolma dataset, training code, 500+ checkpoints, OLMoTrace tool
- Gap: OLMo does not yet provide a sha256-match guarantee from source build

### The SHA256 Reproducibility Problem
GPU floating-point arithmetic is non-associative — parallel operations accumulate rounding errors in different orders, producing slightly different weights each run even with identical code and data. **This is solvable:**
- **SGLang deterministic inference** (`--enable-deterministic-inference` flag) — batch-invariant kernels achieving bit-exact outputs
- **LayerCast** (FP32 computation, FP16 storage) — near-perfect reproducibility, validated across multiple GPU units
- **Constraint**: bit-exact reproducibility requires the same GPU model + driver version + CUDA version + PyTorch version. This is analogous to "same compiler version" in Linux — a reasonable, well-precedented constraint.

### The Autoresearch Foundation (Karpathy, March 2026)
Repo: https://github.com/karpathy/autoresearch

This is the most important new piece of the puzzle. It provides:
- Train from true **layer zero** (random init, no inherited weights)
- **Git as the complete provenance trail** — every architectural decision, hyperparameter change, experiment kept or discarded is a git commit with a measurable outcome (val_bpb metric)
- AI agent autonomously runs 5-minute training experiments in a loop, accumulates commits
- ~630 lines of code (fits in LLM context window), MIT licensed
- `uv.lock` pins all Python dependencies
- Single metric (val_bpb — validation bits per byte, lower = better) makes experiments objectively comparable
- Human programs `program.md` (the research intent); agent modifies `train.py` (the implementation)

**What autoresearch contributes that nothing else does:** the research *process* provenance — not just what the model is, but how every decision was made and why, from the very first training step.

### Legally Clean Training Data
- **Common Pile** (EleutherAI) — 8TB, strictly public domain + openly licensed text, 30 sources
- **Dolma** (Ai2) — 3T tokens, open corpus used by OLMo, documented and versioned
- **KL3M** — 1.2T tokens of US federal government public domain text
- For specialized/niche models: domain-specific datasets can be fully curated, documented, and distributed

---

## The Complete Technical Stack

For bit-exact sha256-verifiable model reproduction:

```
Layer 1: Research provenance
  → autoresearch (git log = complete decision history from commit zero)

Layer 2: Training data
  → Common Pile / Dolma / domain-specific curated corpus
  → Full provenance: source URLs, licenses, cleaning code, sha256 of dataset

Layer 3: Software environment (hermetic build container)
  → Docker image with pinned:
      - Ubuntu 22.04.x (specific point release)
      - CUDA 12.x.x (exact version)
      - PyTorch 2.x.x+cuXXX (exact build hash)
      - Python 3.10.x
      - All deps via uv.lock
  → Docker image itself has a published sha256

Layer 4: Deterministic training kernels
  → SGLang batch-invariant kernels OR LayerCast FP32 mode
  → Disable non-deterministic CUDA ops (torch.use_deterministic_algorithms(True))
  → Fixed random seed in prepare.py (already done in autoresearch)

Layer 5: Verification artifacts (published alongside model)
  → sha256sum of model weights at each checkpoint
  → sha256sum of Docker image
  → sha256sum of training dataset archive
  → Git commit hash of training code used
  → Hardware specification (GPU model, driver version)
```

**Verification by a community member:**
```bash
# Pull pinned build environment
docker pull <registry>/verifiable-llm-build@sha256:<image_hash>

# Run training (same GPU model required)
docker run --gpus all verifiable-llm-build ./train.sh

# Verify
sha256sum output/model_final.bin
# Compare to published: <expected_hash>
```

---

## What Does Not Yet Exist (The Gap to Fill)

No project currently assembles all five layers above with reproducibility as the *explicit primary goal*. The components exist; the integration does not. Specifically missing:

1. **autoresearch extended with deterministic kernels** — the project currently doesn't guarantee bit-exact weight reproduction across runs; adding SGLang's batch-invariant kernels and FP32 mode would close this
2. **Published sha256 of training outputs** — autoresearch logs val_bpb per commit but doesn't publish weight fingerprints
3. **Hermetic Docker build environment** — autoresearch uses uv.lock for Python but doesn't pin CUDA/GPU environment
4. **Dataset sha256 verification step** in prepare.py — currently downloads data without publishing expected hashes
5. **Community verification tooling** — a simple script that: pulls container, runs training, compares sha256 to published value, reports pass/fail

---

## Immediate Next Steps (Suggested)

**Phase 1 — Proof of concept (single GPU, small model)**
- [ ] Fork autoresearch
- [ ] Add `torch.use_deterministic_algorithms(True)` and LayerCast/FP32 mode to train.py
- [ ] Wrap in a pinned Docker image (CUDA + PyTorch + Python exact versions)
- [ ] Add sha256 verification of dataset in prepare.py
- [ ] Run two independent training runs on same GPU model, compare weight sha256
- [ ] Document: "verified reproducible on [GPU model] + [driver] + [CUDA] + [PyTorch]"
- [ ] Publish: model weights + sha256 + Docker image + sha256 + dataset + sha256

**Phase 2 — Community verification**
- [ ] Write `verify.sh`: pulls container, runs training, compares sha256, reports result
- [ ] Invite community members with same GPU model to run verify.sh and report results
- [ ] Document hardware compatibility matrix (which GPU models produce matching sha256)

**Phase 3 — Specialized domain model**
- [ ] Choose a niche domain with a legally clean, fully documentable dataset
- [ ] Train a small (1–7B) specialized model using the Phase 1 pipeline
- [ ] Full public release: dataset + build environment + training history (git log) + weights + sha256

---

## Key References

| Resource | URL | Why it matters |
|---|---|---|
| autoresearch (Karpathy) | https://github.com/karpathy/autoresearch | Layer-zero training + git provenance |
| nanochat (Karpathy) | https://github.com/karpathy/nanochat | Parent project, wider platform support |
| OLMo (Ai2) | https://allenai.org/olmo | Closest existing truly open model |
| Dolma dataset | https://huggingface.co/datasets/allenai/dolma | Open training corpus |
| Common Pile (EleutherAI) | https://arxiv.org/abs/2506.05209 | Legally clean dataset initiative |
| SGLang deterministic | https://lmsys.org/blog/2025-09-22-sglang-deterministic/ | Deterministic inference kernels |
| LayerCast reproducibility | https://arxiv.org/abs/2506.09501 | FP32 determinism approach |
| Ingonyama deterministic GEMM | https://www.ingonyama.com/post/solving-reproducibility-challenges-in-deep-learning-and-llms-our-journey | Deterministic CUDA kernels for ZKP |
| Reproducible Builds project | https://reproducible-builds.org | Methodology reference from software world |

---

## Philosophical Framing (for README / community communications)

The goal of this project is not to produce the most capable model. It is to produce the most *verifiable* model — one where capability and trustworthiness are independently auditable by any member of the community.

The analogy:
- **Linux**: the source code is the law. You can read it, build it, verify the binary.
- **Bitcoin**: the protocol enforces the rules. No participant is exempt.
- **This project**: the git log is the history. The sha256 is the fingerprint. No claim requires trust.

The intended community ethos is one of radical transparency: every decision documented, every artifact fingerprinted, every build reproducible. Not because we assume bad actors, but because a system that *structurally cannot* hide bad behavior is more trustworthy than one that merely *promises* not to behave badly — regardless of who builds it.

---

*Handoff prepared from research conversation, March 2026. Continue in Claude Code by reading this file and the autoresearch README.*
