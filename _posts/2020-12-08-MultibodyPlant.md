---
layout: post
title: "Drake MultibodyPlant"
categories: study
author: "Jixiang Zhang"
---

MultibodyPlant 表示互连的多体物理系统。`model_instance_name[i]` 表示所有模型实例（model instances），且 `world_model_instance()` 和 `default_model_instance()` 一直都会存在。

MultibodyPlant 提供的 API 接口：
- Ports
- Construction
- Geometry
- Contact modeling
- State access and modification
- Parameters
- Free bodies
- Kinematics and dynamics
- System matrices
- Introspection

### Model Instances

### System dynamics

### Loading models from SDF files

### Working with SceneGraph

#### Adding a MultibodyPlant connected to a SceneGraph to your Diagram
#### Registering geometry with a SceneGraph
#### Accessing point contact parameters

### Working with MultibodyElement parameters
### Adding modeling elements
### Modeling contact
### Energy and Power
### Finalize() stage

### References
- [Drake Concepts](https://drake.guzhaoyuan.com/introduction/drake-concept)
- [Drake Multibody](https://drake.guzhaoyuan.com/introduction/drake-multibody)
- [Drake's documentation of Multibody](https://drake.mit.edu/doxygen_cxx/classdrake_1_1multibody_1_1_multibody_plant.html#details)
- [Featherstone 2008] Featherstone, R., 2008. Rigid body dynamics algorithms. Springer.
- [Jain 2010] Jain, A., 2010. Robot and multibody dynamics: analysis and algorithms. Springer Science & Business Media.
- [Seth 2010] Seth, A., Sherman, M., Eastman, P. and Delp, S., 2010. Minimal formulation of joint motion for biomechanisms. Nonlinear dynamics, 62(1), pp.291-303.