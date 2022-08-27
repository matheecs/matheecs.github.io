---
geometry: margin=2cm
output: pdf_document
mainfont: MiSans
colorlinks: true
linkcolor: blue
urlcolor: red
toccolor: gray
title: 机器人感知规划相关资料
author: matheecs
---

### Framework

- SLAM
- Perception
- Planning
- 中间件 ROS, 以上算法基于 ROS 开发

### SLAM = 定位 + 建图算法

机器人导航依赖于定位信息和平面栅格地图，而定位依赖于已知点云地图，故首先需要用SLAM算法构建高精度的点云地图。

常用地图构建算法：
- [Faster-LIO 激光-惯导融合的点云地图构建方案](https://github.com/gaoxiang12/faster-lio)
- [LVI-SAM 激光-视觉-惯导融合的点云地图构建方案](https://github.com/TixiaoShan/LVI-SAM)
- [slam_gmapping 基于激光的平面栅格地图构建方案](https://github.com/ros-perception/slam_gmapping)

常用实时定位算法：
- [hdl_localization 基于点云地图的定位方案](https://github.com/koide3/hdl_localization)
- [amcl 基于平面栅格地图和蒙特卡罗定位方案](https://wiki.ros.org/amcl?distro=noetic)

### Perception 感知算法

地图用来表示机器人周围的环境信息，比如是否有障碍物、是否可以通行、地面坡度、地面高度信息等，作为规划的输入信息。

- [costmap_2d 用于2D导航的平面栅格地图](https://wiki.ros.org/costmap_2d)
- [GridMap 用于导航的2.5维栅格地图](https://github.com/ANYbotics/grid_map)
- [Elevation Mapping cupy 在线构建 GridMap](https://github.com/leggedrobotics/elevation_mapping_cupy)

### Planning 规划算法

规划算法分为前端的路径搜索与后端的轨迹规划算法。路径搜索用来计算出一条无碰撞的从起点到终点的路径，而轨迹规划则基于路径和机器人运动学/动力学模型、其他约束条件计算出一条平滑的运动轨迹，作为底层运动控制的输入。

**前端：全局路径规划**

- [global_planner 采用 A*/Dijkstra 的全局路径规划方案](https://wiki.ros.org/global_planner?distro=noetic)
- [sampling-based-path-finding 基于采样的路径搜索算法](https://github.com/ZJU-FAST-Lab/sampling-based-path-finding)

**后端：局部轨迹规划**

- [teb_local_planner 基于图优化的局部路径规划方案](https://wiki.ros.org/teb_local_planner)
- [mpc_local_planner 基于MPC的局部路径规划方案](https://github.com/rst-tu-dortmund/mpc_local_planner)

### References

- [ROS Tutorials](https://wiki.ros.org/ROS/Tutorials)
- [PythonRobotics 机器人算法集Demo](https://github.com/AtsushiSakai/PythonRobotics)
- [Apollo 自动驾驶开源方案](https://github.com/ApolloAuto/apollo)
- [Apollo 智能驾驶入门课程](https://developer.apollo.auto/devcenter/coursetable_cn.html?target=1)
- [Blog: 构建 2.5D/3D 导航地图](https://matheecs.tech/study/2021/01/21/dense-mapping.html)
- [Blog: 解析Spot机器人](https://matheecs.tech/study/2021/11/29/spot.html)
- [Atlas 跑酷背后的原理](https://matheecs.tech/study/2021/11/26/atlas.html)