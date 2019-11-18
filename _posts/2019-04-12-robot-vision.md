---
layout: math
title: 我所理解的机器人视觉
---

### 背景

计划在毕业前总结读研时学习机器人、计算机视觉、SLAM为系列博文，同时会**结合源代码分析**算法，**强调算法的实现**，采用图解的方式培养**直观理解能力**。

最近学习CMU 16-385的[计算机视觉课程](http://www.cs.cmu.edu/~16385/s17/)，[Kris Kitani](http://www.cs.cmu.edu/~kkitani/)的 CV 讲义非常易于阅读和理解，有必要分享给大家，好比《视觉SLAM十四讲》之于入门 SLAM 的作用。系统整理所学的东西，同时巩固基础。

### 代码实现参考方案

1. [MRPT](https://github.com/MRPT/mrpt)
   - Eigen3
   - OpenCV
   - wxWidgets
   - opengl+glut
   - SSE*
   - libftdi
   - libpcap
   - libusb
   - ZeroMQ (libzmq)
   - PCL
   - LAS (liblas)
   - SuiteSparse
2. OpenCV
3. Eigen3
4. Sophus
5. g2o
6. Ceres
7. Caffe

### 大纲

![](/images/framework.jpg)

# Part 1 **计算机视觉**

## 1 图像处理

### 数字图像的流水线

![](/images/pipeline.jpg)

### 图像滤波
### 像素处理
### 区块滤波
### 高斯滤波
### 降采样
### 金字塔（Gaussian+Laplacian）
### 图像梯度与梯度滤波（Sobel滤波+Laplace滤波/LoG）
### 滤波与卷积

## 2 Hough 变换（**Hough Transform**实践）

### 线段提取
### 线段参数化
### Hough 变换（核心思路: edges **vote** for the possible models）
### 通用型 Hough 变换
### Hough 变换的应用

## 3 角点检测

### 角点定义
### 二次型与特征向量的可视化
### Harris 角点
### 多尺度检测

## 4 视觉识别（**Bag of Words**实践）

### 设计描述子
### 特征描述子：MOPS、GIST、HOG、Textons、SURF、BRIEF
### Scale Invariant Feature Transform（SIFT）
### 目标识别：分类/检测/分割

- 特征匹配（k-d 树、FLANN）
- 空间推理
- 区域分类 $\to$ CNN

### 概率基础：随机变量、Bayes’ Rule、Marginalization、Conditioning
### BoW

- TF-IDF
- 词典学习$\to$编码$\to$分类
- K-means 聚类

### 分类算法

- $k$近邻法
- 朴素贝叶斯法
- 支持向量机

## 5 卷积神经网络（**CNN**实践）

### 感知机
### 神经网络
### 梯度下降的可视化
### Back-Propagation 算法
### 深度学习
### 基本元素：Conv+ReLU+Pool+Softmax
### LeNet$\to$**AlexNet**$\to$VGGNet$\to$GoogLeNet$\to$ResNet

## 6 2D-2D 变换（**Homography**实践）

### 2D变换：平移、欧式、相似、仿射、射影
### 2D对齐：最小二乘法
### 2D对齐：直接线性变换（DLT） $\mathbf{A}\mathbf{h}=\mathbf{0} \to \mathbf{SVD: A=U\Sigma V^{T}}$
### 2D对齐：RANSAC
### 四点法计算 $\mathbf{H}$

## 7 多视图几何

### 相机矩阵（3D-2D）$\mathbf{P}_{3 \times 4}=\mathbf{K[R \mid t]}$
### 相机模型
### 相机位姿估计

- DLT 计算 $\mathbf{P}$
- SVD
- RQ 分解计算 $\mathbf{K,R}$

### 三角测量

- $\mathbf{x=\alpha PX}$ 等价于 $\mathbf{x \times PX=0}$

## 8 运动恢复结构（**SfM**实践）

### 对极几何
### 本质矩阵$\mathbf{E}$（已标定相机，归一化坐标）
### 基础矩阵$\mathbf{F}$（未标定相机，像  素坐标）

- 八点法 SVD

### 三维重建

- Step1: 八点法计算$\mathbf{F}$;
- Step2: $\mathbf{P,P^{\prime}}$;
- Step3: 三角测量计算$\mathbf{X}$.

## 9 双目视觉

### 双目对齐
### 立体匹配

- SSD、SAD、NCC

### 能量最小化

- 动态规划

## 10 光流

### 灰度不变与微运动假设

- 泰勒展开

### Constant Flow（Lucas-Kanade 光流）

- Harris 角点
- Aperture Problem

### Horn-Schunck 光流
### 图像对齐（Image Alignment）

- Lucas Kanade: $\sum_{\boldsymbol{x}}[I(\mathbf{W}(\boldsymbol{x} ; \boldsymbol{p}+\Delta \boldsymbol{p}))-T(\boldsymbol{x})]^{2}$
- Baker-Matthews: $\sum_{\boldsymbol{x}}\left[I(\mathbf{W}(\mathbf{W}(\boldsymbol{x} ; \Delta \boldsymbol{p}) ; \boldsymbol{p})-T(\boldsymbol{x})]^{2}\right.​$

## 11 跟踪（**Lucas Kanade Tracking**实践和**Mean-shift Tracking**实践）

### Kanade-Lucas-Tomasi (KLT) 跟踪
### Mean-Shift 跟踪

- Mean Shift 算法
- Kernel 密度估计
- Mean-Shift 目标跟踪（粒子滤波？）

## 12 卡尔曼滤波

### 滤波（当前状态） vs 平滑 vs 预测
### 稳定性假设、马尔科夫假设和先验状态条件
### 卡尔曼滤波
### EKF
### MonoSLAM

# Part 2 **视觉SLAM**

## 13 三维几何（Eigen实践）

### 旋转矩阵
### 旋转向量
### 欧拉角
### 四元数

## 14 李群与李代数（Sophus实践）

### 扰动模型

## 15 粒子滤波

### 贝叶斯滤波
### Monte Carlo 采样
### 重要性采样
### SIS
### 重采样
### SIR

## 16 图优化与Bundle Adjustment（g2o与Ceres实践）

### Gaussian-Newton法
### Levenberg-Marquardt法

## 17 因子图、马尔科夫随机场与贝叶斯网络

### 因子图：$\mathbf{J}$
### 马尔科夫随机场：$\mathbf{H}$
### 贝叶斯网络：$\mathbf{R}$

## 18 开源方案分析

### S-MSCKF
### ORB-SLAM2
### SVO

### VIO

- IMU传感器
- IMU预积分
- FEJ

## 19 重定位

# Part 3 **应用：运动规划**

## 20 Adaptive Monte Carlo Localization

![](/images/MCL.jpg)

## 21 A Star Algorithm

![](/images/AStar.jpg)

## 22 Dynamic Window Approach

```pseudocode
BEGIN DWA(robotPose,robotGoal,robotModel)
   desiredV = calculateV(robotPose,robotGoal)
   laserscan = readScanner()
   allowable_v = generateWindow(robotV, robotModel)
   allowable_w  = generateWindow(robotW, robotModel)
   for each v in allowable_v
      for each w in allowable_w
      dist = find_dist(v,w,laserscan,robotModel)
      breakDist = calculateBreakingDistance(v)
      if (dist > breakDist)  //can stop in time
         heading = hDiff(robotPose,goalPose, v,w)
         clearance = (dist-breakDist)/(dmax - breakDist)
         cost = costFunction(heading,clearance, abs(desired_v - v))
         if (cost > optimal)
            best_v = v
            best_w = w
            optimal = cost
    set robot trajectory to best_v, best_w
END
```

References

1. [Dynamic Window Algorithm motion planning](http://adrianboeing.blogspot.com/2012/05/dynamic-window-algorithm-motion.html)

## 23 行为树

![](/images/decisionBT.jpg)

References

1. [Introduction to the A* Algorithm](https://www.redblobgames.com/pathfinding/a-star/introduction.html)

# **附录**

## A. C++

## B. ROS

## C. Linux

## D. Python

## E. 最小二乘法

## F. RANSAC

## G. 矩阵分解

### **SVD!!!**

- Total Least Squares
- Solution is the eigenvector corresponding to smallest eigenvalue of $\mathbf{A^TA}$
- Solution is the column of V corresponding to smallest singular value $\mathbf{A=U\Sigma V^T}$
- 主成分分析

### QR
### Cholesky

## H. 矩阵求导

## I. 概率基础

- Chi-square 分布
  - 卡方检验
- Mahalanobis 距离
- Kullback-Leibler Divergence（KLD）
- 重采样方法
  - Multinomial
  - Residual
  - Stratified
  - Systematic
- 随机数生成
  - MT19937
- 图模型

![](/images/reward.jpg)

