---
layout: post
title: "相机和IMU同步的ROS方案[译]"
categories: study
author: "Grau GmbH"
---

原文：[ROS CAMERA AND IMU SYNCHRONIZATION](http://grauonline.de/wordpress/?page_id=1951)



### Idea

针对 VIO 和 SLAM 应用，我们需要实现相机和 IMU 的硬件时间同步(微秒级的精度)：

![](http://grauonline.de/wordpress/wp-content/uploads/bluefox2_mpu6050_synchronize.png)

```
time:   0 ms, IMU data, camera image #0
time:   5 ms, IMU data
time:  10 ms, IMU data
...
time:  50 ms, IMU data, camera image #1
time:  55 ms, IMU data
time:  60 ms, IMU data
...
time: 100 ms, IMU data, camera image #2
...
```

### 我的配置

我将介绍自己的配置细节，同样适合于你们自己的硬件。

- 单目全局相机（型号：mvBlueFox-MLC200wG, ON Semiconductor MT9V034 digital image sensor），USB 接口（注：含用于硬件同步的**外部触发引脚**）
- 132 度鱼眼镜头
- IMU（MPU6050），与 Arduino Nano 相连

![](http://grauonline.de/wordpress/wp-content/uploads/visual_intertial-300x225.jpg)

### 概述

**Arduino** 以毫秒级精度计算 **IMU** 测量（200Hz）数据的时间戳。于特定时间戳（20Hz）经过触发线触发**相机**捕获图像。时间戳和触发计数器数据会发送给 **PC**（IMU 节点）。IMU 节点从 Arduino 接收 IMU 数据，并通过一个新的 ROS TimeRefere 消息（topic/imu/trigger_time）发布时间数据。相机节点订阅时间数据为每一种图像校准时间。故消息流如下：

IMU –> Arduino –> PC (ROS IMU node) –> ROS camera node

ROS trigger_time 通过 IMU 节点发布消息如下所示：

```
/imu/trigger_time  (TimeReference):

header.seq   --> image sequence (triggerCounter)
header.stamp --> timestamp of image
```

### Arduino

1. 先把相机的触发线（digital input 0）和 Arduino 的 D5 引脚相连。

2. 安装 [ROS mpu6050_serial_to_imu](https://github.com/fsteinhardt/mpu6050_serial_to_imu) 包

3. 编写程序 src/MPU6050.ino

   1. 给 IMU 数据（200Hz）加上时间戳

   2. 以 20Hz 的频率触发相机

        ```C++
        volatile unsigned long irqTimestamp = 0;
        volatile unsigned long triggerCounter = 0;
        volatile byte irqCounter = 0;

        // called by MPU6050 interrupt 
        void dmpDataReady() {
            irqTimestamp = millis();
            irqCounter++;
            if (irqCounter == 10){ // trigger cam @20 Hz
              digitalWrite(TRIGGER_PIN, HIGH);
              digitalWrite(TRIGGER_PIN, LOW);
              triggerCounter++;
              irqCounter = 0;   
            } 
        }
        ```

4. 在发送给 PC 的消息中添加时间戳和触发计数器。利用时间戳和触发计数器，PC可以实现

   1. 校准 IMU 时间
   2. 校准相机时间和确定图像编号


### ROS: mpu6050_serial_to_imu

1. 编写  src/mpu6050_serial_to_imu_node.cpp，用来接收来自 Arduino 的时间戳和触发计数器的消息。

2. 添加发布“trigger_time”，用于发布 IMU 节点的时间戳。相机节点随后订阅它们：

   ```c++
   ros::Publisher trigger_time_pub = nh.advertise<sensor_msgs::TimeReference>("trigger_time", 50);
   ```

3. 发布消息：

   ```c++
   ros::Time measurement_time(ts / 1000, (ts % 1000) * 1000*1000);  // sec, nsec       

   ros::Time time_ref(0, 0);
   trigger_time_msg.header.frame_id = frame_id;
   trigger_time_msg.header.stamp = measurement_time;
   trigger_time_msg.time_ref = time_ref;          trigger_time_pub.publish(trigger_time_msg);
   ```

   ### ROS: bluefox2

   1. 安装 [ROS bluefox2](https://github.com/KumarRobotics/bluefox2) 包

   2. 编写 src/single/single_node.cpp，用于相机节点订阅新的 trigger_time 消息：

      ```c++
      subTimeRef = pnh.subscribe("/imu/trigger_time", 1000, &bluefox2::SingleNode::callback, this);

      void SingleNode::callback(const sensor_msgs::TimeReference::ConstPtr &time_ref) {
        bluefox2::TriggerPacket_t pkt;
        pkt.triggerTime = time_ref->header.stamp;
        pkt.triggerCounter = time_ref->header.seq;     
        fifoWrite(pkt);
      }
      ```

   3. 修改 ‘Aquire’ 方法，为每张图片盖上时间戳：

      ```c++
      void SingleNode::Acquire() {
        while (is_acquire() && ros::ok()) {
          bluefox2_ros_->RequestSingle();
          // wait for new trigger packet to receive
          TriggerPacket_t pkt;
          while (!fifoRead(pkt)) {    
            ros::Duration(0.001).sleep();
          }
          // a new video frame was captured
          // check if we need to skip it if one trigger packet was lost
          if (pkt.triggerCounter == nextTriggerCounter) {
                bluefox2_ros_->PublishCamera(pkt.triggerTime);
          } else { 
            ROS_WARN("trigger not in sync (seq expected %10u, got %10u)!",
               nextTriggerCounter, pkt.triggerCounter);     
          } 
          nextTriggerCounter++;
          Sleep();
        }
      }
      ```

   4. 修改相机启动文件（launch/single_node.launch）

      1. 关闭自动曝光时间，采用固定曝光时间。以此才能根据触发时间戳计算最后图像的时间戳
      2. 用触发引脚（高电平触发）作为 camera digital input 0：

      ```c++
      <arg name="aec" default="false"/>
      <arg name="expose_us" default="15000"/>
      <!-- Trigger mode (ctm): 1=on demand (default), 3=hardware trigger -->     
      <arg name="ctm" default="3"/> 
      <arg name="cts" default="0"/>
      ```

   5. 为了让 IMU 节点和相机节点同时启动，在相机节点启动文件（launch/single_node.launch）里加入 IMU 节点：

      ```c++
      <node pkg="mpu6050_serial_to_imu" type="mpu6050_serial_to_imu_node" name="mpu6050_serial_to_imu_node" required="true">
            <param name="port" value="/dev/ttyUSB0"/>
      </node>
      ```

   6. 运行相机启动文件：

      ```shell
      $ roslaunch bluefox2 single_node.launch
      ```

   7. 确认未遗漏任何时间消息。

   ### 下载

   [ROS_bluefox2_mpu6050_synchronization_hack](http://grauonline.de/wordpress/wp-content/uploads/bluefox2_mpu6050_synchronization.tar.gz)