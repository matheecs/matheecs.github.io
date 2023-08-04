---
layout: post
title: "MuJoCo Solver notes"
categories: study
author: "Jixiang Zhang"
---

CITE <https://mujoco.readthedocs.io/en/latest/modeling.html#solver-parameters>

# Impedance

The impedance $d\in (0,1)$ determines a constraint’s ability to generate force. Small values of $d$ correspond to weak constraints while large values of $d$ correspond to strong constraints. Impedance is set using the `solimp` attribute.

<p align="center">
  <img src="https://mujoco.readthedocs.io/en/latest/_images/impedance.png" width="500"/>
</p>

Recall that $d$ must lie between 0 and 1; internally MuJoCo clamps it to the range [`mjMINIMP`, `mjMAXIMP`] which is currently set to [`0.0001`, `0.9999`]. It causes the solver to interpolate between the **unforced acceleration** $a_0$ and **reference acceleration** $a_{ref}$. The user can set $d$ to a constant, or take advantage of its interpolating property and make it position-dependent, i.e., a function of the constraint violation $r$. Position-dependent impedance can be used to model soft contact layers around objects, or define equality constraints that become stronger with larger violation (so as to approximate backlash, for example). The shape of the function $d(r)$ is determined by the element-specific parameter vector `solimp`.

# Reference

$$
a_{ref} = -bv-kr
$$

The reference acceleration $a_{ref}$ determines what the constraint is trying to achieve (as opposed to how well it can achieve it). This acceleration is defined by two numbers, a stiffness $k$ and damping $b$ which can be set directly or re-parameterized as the time-constant and damping ratio of a mass-spring-damper system (a harmonic oscillator). The reference acceleration is controlled by the `solref` attribute.

<p align="center">
  <img src="https://tajimarobotics.com/wp-content/uploads/2017/12/MSD-System-04-1024x476.png" width="500"/>
</p>
