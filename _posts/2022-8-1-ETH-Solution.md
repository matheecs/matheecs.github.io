---
layout: post
title: "解析ETH局部规划"
categories: study
author: "Jixiang Zhang"
---

[ANYmal Rough Terrain Planner](https://github.com/leggedrobotics/art_planner)

## 局部规划器

### Dependencies

| Names         | Function |
| ------------- | -------- |
| OMPL          | 规划框架 |
| ODE           | 碰撞检测 |
| OpenCV        | 图像处理 |
| grid_map_core | 地图数据 |
| Boost         | Graph/A* |

**Terminology**

- **Clearance** : 机器人与障碍物的距离
- **Milestone** : 图的节点

### OMPL

> LazyPRM is a planner that constructs a roadmap of **milestones** that approximate the connectivity of the state space, just like PRM does. The difference is that the planner uses lazy collision checking.

<!-- ![api](images/ompl_api.png) -->
<p align="center">
  <img src="images/ompl_api.png" width="500"/>
</p>

[An OMPL Tutorial with Examples](https://www.youtube.com/watch?v=yggi7QjfOUM)

**Tech Areas**

- Mobility (S3.1)
  - Walking & Flying Robots
- Perception (S3.2)
  - Calibration
  - SLAM
  - Elevation Mapping
- Autonomy
  - Exploration Planner
  - Navigation Planner
  - BT
- Networking
- Artifacts Scoring
- Single Human Supervisor

## Ref

- [Video:DARPA SubT Challenge - Technology Overview by Our Winning Team CERBERUS](https://www.youtube.com/watch?v=lrLPMoSLvVo)
- [Team CERBERUS Homepage](https://www.subt-cerberus.org)
- [Report Document](https://arxiv.org/pdf/2207.04914.pdf)
- [nimbro_network](https://github.com/AIS-Bonn/nimbro_network)
- [BehaviorTree.CPP](https://github.com/BehaviorTree/BehaviorTree.CPP)
- [tmux](https://github.com/tmux/tmux)
  - [Tmux 使用教程](http://www.ruanyifeng.com/blog/2019/10/tmux.html)
- [PlotJuggler](https://github.com/facontidavide/PlotJuggler)
