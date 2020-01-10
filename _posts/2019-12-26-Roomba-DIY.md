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
  <!-- ![IMG_7102](https://tva4.sinaimg.cn/large/d494c514ly1gafm8tf2gkj21kq19dtpj.jpg) -->

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

<!-- ![坐标关系](https://tva2.sinaimg.cn/large/d494c514ly1gaf3s8f17uj20d608qt8r.jpg) -->

![nav configuration](https://tvax2.sinaimg.cn/large/d494c514ly1gag51ui1yjj20lo08vwfx.jpg)

### 利用laser_filters裁剪雷达数据

<!-- ![laser scanner view](https://tva4.sinaimg.cn/large/d494c514ly1gaikgo5td2j20dy05ht9r.jpg) -->

<!-- ![S1](https://tvax1.sinaimg.cn/large/d494c514ly1gainldvkgrj20s80kun0j.jpg) -->

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

![scan_to_scan_filter_chain](https://tvax2.sinaimg.cn/large/d494c514ly1galeiijyedj20j40a4gmf.jpg)

### RViz导入模型与底盘驱动测试

![aha](https://tva2.sinaimg.cn/large/d494c514ly1gakn9ag6zej21fj0tctce.jpg)
![aha-bot](https://tvax2.sinaimg.cn/large/d494c514ly1gakxug6l2kj20zk0zf154.jpg)

### TODO

1. 激光数据去畸变
2. 压缩mesh模型，提高RViz加载速度

### udev rule

[lino_udev](https://github.com/linorobot/lino_udev)

### EKF

![截屏2020-01-10下午10.45.30](https://tva3.sinaimg.cn/large/d494c514gy1garummvn9zj20n704u3z3.jpg)

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