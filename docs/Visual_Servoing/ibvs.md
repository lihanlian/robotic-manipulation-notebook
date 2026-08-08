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

## 1. Clean up the notation first

The lecture slides use the symbol $$\mathbf{v}$$ for the six-dimensional camera velocity. To avoid confusing this symbol with the vertical pixel coordinate $$v$$, denote the camera twist by

$$
\boldsymbol{\nu}_c
=
\begin{bmatrix}
v_x & v_y & v_z & \omega_x & \omega_y & \omega_z
\end{bmatrix}^{T}.
$$

The first three components are translational velocity,

$$
\mathbf{v}_c
=
\begin{bmatrix}
v_x \\
v_y \\
v_z
\end{bmatrix},
$$

and the final three components are angular velocity,

$$
\boldsymbol{\omega}_c
=
\begin{bmatrix}
\omega_x \\
\omega_y \\
\omega_z
\end{bmatrix}.
$$

A fixed 3D point, expressed in the current camera frame, is

$$
P
=
\begin{bmatrix}
X \\
Y \\
Z
\end{bmatrix}.
$$

Its normalized image coordinates are

$$
s
=
\begin{bmatrix}
x \\
y
\end{bmatrix}
=
\begin{bmatrix}
X/Z \\
Y/Z
\end{bmatrix}.
$$

The objective is to derive

$$
\dot{s}=L_s\boldsymbol{\nu}_c,
$$

where $$L_s\in\mathbb{R}^{2\times6}$$ is the interaction matrix, or image Jacobian, for one point feature.

## 2. Why a fixed point moves in the camera frame

Suppose the world point is fixed but the camera moves. A point expressed in the camera frame is generally

$$
P_c=R_{cw}(P_w-C_w),
$$

where $$P_w$$ is the fixed world point, $$C_w$$ is the camera center in world coordinates, and $$R_{cw}$$ rotates world-frame vectors into camera coordinates.

Even though $$P_w$$ is fixed, $$P_c$$ changes because the camera coordinate frame moves. The rigid-body point-motion equation is

$$
\boxed{
\dot{P}
=
-\mathbf{v}_c
-
\boldsymbol{\omega}_c\times P
}.
$$

The minus signs appear because the expression describes a fixed world point as observed from a moving camera:

- If the camera translates right, the fixed point appears to move left.

- If the camera rotates in one direction, the scene appears to rotate in the opposite direction.

> A notation such as $$\dot{P}=-\mathbf{v}_{\mathrm{cam}}\times P$$ is often shorthand for the action of a six-dimensional twist. An ordinary cross product is only defined for two 3D vectors. The explicit and clearer form is
>
> $$
> \dot{P}
> =
> -\mathbf{v}_c
> -
> \boldsymbol{\omega}_c\times P.
> $$

## 3. Expand the rotational cross product

The cross product is

$$
\boldsymbol{\omega}_c\times P
=
\begin{bmatrix}
\omega_yZ-\omega_zY \\
\omega_zX-\omega_xZ \\
\omega_xY-\omega_yX
\end{bmatrix}.
$$

Therefore,

$$
-\boldsymbol{\omega}_c\times P
=
\begin{bmatrix}
-\omega_yZ+\omega_zY \\
-\omega_zX+\omega_xZ \\
-\omega_xY+\omega_yX
\end{bmatrix}.
$$

Adding translation gives

$$
\dot{P}
=
-\mathbf{v}_c
-
\boldsymbol{\omega}_c\times P,
$$

so the three coordinate equations are

$$
\boxed{
\dot{X}
=
-v_x-\omega_yZ+\omega_zY
},
$$

$$
\boxed{
\dot{Y}
=
-v_y-\omega_zX+\omega_xZ
},
$$

$$
\boxed{
\dot{Z}
=
-v_z-\omega_xY+\omega_yX
}.
$$

## 4. Understanding each term physically

Consider

$$
\dot{X}
=
-v_x-\omega_yZ+\omega_zY.
$$

The term $$-v_x$$ comes from camera translation along the camera $$x$$-axis. The term $$-\omega_yZ$$ comes from rotation about the camera $$y$$-axis, and $$+\omega_zY$$ comes from rotation about the optical $$z$$-axis.

Similarly,

$$
\dot{Y}
=
-v_y-\omega_zX+\omega_xZ
$$

contains the effects of translation along $$y$$, rotation about $$z$$, and rotation about $$x$$. Finally,

$$
\dot{Z}
=
-v_z-\omega_xY+\omega_yX
$$

contains the effects of translation along $$z$$, rotation about $$x$$, and rotation about $$y$$.

## 5. Define the normalized image coordinates

The slides first use normalized image coordinates:

$$
x=\frac{X}{Z},
\qquad
y=\frac{Y}{Z}.
$$

These coordinates correspond to projection onto a virtual image plane at unit distance from the camera center. In this normalized coordinate system, the focal-length scaling has temporarily been removed. The visual feature is

$$
s
=
\begin{bmatrix}
x \\
y
\end{bmatrix},
$$

and its time derivative is

$$
\dot{s}
=
\begin{bmatrix}
\dot{x} \\
\dot{y}
\end{bmatrix}.
$$

## 6. Derive $$\dot{x}$$ using the quotient rule

Start from

$$
x=\frac{X}{Z}.
$$

Differentiate using the quotient rule:

$$
\dot{x}
=
\frac{\dot{X}Z-X\dot{Z}}{Z^2}.
$$

Since $$X=xZ$$, this can also be written as

$$
\dot{x}
=
\frac{\dot{X}-x\dot{Z}}{Z}.
$$

Substitute

$$
\dot{X}
=
-v_x-\omega_yZ+\omega_zY
$$

and

$$
\dot{Z}
=
-v_z-\omega_xY+\omega_yX.
$$

Then

$$
\begin{aligned}
\dot{x}
&=
\frac{
-v_x-\omega_yZ+\omega_zY
-x(-v_z-\omega_xY+\omega_yX)
}{Z}
\\[4pt]
&=
\frac{
-v_x-\omega_yZ+\omega_zY
+xv_z+x\omega_xY-x\omega_yX
}{Z}.
\end{aligned}
$$

Use $$X=xZ$$ and $$Y=yZ$$:

$$
\begin{aligned}
\dot{x}
&=
\frac{
-v_x-\omega_yZ+\omega_zyZ
+xv_z+x\omega_xyZ-x\omega_yxZ
}{Z}
\\[4pt]
&=
-\frac{v_x}{Z}
+\frac{xv_z}{Z}
+xy\omega_x
-(1+x^2)\omega_y
+y\omega_z.
\end{aligned}
$$

Therefore,

$$
\boxed{
\dot{x}
=
-\frac{1}{Z}v_x
+\frac{x}{Z}v_z
+xy\omega_x
-(1+x^2)\omega_y
+y\omega_z
}.
$$

There is no $$v_y$$ term because a pure camera translation along $$y$$ does not directly change the normalized horizontal coordinate $$x$$.

## 7. Write the first row of the interaction matrix

Using the velocity ordering

$$
\boldsymbol{\nu}_c
=
\begin{bmatrix}
v_x & v_y & v_z & \omega_x & \omega_y & \omega_z
\end{bmatrix}^{T},
$$

we can write

$$
\dot{x}
=
\begin{bmatrix}
-\dfrac{1}{Z}
&
0
&
\dfrac{x}{Z}
&
xy
&
-(1+x^2)
&
y
\end{bmatrix}
\boldsymbol{\nu}_c.
$$

Thus, the first row is

$$
L_x
=
\begin{bmatrix}
-\dfrac{1}{Z}
&
0
&
\dfrac{x}{Z}
&
xy
&
-(1+x^2)
&
y
\end{bmatrix}.
$$

## 8. Derive $$\dot{y}$$

Start from

$$
y=\frac{Y}{Z}.
$$

Differentiate:

$$
\dot{y}
=
\frac{\dot{Y}Z-Y\dot{Z}}{Z^2}.
$$

Since $$Y=yZ$$,

$$
\dot{y}
=
\frac{\dot{Y}-y\dot{Z}}{Z}.
$$

Substitute

$$
\dot{Y}
=
-v_y-\omega_zX+\omega_xZ
$$

and

$$
\dot{Z}
=
-v_z-\omega_xY+\omega_yX.
$$

Then

$$
\begin{aligned}
\dot{y}
&=
\frac{
-v_y-\omega_zX+\omega_xZ
-y(-v_z-\omega_xY+\omega_yX)
}{Z}
\\[4pt]
&=
\frac{
-v_y-\omega_zX+\omega_xZ
+yv_z+y\omega_xY-y\omega_yX
}{Z}.
\end{aligned}
$$

Use $$X=xZ$$ and $$Y=yZ$$:

$$
\begin{aligned}
\dot{y}
&=
\frac{
-v_y-\omega_zxZ+\omega_xZ
+yv_z+y\omega_xyZ-y\omega_yxZ
}{Z}
\\[4pt]
&=
-\frac{v_y}{Z}
+\frac{yv_z}{Z}
+(1+y^2)\omega_x
-xy\omega_y
-x\omega_z.
\end{aligned}
$$

Therefore,

$$
\boxed{
\dot{y}
=
-\frac{1}{Z}v_y
+\frac{y}{Z}v_z
+(1+y^2)\omega_x
-xy\omega_y
-x\omega_z
}.
$$

## 9. Write the second row of the interaction matrix

The previous equation becomes

$$
\dot{y}
=
\begin{bmatrix}
0
&
-\dfrac{1}{Z}
&
\dfrac{y}{Z}
&
1+y^2
&
-xy
&
-x
\end{bmatrix}
\boldsymbol{\nu}_c.
$$

Thus,

$$
L_y
=
\begin{bmatrix}
0
&
-\dfrac{1}{Z}
&
\dfrac{y}{Z}
&
1+y^2
&
-xy
&
-x
\end{bmatrix}.
$$

## 10. The complete interaction matrix

Stack the two rows:

$$
\begin{bmatrix}
\dot{x} \\
\dot{y}
\end{bmatrix}
=
L_s
\begin{bmatrix}
v_x \\
v_y \\
v_z \\
\omega_x \\
\omega_y \\
\omega_z
\end{bmatrix},
$$

with

$$
\boxed{
L_s
=
\begin{bmatrix}
-\dfrac{1}{Z}
&
0
&
\dfrac{x}{Z}
&
xy
&
-(1+x^2)
&
y
\\[6pt]
0
&
-\dfrac{1}{Z}
&
\dfrac{y}{Z}
&
1+y^2
&
-xy
&
-x
\end{bmatrix}
}.
$$

This is the standard point-feature interaction matrix for normalized image coordinates.