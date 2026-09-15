---
title: VoxPoser
parent: VLA
nav_order: 8
---

# VoxPoser
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

VoxPoser is a zero-shot framework that synthesizes robot manipulation trajectories by composing 3D value maps with large language models (LLMs) and vision-language models (VLMs). Unlike the learned policies in this chapter, VoxPoser requires no training data — it extracts affordances and constraints directly from foundation models at test time.

## ▪ How It Works

1. **Instruction decomposition**: given a free-form language instruction, an LLM writes code (in the style of Language Model Programs from Code as Policies) that breaks the task into sub-tasks and invokes perception and planning APIs.
2. **Value-map composition**: for each sub-task, the generated code queries a VLM (e.g., an open-vocabulary detector) to localize objects of interest and composes two voxelized 3D value maps over the workspace:
   * an **affordance map** indicating where the end effector (or an object) should go;
   * a **constraint map** indicating regions to avoid, which is subtracted from the affordance map.
3. **Trajectory synthesis**: a motion planner optimizes a trajectory through the composed value map, and a controller (optionally with a learned dynamics model for MPC) executes the resulting waypoints.

## ▪ Properties

* **Zero-shot**: no task-specific training data or environment interaction is needed.
* **Composable**: value maps compose across sub-tasks, so long-horizon instructions are handled by sequential composition.
* **Interpretable**: the intermediate 3D value maps can be visualized, making failures easier to diagnose than in end-to-end policies.
* **Open-world**: because perception is grounded by open-vocabulary VLMs, instructions can refer to objects and attributes never seen during any training.

The released implementation demonstrates VoxPoser in RLBench; real-world deployment pairs it with an open-vocabulary detection + segmentation + tracking perception pipeline and an operational-space controller.

----
Reference:

- <i class="fa-brands fa-github" aria-hidden="true"></i> [VoxPoser GitHub repository](https://github.com/huangwl18/VoxPoser).
- <i class="fa-solid fa-book" aria-hidden="true"></i> [VoxPoser: Composable 3D Value Maps for Robotic Manipulation with Language Models](https://arxiv.org/abs/2307.05973).
