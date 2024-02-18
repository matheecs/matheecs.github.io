---
layout: post
title: "局域网内多Master通信方法"
categories: tech
author: "Jixiang Zhang"
---

![](images/ros_multimaster.png)

视频效果：

<video style="display:block; width:100%; height:auto;" src="{{site.baseurl}}/files/icra2019.mp4"  controls preload></video>

本文局域网特指单网络环境，比如只有一个路由器的情况。

使用 `multimaster_fkie` 方案，它主要包含的软件包：

- `master_discovery`：作用是周期性广播消息；检测本地 master 消息的变化并告知其他的 master；
- `master_sync`：利用`master_discovery`提供的数据为本地 master 注册远程的 topic 和 service。

## 配置步骤

1. 修改`/etc/hosts`绑定主机名（Host）与IP
2. 修改`~/.bashrc` 的 ROS_MASTER_URI 并 `$source ~/.bashrc`
3. 开启组播（multicast）并确认，默认的组播IP地址是 `224.0.0.1`，确认是否开启`$ping 224.0.0.1`

## 示例

```xml
<node name="master_discovery" pkg="master_discovery_fkie" type="master_discovery">
    <param name="mcast_group" value="224.0.0.1" />
</node>
<node name="master_sync" pkg="master_sync_fkie" type="master_sync">
    <rosparam param="sync_topics">
    [/master/goal_task,/wing/goal_task]
    </rosparam>
    <rosparam param="ignore_services">[/*]</rosparam>
</node>

```

topic 传递流：

```xml
master —— /master/goal_task ——> wing
wing —— /wing/goal_task ——> master
```

如何在 launch 文件中统一修改 topic 的命名空间：

```xml
<group ns="master">
 ...
</group>
```

## Debug-**Namespace**（命名空间）

1. base
2. relative/name
3. /global/name
4. ~private/name

## References

1. [multimaster FKIE](http://fkie.github.io/multimaster_fkie/index.html)
2. [**Multi-master ROS systems**](http://digital.csic.es/bitstream/10261/133333/1/ROS-systems.pdf)
3. [Multimaster ROS configuration and multimaster_fkie](http://www.huyaoyu.com/technical/2018/08/27/multimaster-ros-configuration-and-multimaster-fkie.html)
