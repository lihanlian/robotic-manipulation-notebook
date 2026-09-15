---
title: Octo
parent: VLA
nav_order: 3
---

# Octo
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

Octo is an open-source generalist robot policy: a transformer-based diffusion policy trained on a diverse mix of about 800k robot trajectories from the Open X-Embodiment dataset. It is designed to be an effective initialization for fine-tuning to new robot setups with modest data and compute budgets.

## ▪ Model Variants

| Model      | Parameters | Inference on 1x NVIDIA 4090 |
| ---------- | ---------- | --------------------------- |
| Octo-Base  | 93M        | ~13 it/sec                  |
| Octo-Small | 27M        | ~17 it/sec                  |

## ▪ Architecture

Octo has a modular structure with three main components:

1. **Tokenizers**: encode each input modality into tokens — a lightweight CNN tokenizer for each RGB camera stream and a text tokenizer for language instructions.
2. **Transformer backbone**: processes the token sequence with a modular, block-wise attention mask that separates observation tokens, task tokens, and readout tokens.
3. **Readout heads**: a diffusion action head predicts action chunks from the readout tokens.

Key design properties:

* **Task flexibility**: the policy can be conditioned on language commands or goal images.
* **Multi-camera support**: multiple RGB inputs (e.g., third-person and wrist cameras) are handled natively.
* **History and chunking**: pretrained with an observation history window of 2 timesteps and an action chunk size of 4.
* **Easy fine-tuning**: new observation tokenizers or action heads can be attached while keeping the pretrained transformer backbone frozen or partially frozen (`head_only`, `head_mlp_only`, or `full` fine-tuning modes).

## ▪ Usage

Pretrained checkpoints are hosted on Hugging Face and can be loaded in a few lines:

```python
from octo.model.octo_model import OctoModel

model = OctoModel.load_pretrained("hf://rail-berkeley/octo-base-1.5")
task = model.create_tasks(texts=["pick up the spoon"])
action = model.sample_actions(observation, task, rng=jax.random.PRNGKey(0))
```

----
Reference:

- <i class="fa-brands fa-github" aria-hidden="true"></i> [Octo GitHub repository](https://github.com/octo-models/octo).
- <i class="fa-solid fa-book" aria-hidden="true"></i> [Octo: An Open-Source Generalist Robot Policy](https://arxiv.org/abs/2405.12213).
