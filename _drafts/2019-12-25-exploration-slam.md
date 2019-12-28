---
layout: post
title: "自主探索SLAM综述"
categories: study
author: "Jixiang Zhang"
---

**综述**

自主移动机器人的核心技术

- 定位：Localization
- 建图：Mapping
- 导航：
  - Motion Planning
  - Exploration
  - Navigation

![截屏2019-12-28下午2.50.27](https://tvax3.sinaimg.cn/mw690/d494c514ly1gacfuqp64aj20ls0f5760.jpg)

“自主探索SLAM技术”的定义：一种融合SLAM与运动规划的技术，具有较高的学术价值和工程难度。

**方案**

(Localization) --> (Mapping) --> (Planning + DFS)

**程序**

- [Turtlebot Autonomous Exploration (3D)](https://github.com/RobustFieldAutonomyLab/turtlebot_exploration_3d)
- [Receding Horizon Next Best View Planning](https://github.com/ethz-asl/nbvplanner)


**文献**

- [Information-Theoretic Exploration with Bayesian Optimization](http://personal.stevens.edu/~benglot/Bai_Wang_Chen_Englot_IROS2016_AcceptedVersion.pdf)