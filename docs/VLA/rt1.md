---
title: RT-1
parent: VLA
nav_order: 1
---

# RT-1
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

RT-1 (Robotics Transformer 1) is one of the first large-scale transformer-based robot policies, introduced by Google DeepMind in 2022. It demonstrated that a single transformer policy trained on a large, diverse real-world dataset can perform hundreds of distinct manipulation tasks and generalize to new tasks, objects, and environments.

## ▪ Dataset

RT-1 was trained on a real-world dataset collected with a fleet of 13 Everyday Robots (EDR) mobile manipulators over roughly 17 months:

* about 130k real robot episodes;
* more than 700 distinct language-instructed tasks;
* data collected with a mix of human demonstrations and autonomous execution.

## ▪ Architecture

RT-1 maps image observations and a language instruction directly to discretized robot actions:

1. **Image encoding**: the camera image is processed by an EfficientNet-B3 pretrained on ImageNet.
2. **Language conditioning**: the instruction is embedded with a Universal Sentence Encoder and injected into the image features through FiLM conditioning.
3. **Token compression**: a TokenLearner module compresses the visual feature maps into a small set of tokens, keeping the transformer tractable.
4. **Transformer backbone**: a decoder-only transformer attends over a short history of visual tokens and outputs action tokens.

## ▪ Action Representation

Actions are discretized into 256 bins per dimension and predicted autoregressively as tokens. The 11-dimensional action vector covers:

* 7 dimensions for the arm (end-effector position, orientation, and gripper);
* 3 dimensions for the mobile base;
* 1 dimension for episode termination.

The discretized, token-based action space lets RT-1 reuse standard language-modeling machinery for control, at the cost of a modest inference rate of about 3 Hz.

----
Reference:

- <i class="fa-solid fa-book" aria-hidden="true"></i> [RT-1: Robotics Transformer for Real-World Control at Scale](https://arxiv.org/abs/2212.06817).
