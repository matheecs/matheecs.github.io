---
layout: post
title: "非线性最小二乘"
categories: study
author: "Jixiang Zhang"
---

Google 的 Ceres 优化库的名称来源于 Gauss 用最小二乘发现的谷神星。该库用于求解以下模型：

$$
\begin{split}\min_{\mathbf{x}} &\quad \frac{1}{2}\sum_{i} \rho_i\left(\left\|f_i\left(x_{i_1}, ... ,x_{i_k}\right)\right\|^2\right) \\
\text{s.t.} &\quad l_j \le x_j \le u_j\end{split}
$$

- `ResidualBlock` $\rho_i\left(\left\|f_i\left(x_{i_1}, ... ,x_{i_k}\right)\right\|^2\right)$
- `CostFunction` $f_i(\cdot)$
- `ParameterBlock` $\left[x_{i_1},... , x_{i_k}\right]$
- `LossFunction` $\rho_i$ 是用于减小外点干扰的标量函数

#### 简单例子

$$
\frac{1}{2}(10 -x)^2.
$$

第一步定义 functor $f(x) = 10 - x$，特别需要注意 `operator()` 函数：

```c++
struct CostFunctor {
   template <typename T>
   bool operator()(const T* const x, T* residual) const {
     residual[0] = T(10.0) - x[0];
     return true;
   }
};
```

第二步构建最小二乘问题，并求解：

```c++
int main(int argc, char** argv) {
  google::InitGoogleLogging(argv[0]);

  // The variable to solve for with its initial value.
  double initial_x = 5.0;
  double x = initial_x;

  // Build the problem.
  Problem problem;

  // Set up the only cost function (also known as residual). This uses
  // auto-differentiation to obtain the derivative (jacobian).
  CostFunction* cost_function =
      new AutoDiffCostFunction<CostFunctor, 1, 1>(new CostFunctor);
  problem.AddResidualBlock(cost_function, NULL, &x);

  // Run the solver!
  Solver::Options options;
  options.linear_solver_type = ceres::DENSE_QR;
  options.minimizer_progress_to_stdout = true;
  Solver::Summary summary;
  Solve(options, &problem, &summary);

  std::cout << summary.BriefReport() << "\n";
  std::cout << "x : " << initial_x
            << " -> " << x << "\n";
  return 0;
}
```

#### 求导

- 数值求导
- 解析求导

#### References

[Non-linear Least Squares](http://ceres-solver.org/nnls_tutorial.html)

最小二乘问题属于数值线性代数的四大矩阵计算问题之一，另外三个是解方程组、特征值问题和奇异值分解。

**敏度分析**：研究计算问题的原始数据有微小的变化将会引起解的多大变化。病态与否由问题本身属性决定，与采用的计算方法无关。

**误差分析**是衡量算法优劣的重要标志。

Levenberg 把梯度下降法和 G-N 法融合得到新的迭代方式

$$
x_{k+1}=x_k-(J^TJ+\lambda I)^{-1}J^Tf
$$

Marquardt 把上式中的单位矩阵替换为 Hessian 矩阵的对角线

$$
x_{k+1}=x_k-(J^TJ+\lambda diag(J^TJ))^{-1}J^Tf
$$
