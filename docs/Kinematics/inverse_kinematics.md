---
title: Inverse Kinematics
parent: Kinematics
nav_order: 4
---

# Inverse Kinematics
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

Inverse kinematics(IK) determines the joint configuration required to place a robot’s end effector at a desired position and orientation. In contrast to forward kinematics, which maps joint variables to a unique pose, inverse kinematics may admit one solution, multiple solutions, infinitely many solutions, or no feasible solution. 

IK can be formulated as either a root-finding problem, where the pose error is driven to zero, or an optimization problem that minimizes the remaining error. Here, we focus on the <span style="color: red"> damped least-squares (DLS) </span> approach (also known as [Levenberg–Marquardt algorithm](https://en.wikipedia.org/wiki/Levenberg%E2%80%93Marquardt_algorithm)) and corresponding implementation can be found at <i class="fa-brands fa-github" aria-hidden="true"></i> [this repository](https://github.com/lihanlian/robot-manipulator-control/blob/main/controller/diffik_dls.py).


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

### Gauss–Newton Method

Numerical inverse kinematics can be formulated as the nonlinear least-squares problem

$$
\min_q \Phi(q),
\qquad
\Phi(q)
=
\frac{1}{2}\|e(q)\|^2,
$$

where $$e(q)$$ measures the difference between the current and desired end-effector poses.

This has the same mathematical structure as nonlinear curve fitting. In curve fitting, model parameters are adjusted until the model output matches the observed data. In IK, the joint configuration $$q$$ is adjusted until the pose predicted by forward kinematics matches the desired pose.

Let $$x(q)$$ denote the end-effector task coordinates obtained from forward kinematics and let $$x_d$$ denote the desired task coordinates. Using the desired-minus-current convention, the error is

$$
e(q)=x_d-x(q).
$$

Around the current estimate $$q^i$$, the forward-kinematics mapping is approximated by its first-order linearization:

$$
x(q^i+\Delta q^i)
\approx
x(q^i)+J(q^i)\Delta q^i.
$$

Here, $$\Delta q^i$$ is a small joint correction and $$J(q^i)$$ is the manipulator Jacobian evaluated at the current configuration.

The error after applying this correction is therefore approximately

$$
\begin{aligned}
e(q^i+\Delta q^i)
&=
x_d-x(q^i+\Delta q^i)\\
&\approx
x_d-\left[x(q^i)+J(q^i)\Delta q^i\right]\\
&=
e(q^i)-J(q^i)\Delta q^i.
\end{aligned}
$$

Gauss–Newton selects the joint correction that minimizes the squared norm of this locally predicted error:

$$
\min_{\Delta q^i}
\frac{1}{2}
\left\|
e(q^i)-J(q^i)\Delta q^i
\right\|^2.
$$

For compactness, let

$$
e=e(q^i),
\qquad
J=J(q^i),
\qquad
\Delta q=\Delta q^i.
$$

Because changing the sign does not change a squared norm,

$$
\|e-J\Delta q\|^2
=
\|J\Delta q-e\|^2.
$$

The local least-squares objective can therefore be written as

$$
\mathcal{L}(\Delta q)
=
\frac{1}{2}
(J\Delta q-e)^T(J\Delta q-e).
$$

First, expand the squared-error term:

$$
\begin{aligned}
(J\Delta q-e)^T(J\Delta q-e)
&=
\Delta q^TJ^TJ\Delta q
-\Delta q^TJ^Te\\
&\quad
-e^TJ\Delta q
+e^Te.
\end{aligned}
$$

The two cross terms are scalars. A scalar is equal to its transpose, so

$$
e^TJ\Delta q
=
\left(e^TJ\Delta q\right)^T
=
\Delta q^TJ^Te.
$$

The expanded objective is therefore

$$
\mathcal{L}(\Delta q)
=
\frac{1}{2}\Delta q^TJ^TJ\Delta q
-
\Delta q^TJ^Te
+
\frac{1}{2}e^Te.
$$

Differentiate the objective with respect to $$\Delta q$$. Because $$J^TJ$$ is symmetric,

$$
\nabla_{\Delta q}
\left(
\frac{1}{2}\Delta q^TJ^TJ\Delta q
\right)
=
J^TJ\Delta q.
$$

The derivative of the cross term is

$$
\nabla_{\Delta q}
\left(
-\Delta q^TJ^Te
\right)
=
-J^Te.
$$

The final term does not depend on $$\Delta q$$, so

$$
\nabla_{\Delta q}
\left(
\frac{1}{2}e^Te
\right)
=
0.
$$

Combining these terms gives

$$
\nabla_{\Delta q}\mathcal{L}
=
J^TJ\Delta q-J^Te.
$$

At the minimum, the gradient is zero:

$$
J^TJ\Delta q-J^Te=0.
$$

Rearranging gives the normal equations

$$
J^TJ\Delta q=J^Te.
$$

If $$J^TJ$$ is invertible, multiply both sides by $$(J^TJ)^{-1}$$:

$$
\Delta q
=
(J^TJ)^{-1}J^Te.
$$

The matrix

$$
J^\dagger
=
(J^TJ)^{-1}J^T
$$

is the left pseudoinverse of $$J$$ when $$J$$ has full column rank. The Gauss–Newton correction can therefore be written as

$$
\Delta q=J^\dagger e,
$$

and the joint configuration is updated according to

$$
q^{i+1}=q^i+\Delta q^i.
$$

Gauss–Newton uses the first-order linearization of the pose residual and approximates the curvature of the nonlinear objective using $$J^TJ$$. Unlike full Newton optimization, it does not require second derivatives of the forward-kinematics or pose-error function.

The undamped solution requires $$J^TJ$$ to be invertible and well conditioned. Near a singular configuration, or for a redundant manipulator with more joints than task dimensions, this condition may not hold. Damped least squares resolves this issue by adding a positive regularization term to the Gauss–Newton normal equations.

### Damped Least Squares

Damped least squares adds a penalty on the size of the joint correction:

$$
\min_{\Delta q}
\frac{1}{2}\|e-J\Delta q\|^2
+
\frac{\rho}{2}\|\Delta q\|^2,
$$

where $$\rho>0$$ is the damping coefficient.

Differentiating this objective and setting the gradient to zero gives

$$
-J^T(e-J\Delta q)+\rho\Delta q=0.
$$

Therefore,

$$
(J^TJ+\rho I)\Delta q=J^Te,
$$

and the damped correction is

$$
\boxed{
\Delta q
=
(J^TJ+\rho I)^{-1}J^Te
}.
$$

Adding $$\rho I$$ makes the matrix invertible even when the Jacobian is singular or rank deficient. It also penalizes excessively large joint corrections, improving numerical stability near singular configurations.

DLS is therefore best understood as a regularized Gauss–Newton method. With a fixed $$\rho$$, it is commonly called fixed-damping DLS. A full Levenberg–Marquardt algorithm adjusts $$\rho$$ according to how successfully each step reduces the objective.

### Interpretation of the Damping Coefficient

The damping coefficient controls the balance between the fast Gauss–Newton direction and the more conservative gradient-descent direction.

When $$\rho$$ is small,

$$
J^TJ+\rho I
\approx
J^TJ,
$$

so

$$
\Delta q
\approx
(J^TJ)^{-1}J^Te.
$$

The update therefore behaves like Gauss–Newton. This usually gives fast convergence when the current configuration is close to a valid solution and the Jacobian is well conditioned.

When $$\rho$$ is large,

$$
J^TJ+\rho I
\approx
\rho I.
$$

Consequently,

$$
\Delta q
\approx
\frac{1}{\rho}J^Te.
$$

For

$$
\Phi(q)=\frac{1}{2}\|e(q)\|^2
$$

with $$e(q)=x_d-x(q)$$, the gradient is

$$
\nabla_q\Phi(q)=-J^Te.
$$

Thus, for large $$\rho$$,

$$
\Delta q
\approx
-\frac{1}{\rho}\nabla_q\Phi(q),
$$

which is a small step in the negative-gradient direction.

The damping coefficient therefore has the following interpretation:

* small $$\rho$$: faster, Gauss–Newton-like motion;
* large $$\rho$$: smaller, more stable gradient-descent-like motion;
* intermediate $$\rho$$: a compromise between convergence speed and robustness.

In an adaptive Levenberg–Marquardt method, $$\rho$$ is reduced when a step successfully decreases the pose error and increased when the local approximation produces a poor step.

The implementation in `diffik_dls.py` uses the equivalent task-space form

$$
\Delta q
=
J^T(JJ^T+\rho I)^{-1}e.
$$

For scalar damping, this is algebraically equivalent to

$$
(J^TJ+\rho I)^{-1}J^Te.
$$

The code uses the task-space form because it solves a system whose size equals the six-dimensional Cartesian task, while the derivation above uses the standard Gauss–Newton form to make the optimization interpretation clearer.


## ▪ Numerical IK versus Differential IK

Numerical IK and differential IK use the same local Jacobian model, but they solve different problems. Numerical IK repeatedly applies a Jacobian-based correction inside an iterative solver to obtain a final joint configuration. Differential IK operates over physical control time and continuously generates joint-velocity commands.

For damped least-squares numerical IK, the solver calculates

$$
\Delta q^i
=
\left(
J(q^i)^TJ(q^i)+\rho I
\right)^{-1}
J(q^i)^Te(q^i),
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

Here, $$i$$ is an internal solver-iteration index, $$\Delta q^i$$ is a joint-configuration correction, and $$\alpha$$ is a dimensionless numerical step size. The correction $$\Delta q^i$$ is not necessarily a physically executable joint velocity because the solver iterations do not represent elapsed physical time.

Differential IK instead begins with the velocity-level kinematic relationship

$$
V_k=J(q_k)\dot q_k.
$$

Given a commanded Cartesian end-effector velocity $$V_{\mathrm{cmd},k}$$, damped least squares calculates

$$
\dot q_k
=
\left(
J(q_k)^TJ(q_k)+\rho I
\right)^{-1}
J(q_k)^T V_{\mathrm{cmd},k}.
$$

Here, $$V_{\mathrm{cmd},k}$$ is a commanded Cartesian twist containing both linear and angular velocity:

$$
V_{\mathrm{cmd},k}
=
\begin{bmatrix}
v_{\mathrm{cmd},k}\\
\omega_{\mathrm{cmd},k}
\end{bmatrix}.
$$

The term “commanded velocity” is more precise than “commanded speed” because $$V_{\mathrm{cmd},k}$$ contains both magnitude and direction. It specifies how the controller would like the end effector to move at control step $$k$$. Because DLS only approximates this command, especially near singularities, the actual end-effector velocity $$J(q_k)\dot q_k$$ may not exactly equal $$V_{\mathrm{cmd},k}$$.

For pose regulation toward a fixed target, $$V_{\mathrm{cmd},k}$$ is commonly chosen as an error-reducing Cartesian velocity:

$$
V_{\mathrm{cmd},k}
=
K_p e(q_k),
$$

where $$K_p$$ is a positive task-space feedback gain. This command converts a pose displacement into a velocity that points toward the desired pose.

In the referenced `diffik_dls.py`, the gain is specified through the response time $$\tau$$:

$$
K_p=\frac{1}{\tau}I.
$$

The commanded task velocity is therefore

$$
V_{\mathrm{cmd},k}
=
\begin{bmatrix}
\dfrac{p_d-p(q_k)}{\tau}\\[8pt]
\dfrac{
\operatorname{Log}
\left(
R_dR(q_k)^T
\right)^\vee
}{\tau}
\end{bmatrix}.
$$

The upper component commands a linear velocity that reduces the position error, while the lower component commands an angular velocity that reduces the orientation error.

The response time $$\tau$$ is not the control timestep. It determines how aggressively the controller attempts to reduce the pose error:

* a smaller $$\tau$$ produces a larger and more aggressive Cartesian velocity;
* a larger $$\tau$$ produces a smaller and smoother Cartesian velocity.

Under ideal tracking, $$\tau$$ acts as a time constant; it does not mean that the error becomes exactly zero after $$\tau$$ seconds.

For tracking a moving Cartesian trajectory, the commanded velocity can also include a feedforward term:

$$
V_{\mathrm{cmd},k}
=
V_{d,k}+K_pe(q_k),
$$

where $$V_{d,k}$$ is the desired trajectory velocity and $$K_pe(q_k)$$ corrects the remaining tracking error. The referenced code uses only the feedback term because it regulates the end effector toward the instantaneous target pose.

After calculating $$\dot q_k$$, differential IK integrates the velocity over the physical control timestep $$\Delta t$$:

$$
q_{k+1}
=
\operatorname{integrate}
\left(
q_k,\Delta t\,\dot q_k
\right).
$$

When position actuators are used, a persistent joint-position reference may instead be integrated:

$$
q_{\mathrm{ref},k+1}
=
\operatorname{integrate}
\left(
q_{\mathrm{ref},k},\Delta t\,\dot q_k
\right).
$$

This is the approach used in `diffik_dls.py`.

| Aspect               | Numerical IK                                                            | Differential IK                                             |
| -------------------- | ----------------------------------------------------------------------- | ----------------------------------------------------------- |
| Primary output       | Final joint configuration $$q^\star$$                                   | Online joint velocity $$\dot q_k$$                          |
| Independent variable | Solver iteration $$i$$                                                  | Physical control step $$k$$                                 |
| Jacobian input       | Pose displacement $$e(q^i)$$                                            | Cartesian velocity $$V_{\mathrm{cmd},k}$$                   |
| Meaning of output    | $$\Delta q^i$$ is a numerical configuration correction                  | $$\dot q_k$$ is a physical velocity command                 |
| Update scale         | Numerical step size $$\alpha$$                                          | Physical timestep $$\Delta t$$                              |
| Update process       | Repeats internally until convergence                                    | Usually performs one update per control cycle               |
| Velocity limits      | Do not directly apply to an internal solver step                        | Must be enforced on the physical velocity command           |
| Typical purpose      | Finding a configuration for a target pose                               | Online Cartesian regulation or trajectory tracking          |
| Execution            | A trajectory generator or controller must move the robot to $$q^\star$$ | Continuously generates commands during operation            |
| Constraints          | Basic DLS does not enforce constraints                                  | Constraints can be incorporated through a velocity-level QP |

Both numerical and differential DLS are unconstrained in their basic forms. Damping reduces the amplification of joint motion near singularities, but it does not impose hard joint-position, velocity, or collision limits. Numerical IK therefore requires explicit joint-limit handling when computing $$q^\star$$, while constrained differential IK and its QP formulation will be discussed later in the Control chapter.

----
Reference:

- <i class="fa-solid fa-book" aria-hidden="true"></i> [MuJoCo XML Reference (actuator)](https://mujoco.readthedocs.io/en/3.3.7/XMLreference.html#actuator-position).
- <i class="fa-brands fa-chrome"></i> [Levenberg–Marquardt algorithm](https://en.wikipedia.org/wiki/Levenberg%E2%80%93Marquardt_algorithm)
