---
layout: post
title: "用 Wireshark 诊断网络故障"
categories: study
author: "Jixiang Zhang"
---

Issues

1. 192.168.1.2   -> TCP-> 8.8.8.8     [TCP Retransmission] & [TCP Port numbers reused]
2. 192.168.1.2 -> TCP-> 192.168.1.254 [TCP Retransmission]
3. 192.168.1.254 ->ICMP-> 192.168.1.2  Destination unreachable (Network unreachable)
4. AAEONTec_92:f2:c8 ->ARP-> Broadcast Who has 192.168.1.123? Tell 192.168.1.120
5. IntelCor_4b:3e:26 ->ARP-> AAEONTec_86:43:34 Who has 192.168.1.102? Tell 192.168.1.2
6. rustdesk `$sudo systemctl disable rustdesk`

Usages

- 分析 -> 专家信息

References

1. [Wireshark User’s Guide](https://www.wireshark.org/docs//wsug_html_chunked/index.html)
