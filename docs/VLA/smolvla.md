---
title: SmolVLA
parent: VLA
nav_order: 7
---

# SmolVLA
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

SmolVLA is a compact, efficient vision-language-action model from Hugging Face, trained entirely on open LeRobot community datasets. It is designed for accessibility: the model is small enough to train and deploy on consumer-grade hardware while remaining competitive with much larger VLA models.

## ▪ Key Properties

* **Compact backbone**: built on SmolVLM-2, a small vision-language model, keeping the total parameter count around 450M.
* **Flow-matching action expert**: a lightweight action expert predicts continuous action chunks with flow matching, similar in spirit to π₀.
* **Efficient training**: trained only on publicly available LeRobot community data, without any proprietary datasets.
* **Asynchronous inference**: the policy server can predict the next action chunk while the robot is still executing the current one, hiding inference latency during deployment.

----
Reference:

- <i class="fa-brands fa-chrome"></i> [SmolVLA on Hugging Face](https://huggingface.co/blog/smolvla) — blog post introducing the model and training setup.
- <i class="fa-solid fa-book" aria-hidden="true"></i> [SmolVLA: A Vision-Language-Action Model for Affordable and Efficient Robotics](https://arxiv.org/abs/2506.01844).
