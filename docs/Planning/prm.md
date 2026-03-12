---
title: PRM
parent: Planning
nav_order: 2
---

# PRM
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

Probabilistic Roadmap Methods (PRM) build a reusable graph of collision-free configurations, then search the graph for a feasible path.

## Core idea

1. Sample many collision-free configurations.
2. Connect nearby samples with local planners.
3. Build a roadmap graph offline.
4. For each query, connect start/goal and run graph search.

## Why use PRM

- Good for multi-query planning problems
- Scales well in high-dimensional configuration spaces
- Reuses the same roadmap across tasks

## Common variants

- Basic PRM
- Lazy PRM
- PRM*
