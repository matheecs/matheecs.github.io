---
layout: post
title: "树莓派那些事儿"
categories: tech
author: "Jixiang Zhang"
---

### 把树莓派变成无线路由器

[RaspAP](https://github.com/billz/raspap-webgui)

自动安装，没有成功

```shell
$ wget -q https://git.io/voEUQ -O /tmp/raspap && bash /tmp/raspap
```

建议查看 **installers** 文件夹内容，然后手动安装，提高安装成功率

```shell
$ bash raspbian.sh
...
```



### 养成备份的好习惯

[BACKUPS](https://www.raspberrypi.org/documentation/linux/filesystem/backup.md)



### 相机使用

开启相机功能

```shell
$ sudo raspi-config
```



##### What causes ENOSPC error when using the Raspberry Pi camera module?

```shell
$ raspistill -o image.jpg
报错
mmal: mmal_vc_component_enable: failed to enable component: ENOSPC
mmal: camera component couldn't be enabled
mmal: main: Failed to create camera component
mmal: Failed to run camera app. Please check for firmware updates
```

Solution

```shell
$ sudo rpi-update
```

