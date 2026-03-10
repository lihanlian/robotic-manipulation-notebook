---
title: RRT
parent: Planning
nav_order: 1
---

# RRT
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

Rapidly-exploring Random Trees (RRT) are sampling-based motion planning methods that incrementally grow a tree in configuration space.

## Core idea

RRT expands from a start state by repeatedly:

1. Sampling a random state.
2. Finding the nearest node already in the tree.
3. Extending toward the sample with a bounded step.
4. Accepting the new node if it is collision-free.

## Why use RRT

- Works well in high-dimensional spaces.
- Handles complex obstacle geometries.
- Produces feasible paths quickly.

## Common variants

- RRT-Connect
- RRT*
- Informed RRT*
