---
layout: post
title: "Drake 使用经验"
categories: study
author: "Jixiang Zhang"
---

Drake 用 Bazel 作为脚手架，搭建了一个面向机器人领域的建模、仿真、优化系统。系统子模块：

- `common/`
  - common
  - proto
  - schema
  - test_utilities
  - trajectories
  - yaml
- `geometry/`
  - geometry
  - proximity
  - query_results
  - render
    - shaders
  - test_utilities
- `lcm/`
  - lcm
- `manipulation/`
  - kinova_jaco
  - kuka_iiwa
  - perception
  - planner
  - schunk_wsg
  - util
- `math/`
  - math
- `multibody/`
  - acrobot
  - free_body
  - inclined_plane
  - kuka_iiwa_robot
  - mass_damper_spring
  - pendulum
  - constraint
  - contact_solvers
  - hydroelastics
  - inverse_kinematics
  - math
  - optimization
  - parsing
  - plant
  - tree
  - triangle_quadrature
- `perception/`
  - perception
- `solvers/`
  - solvers
  - fbstab
    - components
  - test_utilities
- `systems/`
  - analysis
    - test_utilities
  - controllers
    - test_utilities
  - estimators
  - framework
    - test_utilities
  - lcm
  - optimization
  - primitives
  - rendering
  - sensors
  - trajectory_optimization

### [MultibodyPlant](https://drake.mit.edu/doxygen_cxx/classdrake_1_1multibody_1_1_multibody_plant.html)

- Input and output ports
- Construction
- Geometry
- State accessors and mutators
- Working with free bodies
- Kinematic and dynamic computations
- System matrix computations
- Introspection