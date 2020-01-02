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
  ![xbox-default-linux](https://tvax3.sinaimg.cn/large/d494c514ly1gaevjuaepsj20p00fan2r.jpg)

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
  ![IMG_7102](https://tva4.sinaimg.cn/large/d494c514ly1gafm8tf2gkj21kq19dtpj.jpg)

### 组装方案

![IMG_7089](https://tvax3.sinaimg.cn/mw690/d494c514ly1gaca6r5h6vj21if1ji1kx.jpg)

### AhaBot

![ahabot](https://tvax1.sinaimg.cn/large/d494c514ly1gaiawqrx4lj20sl0dc76e.jpg)

DIY路线

1. Setup
2. Bringup
3. Basic Operation
4. SLAM
5. Navigation

![坐标关系](https://tva2.sinaimg.cn/large/d494c514ly1gaf3s8f17uj20d608qt8r.jpg)

![nav configuration](https://tvax2.sinaimg.cn/large/d494c514ly1gag51ui1yjj20lo08vwfx.jpg)

### 参考文献

- [Roomblock](https://www.instructables.com/id/Roomblock-a-Platform-for-Learning-ROS-Navigation-W/)
- [Roomba](http://wiki.ros.org/Robots/Roomba)
- [Running the navigation stack on the Roomba](http://wiki.ros.org/lse_roomba_toolbox/Tutorials/navigation%20on%20the%20Roomba)
- [TurtleBot3手册](http://emanual.robotis.com/docs/en/platform/turtlebot3/setup/#setup)
- [Coordinate Frames for Mobile Platforms](https://www.ros.org/reps/rep-0105.html)
- [Setup and Configuration of the Navigation Stack on a Robot](http://wiki.ros.org/navigation/Tutorials/RobotSetup#Navigation_Stack_Setup)