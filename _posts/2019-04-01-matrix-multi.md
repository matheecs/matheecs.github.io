---
layout: post
title: "矩阵相乘的时间复杂度"
categories: study
author: "Jixiang Zhang"
---

#### Strassen 算法

采用分治的算法（Divide and Conquer）思想，把矩阵相乘的时间复杂度从 $O(n^3)$ 提升到 $O(n^{\log_2 7})=O(n^2.81)$

$$
\mathbf{C} = \mathbf{A} \mathbf{B} \qquad \mathbf{A},\mathbf{B},\mathbf{C} \in R^{2^n \times 2^n}
$$

$$
\mathbf{A} =
\begin{bmatrix}
\mathbf{A}_{1,1} & \mathbf{A}_{1,2} \\
\mathbf{A}_{2,1} & \mathbf{A}_{2,2}
\end{bmatrix}
\mbox { , }
\mathbf{B} =
\begin{bmatrix}
\mathbf{B}_{1,1} & \mathbf{B}_{1,2} \\
\mathbf{B}_{2,1} & \mathbf{B}_{2,2}
\end{bmatrix}
\mbox { , }
\mathbf{C} =
\begin{bmatrix}
\mathbf{C}_{1,1} & \mathbf{C}_{1,2} \\
\mathbf{C}_{2,1} & \mathbf{C}_{2,2}
\end{bmatrix}
$$

根据矩阵乘法定义计算得（**8次乘法运算**）

$$
\mathbf{C}_{1,1} = \mathbf{A}_{1,1} \mathbf{B}_{1,1} + \mathbf{A}_{1,2} \mathbf{B}_{2,1}\\
\mathbf{C}_{1,2} = \mathbf{A}_{1,1} \mathbf{B}_{1,2} + \mathbf{A}_{1,2} \mathbf{B}_{2,2}\\
\mathbf{C}_{2,1} = \mathbf{A}_{2,1} \mathbf{B}_{1,1} + \mathbf{A}_{2,2} \mathbf{B}_{2,1}\\
\mathbf{C}_{2,2} = \mathbf{A}_{2,1} \mathbf{B}_{1,2} + \mathbf{A}_{2,2} \mathbf{B}_{2,2}
$$

Strassen 算法通过定义中间变量减少乘法运算（**7次乘法运算**）

$$
\begin{align}
\mathbf{M}_{1} &:= (\mathbf{A}_{1,1} + \mathbf{A}_{2,2}) (\mathbf{B}_{1,1} + \mathbf{B}_{2,2})\\
\mathbf{M}_{2} &:= (\mathbf{A}_{2,1} + \mathbf{A}_{2,2}) \mathbf{B}_{1,1}\\
\mathbf{M}_{3} &:= \mathbf{A}_{1,1} (\mathbf{B}_{1,2} - \mathbf{B}_{2,2})\\
\mathbf{M}_{4} &:= \mathbf{A}_{2,2} (\mathbf{B}_{2,1} - \mathbf{B}_{1,1})\\
\mathbf{M}_{5} &:= (\mathbf{A}_{1,1} + \mathbf{A}_{1,2}) \mathbf{B}_{2,2}\\
\mathbf{M}_{6} &:= (\mathbf{A}_{2,1} - \mathbf{A}_{1,1}) (\mathbf{B}_{1,1} + \mathbf{B}_{1,2})\\
\mathbf{M}_{7} &:= (\mathbf{A}_{1,2} - \mathbf{A}_{2,2}) (\mathbf{B}_{2,1} + \mathbf{B}_{2,2})
\end{align}
$$

则

$$
\begin{align}
\mathbf{C}_{1,1} &= \mathbf{M}_{1} + \mathbf{M}_{4} - \mathbf{M}_{5} + \mathbf{M}_{7}\\
\mathbf{C}_{1,2} &= \mathbf{M}_{3} + \mathbf{M}_{5}\\
\mathbf{C}_{2,1} &= \mathbf{M}_{2} + \mathbf{M}_{4}\\
\mathbf{C}_{2,2} &= \mathbf{M}_{1} - \mathbf{M}_{2} + \mathbf{M}_{3} + \mathbf{M}_{6}
\end{align}
$$

我们还能用**缓存**来提升效率。更快的算法可参考 [Coppersmith–Winograd algorithm](https://en.wikipedia.org/wiki/Coppersmith%E2%80%93Winograd_algorithm#cite_note-coppersmith-1)。

##### Referencs

1. [Strassen algorithm](https://en.wikipedia.org/wiki/Strassen_algorithm)
2. [Cache-oblivious algorithm](https://en.wikipedia.org/wiki/Cache-oblivious_algorithm)

