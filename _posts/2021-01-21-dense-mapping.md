---
layout: post
title: "构建 2.5D/3D 导航地图"
categories: study
author: "Jixiang Zhang"
---

人类与足式机器人的活动区域位于地球表面，属于**曲面**（2.5D $\in$ 3D）。目前大部分移动机器人自主导航时仅能感知 2D 平面空间信息，如常用的 [costmap_2d](https://github.com/ros-planning/navigation/tree/noetic-devel/costmap_2d)，缺点在于难以适应复杂地形，如斜坡、楼梯等。为此，足式机器人需要实时构建稠密的 2.5D 或 3D 地图。

常用地图

* [~~costmap_2d~~](https://github.com/ros-planning/navigation/tree/noetic-devel/costmap_2d)

![](http://wiki.ros.org/costmap_2d?action=AttachFile&do=get&target=costmap_rviz.png)

* [**OpenVDB** - Sparse volume data structure and tools](https://github.com/AcademySoftwareFoundation/openvdb)
  * [Spatio-Temporal Voxel Layer](https://github.com/SteveMacenski/spatio_temporal_voxel_layer)

![](https://user-images.githubusercontent.com/14944147/37010885-b18fe1f8-20bb-11e8-8c28-5b31e65f2844.gif)

* [**Grid Map** - Universal grid map library for mobile robotic mapping](https://github.com/ANYbotics/grid_map)
  * [Robot-Centric Elevation Mapping](https://github.com/anybotics/elevation_mapping)

![](https://github.com/ANYbotics/elevation_mapping/raw/master/elevation_mapping_demos/doc/elevation_map.jpg)

![](https://github.com/ANYbotics/grid_map/raw/master/grid_map_rviz_plugin/doc/grid_map_rviz_plugin_example.png)

* [**OctoMap**](https://github.com/OctoMap/octomap)

![](http://octomap.github.io/freiburg_079_big.png)

* [**Voxblox**](https://github.com/ethz-asl/voxblox)
  * [OpenChisel](https://github.com/personalrobotics/OpenChisel)
  * [Fast-Planner/plan_env](https://github.com/HKUST-Aerial-Robotics/Fast-Planner)
  * [FIESTA](https://github.com/HKUST-Aerial-Robotics/FIESTA)

![](https://i.imgur.com/pvHhVsL.png/)

* [**Point Cloud** Library](https://github.com/PointCloudLibrary/pcl)

地图解析

* 路面 Terrain
* 障碍 Obstacle

![](/images/densemap.jpg)
