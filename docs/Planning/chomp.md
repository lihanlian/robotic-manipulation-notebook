---
title: CHOMP
parent: Motion Planning
nav_order: 3
---

# CHOMP
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

CHOMP (Covariant Hamiltonian Optimization for Motion Planning) is a trajectory optimization method that improves an initial path into a smooth, collision-free motion.

## Core idea

1. Start from an initial trajectory guess.
2. Define a cost that combines smoothness and obstacle avoidance.
3. Iteratively optimize the trajectory by gradient-based updates.

## Why CHOMP for manipulation

- Produces smooth, executable trajectories for arm motion.
- Handles cluttered environments through obstacle-cost gradients.
- Integrates well with kinematic and collision constraints.

## Practical notes

- Quality depends on the initial trajectory.
- Good signed-distance fields improve obstacle gradients.
- Often used with post-processing or time-parameterization.
