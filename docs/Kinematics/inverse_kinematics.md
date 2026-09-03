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

Inverse kinematics(IK) determines the joint configuration required to place a robot’s end effector at a desired position and orientation. In contrast to forward kinematics, which maps joint variables to a unique pose, inverse kinematics may admit one solution, multiple solutions, infinitely many solutions, or no feasible solution. 

IK can be formulated as either a root-finding problem, where the pose error is driven to zero, or an optimization problem that minimizes the remaining error. Here, we focus on the <span style="color: red"> damped least-squares (DLS) </span> approach and corresponding implementation can be found at <i class="fa-brands fa-github" aria-hidden="true"></i> [this repository](https://github.com/lihanlian/robot-manipulator-control/blob/main/controller/diffik_dls.py).


## ▪ IK Problem Formulation

IK seeks a joint configuration $$q^\star$$ whose forward kinematics matches a desired end-effector pose:

$$
T(q^\star)=T_d.
$$

Here, $$T(q)$$ is the forward-kinematics function that maps a joint configuration $$q$$ to the corresponding end-effector pose, while $$T_d$$ is the desired pose.

Equivalently, IK searches for a configuration at which the pose error is zero:

$$
e(q^\star)=0.
$$

Here, $$e(q)$$ is a pose-error function that measures the position and orientation difference between the current pose $$T(q)$$ and the desired pose $$T_d$$. When $$e(q)=0$$, the end effector has reached the desired pose.

Analytical IK derives joint solutions directly from robot geometry and can be extremely fast when a closed-form solution exists. However, the derivation is robot-specific and is generally unavailable for arbitrary or redundant manipulators. This section therefore focuses on numerical IK, which can be applied to a much wider range of robot structures.

Numerical inverse kinematics solves this nonlinear equation iteratively. Starting from an initial configuration $$q^0$$, it repeatedly calculates a local joint correction until the pose error becomes sufficiently small.

### Local Linearization

The Jacobian provides the fundamental relationship between joint velocity and end-effector velocity:

$$
V=J(q)\dot q.
$$

Here, $$\dot q\in\mathbb{R}^n$$ is the joint-velocity vector, while $$V\in\mathbb{R}^6$$ is the end-effector velocity containing both linear and angular velocity. The Jacobian $$J(q)\in\mathbb{R}^{6\times n}$$ depends on the current joint configuration and describes how motion of each joint affects the end effector. Each column represents the instantaneous end-effector motion produced by one joint moving at unit velocity.

The forward-kinematics mapping is nonlinear, so the complete IK problem is generally not solved in one step. Numerical IK applies the same local relationship to small displacements. Around the current estimate $$q^i$$, a small joint correction $$\Delta q^i$$ produces an approximately linear end-effector correction:

$$
J(q^i)\Delta q^i
\approx
e(q^i),
$$

where $$e(q^i)$$ is the current pose error. This equation is used to calculate a joint correction that moves the end effector toward the desired pose.

Because the approximation is only accurate near the current configuration, the solver repeatedly:

1. computes the current end-effector pose;
2. evaluates the current pose error;
3. computes the current Jacobian;
4. calculates and applies the next joint correction.

### Pose-Error Convention

A pose error describes the correction required to move the end effector from its current pose to the desired pose. Damped least squares does not require one particular error representation, but the error vector and Jacobian must use the same reference frame and component ordering.

The subscripts indicate what each error represents:

* $$e_p$$: position error, where $$p$$ stands for position;
* $$e_R$$: orientation error, where $$R$$ refers to the rotation matrix;
* $$e_b$$: full pose error expressed in the body frame, where $$b$$ stands for body;
* $$e$$: the complete error vector used by the solver.

A full body-frame pose error can be defined as

$$
e_b(q)
=
\operatorname{Log}
\left(
T(q)^{-1}T_d
\right)^\vee.
$$

Here, $$T(q)$$ is the current end-effector pose and $$T_d$$ is the desired pose. <span style="color: red"> The relative transformation $$T(q)^{-1}T_d$$ describes the motion from the current pose to the desired pose, expressed in the current end-effector frame. </span> Its logarithm produces a six-dimensional error containing both translational and rotational corrections. Because $$e_b(q)$$ is expressed in the body frame, it should be paired with a body Jacobian.

The aforementioned implementation (`diffik_dls.py`) instead uses separate position and orientation errors expressed in a world-aligned frame. The position error is

$$
e_p(q)=p_d-p(q),
$$

where $$p(q)$$ is the current end-effector position and $$p_d$$ is the desired position.

The order is desired minus current because the result points from the current position toward the target. This sign can be derived from the local position approximation

$$
p(q+\Delta q)
\approx
p(q)+J_p(q)\Delta q.
$$

To make the next position approach $$p_d$$, set

$$
p(q)+J_p(q)\Delta q
\approx
p_d.
$$

Rearranging gives

$$
J_p(q)\Delta q
\approx
p_d-p(q)
=
e_p(q).
$$

Therefore, using desired minus current allows the correction to appear directly on the right-hand side of the IK equation. If the error were instead defined as current minus desired, the correction equation would require an additional negative sign:

$$
J_p(q)\Delta q
\approx
-\left(p(q)-p_d\right).
$$

Both conventions are mathematically valid, but the error definition and update equation must be consistent. Using desired minus current is intuitive because a positive feedback gain produces motion toward the target.

Orientations cannot be subtracted directly like position vectors. Instead, the world-aligned orientation error is defined using the relative rotation

$$
e_R(q)
=
\operatorname{Log}
\left(
R_dR(q)^T
\right)^\vee,
$$

where $$R(q)$$ is the current orientation and $$R_d$$ is the desired orientation. The multiplication order follows from

$$
R_{\mathrm{err}}R(q)=R_d,
$$

which gives

$$
R_{\mathrm{err}}
=
R_dR(q)^T.
$$

Thus, $$R_dR(q)^T$$ is the rotation that transforms the current orientation toward the desired orientation, expressed in the world-aligned frame. The logarithm converts this rotation matrix into a three-dimensional rotation vector.

The complete error used by the code is

$$
e(q)
=
\begin{bmatrix}
e_p(q)\\
e_R(q)
\end{bmatrix}
\in\mathbb{R}^6,
$$

with position error first and orientation error second.

The matching Jacobian is

$$
J(q)
=
\begin{bmatrix}
J_p(q)\\
J_\omega(q)
\end{bmatrix}
\in\mathbb{R}^{6\times n},
$$

where $$J_p(q)$$ maps joint motion to linear end-effector motion and $$J_\omega(q)$$ maps joint motion to angular end-effector motion.

This ordering matches the code:

* `self._jacobian[:3]` contains $$J_p$$, the translational Jacobian;
* `self._jacobian[3:]` contains $$J_\omega$$, the rotational Jacobian;
* `self._task_velocity[:3]` contains the commanded linear velocity;
* `self._task_velocity[3:]` contains the commanded angular velocity.


### Iterative Algorithm

Starting from an initial guess $$q^0$$, one solver iteration performs:

1. Compute the current pose $$T(q^i)$$.
2. Compute the pose error $$e(q^i)$$.
3. Stop if the position and orientation errors are sufficiently small.
4. Compute the matching Jacobian $$J(q^i)$$.
5. Calculate the damped least-squares correction.
6. Update the configuration:

$$
q^{i+1}
=
q^i+\alpha\Delta q^i,
$$

where $$0<\alpha\leq1$$ is an optional step size.

For configurations that do not form a Euclidean vector space, the more general update is

$$
q^{i+1}
=
\operatorname{integrate}
\left(
q^i,\alpha\Delta q^i
\right).
$$

The Jacobian and pose error are then recomputed because the linear approximation changes with the configuration.

## ▪ Numerical Inverse Kinematics

### Root-Finding and Newton–Raphson Intuition

Consider a general root-finding problem:

$$
r(q)=0.
$$

Linearizing the residual around the current estimate $$q^i$$ gives

$$
r(q^i+\Delta q^i)
\approx
r(q^i)+J_r(q^i)\Delta q^i,
$$

where

$$
J_r(q^i)
=
\frac{\partial r}{\partial q}
\bigg|_{q=q^i}
$$

is the Jacobian of the residual.

Newton–Raphson chooses a correction that makes the local approximation equal to zero:

$$
r(q^i)+J_r(q^i)\Delta q^i=0.
$$

Therefore,

$$
J_r(q^i)\Delta q^i
=
-r(q^i).
$$

If the pose error uses the desired-minus-current convention,

$$
e(q)=-r(q),
$$

then the correction must approximately satisfy

$$
J(q^i)\Delta q^i
=
e(q^i).
$$

For a square and nonsingular Jacobian, this gives

$$
\Delta q^i
=
J(q^i)^{-1}e(q^i).
$$

The next estimate is then

$$
q^{i+1}
=
q^i+\Delta q^i.
$$

This is the Newton–Raphson interpretation of numerical IK: locally linearize the nonlinear root equation, solve the linearized equation, update the configuration, and repeat.

Robot Jacobians, however, may be nonsquare or nearly singular. Direct Jacobian inversion is therefore generally unsuitable.

### Damped Least-Squares Formulation

Let

$$
J\in\mathbb{R}^{m\times n},
$$

where $$m$$ is the task dimension and $$n$$ is the number of joint-velocity variables. For full pose control, $$m=6$$.

Damped least squares calculates the joint correction by solving

$$
\min_{\Delta q}
\;
\frac{1}{2}
\left\|
J\Delta q-e
\right\|^2
+
\frac{\rho}{2}
\|\Delta q\|^2,
$$

where $$\rho>0$$ is the regularization coefficient.

The first term minimizes the locally predicted pose error:

$$
\frac{1}{2}
\left\|
J\Delta q-e
\right\|^2.
$$

The second term penalizes excessively large joint corrections:

$$
\frac{\rho}{2}
\|\Delta q\|^2.
$$

The code adds `config.damping` directly to $$JJ^T$$:

$$
JJ^T+\texttt{damping}\,I.
$$

Therefore, this section uses

$$
\rho
=
\texttt{config.damping}.
$$

Many textbooks instead write the regularizer as $$\lambda^2I$$. Under that convention,

$$
\rho=\lambda^2.
$$

Thus, the code variable called `damping` corresponds to the complete regularization coefficient, not necessarily to the unsquared textbook parameter $$\lambda$$.

### Step-by-step derivation of the DLS solution

Define the damped least-squares objective

$$
\mathcal{L}(\Delta q)
=
\frac{1}{2}
\left(
J\Delta q-e
\right)^T
\left(
J\Delta q-e
\right)
+
\frac{\rho}{2}
\Delta q^T\Delta q.
$$

First, expand the squared-error term:

$$
\begin{aligned}
\left(
J\Delta q-e
\right)^T
\left(
J\Delta q-e
\right)
&=
\Delta q^TJ^TJ\Delta q
-
\Delta q^TJ^Te\\
&\quad
-
e^TJ\Delta q
+
e^Te.
\end{aligned}
$$

Because both cross terms are scalars,

$$
e^TJ\Delta q
=
\Delta q^TJ^Te.
$$

Therefore,

$$
\mathcal{L}(\Delta q)
=
\frac{1}{2}
\Delta q^TJ^TJ\Delta q
-
\Delta q^TJ^Te
+
\frac{1}{2}e^Te
+
\frac{\rho}{2}
\Delta q^T\Delta q.
$$

Differentiate with respect to $$\Delta q$$:

$$
\nabla_{\Delta q}\mathcal{L}
=
J^TJ\Delta q
-
J^Te
+
\rho\Delta q.
$$

Equivalently,

$$
\nabla_{\Delta q}\mathcal{L}
=
J^T
\left(
J\Delta q-e
\right)
+
\rho\Delta q.
$$

At the minimum, the gradient is zero:

$$
J^TJ\Delta q
-
J^Te
+
\rho\Delta q
=
0.
$$

Collecting the terms that multiply $$\Delta q$$ gives

$$
\left(
J^TJ+\rho I_n
\right)
\Delta q
=
J^Te.
$$

These are called the regularized normal equations.

Because $$\rho>0$$, the matrix

$$
J^TJ+\rho I_n
$$

is invertible. Therefore,

$$
\Delta q
=
\left(
J^TJ+\rho I_n
\right)^{-1}
J^Te.
$$

This is the joint-space or primal form of the damped least-squares solution.

### Why two equivalent DLS formulas appear

The joint-space form is

$$
\Delta q
=
\left(
J^TJ+\rho I_n
\right)^{-1}
J^Te.
$$

In this expression, $$J^T$$ appears after the inverse term.

An equivalent task-space form is

$$
\Delta q
=
J^T
\left(
JJ^T+\rho I_m
\right)^{-1}
e.
$$

In this expression, $$J^T$$ appears before the inverse term.

The two expressions are equivalent because

$$
\left(
J^TJ+\rho I_n
\right)J^T
=
J^T
\left(
JJ^T+\rho I_m
\right).
$$

To verify this identity, expand the left-hand side:

$$
\begin{aligned}
\left(
J^TJ+\rho I_n
\right)J^T
&=
J^TJJ^T+\rho J^T\\
&=
J^T
\left(
JJ^T+\rho I_m
\right).
\end{aligned}
$$

Now multiply from the left by

$$
\left(
J^TJ+\rho I_n
\right)^{-1}
$$

and from the right by

$$
\left(
JJ^T+\rho I_m
\right)^{-1}.
$$

This gives the push-through identity

$$
\left(
J^TJ+\rho I_n
\right)^{-1}J^T
=
J^T
\left(
JJ^T+\rho I_m
\right)^{-1}.
$$

Therefore,

$$
\boxed{
\left(
J^TJ+\rho I_n
\right)^{-1}J^T
=
J^T
\left(
JJ^T+\rho I_m
\right)^{-1}
}
$$

and both formulas produce the same damped least-squares correction.

The placement of $$J^T$$ is not caused by commuting $$J^T$$ with an inverse. Matrix multiplication is generally not commutative. The equivalence follows from the specific push-through identity derived above.

The dimensions also explain why the two formulas look different:

$$
J\in\mathbb{R}^{m\times n},
$$

$$
J^T\in\mathbb{R}^{n\times m}.
$$

The joint-space form inverts an $$n\times n$$ matrix:

$$
J^TJ+\rho I_n
\in
\mathbb{R}^{n\times n}.
$$

The task-space form inverts an $$m\times m$$ matrix:

$$
JJ^T+\rho I_m
\in
\mathbb{R}^{m\times m}.
$$

For a redundant manipulator with $$n>m$$, the task-space form usually requires inverting the smaller matrix.

For a full-pose task,

$$
m=6.
$$

Therefore, the task-space form only requires solving a $$6\times6$$ linear system, even when the robot has seven or more joints.

### DLS form used by `diffik_dls.py`

The attached implementation uses

$$
\dot q
=
J^T
\left(
JJ^T+\rho I_6
\right)^{-1}
V_{\mathrm{cmd}}.
$$

This corresponds directly to:

```python
controlled_jacobian.T @ np.linalg.solve(
    controlled_jacobian @ controlled_jacobian.T + self._regularizer,
    task_velocity,
)
```

## ▪ Numerical IK versus Differential IK

Numerical IK and differential IK use the same local Jacobian model, but they use it for different purposes. Numerical IK repeats the Jacobian-based update as an internal solver step until it obtains a final joint configuration. Differential IK applies the mapping during physical control time to generate joint-velocity commands.

For damped least-squares numerical IK, the solver computes

$$
\Delta q^i
=
J(q^i)^T
\left(
J(q^i)J(q^i)^T+\lambda^2I
\right)^{-1}
e(q^i),
$$

followed by

$$
q^{i+1}
=
\operatorname{integrate}
\left(
q^i,\alpha\Delta q^i
\right).
$$

Here, $$i$$ is an internal solver-iteration index, and $$\Delta q^i$$ is a joint-configuration correction. It is not necessarily a physically executable joint velocity.

For damped least-squares differential IK, the controller computes

$$
\dot q_k
=
J(q_k)^T
\left(
J(q_k)J(q_k)^T+\lambda^2I
\right)^{-1}
V_{\mathrm{cmd},k},
$$

and integrates the velocity over the physical control timestep:

$$
q_{k+1}
=
\operatorname{integrate}
\left(
q_k,\Delta t\,\dot q_k
\right).
$$

Here, $$k$$ represents physical control time, $$\dot q_k$$ is measured in joint units per second, and $$\Delta t$$ is the controller timestep.

| Aspect | Numerical IK | Differential IK |
|---|---|---|
| Primary output | Final joint configuration $$q^\star$$ | Online joint velocity $$\dot q_k$$ |
| Independent variable | Solver iteration $$i$$ | Physical control step $$k$$ |
| Jacobian input | Pose error $$e(q^i)$$ | Commanded Cartesian velocity $$V_{\mathrm{cmd},k}$$ |
| Update process | Repeats internally until convergence | Usually performs one update per control cycle |
| Meaning of update | $$\Delta q^i$$ is a numerical configuration correction | $$\dot q_k$$ is a physical velocity command |
| Velocity limits | Not automatically meaningful or enforced | Must be enforced on the commanded velocity |
| Typical purpose | Solving for a static target configuration | Online Cartesian tracking and feedback control |
| Execution | Requires a trajectory generator or controller to reach $$q^\star$$ | Directly generates commands during robot operation |
| Constraints | Basic DLS does not enforce joint or collision constraints | Constraints can be incorporated through a velocity-level QP |

Damped least squares alone does not enforce joint-position or joint-velocity limits in either formulation. Numerical IK requires a constrained solver when the final configuration must remain within joint bounds. Differential IK commonly uses a quadratic program to enforce velocity limits and predictive joint-position constraints during online control.

The remainder of this section focuses on numerical IK as a configuration solver. Differential IK and its constrained QP formulation are developed later as online control methods.

----
Reference:

- <i class="fa-solid fa-book" aria-hidden="true"></i> [MuJoCo XML Reference (actuator)](https://mujoco.readthedocs.io/en/3.3.7/XMLreference.html#actuator-position).
- <i class="fa-brands fa-chrome"></i> [Levenberg–Marquardt algorithm](https://en.wikipedia.org/wiki/Levenberg%E2%80%93Marquardt_algorithm)
