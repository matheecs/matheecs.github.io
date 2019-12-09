---
layout: post
title: "科研那些事儿"
categories: study
author: "Jixiang Zhang"
---

### Shadowsocks使用方式

1. 服务器+客户端模式
2. 路由器模式

### 安装步骤

1. 第一步需要购买虚拟专用服务器（VPS），部署VPS作为服务端。推荐 [Vultr](https://www.vultr.com/) 提供商，支持支付宝付款。选择购买位于美国的服务器，建议选择安装好 Ubuntu 16.04 系统的服务器。购买好后开始部署。采用系统自带的终端即可SSH登录服务器。执行以下命令：

   ```shell
   $ wget --no-check-certificate -O shadowsocks-all.sh https://raw.githubusercontent.com/teddysun/shadowsocks_install/master/shadowsocks-all.sh
   $ chmod +x shadowsocks-all.sh
   $ ./shadowsocks-all.sh 2>&1 | tee shadowsocks-all.log
   ```

   根据提示输入端口、加密方式等信息。

   - 重启即可。

   - [可选]安装 BBR 加速算法：

      ```shell
      $ wget --no-check-certificate https://github.com/teddysun/across/raw/master/bbr.sh && chmod +x bbr.sh && ./bbr.sh
      ```

2. 第二步，在自己的设备上安装 Shadowsocks 客户端（Win/macOS/Linux/iOS/Android/OpenWRT），输入账号信息，开启愉快的科研生活。

3. 安装浏览器插件 SwitchyOmega。

4. TODO：如何实现 Ubuntu 下全局代理？

### 参考文章

1. [自建 Shadowsocks 翻墙服务器](https://medium.com/@okjaketo/%E8%87%AA%E5%BB%BA-shadowsocks-%E7%BF%BB%E5%A2%99%E6%9C%8D%E5%8A%A1%E5%99%A8-35bd66b29fb3)
2. [Shadowsocks 一键安装脚本](https://teddysun.com/486.html)
3. [一键安装最新内核并开启 BBR 脚本](https://teddysun.com/489.html)
4. [Shadowsocks Troubleshooting](https://teddysun.com/399.html)
5. [shadowsocks客户端](https://shadowsocks.org/en/download/clients.html)
6. [shadowsocks-libev](https://github.com/shadowsocks/shadowsocks-libev)
7. [使用SwitchyOmega设置Chrome代理](https://blog.csdn.net/qq_31851531/article/details/78410146)
8. [scientific_network_summary](https://github.com/crifan/scientific_network_summary)