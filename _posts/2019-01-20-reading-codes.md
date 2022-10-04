---
layout: post
title: "C++笔记"
categories: study
author: "Jixiang Zhang"
---

## 参考网址

- [cplusplus.com](http://www.cplusplus.com)
- [C++ reference](https://en.cppreference.com/w/)

## C++代码优化策略《C++性能优化指南》

1. 用好的编译器并用好编译器
2. 使用更好的算法
3. 使用更好的库
4. 减少内存分配和复制
5. 移除计算
6. 使用更好的数据结构
7. 提高并发性
8. 优化内存管理

## 阅读项目

1. [**manif**](https://github.com/artivis/manif) & **Eigen**
2. [**RoboRTS**](https://github.com/RoboMaster/RoboRTS)
3. [**S-MSCKF**](https://github.com/KumarRobotics/msckf_vio)
4. [**Caffe**](https://github.com/BVLC/caffe)
5. [**ORB-SLAM2**](https://github.com/raulmur/ORB_SLAM2)

## 语法笔记

1. Type alias, alias template (since C++11)

   ```c++
   using identifier attr(optional) = type-id ;
   ```

2. `#`在宏展开的时候会将后面的参数替换成字符串。

3. `##`将前后两个的单词拼接在一起。

4. std::thread

   ```c++
   void Command();
   ...
   auto command_thread= std::thread(Command);
   ```

5. std::enable_shared_from_this 能让其一个对象（假设其名为 t ，且已被一个 std::shared_ptr 对象 pt 管理）安全地生成其他额外的 std::shared_ptr 实例（假设名为 pt1, pt2, ... ） ，它们与 pt 共享对象 t 的所有权。

6. Writing generic code

   ```c++
   #include <iostream>
   #include <manif/manif.h>
   
   using namespace manif;
   
   template <typename Derived>
   void print(const LieGroupBase<Derived>& g)
   {
     std::cout << "Group degrees of freedom : " << g::DoF << std::endl;
     std::cout << "Group underlying representation vector size : " << g::RepSize << std::endl;
     std::cout << "Current values : " << g << std::endl;
   }
   
   int main()
   {
     SE2d p_2d;
     print(p_2d);
   
     SE3d p_3d;
     print(p_3d);
   }
   ```

   ```c++
   #include <iostream>
   #include <manif/manif.h>
   
   using namespace manif;
   
   template <typename DerivedA, typename DerivedB>
   void ominusSquaredNorm(const LieGroupBase<DerivedA>& g0, const LieGroupBase<DerivedB>& g1)
   {
     return (g0-g1).coeffs().squaredNorm();
   }
   ```

7. `std::make_shared` vs `std::shared_ptr`

8. `()`不可少

   ```c++
   auto chassis_executor = std::make_shared<roborts_decision::ChassisExecutor>();
   ```
