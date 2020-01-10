---
layout: post
title: "ArduPilot的历史"
categories: study
author: "Jixiang Zhang"
---

**硬件**

![History of ArduPilot](https://tvax1.sinaimg.cn/large/d494c514ly1gaqlp88gwmj20m809pwf6.jpg)
![Simple Overview of ArduPilot Operation](https://tvax4.sinaimg.cn/large/d494c514ly1gaqm3s7m6fj20k10f20ts.jpg)

**软件**

- [**ArduPilot**](https://github.com/ArduPilot/ardupilot)
  - Copter
  - Plane
  - **Rover**
    - **Boat**
  - Antenna Tracker
- [~~PX4~~](https://github.com/PX4/Firmware)
- [~~Paparazzi~~](https://github.com/paparazzi/paparazzi)

**DIY清单**

- **CUAV V5 Nano 飞控**
  ![CUAV V5 Nano](https://tvax1.sinaimg.cn/large/d494c514ly1garhmxzhn8j21u01sk78o.jpg)
- **GPS + Compass**
- **数传**
- **Sonar Sensors**
- **RC**
  - FlySky FS-I6
- 锂电池&充电器
- 地面站
  - Mission Planner
  - QGroundControl
  - APM Planner 2

**电机布局**

- Separate Steering and Throttle
  ![](https://tvax4.sinaimg.cn/large/d494c514ly1garihb5ij8j20nj0f6ac5.jpg)
- Skid Steering
  ![](https://tvax3.sinaimg.cn/large/d494c514ly1gariillt36j20ni0aiq4s.jpg)
- Omni Vehicles
  ![](https://tva3.sinaimg.cn/large/d494c514ly1garij3k0rkj20s20amq3t.jpg)

**无人船配置**

- `FRAME_CLASS` = 2
- Boats appear as boats on the ground station (version 3.3.0 and higher)
- In `Auto`, `Guided`, `RTL` and `SmartRTL` modes the vehicle will attempt to maintain its position even after it reaches its destination (version 3.2.0 and higher)
- `Vectored Thrust` can be enabled to improve steering response on boats which use the steering servo to aim the motors (version 3.3.1 and higher)
- `Loiter mode` for holding position
- `Echosounders` for underwater mapping

**准备工作**

- RC标定
- 加速度计标定
- 指南针标定
- RC飞行模式配置
- **电机配置**
  - 电调类型 `MOT_PWM_TYPE`
    - Normal
    - Brushed With Relay
    - Brushed BiPolar

**Mission Planning**

**参考文献**

- [Rover Home](https://ardupilot.org/rover/index.html)