---
title: GR-ConvNet
parent: Grasping
nav_order: 3
---

# GR-ConvNet
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

GR-ConvNet (Generative Residual Convolutional Neural Network) predicts antipodal grasp configurations from RGB-D images using a generative residual architecture.

## Core ideas

- Image-based grasp generation from depth (and optionally RGB)
- Residual convolutional network for efficient inference
- Outputs grasp quality, angle, and width maps

## Typical workflow

1. Capture an RGB-D image of the scene.
2. Run GR-ConvNet to predict grasp quality, orientation, and width.
3. Sample peak-quality pixel locations as grasp candidates.
4. Convert image-plane grasps to robot/gripper poses and execute.

## Manipulation relevance

- Fast planar/antipodal grasping from camera views
- Common baseline for learning-based grasp detection
- Complements 6-DoF methods such as GraspNet in cluttered scenes
