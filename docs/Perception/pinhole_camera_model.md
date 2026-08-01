---
title: Pinhole Camera Model
parent: Perception
nav_order: 1
---

# Pinhole Camera Model
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

The pinhole camera model relates 3D points in the world to 2D pixel coordinates through perspective projection.

## Projection

A 3D point in the camera frame is mapped to the image plane by:

\[
u = f_x \frac{X}{Z} + c_x, \quad
v = f_y \frac{Y}{Z} + c_y
\]

where \((f_x, f_y)\) are focal lengths and \((c_x, c_y)\) is the principal point.

## Why it matters in manipulation

- Foundation for camera calibration
- Used in pose estimation and visual servoing
- Links geometry of the scene to image measurements
