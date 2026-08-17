---
title: GraspNet
parent: Grasping
nav_order: 2
---

# GraspNet
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

GraspNet is a large-scale 6-DoF grasp detection framework that predicts diverse grasp poses from point-cloud observations of cluttered scenes.

## Core ideas

- Dense grasp proposals over observed 3D geometry
- Grasp quality scoring for ranking candidates
- Dataset and benchmark support for large-scale evaluation

## Typical workflow

1. Acquire an RGB-D observation and build a point cloud.
2. Generate candidate 6-DoF grasps on the scene.
3. Score and filter grasps by quality and collision checks.
4. Select and execute a high-quality feasible grasp.

## Manipulation relevance

- Strong baseline for cluttered-scene 6-DoF grasping
- Useful for comparing learned grasp detectors
- Connects perception outputs to executable grasp poses
