---
layout: post
title: "机器人底层驱动"
categories: study
author: "Jixiang Zhang"
---

<!-- ### RS232与TTL

![RS232和TTL的时序对比](https://tva4.sinaimg.cn/large/d494c514ly1gaj9iefrmmj20k00bwjtn.jpg) -->

### 2020.01.04四轮差速底盘测试记录

测试方案：ROS上层**只发线/角速度**控制指令，未用航迹推演 odom/tf 反馈的数据，相当于开环控制。

![电气结构.jpg](https://i.loli.net/2019/12/25/l5C2e3wLmWvirHs.jpg)

#### 线速度测试

线速度 0.5——1.5 m/s 时，目标位移 10 m，实际 9.7——9.9 m，速度越大误差越大。

#### 角速度测试

角速度 0.5——1.0 rad/s 时，目标角位移 5*(2*PI) rad，实际 2.6*(2*PI) rad，大概是目标值的一半，推测与轮子打滑有关。

#### 存在的问题

- 角速度误差偏大，50%左右
  - TODO：标定角速度的系数`scale`
  - Question：角速度系数和线速度系数会相互影响吗？
- 一个编码器发出异常声音，且运行时会抖动

#### 测试用程序

```python
#!/usr/bin/env python

import rospy
from geometry_msgs.msg import Twist
from math import pi

class OutAndBack():
    def __init__(self):
        # Give the node a name
        rospy.init_node('out_and_back', anonymous=False)

        # Set rospy to execute a shutdown function when exiting
        rospy.on_shutdown(self.shutdown)

        # Publisher to control the robot's speed
        self.cmd_vel = rospy.Publisher('/cmd_vel', Twist, queue_size=1)

        # How fast will we update the robot's movement?
        rate = 50

        # Set the equivalent ROS rate variable
        r = rospy.Rate(rate)

        # Set the forward linear speed to 0.2 meters per second
        linear_speed = 0.2

        # Set the travel distance to 1.0 meters
        goal_distance = 1.0

        # How long should it take us to get there?
        linear_duration = goal_distance / linear_speed

        # Set the rotation speed to 1.0 radians per second
        angular_speed = 1.0

        # Set the rotation angle to Pi radians (180 degrees)
        goal_angle = pi

        # How long should it take to rotate?
        angular_duration = goal_angle / angular_speed

        # Initialize the movement command
        move_cmd = Twist()

        # 1. Linear Speed test:
        move_cmd.linear.x = linear_speed
        ticks = int(linear_duration * rate)
        for t in range(ticks):
            self.cmd_vel.publish(move_cmd)
            r.sleep()
        move_cmd = Twist()
        self.cmd_vel.publish(move_cmd)
        rospy.sleep(1)

        # 2. Rotate Speed test:
        # move_cmd.angular.z = angular_speed
        # ticks = int(angular_duration * rate)
        # for t in range(ticks):
        #     self.cmd_vel.publish(move_cmd)
        #     r.sleep()
        # move_cmd = Twist()
        # self.cmd_vel.publish(move_cmd)
        # rospy.sleep(1)

        # Stop the robot
        self.cmd_vel.publish(Twist())

    def shutdown(self):
        # Always stop the robot when shutting down the node.
        rospy.loginfo("Stopping the robot...")
        self.cmd_vel.publish(Twist())
        rospy.sleep(1)

if __name__ == '__main__':
    try:
        OutAndBack()
    except:
        rospy.loginfo("Out-and-Back node terminated.")
```

启动底盘后执行测试脚本

```bash
roslaunch xqserial_server xqserial.launch
rosrun rbx1_nav timed_out_and_back.py
```