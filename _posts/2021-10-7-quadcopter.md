---
layout: post
title: "智能无人机笔记🚁"
categories: study
author: "Jixiang Zhang"
---

《智能无人机》线下课程由[深蓝学院](https://www.shenlanxueyuan.com)和[FAST Lab](http://zju-fast.com)组织举办，为期五天时间。无人机特指旋翼无人机。课程内容安排

| 时间    |      主讲人 |    内容    |
| :------ | ----------: | :--------: |
| 10/2 AM |     Fei Gao | 无人机概论 |
| 10/2 PM |    Neng Pan | 软硬件DIY  |
| 10/3    | Zhepei Wang | 控制与规划 |
| 10/4    | Zhepei Wang |  GCOPTER   |
| 10/5    |  Hongkai Ye |  路径规划  |
| 10/6    |  Hongkai Ye |  全局规划  |

![](/images/quadcopter-class-notes.jpg)

本课程自底向上介绍了无人机系统，从硬件到软件，从理论到实践，从控制到规划。重点讲解了无人机的最新规划控制算法。

---

### 第1天 概论与DIY

无人机系统子模块

| 模块 |                   功能 |
| :--- | ---------------------: |
| 传感 |      IMU RGBD相机 雷达 |
| 估计 | SLAM VIO LIO Vicon UWB |
| 建图 |      滚动栅格地图 ESDF |
| 规划 |      前端搜索 后端优化 |
| 控制 |           线性控制 SE3 |

需注意 SLAM 中的稀疏地图不同于导航规划所用的稠密地图，前者的作用是辅助定位，通常不能直接用于导航规划。控制器默认 INPUT 为 {推力 + 姿态}，更底层的功能则由 PX4 飞控完成。我们常把旋翼无人机当作一个在构型空间的质点，需先对地图进行膨胀处理；而作为一个多面体时，需做碰撞检测。

路径搜索算法：Dijkstra、A* 和 RRT 算法。规划优化算法：min snap 算法。

无人机模型具有**微分平坦**的特点。

**规划是开环的控制；控制是闭环的规划**。规划考虑长期过程；控制考虑短期过程。

闭环控制解决**模型误差和状态估计误差**的问题。

根据**力效表**选择电机、电池、电调。

飞控相当于无人机的**小脑**；NX/树莓派相当于无人机的**大脑**。

常用相机有 D435、T265。

Linux基础：文件目录、常用命令、进程管理。

ROS基础：文件系统、Topic、launch 文件参数功能、rqt 工具、RViz、Gazebo。

---

### 第2天 控制与规划

规划算法分类

| 算法类型           |             特点 |
| :----------------- | ---------------: |
| Optimization-based |  局部 local 最优 |
| Sampling-based     | 全局 global 最优 |
| Search-based       | 全局 global 最优 |

常采用全局方法结合局部方法 **Coarse-to-Fine**：先用全局方法寻找拓扑结构，提供一个粗糙的初值，再用局部方法。

模型越简单，系统越稳定。

控制流：Waypoints -> 位置控制 -> 姿态控制 -> 角速度控制 -> 电机控制。IMU 能直接测量角速度，故角速度环控制效果好。穿越机采用角速度控制。无人机低速飞行时忽略空气阻力，低速常指速度小于 8m/s。电机模型的滞后参数。

无人机欧拉角定义采用 ZXY 顺序，对应 Yaw-Roll-Pitch，ZXY 顺序易解释**微分平坦**。实验时采用 ZYX 顺序。Eigen 转欧拉角有 BUG，导致控制异常，建议用[解析公式](https://en.wikipedia.org/wiki/Euler_angles)转换。

牛顿方程描述平动；欧拉方程描述转动。

因为二阶系统 xdd = u 含双微分，所以 PID 控制时不需要积分项I，故用 PD 控制器。物理直观含义：比例项P 解释为弹簧；微分项D 解释为阻尼。Noise filter 后再计算微分项。积分项I 会影响系统的稳定性。线性控制在平衡态附近做线性化，线性模型适用条件为倾斜角度不超过 20度。

无人机微分平坦原理用于控制和规划。

前馈的定义。

欧拉角的奇异性（分母为零）发生的三种情况：推力为零；倒转 180度；俯仰角正负 90度。

无人机控制的问题：奇异性；偏航角响应慢；不考虑空气阻力。

平坦空间 vs 构型空间 vs 工作空间。

如何处理约束：规划自顶向下；MPC 自底向上。

大于 9 次的高阶多项式存在数值稳定性的问题。

从手工设计参数到只需设计目标函数。

平坦表示系统没有内动态。

Yaw 单独控制。

有约束优化问题：不要把所有约束喂给求解器，只给关键参数。

min snap QP 迭代法复杂度 O(n*n*log(n))，分离 XYZ 求解效率更高；闭式解析法复杂度 O(n*n*n)。

时间归一化提升数值稳定性（条件数）。

---

### 第3天 GCOPTER 或无人机轨迹优化终极方案

之前方法存在的缺点：Heuristic 太保守、约束太多、非时间最优、数值迭代存在数值稳定性差的问题、微分平坦把无人机当作一个质点。

运动规划的地图表征：Ray casting 局部栅格地图；全局地图 OpenVDB/nanoVDB(GPU 加速)。点云 KD树（有增量式版本 iKD树）。

ESDF = 栅格地图 + 离障碍物的距离，考虑 GPU/FPGA 并行计算。

从计算几何/图形学/CFD 领域寻找思路和方法。

安全走廊 Corridor 几何体生成算法 IRIS …… Corridor 对优化更友好。

用 GCOPTER 论文中定理 2 最优性的充要条件快速求解 min snap。

把原始地图转换为适用规划的地图。

路径规划 Graph-based：动态图 PRM/RRT*，概率完备；静态图 A*/JPS，伪距离最优。

轨迹规划 Optimization-based：涉及动力学，属于局部方法，须与全局方法结合使用。用全局方法克服非凸性，找拓扑结构。

考虑空气阻力的无人机模型。

ETH 最新 SQP 求解器 HPIPM。

从状态空间求解到平坦空间求解，变量和约束大大减少。

GCOPTER 把轨迹表征为 {waypoint + time}。学习 CFD 的思路，把 waypoint + time 作为关键参数。MINCO 具备线性复杂度。不要把等式约束喂给求解器。用微分几何方法处理安全走廊几何体约束。

---

### 第4天 路径规划

路径规划常用于 GAME 游戏领域。

路径规划算法分为 Search-based、Sampling-based。

构型空间与时间无关，状态空间与时间有关。

构型空间中把无人机抽象为一个点。

Search-based 方法：Dijkstra、A*、JPS。核心循环步骤：**Remove** 取出节点，**Expansion** 扩展领域节点，**Push** 加新节点。

广度优先 vs 深度优先。

BFS -> Dijkstra(f=g+0) -> A*(f=g+h) -> JPS。

Heuristic is a GUESS of how close you are to the target。

A* 最优性条件：**Admissible** h(n) <= h*(n)。A* 效率最优条件：**Consistent** h(x) <= d(x,y)+h(y) Consistent 条件强于 Admissible，保证访问最少的节点数量。

Sub-optimal solution 得到 Weighted A*，牺牲最优性换取速度。

栅格地图的曼哈顿距离不是 Admissible 的。

栅格地图 Tie breaker 解决方法 h = h + cross*0.001 不适合有障碍的场景。

针对栅格地图的 JPS 方法，解决对称路径问题，邻居扩展方式不同。不适合迷宫、多障碍物的场景。

动态规划思想。Direct 方法；Successive Approximation 方法 (Pull法、Push法)。

Dijkstra vs Pull法：前者要求边代价为正，且升序跟更新；后者会多次访问同一个节点。

Sampling-based 和 Search-based 本质上都是 Search。前者采用隐式图；后者采用显示图，又分为增量式和 Batch式（如BIT*）。

PRM -> RRT -> RRT* -> RRT#。

核心循环步骤：Exploration 探索解空间 and Exploitation。

PRM = Learning Phase 利用先验或者学习方法 + Query Phase。缺点是效率低，实际只需要一条路径。改进用 Lazy collision checking。RRT 构建了一个树，边构图边探索。Bidirectional RRT 加速找到第一个解。RRT* 渐近最优。RRT# 找到第一个解后加速收敛，解决了 over-exploration 和 under-exploration 的问题。

---

### 第5天 全局规划

Informed-RRT*、GuILD 只适用于 L2 norm。Informed-RRT* 改变了采样集合。

Kinodynamic RRT* 适用于全局大空间，引入时间。前向/后向可达解。路径规划提取骨架，然后在附近采样。全局采样 + 局部优化。**STD-Trees** 时空形变树，Kinodynamic RRT* 引入局部优化思想来提升轨迹收敛速度，优化树的质量。形变单元的代价设计。采用 Penaty 把有约束问题转化为无约束问题。考虑到 SDF 不连续采用 LMBM。

---

### 总结

课程特色：理论结合实践，学员从零开始搭建无人机，先仿真再实验；小组形式，组员互助；线下形式，学员能与老师面对面交流；国内顶级教育资源。唯一缺点是五天学习时间相对紧凑，部分知识点消化不足。

#### 仿真代码框架

![](/images/sim_ros.jpg)

#### 实机代码框架

![](/images/real_ros.jpg)

#### MINCO 实验

![](/images/minco.gif)

#### min snap 实验

![](/images/minsnap.gif)

### References

- [FAST Lab 实验室主页](http://zju-fast.com)
- [ZJU FAST Lab 代码主页](https://github.com/ZJU-FAST-Lab)
- [GCOPTER](https://github.com/ZJU-FAST-Lab/GCOPTER)
- [FASTLAB 自主导航无人机硬件](https://github.com/ZJU-FAST-Lab/Fast-Drone-250)
- [自主旋翼无人机导论](https://www.shenlanxueyuan.com/course/385)

最后感谢各位老师助教同学和深蓝学院工作人员，特别是第三组的小伙伴，一起努力实现了线性控制器、SE3控制器、min snap和MINCO轨迹规划及其实验。

<!-- ![](/images/group3.jpg) -->