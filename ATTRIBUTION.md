# Attribution

Fishbowl's training foundation is derived from [autoresearch](https://github.com/karpathy/autoresearch) by Andrej Karpathy.

- **Original repo:** https://github.com/karpathy/autoresearch
- **Description:** AI agents running research on single-GPU nanochat training automatically
- **Files derived:** `train.py`, `prepare.py`, `pyproject.toml`, `uv.lock`, `.python-version`

Fishbowl extends autoresearch with deterministic training, hermetic build environments, and sha256 verification — making the training process independently reproducible and verifiable.
