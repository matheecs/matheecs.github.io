---
layout: post
title: "飞行机器人视觉导航笔记"
categories: study
author: "Jixiang Zhang"
---

Homepage: [Visual Navigation for Flying Robots](https://vision.in.tum.de/teaching/ss2013/visnav2013)

Lectures: [visnav2013lecturenotes](https://vision.in.tum.de/_media/teaching/ss2013/visnav2013/visnav2013lecturenotes.pdf)

##### Research Goal

> Apply solutions from computer vision to real-world problems in robotics.

##### Course Material

1. Probabilistic Robotics
2. Computer Vision: Algorithms and Applications

##### Lecture Plan

Basics on Mobile Robotics $\to$ Camera-based Localization and Mapping $\to$ Advanced Topics

##### Safety Warning!!!

1. Quadrocopters are dangerous objects
2. Read the instructions carefully before you start
3. Always use the protective hull
4. If somebody gets injured, report to us so that we can improve safety guidelines
5. If something gets damaged, report it to us so that we can fix it
6. NEVER TOUCH THE PROPELLORS
7. DO NOT TRY TO CATCH THE QUADROCOPTER WHEN IT FAILS – LET IT FALL/CRASH!

##### Robot Design

Imagine that we want to build a robot that has to perform navigation tasks...

How would you tackle this?
- What hardware would you choose?

- What software architecture would you choose?

##### Robot Hardware/Components

1. Sensors
2. Actuators
3. Control Unit/Software

##### Software Architecture

##### **Computer Program $\neq$ Robot Program**

1. Classical robotics (Computer Programming Method, But **A Robot IS NOT A Computrer**)
   $$
   Sense \to Plan \to Act
   $$

2. Reactive paradigms (Rodney Brooks 1986)
   $$
   Sense \to Act
   $$
   Such as Roomba Robot. ![](/images/bbr.jpg)

3. Hybrid approaches

4. Current trends

##### Design Steps [jones1999mobile]

1. What is the robot supposed to do?
2. What is the simplest way to accomplish the task?
3. What mechanical platform is needed?
4. What information does the robot need?
5. What sensors can supply this information most effectively?
6. How can the problem be **decomposed** into **behaviors**?

##### Best Practices for Robot Architectures

- Modular
- Robust
- De-centralized
- Facilitate software re-use
- Hardware and software abstraction
- Provide introspection
- Data logging and playback
- Easy to learn and to extend

##### Robotic Middleware

- Provides infrastructure
- Communication between modules
- Data logging facilities
- Tools for visualization
- Several systems available 
  - Open-source: ROS (Robot Operating System), Player/Stage, CARMEN, YARP, OROCOS
  - Closed-source: Microsoft Robotics Studio

##### Communication Paradigms

- Message-based communication
- Direct (shared) memory access

##### Forms of Communication

- Push
- Pull
- Publisher/subscriber
- Publish to *blackboard*
- Remote procedure calls / service calls
- Preemptive tasks / actions

##### Useful Tools (ROS)

- roscreate-pkg
- rosmake
- roscore
- rosnode list/info
- rostopic list/echo
- rosbag record/play
- rosrun

##### Geometric Primitives in 2D

Line joining two points
$$
\tilde l = \tilde x_1 \times \tilde x_2
$$
Intersection point of two lines
$$
\tilde x = \tilde l_1 \times \tilde l_2
$$

##### Geometric Primitives in 3D

- 3D point
  $$
  x =
  \begin{bmatrix}
  x\\
  y\\
  z\\
  \end{bmatrix}
  \in
  \mathbf{R^3}
  $$










- Augmented vector
  $$
  \bar x =
  \begin{bmatrix}
  x\\
  y\\
  z\\
  1
  \end{bmatrix}
  \in
  \mathbf{R^4}
  $$










- Homogeneous coordinates
  $$
  \tilde x =
  \begin{bmatrix}
  \tilde x\\
  \tilde y\\
  \tilde z\\
  \tilde w
  \end{bmatrix}
  \in
  \mathbf{P^3}
  $$










##### Scientific Research

**Be Creative & Do Research** $\to$ Write and  Submit Paper $\to$ Prepare Talk/Poster $\to$ Present Work at Conference $\to$ Talk with People & Get Inspired $\to$ ...



![](/images/state.jpg)



##### Sensor Model

$$
z = h(x)
$$

Goal: Infer the state of the world from sensor readings
$$
x=h^{-1}(x)
$$

##### Motion Model

$$
x^{\prime}=g(x,u)
$$

![](/images/control.jpg)

##### Assumptions of Cascaded Control

- Dynamics of inner loops is so fast that it is not visible from outer loops
- Dynamics of outer loops is so slow that it appears as static to the inner loops

Example

1. Motor control happens on motor boards (controls every motor tick)
2. Attitude control implemented on microcontroller with hard real-time (at 1000 Hz)
3. Position control (at 10 – 250 Hz)
4. Trajectory (waypoint) control (at 0.1 – 1 Hz)

##### Smith Predictor

- Allows for higher gains

- Requires (accurate) model of plant

Why is this *unrealistic* in practice?

#### Mechanical Equivalent

> PD Control is equivalent to adding spring-dampers between the desired values and the current position

![](/images/PD.jpg)

##### Advanced Control Techniques

- Adaptive control
- Robust control
- Optimal control
- Linear-quadratic regulator (LQR)
- Reinforcement learning
- Inverse reinforcement learning
- ... and many more



##### Robust Error Metrics

1. Sum of Squared Differences (SSD)
2. Sum of absolute differences (SAD, L1 norm)
3. Sum of truncated errors
4. Geman-McClure



##### Exposure Differences？



##### Ideas for Your Mini-Project

- Person following (colored shirt or wearing a marker)
- Flying camera for taking group pictures (possibly using the OpenCV face detector)
- Fly through a hula hoop (brightly colored, white background)
- Navigate through a door (brightly colored)
- Navigate from one room to another (using ground markers)
- Avoid obstacles using optical flow
- Landing on a marked spot/moving target
- **Your own idea here – be creative!**
- ...

##### Four Important SfM Problems

- Camera calibration / resection

  Known 3D points, observe corresponding 2D points, compute camera pose

- Point triangulation

  Known camera poses, observe 2D point correspondences, compute 3D point

- Motion estimation

  Observe 2D point correspondences, compute camera pose (up to scale)

- Bundle adjustment / visual SLAM

  Observe 2D point correspondences, compute camera pose and 3D points (up to scale)

##### SVD



##### RANSAC

**Goal**: Robustly fit a model to a data set   which contains outliers
**Algorithm**:

1. Randomly select a (minimal) subset
2. Instantiate the model from it
3. Using this model, classify the all data points as inliers or outliers
4. Repeat 1-3 for N iterations
5. Select the largest inlier set, and re-estimate the model from all points in this set

##### Derivatives of the Error Terms

Jacobian is **sparse**
$$
J_{ij}(\mathbf{x})=(\mathbf{0} \, \cdots \, \dfrac{\partial{e_{ij}(\mathbf{x})}}{\partial{\mathbf{c}}_i} \, \cdots \,\dfrac{\partial{e_{ij}(\mathbf{x})}}{\partial{\mathbf{c}}_j} \, \cdots \, 0)
$$
We have to solve
$$
H\Delta\mathbf{x}=-\mathbf{b}
$$
Hessian is

- positive semi-definit
- symmetric
- sparse

##### Motion Planning Sub-Problems

1. C-Space discretization (generating a graph / roadmap) 
2. Searchalgorithms (Dijkstra’s algorithm, A*, ...) 
3. Re-planning (D*, ...) 
4. Path tracking (PID control, potential fields, funnels, ...) 

##### Navigation with Funnels



##### Motion Planning in ROS

- Executive: state machine (**move_base**)
- Global costmap: grid with inflation (**costmap_2d**)
- Global path planner: Dijkstra (Dijkstra, **navfn**)
- Local costmap (**costmap_2d**)
- Local planner: Dynamic window approach (**base_local_planner**)

##### Information Theory

> **Entropy** is a general measure for the uncertainty of a probability distribution