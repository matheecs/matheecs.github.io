---
layout: post
title: "稀疏矩阵表示方法"
categories: study
author: "Jixiang Zhang"
---

SparseMatrix

<p align="center">
  <img src="/images/SparseMatrix.png" width="500"/>
</p>

```text
0	3	0	0	0
22	0	0	0	17
7	5	0	1	0
0	0	0	0	0
0	0	14	0	8
```

Column major representation

```text
Values:	        22	7	3	5	14	1	17	8
InnerIndices:	1	2	0	2	4	2	1	4
OuterStarts:	0	2	4	5	6	8
```

Values 存储非零元素值；InnerIndices 值所在行数；OuterStarts 表示第i列有效元素从第InnerIndices[i]行开始，OuterStarts[end] 表示非零元素的总数。

**Ref**

1. [稀疏矩阵（sparse matrix）的基本数据结构实现](https://zhuanlan.zhihu.com/p/22711401)
2. [Eigen: Sparse matrix manipulations](https://eigen.tuxfamily.org/dox/group__TutorialSparse.html)