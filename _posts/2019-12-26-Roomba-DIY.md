---
layout: post
title: "低成本改造扫地机"
categories: study
author: "Jixiang Zhang"
---

**如何把两轮差速底盘、激光传感器和PC组装成一台自主导航机器人？**

### 所需硬件（Cost < ¥1000）

- Roomba 600
- RPLIDAR S1/A1/A2/A3
- 自己的笔记本
- Xbox 360 手柄（阿修罗2无线版）
  ![xbox-default-linux](https://i0.wp.com/tvax3.sinaimg.cn/large/d494c514ly1gaevjuaepsj20p00fan2r.jpg)

  所有通道（8个模拟[-1.0,+1.0] + 11个数字{0,1}）默认初始值：

  ```
  axes: [-0.0, -0.0, -0.0, -0.0, 1.0, 1.0, -0.0, -0.0]
  buttons: [0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0]
  ```

### 所需软件

- [create_autonomy](https://github.com/AutonomyLab/create_autonomy#create_autonomy)
- [RPLIDAR ROS package](https://github.com/slamtec/rplidar_ros)
- [Roomblock](https://github.com/tork-a/roomblock)
- [xboxdrv](https://gitlab.com/xboxdrv/xboxdrv/)
- [Joystick Drivers](https://github.com/ros-drivers/joystick_drivers)
  <!-- ![IMG_7102](https://i0.wp.com/tva4.sinaimg.cn/large/d494c514ly1gafm8tf2gkj21kq19dtpj.jpg) -->

### 组装方案

![IMG_7089](https://i0.wp.com/tvax3.sinaimg.cn/mw690/d494c514ly1gaca6r5h6vj21if1ji1kx.jpg)

### AhaBot

![ahabot](https://i0.wp.com/tvax1.sinaimg.cn/large/d494c514ly1gaiawqrx4lj20sl0dc76e.jpg)

DIY路线

1. Setup
2. Bringup
3. Basic Operation
4. SLAM
5. Navigation

<!-- ![坐标关系](https://i0.wp.com/tva2.sinaimg.cn/large/d494c514ly1gaf3s8f17uj20d608qt8r.jpg) -->

![nav configuration](https://i0.wp.com/tvax2.sinaimg.cn/large/d494c514ly1gag51ui1yjj20lo08vwfx.jpg)

### 利用laser_filters裁剪雷达数据

<!-- ![laser scanner view](https://i0.wp.com/tva4.sinaimg.cn/large/d494c514ly1gaikgo5td2j20dy05ht9r.jpg) -->

<!-- ![S1](https://i0.wp.com/tvax1.sinaimg.cn/large/d494c514ly1gainldvkgrj20s80kun0j.jpg) -->

新建文件`angular_bounds_filter_example.launch`：

```xml
<launch>
<node pkg="laser_filters" type="scan_to_scan_filter_chain" output="screen" name="laser_filter">
    <rosparam command="load" file="path to angular_bounds_filter.yaml" />
</node>
</launch>
```

和`angular_bounds_filter.yaml`：

```yaml
scan_filter_chain:
- name: angle
  type: laser_filters/LaserScanAngularBoundsFilterInPlace
  params:
    lower_angle: -1.57
    upper_angle: 1.57
```

最后加载`angular_bounds_filter_example.launch`文件。

![scan_to_scan_filter_chain](https://i0.wp.com/tvax2.sinaimg.cn/large/d494c514ly1galeiijyedj20j40a4gmf.jpg)

### RViz导入模型与底盘驱动测试

![aha](https://i0.wp.com/tva2.sinaimg.cn/large/d494c514ly1gakn9ag6zej21fj0tctce.jpg)
![aha-bot](https://i0.wp.com/tvax2.sinaimg.cn/large/d494c514ly1gakxug6l2kj20zk0zf154.jpg)

### TODO

1. 激光数据去畸变
2. 压缩mesh模型，提高RViz加载速度

### udev rule

[lino_udev](https://github.com/linorobot/lino_udev)

### EKF

![截屏2020-01-10下午10.45.30](https://i0.wp.com/tva3.sinaimg.cn/large/d494c514gy1garummvn9zj20n704u3z3.jpg)

### 王晋泰《自主导航机器人制作十日谈》课程内容

1. 项目准备
   1. 课程内容解读
   2. 软件环境配置
2. 项目分析与采购技巧
   1. 自主导航机器人能力的分析
   2. 硬件选型及其注意事项
   3. 物料采购技巧
3. 开源硬件与开源项目
   1. Arduino开源硬件及编程
   2. 底盘控制器开源项目分析
   3. 原理草图绘制
4. 底盘控制器电路设计与制作
   1. 控制板制作方式分析
   2. 布局草图绘制
   3. 焊接并连接线路
   4. 控制板软硬件测试
5. 机器人本体组装
   1. 差速模型解读
   2. 机器人本体设计与组装
   3. 记录机器人差速模型参数
6. 机器人操作系统的使用与配置
   1. Linux与ROS基础命令
   2. 底盘控制器驱动功能包配置
   3. 底盘控制器驱动功能包调试
7. 功能包与移动控制
   1. ROS常见功能包演示
   2. 移动控制功能整合
   3. 使用手机控制机器人
8. 激光雷达与可视化工具
   1. 激光雷达传感器使用
   2. 深度相机传感器使用
   3. 机器人操作系统可视化工具
9. 自主导航
   1. 仿真环境中的自主导航
   2. 自主导航功能包
   3. 自主导航功能的整合与调试
10. 回顾与总结
    1. 知识的类型
    2. 抽象层次
    3. 学习方法分享

### 我最想设计的一门移动机器人课程

1. Concepts of Robotics
2. Perception
3. Planning & Control
4. ROS, C++, Ubuntu, Arduino
5. Encoder, Motor, Power
6. URDF Modeling and Simulation
7. SLAM and Navigation

### 参考文献

- [Roomblock](https://www.instructables.com/id/Roomblock-a-Platform-for-Learning-ROS-Navigation-W/)
- [Roomba](http://wiki.ros.org/Robots/Roomba)
- [Running the navigation stack on the Roomba](http://wiki.ros.org/lse_roomba_toolbox/Tutorials/navigation%20on%20the%20Roomba)
- [TurtleBot3手册](http://emanual.robotis.com/docs/en/platform/turtlebot3/setup/#setup)
- [Coordinate Frames for Mobile Platforms](https://www.ros.org/reps/rep-0105.html)
- [Setup and Configuration of the Navigation Stack on a Robot](http://wiki.ros.org/navigation/Tutorials/RobotSetup#Navigation_Stack_Setup)
- ~~[ROS中发布激光扫描消息](https://www.cnblogs.com/21207-iHome/p/7840129.html)~~
- ~~[rplidar 扫描角度设置](https://www.cnblogs.com/lovebay/p/10400762.html)~~
- ~~[裁剪rplidar的扫描数据](https://blog.csdn.net/sunyoop/article/details/78302090)~~
- ~~[通过修改rplidar的ros包源码达到屏蔽雷达角度的目的](https://blog.csdn.net/dzhongjie/article/details/84575482)~~
- [ROS机器人底盘(38)-laser_filters的使用（3）-屏蔽指定角度](https://www.jianshu.com/p/fc4b57fd6006)
- [小强ROS机器人教程](https://xq-manual.bwbot.org/)
- [ROS By Example Volume 1](https://github.com/pirobot/rbx1)
- [urdf-XML-link](http://wiki.ros.org/urdf/XML/link)
- [urdf-XML-joint](http://wiki.ros.org/urdf/XML/joint)
- [udev — Linux dynamic device management](https://mirrors.edge.kernel.org/pub/linux/utils/kernel/hotplug/udev/udev.html)
- [Writing udev rules](http://www.reactivated.net/writing_udev_rules.html)
- [激光slam理论与实践](https://blog.csdn.net/qq_29230261/article/details/83146147)
- [ROS 学习系列 -- 执行turtlebot navigation的方法](https://blog.csdn.net/crazyquhezheng/article/details/49909817)
- [**linorobot**](https://github.com/linorobot/linorobot)
  > Linorobot is a suite of Open Source ROS compatible robots that aims to provide students, developers, and researchers a low-cost platform in creating new exciting applications on top of ROS (Robot Operating System). Linorobot supports different robot bases you can build from the ground up.
- [onine](https://github.com/grassjelly/onine)
  > ROS based service robot
