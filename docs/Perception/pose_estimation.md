---
title: Pose Estimation
parent: Perception
nav_order: 2
---

# Pose Estimation
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

Pose estimation infers 3D position and orientation from sensor observations.

## Common inputs

- RGB images
- RGB-D images
- Point clouds
- Fiducial markers

## Typical pipeline

1. Detect keypoints or object regions.
2. Match features with model priors.
3. Solve for 6D pose.
4. Refine and track over time.

## Why it matters in manipulation

- Enables closed-loop grasping
- Supports hand-eye coordination
- Improves robustness to scene changes
