---
layout: post
title: "移动机器人运动规划笔记"
categories: study
author: "Jixiang Zhang"
---

[**高飞老师**](https://ustfei.com/)：

> 在我HKUST读博时候，计算机视觉课，老师扔来一篇08年的CVPR，“同学们，两周以内实现出来。不组队，自己做。最后一个一个demo程序给老师看。”没有任何框架代码，从0开始写。或者我导师沈老师的课，这个星期写A star，下个星期minimum snap，下下个星期EKF。全都是C++。你不上过这种硬核的课，怎么知道什么才叫“上课”。

编程环境

1. MATLAB

- Keyboard-Shorcuts 设置为 `Windows Default Set`

2. C++

**A*算法** $f(n) = g(n) + h(n)$

1. Remove
   - OpenList 数据结构 $\{bool,X ,Y,Parent X,Parent Y,h(n),g(n),f(n)\}$
2. Expansion
3. Push

**RRT**

参考文献

- [Planning Algorithms](http://planning.cs.uiuc.edu/node1.html)
