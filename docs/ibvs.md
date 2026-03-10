---
title: IBVS
parent: Visual Servoing
nav_order: 1
---

# IBVS
{: .no_toc }

Image-Based Visual Servoing (IBVS) uses image-space features directly in the feedback loop.
{: .fs-6 .fw-300 }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Core idea

1. Detect visual features in the image plane.
2. Define image error relative to desired feature positions.
3. Use an image Jacobian (interaction matrix) to compute camera/robot velocity commands.

## Typical advantages

- Directly uses image measurements
- Less dependence on full 3D pose estimation
- Effective for feature-tracking tasks

## Typical challenges

- Interaction matrix can become singular
- Sensitive to feature loss and noise
- May yield unintuitive Cartesian trajectories
