---
title: Inverse Kinematics
parent: Kinematics
nav_order: 3
---

# Inverse Kinematics
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

Inverse kinematics determines the joint configuration required to place a robot’s end effector at a desired position and orientation. In contrast to forward kinematics, which maps joint variables to a unique pose, inverse kinematics may admit one solution, multiple solutions, infinitely many solutions, or no feasible solution. It can be formulated as either a root-finding problem, where the pose error is driven to zero, or an optimization problem that minimizes the remaining error. Common approaches include analytical solutions and iterative numerical methods based on the manipulator Jacobian. Their performance depends on robot geometry, target reachability, singularities, constraints, and initial configuration.


## IK Problem Formulation

- Pose combines translation and rotation
- Common orientation representations include rotation matrices, Euler angles, and quaternions
- Transformations map points and frames between coordinate systems


## Pose Error

## Numerical Inverse Kinematics

## Numerical IK versus Differential IK

----
Reference:

- <i class="fa-solid fa-book" aria-hidden="true"></i> [MuJoCo XML Reference (actuator)](https://mujoco.readthedocs.io/en/3.3.7/XMLreference.html#actuator-position).
