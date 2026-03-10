---
title: PBVS
parent: Visual Servoing
nav_order: 2
---

# PBVS
{: .no_toc }

Position-Based Visual Servoing (PBVS) estimates the target pose and controls the robot in Cartesian space.
{: .fs-6 .fw-300 }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Core idea

PBVS closes the loop on 3D pose error:

1. Estimate camera/target relative pose from image measurements.
2. Compute pose error in SE(3).
3. Apply a control law to reduce translational and rotational error.

## Typical advantages

- Intuitive geometric interpretation
- Smooth Cartesian trajectories
- Easy integration with model-based controllers

## Typical challenges

- Sensitive to pose-estimation errors
- Requires camera calibration accuracy
- Can degrade with poor visual features
