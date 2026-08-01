---
title: Behavior Cloning
parent: Imitation Learning
nav_order: 1
---

# Behavior Cloning
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

Behavior cloning learns a policy by supervised learning on expert demonstration data, mapping observations to actions.

## Typical pipeline

1. Collect expert demonstrations.
2. Train a policy to imitate expert actions.
3. Deploy the learned policy.

## Why it matters in manipulation

- Simple and widely used baseline for learning from demos
- Foundation for methods such as DAgger, ACT, and Diffusion Policy
- Sensitive to covariate shift when the policy leaves the expert distribution
