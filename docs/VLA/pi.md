---
title: Pi
parent: VLA
nav_order: 5
---

# Pi
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

The π (Pi) models are a family of vision-language-action models developed by Physical Intelligence and released through the open-source `openpi` repository. The repo provides base checkpoints pretrained on more than 10k hours of robot data, together with training and fine-tuning code in both JAX and PyTorch.

## ▪ Model Family

* **π₀**: a flow-based VLA. A pretrained VLM backbone (PaliGemma) encodes images and language, and a smaller *action expert* generates continuous action chunks with flow matching, enabling high-frequency (up to 50 Hz) dexterous control.
* **π₀-FAST**: an autoregressive variant built on the FAST action tokenizer, which compresses action chunks into a short sequence of discrete tokens for language-model-style generation.
* **π₀.₅**: an upgraded version of π₀ trained with *knowledge insulation*, giving substantially better open-world generalization; the repo currently supports it with the flow-matching action head.

## ▪ Checkpoints

Base checkpoints (`pi0_base`, `pi0_fast_base`, `pi05_base`) are intended for fine-tuning, while "expert" checkpoints fine-tuned on specific platforms — DROID, ALOHA, and LIBERO — can often run zero-shot on those setups. Checkpoints are downloaded automatically from `gs://openpi-assets` and cached locally.

## ▪ Usage

Running a pretrained policy takes a few lines:

```python
from openpi.training import config as _config
from openpi.policies import policy_config
from openpi.shared import download

config = _config.get_config("pi05_droid")
checkpoint_dir = download.maybe_download("gs://openpi-assets/checkpoints/pi05_droid")
policy = policy_config.create_trained_policy(config, checkpoint_dir)
action_chunk = policy.infer(example)["actions"]
```

Fine-tuning uses LeRobot-format datasets: convert the data, compute normalization statistics with `scripts/compute_norm_stats.py`, train with `scripts/train.py`, and serve the resulting checkpoint through a websocket policy server (`scripts/serve_policy.py`) so the robot client can stream observations and receive action chunks remotely.

----
Reference:

- <i class="fa-brands fa-github" aria-hidden="true"></i> [openpi GitHub repository](https://github.com/Physical-Intelligence/openpi).
- <i class="fa-solid fa-book" aria-hidden="true"></i> [π₀: A Vision-Language-Action Flow Model for General Robot Control](https://arxiv.org/abs/2410.24164).
- <i class="fa-solid fa-book" aria-hidden="true"></i> [π₀.₅: a Vision-Language-Action Model with Open-World Generalization](https://arxiv.org/abs/2504.16054).
