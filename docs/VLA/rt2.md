---
title: RT-2
parent: VLA
nav_order: 2
---

# RT-2
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

RT-2 (Robotics Transformer 2) is a vision-language-action (VLA) model from Google DeepMind that fine-tunes large web-pretrained vision-language models (VLMs) to directly output robot actions. The key idea is that web-scale pretraining endows the policy with semantic and visual generalization that robot data alone cannot provide.

## ▪ From VLM to VLA

RT-2 instantiates the VLA recipe as follows:

1. **Backbone**: start from a large VLM pretrained on web data — either PaLI-X (55B) or PaLM-E (12B).
2. **Actions as text**: continuous robot actions are discretized into 256 bins per dimension and represented as strings of digits, so they can be generated with the model's existing text-token output space.
3. **Co-fine-tuning**: the VLM is fine-tuned on a mixture of robot trajectory data (from RT-1) and the original web data, which prevents catastrophic forgetting and preserves generalization.

## ▪ Emergent Generalization

Because RT-2 inherits web-scale knowledge from its VLM backbone, it exhibits emergent capabilities not present in the robot training data, such as:

* recognizing and manipulating novel objects and object categories;
* following instructions that require semantic reasoning (e.g., "pick up the extinct animal");
* generalizing to new backgrounds, lighting conditions, and environments.

## ▪ RT-2-X

A later variant, RT-2-X, trains the same architecture on the Open X-Embodiment dataset — a pooled collection of robot data from many different embodiments — showing that cross-embodiment training transfers skills across robots and improves performance even on the source robots.

----
Reference:

- <i class="fa-solid fa-book" aria-hidden="true"></i> [RT-2: Vision-Language-Action Models Transfer Web Knowledge to Robotic Control](https://arxiv.org/abs/2307.15818).
- <i class="fa-solid fa-book" aria-hidden="true"></i> [Open X-Embodiment: Robotic Learning Datasets and RT-X Models](https://arxiv.org/abs/2310.08864).
