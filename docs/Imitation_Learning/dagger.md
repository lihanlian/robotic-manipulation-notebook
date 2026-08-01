---
title: DAgger
parent: Imitation Learning
nav_order: 2
---

# DAgger
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

DAgger (Dataset Aggregation) iteratively improves a cloned policy by querying an expert on states visited by the learner and aggregating those labeled samples into the training set.

## Typical pipeline

1. Train an initial policy with behavior cloning.
2. Roll out the learner policy and collect visited states.
3. Query the expert for actions on those states.
4. Aggregate the new data and retrain.

## Why it matters in manipulation

- Reduces covariate shift relative to pure behavior cloning
- Improves robustness when the policy encounters novel states
- Requires interactive access to an expert during training
