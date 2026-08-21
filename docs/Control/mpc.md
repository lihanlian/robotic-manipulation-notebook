---
title: Model Predictive Control
parent: Control
nav_order: 5
---

# MPC
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

Model Predictive Control (MPC) optimizes control actions over a finite prediction horizon using a dynamics model, then applies the first action and re-plans at the next step.

## Typical pipeline

1. Predict future states with a system model.
2. Optimize a cost over the horizon (tracking, constraints, effort).
3. Apply the first control input.
4. Receed and repeat with updated observations.

## Why it matters in manipulation

- Handles constraints on joint limits, contact, and workspace
- Useful for contact-rich and tracking tasks
- Bridges trajectory planning and closed-loop control
