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

### Ubuntu: udev rules for USB scanner

How to create a udev rule to get a USB scanner working in Ubuntu jaunty.

Assumes that: (1) user is a member of group `scanner`, (2) sane is installed.

Symptoms of problem: "sane-find-scanner" detects the existence of the scanner, but "scanimage -L" does not report it when run with user permissions, only when run with root permissions. Scanning works as root, but not as a normal user.

1) Run `lsusb` to find the scanner's manufacturer and device IDs. Example output:

```bash
$ lsusb
Bus 003 Device 014: ID 04b8:012e Seiko Epson Corp.
Bus 003 Device 012: ID 046d:c408 Logitech, Inc. Marble Mouse (4-button)
Bus 003 Device 011: ID 05e3:0608 Genesys Logic, Inc. USB-2.0 4-Port HUB
Bus 003 Device 001: ID 0000:0000
Bus 002 Device 001: ID 0000:0000
Bus 001 Device 013: ID 04b8:0005 Seiko Epson Corp. Stylus Printer
Bus 001 Device 001: ID 0000:0000
```

In the above example, the first line relates to the scanner... in this case an Epson Perfection V200... it is somewhat unhelpful that this particular scanner's text ID string is nothing more than "Seiko Epson Corp.", but it is easy to determine that that device is indeed the scanner by running "lsusb" both with and without the scanner plugged in.

The four-digit hex number `04b8` is the manufacturer ID (observe that the Epson printer on the penultimate line has the same manufacturer ID), and `012e` is the device ID.

2) Create a file `/etc/udev/rules.d/40-scanner.rules` containing the following (your browser may have wrapped this; it should be all on one line):

```
SUBSYSTEMS=="usb", ATTRS{idVendor}=="04b8", ATTRS{idProduct}=="012e", ENV{libsane_matched}="yes", GROUP="scanner"
```

Of course you will substitute the appropriate values for your scanner, obtained from `lsusb`, for the idVendor and idProduct entries.

The SUBSYSTEMS and ATTRS entries have double == signs whereas the ENV and GROUP entries have single = signs. This is not a typo.

3) Restart udev:

```bash
$ sudo /etc/init.d/udev restart
* Stopping kernel event manager... [ OK ]
* Starting kernel event manager... [ OK ]
```

4) Unplug the scanner and plug it in again.

5) Check that it has obtained the correct permissions. Run "lsusb" again to determine the bus and device IDs - they will have changed due to the unplugging and replugging. Then check the appropriate device node in /dev/bus/usb:

```bash
$ lsusb
Bus 003 Device 015: ID 04b8:012e Seiko Epson Corp.
Bus 003 Device 012: ID 046d:c408 Logitech, Inc. Marble Mouse (4-button)
Bus 003 Device 011: ID 05e3:0608 Genesys Logic, Inc. USB-2.0 4-Port HUB
Bus 003 Device 001: ID 0000:0000
Bus 002 Device 001: ID 0000:0000
Bus 001 Device 013: ID 04b8:0005 Seiko Epson Corp. Stylus Printer
Bus 001 Device 001: ID 0000:0000
$ ls -l /dev/bus/usb/003/015
crw-rw-r-- 1 root scanner 189, 269 Apr 24 19:10 /dev/bus/usb/003/015
```

If the output corresponds with the above, all should be well, and running "scanimage -L" with user permissions should now pick the scanner up:

```bash
$ scanimage -L
device `epkowa:libusb:003:015' is a Epson Perfection V200 flatbed scanner
```

Note that the V200 requires a driver; this document does not cover that, only the setting of the permissions.

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