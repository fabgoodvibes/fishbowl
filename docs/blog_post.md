# Can't Be Evil: Toward Fully Verifiable, Reproducible AI

*A case for applying the free software ethos — "rules without rulers" — to language model development.*

---

## The Problem Nobody Is Naming Clearly

The AI field has a trust problem, and it's not the one most people are talking about.

The common framing is: *can we trust what AI says?* But there's a deeper, structural question underneath it: *can we trust how AI was built?* And crucially — do we have to take anyone's word for it?

Right now, the answer is yes. We have to take their word for it. Even the most "open" models available today only release their weights — the finished numerical artifact — without the training data, the training code, the data curation decisions, or the reward modeling choices that shaped the model's behavior. You can download the model. You cannot verify how it came to be.

This is the same as a pharmaceutical company releasing a drug and saying "trust us, we ran the trials" — without publishing the trial data. In any other field of engineering or science, we'd call this unacceptable. In AI, we've normalized it.

The issue is not malice. It's that the current structure makes malice *undetectable*. A model could have been trained to behave subtly differently under specific trigger conditions. Its training data could have been curated to instill particular biases or knowledge gaps. None of this would be visible from the weights alone, and none of it would be caught by standard benchmarks. The problem isn't "don't be evil" — it's that there's currently no architecture that enforces *can't be evil*.

This isn't a geopolitical argument. It applies equally to any sufficiently powerful and opaque actor: a corporation, a government, a research lab, a well-funded individual. Opacity at the foundation of a technology that millions of people are beginning to depend on is a structural risk, regardless of who holds the opacity.

---

## What "Open" Actually Means (And Doesn't)

The term "open source" is being used loosely in AI in ways that would not pass muster in the free software community that coined it.

A truly open model — in the spirit of Linux, or the FSF's four freedoms — would require all of the following to be publicly available and independently reproducible:

- **Model weights** — the numerical parameters (what most today call "open weight")
- **Training dataset** — every document, its source, its license, and the code used to clean and filter it
- **Training code and hyperparameters** — the exact recipe, reproducible from scratch
- **All intermediate checkpoints** — so the training *process* can be studied, not just the outcome
- **Evaluation methodology** — including what the model fails at, not just where it excels
- **The research decision log** — why this architecture, why this data mixture, why these choices

Almost no model in existence releases all six. Most release only the first. The closest existing effort is OLMo from the Allen Institute for AI, which releases weights, training data (the Dolma corpus), training code, and evaluation tooling — and even built OLMoTrace, a tool that lets you trace any model output back to specific documents in the training set. That is the correct direction. But even OLMo does not yet provide the guarantee that running its training code on your hardware produces the same sha256 fingerprint as the released weights.

That last step — the sha256 guarantee — is the difference between "we published everything" and "you can verify everything." It is the difference between *don't be evil* and *can't be evil*.

---

## The Technical Path: From Layer Zero to Verified Fingerprint

Every technical component needed to build a fully verifiable, reproducible AI training pipeline already exists. What doesn't yet exist is the project that assembles them with reproducibility as the explicit primary goal.

The complete stack:

**1. Train from true layer zero.** No pre-trained weights, no inherited opacity. Every decision that shapes the model is recorded. Karpathy's `autoresearch` project (March 2026) demonstrates this: an AI agent runs training experiments on a minimal GPT implementation, with every architectural change and experimental outcome committed to git. The git history *is* the provenance trail — complete from the first commit, auditable by anyone.

**2. Use a legally clean, fully documented dataset.** EleutherAI's Common Pile (public domain and openly licensed text) and Ai2's Dolma corpus show that competitive LLM training is possible on fully auditable data. For specialized domain models, this is even more tractable — the dataset can be small enough to be completely documented and distributed alongside the model.

**3. Pin the entire software environment.** A hermetic build container (Docker, pinned to specific CUDA, PyTorch, Python, and all dependency versions) makes the software stack independently reproducible. `uv.lock` in autoresearch already does this for the Python layer.

**4. Use deterministic training kernels.** GPU floating-point arithmetic is non-associative — parallel operations accumulate rounding errors in different orders, producing slightly different weights each run. This is solvable: SGLang's batch-invariant kernels and the LayerCast approach (FP32 computation with FP16 storage) both achieve bit-exact reproducibility, validated across multiple physical GPU units of the same model.

**5. Publish sha256 fingerprints of weights at every checkpoint.** Once steps 1–4 are in place, any community member with the same GPU model running the same container on the same dataset arrives at the same weights. Checking it is as simple as `sha256sum model.bin`.

The Linux kernel analogy holds precisely: you don't expect `gcc 13.2` and `gcc 14.1` to produce the same binary. The constraint is "same compiler" — here it's "same GPU model and software stack." That is a well-precedented and entirely reasonable constraint. It doesn't undermine the reproducibility claim; it defines its scope, just as Linux does.

---

## Why This Matters: Rules Without Rulers

The Linux kernel's power isn't just technical. It's social and philosophical: the code is the law. No individual, company, or government can unilaterally change what Linux does without the change being visible to every person who can read code. The rules exist without requiring trust in any ruler.

Bitcoin applied this idea to money: the protocol enforces the rules, and no participant can exempt themselves from them.

This project asks: can we build AI with the same property?

Not AI that *promises* to be transparent. Not AI from organizations that say they have good values. AI where the training process is verifiable by any technically capable third party, where the history of every decision lives in a public git log, and where the resulting weights carry a cryptographic fingerprint that anyone can check.

For small, specialized models trained on domain-specific, legally clean data — the barriers are engineering problems, not fundamental ones. A 1–7B parameter model for a specific domain can be trained on a single consumer GPU, documented completely, and made fully verifiable. The technology is ready.

The missing piece is the project that assembles it with reproducibility as the first-class goal — and the community that holds it to that standard.

That project begins with a single GPU, a clean dataset, a pinned software environment, and a git log that starts at commit zero.

---

*This post summarizes a research conversation exploring open-weight vs. truly open-source AI, reproducibility barriers in LLM training, and the autoresearch framework by Andrej Karpathy as a foundation for verifiable AI development from layer zero.*
