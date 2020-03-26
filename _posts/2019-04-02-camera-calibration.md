---
layout: post
title: "张氏相机标定"
categories: study
author: "Jixiang Zhang"
---

#### 预备知识

##### 相机投影方程

$$
s\tilde{\mathbf{m}}=\mathbf{K}[\mathbf{R\quad t}]\tilde{\mathtt{M}}
$$

##### 内参矩阵

$$
\mathbf{K}=
\begin{bmatrix}
\alpha & \gamma & u_0\\
0 & \beta & v_0\\
0 & 0 & 1
\end{bmatrix}
$$

##### 单应矩阵（假设空间平面位于 $Z=0$，相对于世界坐标系）

$$
s
\begin{bmatrix}
u\\
v\\
1\\
\end{bmatrix}
=
\mathbf{K}[\mathbf{r}_1\quad \mathbf{r}_2\quad \mathbf{r}_3\quad \mathbf{t}]
\begin{bmatrix}
X\\
Y\\
0\\
1
\end{bmatrix}
=
\mathbf{K}[\mathbf{r}_1\quad \mathbf{r}_2\quad \mathbf{t}]
\begin{bmatrix}
X\\
Y\\
1
\end{bmatrix}
$$

简写为

$$
s\tilde{\mathbf{m}}=\mathbf{H}\tilde{\mathtt{M}} \quad \mathsf{with} \quad \mathbf{H}=\mathbf{K}[\mathbf{r}_1\quad \mathbf{r}_2\quad \mathbf{t}],\quad\tilde{\mathtt{M}}=[X,Y,1]^T.
$$

##### 内参约束

记

$$
[\mathbf{h}_1\quad \mathbf{h}_2\quad \mathbf{h}_3]=\lambda\mathbf{K}[\mathbf{r}_1\quad \mathbf{r}_2\quad \mathbf{t}]
$$

因 $\mathbf{r}_1\,,\mathbf{r}_2$ 正交，有内参约束

$$
\left\{
\begin{aligned}
&\mathbf{h}_1^T\mathbf{K}^{-T}\mathbf{K}^{-1}\mathbf{h}_2=0\\
&\mathbf{h}_1^T\mathbf{K}^{-T}\mathbf{K}^{-1}\mathbf{h}_1=\mathbf{h}_2^T\mathbf{K}^{-T}\mathbf{K}^{-1}\mathbf{h}_2
\end{aligned}
\right.
$$


#### 标定方法

##### 解析法

根据内参约束构造方程（n 表示图像数量）

$$
\mathbf{V}_{2n\times 6}\mathbf{b}=0
$$

方程的解 $\mathbf{b}$ 是 $\mathbf{V}^T\mathbf{V}$ 的最小特征值。然后依次计算出内参 $\mathbf{K}$ 、外参 $[\mathbf{r}_1\quad \mathbf{r}_2\quad \mathbf{r}_3\quad \mathbf{t}]$。

##### 非线性优化

最小化重投影误差

$$
\sum_{i=1}^{n}\sum_{j=1}^{m}||\mathbf{m}_{ij}-\hat{\mathbf{m}}(\mathbf{K},\mathbf{R}_i,\mathbf{t}_i,\mathtt{M}_j)||^2
$$

式中 $\hat{\mathbf{m}}(\mathbf{K},\mathbf{R}_i,\mathbf{t}_i,\mathtt{M}_j)$ 表示空间点 $\mathtt{M}_j$ 投影到第 $i$ 张图像上的像素坐标。常用 LM 迭代优化，而**初值**采用解析法的结果。

##### 考虑镜头畸变的优化

前面的标定假设相机无畸变，而实际相机存在**径向畸变**和切向畸变。下面以径向畸变为例

$$
\begin{aligned}
x_{distorted}&=x[1+k_1(x^2+y^2)+k_2(x^2+y^2)^2]\\
y_{distorted}&=y[1+k_1(x^2+y^2)+k_2(x^2+y^2)^2]\\
\end{aligned}
$$

式中 $x,y,x_{distorted},y_{distorted}$ 为归一化坐标，其像素坐标

$$
\begin{aligned}
u_{distorted}&=u_0+\alpha x_{distorted}\\
v_{distorted}&=v_0+\beta y_{distorted}\\
\end{aligned}
$$

综合以上构造线性方程

$$
\mathbf{D}\mathbf{k}=\mathbf{d} \quad \mathsf{with} \quad \mathbf{k}=[k_1,k_2]^T
$$

它的最小二乘解

$$
\mathbf{k}=(\mathbf{D}^T\mathbf{D})^{-1}\mathbf{D}^T\mathbf{d}
$$

##### 完整的优化模型

$$
\sum_{i=1}^{n}\sum_{j=1}^{m}||\mathbf{m}_{ij}-\mathbf{m}_{distorted}(\mathbf{K},\mathbf{k},\mathbf{R}_i,\mathbf{t}_i,\mathtt{M}_j)||^2
$$

#### 标定步骤总结

1. 标定之前需要准备一块平整的标定板；
2. 旋转标定板或相机采集若干不同图像；
3. 特征提取；
4. 用解析法估计内参 $\mathbf{K}$ 和外参 $[\mathbf{r}_1\quad \mathbf{r}_2\quad \mathbf{r}_3\quad \mathbf{t}]$；
5. 估计径向畸变系数 $\mathbf{k}=[k_1,k_2]^T​$；
6. BA优化全部参数 $\mathbf{K},\mathbf{k},\mathbf{R}_i,\mathbf{t}_i,\mathtt{M}_j$。

注：纯平移是会出现退化的情况。

##### References

1. **A Flexible New Technique for Camera Calibration**

