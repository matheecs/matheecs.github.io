---
layout: post
title: "State Estimation for Robotics 笔记"
categories: study
author: "Jixiang Zhang"
---

某某说：*State Estimation for Robotics* 是学习 SLAM 的必读书籍。看完高翔的 *视觉 SLAM 十四讲*，再读 *State Estimation for Robotics*，把十四讲里的一些概念彻底弄明白。

### 内容简介

第 1、2 和 6 章是介绍和理论铺垫，包括概率论和三维空间几何。第 3 章系统介绍了 Linear-Gaussian 估计，包括离散和连续、Batch 和 Recursive 的估计。第 4 和 7 章是全书**核心**，前者介绍实际面对的 **Nonlinear Non-Gaussian** 估计问题，因为旋转不属于*线性空间*，所以后者引入了 **Matrix Lie Group** ，包含*流形空间 SO(3) SE(3)*。第 5、8、9 和 10 章讨论如何用状态估方法解决真实世界遇到的问题，第 9 章详细介绍了 SLAM 问题。

### 两种范式

- Bayesian inference
- Maximum A Posteriori

### 非线性处理方法

- Monte Carlo Method
- Linearization
- Sigmapoint Transformation

### BA 与 SLAM 的区别

$$
\mathbf{Prior} + \mathbf{BA ( ML )} \to \mathbf{SLAM ( MAP )}
$$

$$
\mathbf{Motion\,Model(Input)} + \mathbf{Mesurement\,Model} \to \mathbf{SLAM}
$$

$$
\mathbf{iRobot Create 2} + \mathbf{Stereo Camera} \to \mathbf{vSLAM}
$$

