---
layout: post
title: "Centroidal Dynamics Notes"
categories: study
author: "Jixiang Zhang"
---

![](/images/RMP.gif)

Cite: [Reaction Mass Pendulum (RMP) : An explicit model for centroidal angular momentum of humanoid robots](https://www.youtube.com/watch?v=Qf3iyco-3rc)

# Formulas

## Task

* $\mathbf{h}_g = A_g(q) \dot{q}$
* $\dot{\mathbf{h}}_g = A_g(q) \ddot{q}\ + \dot{A_g}(q,\dot{q})\dot{q}$
* $\mathbf{h}_g = (m\dot{c}, L_g)$
* $\dot{\mathbf{h}}_g = (m\ddot{c}, \dot{L}_g)$
* $\mathbf{h}_g = I_g v_{mean}$

$$
d\dot{\mathbf{h}}_g = \frac{\partial \dot{\mathbf{h}}_g}{\partial q} dq
                    + \frac{\partial \dot{\mathbf{h}}_g}{\partial \dot{q}} d\dot{q}
                    + \frac{\partial \dot{\mathbf{h}}_g}{\partial \ddot{q}} d\ddot{q}
$$

## Control (contact_forces/contact_wrenches)

* $\mathbf{f} = \dot{\mathbf{h}}_g$

Cite:

* [Pinocchio](https://github.com/stack-of-tasks/pinocchio.git)
* [Mapping between OCS2 and pinocchio state and input](https://github.com/leggedrobotics/ocs2/issues/53#issue-1393265676)
