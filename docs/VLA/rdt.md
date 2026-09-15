---
title: RDT
parent: VLA
nav_order: 6
---

# RDT
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

RDT-1B (Robotics Diffusion Transformer) is a 1B-parameter diffusion foundation model for bimanual manipulation. Pretrained on more than 1M multi-robot episodes, it uses diffusion in a continuous action space to capture the multimodal action distributions that arise in dexterous dual-arm tasks.

## ▪ Architecture

* **Backbone**: a Diffusion Transformer (DiT) that iteratively denoises a chunk of future actions, predicting the next 64 robot actions per inference.
* **Vision encoder**: SigLIP (`siglip-so400m-patch14-384`) encodes RGB images from up to three camera views.
* **Language encoder**: T5-XXL (`t5-v1_1-xxl`) encodes the language instruction; because of its size, language embeddings are often precomputed and cached.
* **Unified action space**: actions from different robots are embedded into a unified 128-dimensional action vector with reserved slots for each physical quantity (joint positions, end-effector poses with 6D rotation, gripper width, and even wheeled base motion), making RDT compatible with single-arm, dual-arm, and mobile manipulators.

## ▪ Training and Deployment

* **Pretraining**: 1M+ episodes from a large multi-robot data collection; a 1M-step RDT-1B checkpoint and a smaller RDT-170M checkpoint are released on Hugging Face.
* **Fine-tuning**: the authors fine-tuned RDT on 6K+ self-collected bimanual episodes and deployed it on ALOHA dual-arm robots, achieving strong dexterity, zero-shot generalization, and few-shot learning. Fine-tuning uses DeepSpeed and typically runs for at least ~150K steps.
* **Benchmarks**: evaluation scripts are provided for the ManiSkill simulation benchmark, and RDT ranks among the top policies on the RoboTwin dual-arm benchmark.

----
Reference:

- <i class="fa-brands fa-github" aria-hidden="true"></i> [RoboticsDiffusionTransformer GitHub repository](https://github.com/thu-ml/RoboticsDiffusionTransformer).
- <i class="fa-solid fa-book" aria-hidden="true"></i> [RDT-1B: a Diffusion Foundation Model for Bimanual Manipulation](https://arxiv.org/abs/2410.07864).
