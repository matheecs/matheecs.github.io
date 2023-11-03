---
layout: post
title: "Effective C++目录"
categories: study
author: "Jixiang Zhang"
---

##### 让自己习惯C++

- 视 C++ 为一个语言联邦
- 尽量以 const，enum，inline 替换 #define
- 尽可能使用 const
- 确定对象被使用前已先被初始化

##### 构造/析构/赋值运算

- 了解 C++ 默默编写并调用哪些函数
- 若不想使用编译器自动生成的函数，就该明确拒绝
- 为多态基类声明 virtual 析构函数
- 别让异常逃离析构函数
- 绝不在构造和析构过程中调用 virtual 函数
- 令 operator= 返回一个 *reference to* ***this**
- 在 operator= 中处理“自我赋值”
- 复制对象时勿忘其每一个成分

##### 资源管理

- 以对象管理资源
- 在资源管理类中小心 *copying* 行为
- 在资源管理类中提供对原始资源的访问
- 成对使用 new 和 delete 时要采取相同形式
- 以独立语句将 new**ed** 对象置入智能指针

##### 设计与声明

- 让接口容易被正确使用，不易被误用
- 设计 class 犹如设计 type
- 宁以 pass-by-reference-to-const 替换 pass-to-value
- 必须返回对象时，别妄想返回其 reference
- 将成员变量声明为 private
- 宁以 non-member、non-friend 替换 member 函数
- 若所有参数皆需类型转换，请为此采用 non-member 函数
- 考虑写出一个不抛异常的 swap 函数

##### 实现

- 尽可能延后变量定义式的出现时间
- 尽量少做转型动作
- 避免返回 handles 指向对象内部成分
- 为“异常安全”而努力是值得的
- 透彻了解 inlining 的里里外外
- 将文件间的编译依存关系降至最低

##### 继承与面向对象设计

- 确定你的 public 继承塑模出 **is-a** 关系
- 避免遮掩继承而来的名称
- 区分接口继承和实现继承
- 考虑 virtual 函数以外的其他选择
- 绝不重新定义继承而来的 non-virtual 函数
- 绝不重新定义继承而来的缺省参数值
- 通过复合塑模出 **has-a** 或“根据某物实现出”
- 明智而审慎地使用 private 继承
- 明智而审慎地使用多重继承

##### 模板与泛型编程

- 了解隐式接口和编译期多态
- 了解 typename 的双重意义
- 学习处理模板化基类内的名称
- 将与参数无关的代码抽离 templates
- 运用成员函数模板接受所有兼容类型
- 需要类型转换时请为模板定义非成员函数
- 请使用 traits classes 表现类型信息
- 认识 template 元编程

##### 定制 new 和 delete

- 了解 new-handler 的行为
- 了解 new 和 delete 的合理替换时机
- 编写 new 和 delete 时需固守常规
- 写了 *placement* new 也要写 *placement* delete

##### 杂项讨论

- 不要轻忽编译器的警告
- 让自己熟悉包括 TR1 在内的标准程序库
- 让自己熟悉 Boost
