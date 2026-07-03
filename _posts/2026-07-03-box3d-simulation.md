---
layout: post
title: "[译] Box3D 仿真"
categories: graphics
author: "Jixiang Zhang"
---

本文翻译自 Box3D 文档：[Simulation](https://github.com/erincatto/box3d/blob/main/docs/simulation.md)。Box3D 采用 [MIT License](https://github.com/erincatto/box3d/blob/main/LICENSE)。

---

# 仿真

刚体仿真是 Box3D 的核心功能。它是 Box3D 中最复杂的部分，也很可能是你与 Box3D 交互最多的部分。仿真层构建在基础层和碰撞层之上，因此在阅读本节之前，你最好已经对这些层有一定了解。

刚体仿真包含：

- 世界（worlds）
- 刚体（bodies）
- 形状（shapes）
- 接触（contacts）
- 关节（joints）
- 事件（events）

这些对象之间存在许多依赖关系，因此很难在不引用其他对象的情况下单独描述其中一个。下面的内容中可能会出现尚未正式说明的对象。因此，你可以先快速浏览本节，再进行细读。

## Id

Box3D 提供 C 接口。在典型的 C/C++ 库中，当你创建一个生命周期较长的对象时，通常会保留指向该对象的指针或智能指针。

Box3D 的方式不同。创建对象时，你得到的不是指针，而是一个 *id*。这个 *id* 充当 [handle](https://en.wikipedia.org/wiki/Handle_(computing))，有助于避免 [dangling pointer](https://en.wikipedia.org/wiki/Dangling_pointer) 问题。

这也允许 Box3D 在内部采用 [data-oriented design](https://en.wikipedia.org/wiki/Data-oriented_design)。这能显著减少缓存未命中，并允许进行 [SIMD](https://en.wikipedia.org/wiki/Single_instruction,_multiple_data) 优化。

因此，你会处理 `b3WorldId`、`b3BodyId` 等类型。它们是很小的不透明结构体，可以像指针一样按值传递。Box3D 的创建函数返回 id，作用于 Box3D 对象的函数也接收 id。

```c
b3BodyId myBodyId = b3CreateBody(myWorldId, &myBodyDef);
```

Box3D 提供了检查 id 是否有效的函数。如果你使用无效 id，Box3D 函数会触发断言。相比使用悬垂指针，这使调试更容易。

```c
if (b3Body_IsValid(myBodyId) == false)
{
    // oops
}
```

空 id 可以通过几种方式建立。你可以使用预定义常量，也可以使用零初始化。

```c
b3BodyId myNullBodyId = b3_nullBodyId;
b3BodyId otherNullBodyId = {0};
```

你可以使用一些辅助宏测试 id 是否为空：

```c
if (B3_IS_NULL(myBodyId))
{
    // do something
}
```

```c
if (B3_IS_NON_NULL(myShapeId))
{
    // do something
}
```

## World

Box3D 的世界包含刚体和关节。它管理仿真的所有方面，并允许进行异步查询，例如 AABB 查询和射线投射。你与 Box3D 的大多数交互都会通过一个 world 对象完成，并使用 `b3WorldId`。

### World Definition

世界通过一个 *definition* 结构创建。这是一个临时结构，可用于配置世界创建时的选项。你**必须**使用 `b3DefaultWorldDef()` 初始化世界定义。

```c
b3WorldDef worldDef = b3DefaultWorldDef();
```

世界定义有许多选项，但多数情况下使用默认值即可。你可能希望设置重力：

```c
// +Y is up; default is {0, -10, 0}
worldDef.gravity = (b3Vec3){0.0f, -10.0f, 0.0f};
```

Box3D 没有内建的“上方向”概念。重力只是一个在每个步进中施加到所有动态刚体上的 `b3Vec3`。

如果你的游戏不需要休眠，可以完全禁用休眠以获得性能提升：

```c
worldDef.enableSleep = false;
```

你也可以配置多线程以提升性能：

```c
worldDef.workerCount = 4;
worldDef.enqueueTask = myAddTaskFunction;
worldDef.finishTask = myFinishTaskFunction;
worldDef.userTaskContext = &myTaskSystem;
```

多线程不是必需的，但它可以显著提升性能。

### World Lifetime

创建世界需要使用世界定义。

```c
b3WorldId myWorldId = b3CreateWorld(&worldDef);

// ... do stuff ...

b3DestroyWorld(myWorldId);

// Nullify id for safety
myWorldId = b3_nullWorldId;
```

你最多可以创建 128 个世界。这些世界彼此不交互，并且可以并行仿真。

销毁一个世界时，其中的每个刚体、形状和关节也会被销毁。这比逐个销毁对象快得多。

### Simulation

世界用于驱动仿真。你需要指定时间步长和子步数量。例如：

```c
float timeStep = 1.0f / 60.0f;
int subStepCount = 4;
b3World_Step(myWorldId, timeStep, subStepCount);
```

完成时间步进后，你可以检查刚体和关节的信息。最常见的做法是从刚体获取位置，以更新游戏对象并渲染它们。更优化的方式是使用 `b3World_GetBodyEvents()`。

你可以在游戏循环中的任意位置执行时间步进，但必须注意操作顺序。例如，如果希望在当前帧获得新刚体的碰撞结果，就必须在时间步进之前创建这些刚体。

你应该使用固定时间步。较大的时间步可以改善低帧率场景下的性能，但通常时间步应为 1/30 秒（30Hz）或更小。1/60 秒（60Hz）的时间步通常能提供高质量仿真。

子步数量用于提高精度。通过子步进，求解器会把时间划分为更小的增量，使刚体每次移动较小的距离。这允许关节和接触以更精细的方式响应。推荐的子步数量是 4。不过，增加子步数量可以进一步提高精度。例如，较长的关节链在更多子步下拉伸更少。

## Rigid Bodies

刚体，简称 *body*，具有位置和速度。你可以对刚体施加力、力矩和冲量。刚体可以是静态、运动学或动态的。

三维刚体有 6 个自由度：3 个平移自由度（沿 x、y、z 的位置）和 3 个旋转自由度（绕 x、y、z 轴旋转）。姿态用四元数（`b3Quat`）表示，角速度和力矩都是 `b3Vec3` 量。

下面是刚体类型的定义。

### Body Types

`b3_staticBody`：静态刚体在仿真中不会运动，其行为类似于具有无限质量的物体。在内部，Box3D 将其质量和质量倒数存为零。静态刚体速度为零。静态刚体不会与其他静态刚体或运动学刚体碰撞。

`b3_kinematicBody`：运动学刚体根据自身速度在仿真中运动。运动学刚体不响应力。运动学刚体通过设置速度来移动。它的行为类似于具有无限质量的物体，但 Box3D 同样将其质量和质量倒数存为零。运动学刚体不会与其他运动学刚体或静态刚体碰撞。通常，如果你希望某个形状被动画驱动，并且不受力或碰撞影响，就应使用运动学刚体。

`b3_dynamicBody`：动态刚体被完整仿真，并根据力和力矩运动。动态刚体可以与所有刚体类型碰撞。动态刚体总是具有有限且非零的质量。

> **注意**：通常不应在创建之后设置刚体的变换。Box3D 会把这种操作视为瞬移，这可能导致不理想的行为或性能问题。

刚体携带形状并在世界中移动它们。Box3D 中的 body 始终是刚体。这意味着附着到同一刚体上的两个形状永远不会相对运动，并且同一刚体上的形状不会彼此碰撞。

形状具有碰撞几何和密度。通常，刚体的质量属性来自其附着的形状。不过，你也可以在刚体构造后覆盖质量属性。

你通常会保存所创建刚体的 id。这样便可以查询刚体位置，用于更新图形实体的位置。你也应保存刚体 id，以便在使用结束时销毁它们。

### Body Definition

创建刚体之前，必须先创建刚体定义（`b3BodyDef`）。刚体定义保存正确创建和初始化刚体所需的数据。

由于 Box3D 使用 C API，因此提供了一个创建默认刚体定义的函数。

```c
b3BodyDef myBodyDef = b3DefaultBodyDef();
```

这能确保刚体定义有效，并且这种初始化是**强制性的**。

Box3D 会从刚体定义中复制数据；它不会保存指向刚体定义的指针。这意味着你可以复用一个刚体定义来创建多个刚体。

### Body Type

如前所述，刚体有三种类型：静态、运动学和动态。`b3_staticBody` 是默认值。你应该在创建时确定刚体类型，因为之后修改刚体类型代价较高。

```c
b3BodyDef bodyDef = b3DefaultBodyDef();
bodyDef.type = b3_dynamicBody;
```

### Position and Orientation

你可以在刚体定义中初始化刚体的位置和姿态。相比在世界原点创建刚体再移动它，这种方式性能好得多。

> **注意**：不要先在原点创建刚体再移动它。如果你在原点创建多个刚体，性能会受到影响。

刚体有两个主要关注点。第一个是刚体原点。形状和关节相对于刚体原点附着。第二个是质心。质心由所附着形状的质量分布决定，也可以使用 `b3MassData` 显式设置。Box3D 内部的许多计算使用质心位置。例如，刚体存储的是质心的线速度，而不是刚体原点的线速度。

<p align="center">
  <img src="https://raw.githubusercontent.com/erincatto/box3d/main/docs/images/center_of_mass.svg" alt="Body Origin and Center of Mass" />
</p>

构建刚体定义时，你可能并不知道质心位于何处。因此，你指定的是刚体原点的位置。你也可以用 `b3Quat` 指定刚体姿态。如果之后修改刚体的质量属性，质心可能会在刚体上移动，但原点位置和姿态不会改变，附着的形状和关节也不会移动。

```c
b3BodyDef bodyDef = b3DefaultBodyDef();
bodyDef.position = (b3Vec3){0.0f, 2.0f, 0.0f};

// Rotate 45 degrees about the Y axis
b3Vec3 axis = {0.0f, 1.0f, 0.0f};
bodyDef.rotation = b3MakeQuatFromAxisAngle(axis, 0.25f * B3_PI);
```

从四元数读回轴和角度：

```c
b3Quat q = b3Body_GetRotation(myBodyId);
float radians;
b3Vec3 axis = b3GetAxisAngle(&radians, q);
```

刚体是一个参考系。你可以在该参考系中定义形状和关节。这些形状和关节锚点在刚体局部坐标系中永远不会移动。

### Damping

阻尼用于降低刚体在世界坐标中的速度。阻尼不同于摩擦，因为摩擦只在接触时发生。阻尼不是摩擦的替代品，两种效应会共同使用。

阻尼参数非负。通常你会使用 0 到 1 之间的阻尼值。线性阻尼一般不理想，因为它会使刚体看起来像在漂浮。

```c
bodyDef.linearDamping = 0.0f;
bodyDef.angularDamping = 0.1f;
```

为提高性能，阻尼是近似计算的。当阻尼值较小时，阻尼效应基本与时间步无关。当阻尼值较大时，阻尼效应会随时间步变化。如果你使用固定时间步（推荐），这通常不是问题。

### Gravity Scale

你可以使用重力缩放来调整单个刚体受到的重力。不过需要谨慎，过大的重力幅值会降低稳定性。

```c
// Set the gravity scale to zero so this body will float
bodyDef.gravityScale = 0.0f;
```

### Sleep Parameters

当刚体静止时，Box3D 可以停止仿真它以节省 CPU 时间。这称为 *sleeping*。当 Box3D 判定一个刚体或一组刚体已经静止时，该刚体会进入休眠状态，此时 CPU 开销很低。如果一个清醒刚体与休眠刚体碰撞，休眠刚体会被唤醒。若附着到刚体的关节或接触被销毁，刚体也会被唤醒。你还可以手动唤醒刚体。

刚体定义允许你指定刚体是否可以休眠，以及刚体创建时是否处于清醒状态。

```c
bodyDef.enableSleep = true;
bodyDef.isAwake = true;
```

如果 `enableSleep` 为 false，`isAwake` 标志会被忽略。

### Motion Locks

在三维中，你有时希望把刚体限制在其六个自由度的某个子集上。例如，门铰链可能只需要绕 Y 轴旋转，平台可能只需要沿 X 轴平移。Box3D 为此提供 `b3MotionLocks`：

```c
b3MotionLocks locks = {0};
locks.linearY  = true;   // prevent translation along Y
locks.angularX = true;   // prevent rotation about X
locks.angularZ = true;   // prevent rotation about Z
bodyDef.motionLocks = locks;
```

你也可以在创建后更新这些锁：

```c
b3Body_SetMotionLocks(myBodyId, locks);
b3MotionLocks current = b3Body_GetMotionLocks(myBodyId);
```

锁定全部三个角轴等价于完全固定旋转。

### Bullets

游戏仿真通常生成一系列以某个帧率播放的变换。这称为离散仿真。在离散仿真中，刚体可能在一个时间步内移动很大距离。如果物理引擎没有处理这种大位移，就可能看到某些对象错误地穿过彼此。这种现象称为 *tunneling*。

<p align="center">
  <img src="https://raw.githubusercontent.com/erincatto/box3d/main/docs/images/tunneling1.svg" alt="Tunneling 1" />
</p>

<p align="center">
  <img src="https://raw.githubusercontent.com/erincatto/box3d/main/docs/images/tunneling2.svg" alt="Tunneling 2" />
</p>

默认情况下，Box3D 使用连续碰撞检测（continuous collision detection, CCD）防止动态刚体穿透静态刚体。这通过从旧位置到新位置扫掠形状完成。引擎会在扫掠过程中查找新碰撞，并计算这些碰撞的撞击时间（time of impact, TOI）。在时间步末尾，刚体会被移动到它们的第一个 TOI。

<p align="center">
  <img src="https://raw.githubusercontent.com/erincatto/box3d/main/docs/images/captured_toi.svg" alt="Captured TOI" />
</p>

<p align="center">
  <img src="https://raw.githubusercontent.com/erincatto/box3d/main/docs/images/missed_toi.svg" alt="Missed TOI" />
</p>

通常动态刚体之间不使用 CCD，这是为了保持合理性能。在某些游戏场景中，你需要动态刚体使用 CCD。例如，你可能希望向一堆动态砖块发射高速子弹。如果没有 CCD，子弹可能穿过砖块。

Box3D 中快速运动的对象可以配置为 *bullets*。bullet 会与所有刚体类型执行 CCD，但**不会**与其他 bullet 执行 CCD。你应根据游戏设计决定哪些刚体应被视为 bullet。如果决定将某个刚体作为 bullet 处理，使用如下设置：

```c
bodyDef.isBullet = true;
```

bullet 标志只影响动态刚体。bullet 应谨慎使用。

### Disabling

你可能希望创建一个刚体，但不让它参与碰撞或仿真。该状态类似于休眠，但刚体不会被其他刚体唤醒，其形状也不会与任何东西碰撞。这意味着该刚体不会参与碰撞、射线投射等。

你可以创建一个禁用状态的刚体，并在之后启用它。

```c
bodyDef.isEnabled = false;

// Later ...
b3Body_Enable(myBodyId);
```

关节可以连接到禁用的刚体。这些关节不会被仿真。启用刚体时，应注意其关节没有被扭曲。

注意，启用一个刚体的代价几乎与从头创建该刚体一样。因此，不应把刚体禁用用于流式世界。对于流式世界，应使用创建/销毁来节省内存。

### User Data

用户数据是一个 void 指针。它提供一个钩子，用于把应用程序对象链接到刚体。你应一致地为所有刚体用户数据使用同一种对象类型。

```c
bodyDef.userData = &myGameObject;
```

当你从射线投射或事件等查询中收到结果，并希望回到游戏对象时，这很有用。你可以使用 `b3Body_GetUserData()` 从刚体获取用户数据。

### Body Lifetime

刚体通过 world id 创建和销毁。这使世界可以用高效分配器创建刚体，并将刚体加入世界数据结构。

```c
b3BodyId myBodyId = b3CreateBody(myWorldId, &bodyDef);

// ... do stuff ...

b3DestroyBody(myBodyId);

// Nullify body id for safety
myBodyId = b3_nullBodyId;
```

Box3D 不会保留对刚体定义或其中数据的引用（用户数据指针除外）。因此你可以创建临时刚体定义，并复用同一个刚体定义。

Box3D 允许你直接使用 `b3DestroyWorld()` 销毁世界，从而避免逐个销毁刚体；该函数会完成所有清理工作。不过，你仍应注意将应用程序中保留的刚体 id 置空。

销毁刚体时，其附着的形状和关节也会自动销毁。这对形状和关节 id 的管理有重要影响。销毁刚体后，应将这些 id 置空。

### Using a Body

创建刚体后，你可以对其执行许多操作，包括设置质量属性、访问位置和速度、施加力，以及变换点和向量。

### Mass Data

刚体具有质量（标量）、质心（`b3Vec3`）和转动惯量张量（`b3Matrix3`）。对于静态刚体，质量和转动惯量设为零。当所有角运动锁均激活时，转动惯量实际为零。

通常，当形状被添加到刚体上时，刚体的质量属性会自动建立。你也可以在运行时调整刚体质量。通常只有在特殊游戏场景需要改变质量时才这样做。

```c
b3MassData myMassData;
myMassData.mass = 10.0f;
myMassData.center = (b3Vec3){0.0f, 0.0f, 0.0f};
myMassData.inertia = b3Mat3_identity; // b3Matrix3 inertia tensor
b3Body_SetMassData(myBodyId, myMassData);
```

直接设置刚体质量后，你可能希望恢复为由形状决定的质量。可以这样做：

```c
b3Body_ApplyMassFromShapes(myBodyId);
```

可通过以下函数访问刚体的质量数据：

```c
float mass = b3Body_GetMass(myBodyId);
b3Matrix3 inertia = b3Body_GetLocalRotationalInertia(myBodyId);
b3Vec3 localCenter = b3Body_GetLocalCenterOfMass(myBodyId);
b3MassData massData = b3Body_GetMassData(myBodyId);
```

### State Information

刚体状态包含许多方面。你可以通过以下函数访问这些状态数据：

```c
b3Body_SetType(myBodyId, b3_kinematicBody);
b3BodyType bodyType = b3Body_GetType(myBodyId);
b3Body_SetBullet(myBodyId, true);
bool isBullet = b3Body_IsBullet(myBodyId);
b3Body_EnableSleep(myBodyId, false);
bool isSleepEnabled = b3Body_IsSleepEnabled(myBodyId);
b3Body_SetAwake(myBodyId, true);
bool isAwake = b3Body_IsAwake(myBodyId);
b3Body_Disable(myBodyId);
b3Body_Enable(myBodyId);
bool isEnabled = b3Body_IsEnabled(myBodyId);
b3Body_SetMotionLocks(myBodyId, locks);
b3MotionLocks locks = b3Body_GetMotionLocks(myBodyId);
```

更多细节请参阅这些函数的注释。

### Position and Velocity

你可以访问刚体的位置和姿态。这在渲染关联的游戏对象时很常见。你也可以设置位置和姿态，尽管这并不常见，因为通常应由 Box3D 来仿真运动。

```c
b3Body_SetTransform(myBodyId, position, rotation);
b3Transform transform = b3Body_GetTransform(myBodyId);
b3Vec3 position = b3Body_GetPosition(myBodyId);
b3Quat rotation = b3Body_GetRotation(myBodyId);
```

你可以访问局部坐标和世界坐标中的质心位置。Box3D 的内部仿真大量使用质心。不过你通常不需要直接访问它，而是会使用刚体变换。例如，你可能有一个盒形刚体，刚体原点在盒子角点，而质心位于盒子中心。

```c
b3Vec3 worldCenter = b3Body_GetWorldCenterOfMass(myBodyId);
b3Vec3 localCenter = b3Body_GetLocalCenterOfMass(myBodyId);
```

你可以访问线速度和角速度。线速度是质心的速度。角速度是一个 `b3Vec3`，其方向为旋转轴，大小为以弧度每秒计的旋转速率。

```c
b3Vec3 linearVelocity = b3Body_GetLinearVelocity(myBodyId);
b3Vec3 angularVelocity = b3Body_GetAngularVelocity(myBodyId);
```

你可以驱动刚体到特定变换。这对运动学刚体很有用。

```c
b3Pos targetPosition = {42.0f, 0.0f, -100.0f};
b3Quat targetRotation = b3MakeQuatFromAxisAngle(b3Vec3_axisY, B3_PI);
b3WorldTransform target = {targetPosition, targetRotation};
float timeStep = 1.0f / 60.0f;
b3Body_SetTargetTransform(myBodyId, target, timeStep, true);
```

### Forces and Impulses

你可以对刚体施加力、力矩和冲量。施加力或冲量时，可以提供负载施加的世界点。这通常会在质心周围产生力矩。

```c
b3Body_ApplyForce(myBodyId, force, worldPoint, wake);
b3Body_ApplyTorque(myBodyId, torque, wake);
b3Body_ApplyLinearImpulse(myBodyId, linearImpulse, worldPoint, wake);
b3Body_ApplyAngularImpulse(myBodyId, angularImpulse, wake);
```

所有力、力矩和冲量值都是世界空间中的 `b3Vec3`。施加力、力矩或冲量可以选择唤醒刚体。如果不唤醒刚体且刚体处于休眠状态，该力或冲量会被忽略。

你也可以向质心施加力和线冲量，以避免产生旋转。

```c
b3Body_ApplyForceToCenter(myBodyId, force, wake);
b3Body_ApplyLinearImpulseToCenter(myBodyId, linearImpulse, wake);
```

> **注意**：由于 Box3D 使用子步进，不应连续多帧施加稳定冲量。应施加力，Box3D 会将其均匀分布到各个子步中，从而得到更平滑的运动。

### Coordinate Transformations

刚体提供了一些实用函数，用于在局部空间和世界空间之间变换点和向量。如果你不理解这些概念，建议阅读 Jim Van Verth 和 Lars Bishop 的《Essential Mathematics for Games and Interactive Applications》。

```c
b3Vec3 worldPoint = b3Body_GetWorldPoint(myBodyId, localPoint);
b3Vec3 worldVector = b3Body_GetWorldVector(myBodyId, localVector);
b3Vec3 localPoint = b3Body_GetLocalPoint(myBodyId, worldPoint);
b3Vec3 localVector = b3Body_GetLocalVector(myBodyId, worldVector);
```

### Accessing Shapes and Joints

你可以访问刚体上的形状。首先可以获取形状数量。

```c
int shapeCount = b3Body_GetShapeCount(myBodyId);
```

如果某些刚体有许多形状，可以分配一个数组；如果你知道数量有限，也可以使用固定大小数组。

```c
b3ShapeId shapeIds[10];
int returnCount = b3Body_GetShapes(myBodyId, shapeIds, 10);

for (int i = 0; i < returnCount; ++i)
{
    b3ShapeId shapeId = shapeIds[i];

    // do something with shapeId
}
```

你也可以用类似方式获取刚体上的关节数组。

### Body Events

虽然你可以在每个时间步后从所有刚体收集变换，但这效率较低。许多刚体可能因为休眠而没有移动。此外，遍历大量刚体会产生许多缓存未命中。

Box3D 提供 `b3BodyEvents`，你可以在每次调用 `b3World_Step()` 后访问它，以获得刚体运动事件数组。由于这些数据是连续存储的，因此对缓存友好。

```c
b3BodyEvents events = b3World_GetBodyEvents(myWorldId);
for (int i = 0; i < events.moveCount; ++i)
{
    const b3BodyMoveEvent* event = events.moveEvents + i;
    MyGameObject* gameObject = event->userData;
    MoveGameObject(gameObject, event->transform);
    if (event->fellAsleep)
    {
        SleepGameObject(gameObject);
    }
}
```

刚体事件还会指示该刚体是否在当前时间步进入休眠。这可能有助于优化你的应用程序。

## Shapes

一个刚体可以有零个或多个形状。具有多个形状的刚体有时称为 *compound body*。

形状保存以下内容：

- 形状基元
- 密度、摩擦和恢复系数（通过 `baseMaterial`）
- 碰撞过滤标志
- 父刚体 id
- 用户数据
- 传感器标志

这些内容将在后续小节说明。

几何类型（球、胶囊、凸包、网格、高度场）在 `collision.md` 中有详细文档。复合形状在 `compound.md` 中说明。本节覆盖形状生命周期和材料属性。

### Shape Lifetime

形状通过初始化形状定义和形状基元来创建。它们会传给特定形状类型对应的创建函数。

```c
b3ShapeDef shapeDef = b3DefaultShapeDef();
shapeDef.density = 10.0f;
shapeDef.baseMaterial.friction = 0.7f;

b3BoxHull box = b3MakeBoxHull(0.5f, 0.5f, 1.0f);
b3ShapeId myShapeId = b3CreateHullShape(myBodyId, &shapeDef, &box.base);
```

这会创建一个凸包形状并将其附着到刚体上。你不必保存形状 id，因为父刚体销毁时该形状会自动销毁。不过，如果之后计划修改其属性，可能仍希望保存形状 id。

你可以在单个刚体上创建多个形状。它们都可以贡献刚体质量。这些形状永远不会彼此碰撞，并且可以相互重叠。

你可以销毁父刚体上的某个形状。这可用于建模可破坏对象。否则，你可以不管该形状，让刚体销毁时处理附着形状的销毁。

```c
b3DestroyShape(myShapeId, true);
```

第二个参数控制是否立即更新父刚体质量。

密度、摩擦和恢复系数等材料属性与形状关联，而不是与刚体关联。由于一个刚体可以附着多个形状，这允许更丰富的设置。例如，可以让车辆后部更重。

### Density

形状密度用于计算父刚体的质量属性。密度可以为零或正数。通常应为所有形状使用相近的密度，这会改善堆叠稳定性。

设置密度时，刚体质量不会自动调整。必须调用 `b3Body_ApplyMassFromShapes()` 才会发生这种更新。通常应在 `b3ShapeDef` 中建立形状密度，并避免之后修改它，因为这可能代价较高，尤其是在复合刚体上。

```c
b3Shape_SetDensity(myShapeId, 5.0f, true);
b3Body_ApplyMassFromShapes(myBodyId);
```

### Friction

摩擦用于让物体沿彼此真实滑动。Box3D 支持静摩擦和动摩擦，但二者使用同一个参数。Box3D 试图精确模拟摩擦，摩擦强度与法向力成正比。这称为 [Coulomb friction](https://en.wikipedia.org/wiki/Friction)。摩擦参数通常设在 0 到 1 之间，但可以是任意非负值。摩擦值为 0 会关闭摩擦，值为 1 表示摩擦很强。当计算两个形状之间的摩擦力时，Box3D 必须组合两个父形状的摩擦参数。组合使用 [geometric mean](https://en.wikipedia.org/wiki/Geometric_mean)：

```c
float mixedFriction = sqrtf(b3Shape_GetFriction(shapeIdA) * b3Shape_GetFriction(shapeIdB));
```

如果某个形状摩擦为零，则混合摩擦也为零。

摩擦作为形状基础表面材料的一部分存储：

```c
shapeDef.baseMaterial.friction = 0.5f;
```

### Restitution

[Restitution](https://en.wikipedia.org/wiki/Coefficient_of_restitution) 用于让物体弹跳。恢复系数通常设为 0 到 1。考虑把一个球丢到桌面上。值为零表示球不会弹起，这称为 *inelastic* 碰撞。值为一表示球的速度会被精确反射，这称为 *perfectly elastic* 碰撞。恢复系数使用如下公式组合。

```c
float mixedRestitution = b3MaxFloat(b3Shape_GetRestitution(shapeIdA), b3Shape_GetRestitution(shapeIdB));
```

这样组合恢复系数，是为了允许你有一个会弹跳的超级球，而不需要地面也具有弹性。

当一个形状产生多个接触时，恢复系数是近似仿真的。这是因为 Box3D 使用顺序求解器。Box3D 还会在碰撞速度较小时使用非弹性碰撞，以防止抖动。参见 `b3WorldDef::restitutionThreshold`。

恢复系数作为形状基础表面材料的一部分存储：

```c
shapeDef.baseMaterial.restitution = 0.3f;
```

### Friction and Restitution Callbacks

高级用户可以使用 `b3FrictionCallback` 和 `b3RestitutionCallback` 覆盖摩擦与恢复系数的混合方式。这些函数应非常轻量，因为它们会被频繁调用。回调会接收两个摩擦（或恢复）值以及来自各形状表面材料的用户材料 id。

```c
float MyFrictionCallback(float frictionA, uint64_t userMaterialIdA,
                         float frictionB, uint64_t userMaterialIdB)
{
    if (userMaterialIdA > userMaterialIdB)
    {
        return frictionA;
    }

    return frictionB;
}

b3WorldDef worldDef = b3DefaultWorldDef();
worldDef.frictionCallback = MyFrictionCallback;
```

### Filtering

碰撞过滤允许你高效地阻止形状之间发生碰撞。例如，假设你制作一个骑自行车的角色。你希望自行车与地形碰撞、角色与地形碰撞，但不希望角色与自行车碰撞，因为它们必须重叠。Box3D 使用类别、掩码和组支持这种碰撞过滤。

Box3D 支持 64 个碰撞类别（存储为 `uint64_t`）。对每个形状，你可以指定其所属类别，也可以指定它能与哪些其他类别碰撞。例如，在多人游戏中，你可以指定玩家之间不碰撞。与其枚举所有“不应碰撞”的情况，我建议枚举所有“应当碰撞”的情况。这样可以避免陷入 [double negatives](https://en.wikipedia.org/wiki/Double_negative)。你可以使用掩码位指定哪些东西可以碰撞。例如：

```c
enum MyCategories
{
    PLAYER  = 0x00000002,
    MONSTER = 0x00000004,
};

b3ShapeDef playerShapeDef  = b3DefaultShapeDef();
b3ShapeDef monsterShapeDef = b3DefaultShapeDef();
playerShapeDef.filter.categoryBits  = PLAYER;
monsterShapeDef.filter.categoryBits = MONSTER;

// Players collide with monsters, but not with other players
playerShapeDef.filter.maskBits = MONSTER;

// Monsters collide with players and other monsters
monsterShapeDef.filter.maskBits = PLAYER | MONSTER;
```

碰撞发生的规则如下：

```c
uint64_t catA  = shapeA.filter.categoryBits;
uint64_t maskA = shapeA.filter.maskBits;
uint64_t catB  = shapeB.filter.categoryBits;
uint64_t maskB = shapeB.filter.maskBits;

if ((catA & maskB) != 0 && (catB & maskA) != 0)
{
    // shapes can collide
}
```

另一个过滤特性是 *collision groups*。碰撞组允许你指定组索引。所有具有相同组索引的形状可以总是碰撞（正索引）或永不碰撞（负索引）。组索引通常用于某种相关对象，例如车辆部件。下例中，shape1 和 shape2 总是碰撞，而 shape3 和 shape4 永不碰撞。

```c
shape1Def.filter.groupIndex =  2;
shape2Def.filter.groupIndex =  2;
shape3Def.filter.groupIndex = -8;
shape4Def.filter.groupIndex = -8;
```

不同组索引形状之间的碰撞根据类别和掩码位过滤。如果两个形状具有相同的非零组索引，则它会覆盖类别和掩码。碰撞组优先级高于类别和掩码。

注意，Box3D 还会自动执行额外的碰撞过滤：

- 静态刚体上的形状只能与动态刚体碰撞。
- 运动学刚体上的形状只能与动态刚体碰撞。
- 同一刚体上的形状永远不会彼此碰撞。
- 你可以选择启用或禁用由关节连接的刚体之间的碰撞。

有时你可能需要在形状创建之后修改碰撞过滤。你可以使用 `b3Shape_GetFilter()` 和 `b3Shape_SetFilter()` 在已有形状上获取和设置 `b3Filter` 结构。改变过滤代价较高，因为它会导致接触被销毁。

### Sensors

有时游戏逻辑需要知道两个形状何时重叠，但不希望产生碰撞响应。这通过传感器实现。传感器是一种检测重叠但不产生响应的形状。

你可以将任何形状标记为传感器。传感器可以是静态、运动学或动态的。记住，一个刚体可以有多个形状，并且可以混合传感器形状和实体形状。传感器也可以检测其他传感器。传感器形状像普通形状一样具有质量。如果不希望传感器有质量，可以将密度设为零。

```c
b3ShapeDef shapeDef = b3DefaultShapeDef();
shapeDef.isSensor = true;
```

对于传感器和非传感器，还必须启用传感器事件。生成传感器事件有性能成本，因此默认禁用。

```c
shapeDef.enableSensorEvents = true;
```

两个参与的形状都必须把该标志设为 true。这允许游戏使用 `b3Shape_EnableSensorEvents` 禁用特定传感器。

传感器在世界步进末尾处理，并立即生成 begin 和 end 事件。用户操作也可能导致重叠开始或结束，这些会在下一时间步处理。这类操作包括：

- 销毁刚体或形状
- 改变形状过滤
- 禁用或启用刚体
- 设置刚体变换
- 在形状上禁用或启用传感器事件

传感器不会检测在一个时间步内穿过传感器形状的对象。因此，传感器没有连续碰撞检测。如果你有高速移动对象或很小的传感器，应使用射线投射或形状投射检测这些事件。

你可以访问上一世界步中的当前传感器重叠。需要小心，因为某些形状 id 可能由于形状被销毁而无效。使用 `b3Shape_IsValid` 确认重叠形状仍然有效。

```c
// First determine the required array capacity to hold all the overlapping shape ids.
int capacity = b3Shape_GetSensorCapacity(sensorShapeId);
b3ShapeId overlaps[64]; // or dynamically allocate capacity items

// Now get all overlaps and record the actual count
int count = b3Shape_GetSensorData(sensorShapeId, overlaps, capacity);

for (int i = 0; i < count; ++i)
{
    b3ShapeId visitorId = overlaps[i];

    // Ensure the visitorId is valid
    if (b3Shape_IsValid(visitorId) == false)
    {
        continue;
    }

    // process overlap using game logic
}
```

传感器重叠也可以通过事件确定，见下文。

### Sensor Events

每次调用 `b3World_Step()` 后都可以获得传感器事件。传感器事件是获取传感器重叠信息的最佳方式。当某个形状开始与传感器重叠时，会产生事件。

```c
b3SensorEvents sensorEvents = b3World_GetSensorEvents(myWorldId);
for (int i = 0; i < sensorEvents.beginCount; ++i)
{
    b3SensorBeginTouchEvent* beginTouch = sensorEvents.beginEvents + i;
    void* myUserData = b3Shape_GetUserData(beginTouch->visitorShapeId);
    // process begin event
}
```

当某个形状停止与传感器重叠时，也会产生事件。处理 end touch 事件时需要小心，因为它们可能在形状被销毁时产生。应使用 `b3Shape_IsValid` 测试形状 id。

```c
for (int i = 0; i < sensorEvents.endCount; ++i)
{
    b3SensorEndTouchEvent* endTouch = sensorEvents.endEvents + i;
    if (b3Shape_IsValid(endTouch->visitorShapeId))
    {
        void* myUserData = b3Shape_GetUserData(endTouch->visitorShapeId);
        // process end event
    }
}
```

传感器事件应在世界步进之后、其他游戏逻辑之前处理。这有助于避免处理陈旧数据。

只有当形状的 `b3ShapeDef::enableSensorEvents` 设为 true 时，才会为该形状启用传感器事件。

> **注**：形状不能开始或停止作为传感器。这类特性会破坏传感器事件，可能导致游戏逻辑 bug。

## Contacts

接触是 Box3D 为管理成对形状之间碰撞而创建的内部对象。它们是游戏刚体仿真的基础。

### Terminology

接触涉及一些重要术语，需要先回顾。

#### contact point

接触点是两个形状相接触的位置。Box3D 用少量点近似接触。

#### contact normal

接触法线是从一个形状指向另一个形状的单位向量。按照约定，法线从 shapeA 指向 shapeB。

#### contact separation

间隔是穿透的反义。形状重叠时，间隔为负。

#### contact manifold

两个凸形状之间的接触可能生成多个接触点。这些点共享同一法线，因此被组织为一个接触流形，它是连续接触区域的近似。

#### normal impulse

法向力是在接触点施加的、用于防止形状穿透的力。为方便起见，Box3D 使用冲量。法向冲量就是法向力乘以时间步。由于 Box3D 使用子步进，这里指的是子步时间步。

#### tangent impulse

切向力在接触点生成，用于仿真摩擦。为方便起见，它也存储为冲量。

#### contact point id

Box3D 会尝试复用某一时间步的接触冲量结果，作为下一时间步的初始猜测。Box3D 使用接触点 id 在时间步之间匹配接触点。这些 id 包含几何特征索引，有助于区分不同接触点。

#### speculative contact

当两个形状相互接近时，即使它们尚未接触，Box3D 也会创建接触点。这使 Box3D 能够预判碰撞以改善行为。推测接触点具有正间隔。

### Contact Lifetime

当两个形状的 AABB（包围盒）开始重叠时，接触被创建。有时碰撞过滤会阻止接触创建。当 AABB 不再重叠时，接触被销毁。

因此，你可能会意识到，可能存在形状尚未真正接触、只是 AABB 重叠时就创建的接触。没错。这是一个“先有鸡还是先有蛋”的问题：在创建接触对象并分析碰撞之前，我们不知道是否需要接触对象。如果形状没有接触，可以立刻删除接触；也可以等待 AABB 停止重叠。Box3D 采用后一种方式，因为它允许系统缓存信息以提升性能。

### Contact Data

如前所述，接触由 Box3D 自动创建和销毁。接触数据不是由用户创建的。不过，你可以访问接触数据。

你可以从形状或刚体获取接触数据。形状上的接触数据是刚体接触数据的子集。接触数据只会为正在接触的接触返回。未接触的接触对应用程序没有有意义的信息。

接触数据以数组形式返回。因此，你可以先询问某个形状或刚体需要多少数组空间。该数量是保守估计，实际接收的接触数可能少于这个数，但不会更多。

```c
int shapeContactCapacity = b3Shape_GetContactCapacity(myShapeId);
int bodyContactCapacity  = b3Body_GetContactCapacity(myBodyId);
```

你可以分配数组空间以在所有情况下获取全部接触数据，也可以使用固定大小数组获取有限数量的结果。

```c
b3ContactData contactData[10];
int shapeContactCount = b3Shape_GetContactData(myShapeId, contactData, 10);
int bodyContactCount  = b3Body_GetContactData(myBodyId, contactData, 10);
```

`b3ContactData` 包含两个形状 id 和流形数组。

```c
for (int i = 0; i < bodyContactCount; ++i)
{
    b3ContactData* data = contactData + i;
    printf("manifold count = %d\n", data->manifoldCount);
}
```

从形状和刚体获取接触数据不是处理接触数据的最高效方式。应改用接触事件。

### Contact Events

每个世界步进后都可以获得接触事件。与传感器事件类似，应在执行其他游戏逻辑之前检索并处理这些事件。否则可能访问到孤立或无效数据。

你可以通过单个数据结构访问所有接触事件。这比使用 `b3Body_GetContactData()` 等函数高效得多。

```c
b3ContactEvents contactEvents = b3World_GetContactEvents(myWorldId);
```

这些数据都不适用于传感器，因为传感器单独处理。所有事件都至少涉及一个动态刚体。

接触事件有三种。

#### Contact Touch Event

当两个形状开始接触时，会记录 `b3ContactBeginTouchEvent`。

```c
for (int i = 0; i < contactEvents.beginCount; ++i)
{
    b3ContactBeginTouchEvent* beginEvent = contactEvents.beginEvents + i;
    ShapesStartTouching(beginEvent->shapeIdA, beginEvent->shapeIdB);
}
```

当两个形状停止接触时，会记录 `b3ContactEndTouchEvent`。

```c
for (int i = 0; i < contactEvents.endCount; ++i)
{
    b3ContactEndTouchEvent* endEvent = contactEvents.endEvents + i;

    // Use b3Shape_IsValid because a shape may have been destroyed
    if (b3Shape_IsValid(endEvent->shapeIdA) && b3Shape_IsValid(endEvent->shapeIdB))
    {
        ShapesStopTouching(endEvent->shapeIdA, endEvent->shapeIdB);
    }
}
```

类似于 `b3SensorEndTouchEvent`，`b3ContactEndTouchEvent` 可能因用户操作产生，例如销毁刚体或形状。这些事件会在下一次 `b3World_Step` 之后与仿真事件一起提供。

只有当形状的 `b3ShapeDef::enableContactEvents` 为 true 时，该形状才会生成 begin 和 end touch 事件。

#### Hit Events

在游戏中，你通常主要关心两个形状以显著速度碰撞时的接触事件，以便播放音效或粒子效果。hit event 正是为此设计。

```c
for (int i = 0; i < contactEvents.hitCount; ++i)
{
    b3ContactHitEvent* hitEvent = contactEvents.hitEvents + i;
    if (hitEvent->approachSpeed > 10.0f)
    {
        // play sound
    }
}
```

只有当形状的 `b3ShapeDef::enableHitEvents` 为 true 时，形状才会生成 hit event。只应为需要 hit event 的形状启用该功能，因为它会产生一些开销。Box3D 也只报告接近速度大于 `b3WorldDef::hitEventThreshold` 的 hit event。

### Contact Filtering

在游戏中，你经常不希望所有对象都发生碰撞。例如，你可能希望创建一扇只有某些角色可以通过的门。这称为接触过滤，因为某些交互被过滤掉了。

接触过滤设置在形状上，并已在 [Filtering](#filtering) 中说明。

### Advanced Contact Handling

#### Custom Filtering Callback

为了获得最佳性能，应使用 `b3Filter` 提供的接触过滤。不过在某些情况下，你可能需要自定义过滤。可以注册一个实现 `b3CustomFilterFcn()` 的自定义过滤回调。

```c
bool MyCustomFilter(b3ShapeId shapeIdA, b3ShapeId shapeIdB, void* context)
{
    MyGame* myGame = context;
    return myGame->WantsCollision(shapeIdA, shapeIdB);
}

// Elsewhere
b3World_SetCustomFilterCallback(myWorldId, MyCustomFilter, myGame);
```

该函数必须是 [thread-safe](https://en.wikipedia.org/wiki/Thread_safety) 的，并且不得读写 Box3D 世界。否则会产生 [race condition](https://en.wikipedia.org/wiki/Race_condition)。

#### Pre-Solve Callback

该回调在碰撞检测之后、碰撞求解之前调用。它让你有机会基于接触几何禁用接触。例如，可以用该回调实现单向平台。

接触会在每次碰撞处理时重新启用，因此你需要在每个时间步都禁用该接触。该函数必须线程安全，并且不得读写 Box3D 世界。

Box3D 的 pre-solve 回调接收两个形状 id、接触点和接触法线：

```c
bool MyPreSolve(b3ShapeId shapeIdA, b3ShapeId shapeIdB,
                b3Vec3 point, b3Vec3 normal, void* context)
{
    MyGame* myGame = context;

    if (myGame->ShouldDisableContact(shapeIdA, shapeIdB, point, normal))
    {
        return false;
    }

    return true;
}

// Elsewhere
b3World_SetPreSolveCallback(myWorldId, MyPreSolve, myGame);
```

注意，该功能当前不适用于高速碰撞，因此在这些情况下可能看到暂停。

## Joints

关节用于将刚体约束到世界或约束到彼此。游戏中的典型例子包括布娃娃、跷跷板和滑轮。关节可以以许多不同方式组合，产生有趣的运动。

一些关节提供限制，使你能控制运动范围。一些关节提供马达，可用于以给定速度驱动关节，直到超过给定力或力矩。还有一些关节提供带阻尼的弹簧。

关节马达可以有多种用途。你可以通过指定与实际位置和期望位置之差成正比的关节速度来控制位置。你也可以用马达仿真关节摩擦：将关节速度设为零，并提供一个小但有意义的最大马达力或力矩。这样马达会试图阻止关节运动，直到负载过大。

### Joint Definition

每种关节类型都有一个关联的关节定义，其中嵌入 `b3JointDef` 基类。所有关节都连接两个不同刚体。其中一个刚体可以是静态刚体。静态和/或运动学刚体之间的关节是允许的，但不会产生效果，并且会消耗一些处理时间。

如果关节连接到禁用刚体，则该关节实际上也被禁用。当关节两端刚体都启用时，关节也会自动启用。换言之，你不需要显式启用或禁用关节。

你可以为任何关节类型指定用户数据，并可以提供一个标志来防止附着刚体彼此碰撞。这是默认行为；若要允许两个连接刚体碰撞，必须将 `collideConnected` 设为 true。

许多关节定义要求你提供一些几何数据。在 Box3D 中，关节使用 *local frames*（`b3Transform localFrameA` 和 `b3Transform localFrameB`），而不是仅使用锚点。局部坐标系同时指定附着点和用于测量关节量的姿态轴。这些坐标系在所附刚体的局部空间中指定。这样，即便当前刚体变换违反关节约束，也能指定关节。

### Joint Lifetime

关节使用各关节类型提供的创建函数创建，并使用共享函数销毁。所有关节类型共享同一个 id 类型 `b3JointId`。

下面是转动关节生命周期示例：

```c
b3RevoluteJointDef jointDef = b3DefaultRevoluteJointDef();
jointDef.base.bodyIdA = myBodyA;
jointDef.base.bodyIdB = myBodyB;
// Set up local frames so the hinge aligns with the desired pivot
jointDef.base.localFrameA = b3Transform_identity;
jointDef.base.localFrameB = b3Transform_identity;

b3JointId myJointId = b3CreateRevoluteJoint(myWorldId, &jointDef);

// ... do stuff ...

b3DestroyJoint(myJointId, false);
myJointId = b3_nullJointId;
```

销毁 id 后将其置空始终是好的做法。

关节生命周期与刚体生命周期相关。关节不能脱离刚体存在。因此，当刚体被销毁时，附着到该刚体的所有关节会自动销毁。这意味着你需要小心，避免在附着刚体被销毁后继续使用关节 id。如果你使用悬垂关节 id，Box3D 会触发断言。

> **注意**：附着刚体被销毁时，关节也会被销毁。

好在你可以检查关节 id 是否有效。

```c
if (b3Joint_IsValid(myJointId) == false)
{
    myJointId = b3_nullJointId;
}
```

### Using Joints

许多仿真在创建关节后直到销毁前都不会再访问它们。不过，关节中包含许多有用数据，可用于创建更丰富的仿真。

首先，你可以从关节获取类型、刚体、局部坐标系和用户数据。

```c
b3JointType jointType = b3Joint_GetType(myJointId);
b3BodyId bodyIdA = b3Joint_GetBodyA(myJointId);
b3BodyId bodyIdB = b3Joint_GetBodyB(myJointId);
b3Transform localFrameA = b3Joint_GetLocalFrameA(myJointId);
b3Transform localFrameB = b3Joint_GetLocalFrameB(myJointId);
void* myUserData = b3Joint_GetUserData(myJointId);
```

所有关节都有反作用力和反作用力矩。反作用力与 [free body diagram](https://en.wikipedia.org/wiki/Free_body_diagram) 相关。Box3D 的约定是，反作用力在锚点处施加到 body B。你可以使用反作用力破坏关节或触发其他游戏事件。这些函数可能会执行一些计算，因此如果不需要结果，就不要调用它们。

```c
b3Vec3 force  = b3Joint_GetConstraintForce(myJointId);
b3Vec3 torque = b3Joint_GetConstraintTorque(myJointId);
```

### Distance Joint

最简单的关节之一是距离关节，它规定两个刚体上两个锚点之间的距离必须保持恒定。指定距离关节时，两个刚体应已经位于合适位置。随后通过局部坐标系指定两个锚点。这些点隐含了距离约束的长度。

下面是距离关节定义示例：

```c
b3DistanceJointDef jointDef = b3DefaultDistanceJointDef();
jointDef.base.bodyIdA = myBodyIdA;
jointDef.base.bodyIdB = myBodyIdB;

// Place anchor A at world point anchorA, measured in body A's local frame
b3Vec3 localAnchorA = b3Body_GetLocalPoint(myBodyIdA, anchorA);
b3Vec3 localAnchorB = b3Body_GetLocalPoint(myBodyIdB, anchorB);
jointDef.base.localFrameA.p = localAnchorA;
jointDef.base.localFrameB.p = localAnchorB;
jointDef.length = b3Distance(anchorA, anchorB);
jointDef.base.collideConnected = true;

b3JointId myJointId = b3CreateDistanceJoint(myWorldId, &jointDef);
```

距离关节也可以变软，类似弹簧-阻尼连接。通过启用弹簧并调节定义中的两个值实现软化：Hertz 和阻尼比。

```c
jointDef.enableSpring = true;
jointDef.hertz = 2.0f;
jointDef.dampingRatio = 0.5f;
```

hertz 是 [harmonic oscillator](https://en.wikipedia.org/wiki/Harmonic_oscillator) 的频率，例如吉他弦。通常该频率应小于时间步频率的一半。因此，如果你使用 60Hz 时间步，距离关节频率应小于 30Hz。原因与 [Nyquist frequency](https://en.wikipedia.org/wiki/Nyquist_frequency) 有关。

阻尼比控制振荡消散速度。阻尼比为一是 [critical damping](https://en.wikipedia.org/wiki/Damping)，可防止振荡。

也可以为距离关节定义最小和最大长度。甚至可以为距离关节添加马达，以动态调整其长度。详见 `b3DistanceJointDef` 和 `DistanceJoint` 示例。

### Revolute Joint

转动关节（也称 *hinge* 或 *pin* 关节）强制两个刚体共享一个公共锚点，并允许围绕单一轴相对旋转，即局部坐标系的 z 轴。它具有一个旋转自由度。

关节角度测量的是 frame B 的 z 轴相对于 frame A 的 z 轴的扭转。

```c
b3RevoluteJointDef jointDef = b3DefaultRevoluteJointDef();
jointDef.base.bodyIdA = myBodyIdA;
jointDef.base.bodyIdB = myBodyIdB;
jointDef.base.localFrameA.p = b3Body_GetLocalPoint(myBodyIdA, worldPivot);
jointDef.base.localFrameB.p = b3Body_GetLocalPoint(myBodyIdB, worldPivot);

b3JointId myJointId = b3CreateRevoluteJoint(myWorldId, &jointDef);
```

在某些情况下，你可能希望控制关节角。为此，转动关节可以仿真关节限制和/或马达。

关节限制强制关节角保持在下限和上限之间。限制会施加所需任意大小的力矩来实现这一点。限制范围应包含零，否则关节会在仿真开始时突然跳动。

关节马达允许你指定关节速度。速度可以为负或为正。马达可以具有无限力矩，但这通常不可取。回想那个永恒问题：

> 当不可阻挡之力遇到不可移动之物，会发生什么？

我可以告诉你，那不会好看。因此，你可以为关节马达提供最大力矩。只要所需力矩没有超过指定最大值，关节马达就会保持指定速度。当超过最大力矩时，关节会减速，甚至反向。

你可以使用关节马达仿真关节摩擦。只需将关节速度设为零，并将最大力矩设为某个小但有意义的值。马达会试图阻止关节旋转，但在显著负载下会让步。

下面是一个启用了限制和马达的转动关节示例。该马达设置用于仿真关节摩擦。

```c
jointDef.base.localFrameA.p = b3Body_GetLocalPoint(myBodyIdA, worldPivot);
jointDef.base.localFrameB.p = b3Body_GetLocalPoint(myBodyIdB, worldPivot);
jointDef.lowerAngle  = -0.5f * B3_PI; // -90 degrees
jointDef.upperAngle  =  0.25f * B3_PI; //  45 degrees
jointDef.enableLimit = true;
jointDef.maxMotorTorque = 10.0f;
jointDef.motorSpeed  = 0.0f;
jointDef.enableMotor = true;
```

你可以访问转动关节的角度、速度和马达力矩。

```c
float angleInRadians = b3RevoluteJoint_GetAngle(myJointId);
float speed          = b3RevoluteJoint_GetMotorSpeed(myJointId);
float currentTorque  = b3RevoluteJoint_GetMotorTorque(myJointId);
```

你也可以在每个步进中更新马达参数。

```c
b3RevoluteJoint_SetMotorSpeed(myJointId, 20.0f);
b3RevoluteJoint_SetMaxMotorTorque(myJointId, 100.0f);
```

关节马达有一些有趣能力。你可以在每个时间步更新关节速度，从而让关节像正弦波一样来回运动，或遵循你想要的任意函数。

```c
// ... Game Loop Begin ...
b3RevoluteJoint_SetMotorSpeed(myJointId, cosf(0.5f * time));
// ... Game Loop End ...
```

### Prismatic Joint

平移关节（也称 *slider* 关节）允许两个刚体沿局部坐标系 A 的 x 轴相对平移。平移关节阻止相对旋转。因此，平移关节具有一个自由度。

平移关节定义类似于转动关节说明：只需将角度替换为平移，将力矩替换为力。借助这一类比，下面给出一个带有关节限制和摩擦马达的平移关节定义示例：

```c
b3PrismaticJointDef jointDef = b3DefaultPrismaticJointDef();
jointDef.base.bodyIdA = myBodyIdA;
jointDef.base.bodyIdB = myBodyIdB;
jointDef.base.localFrameA.p = b3Body_GetLocalPoint(myBodyIdA, worldPivot);
jointDef.base.localFrameB.p = b3Body_GetLocalPoint(myBodyIdB, worldPivot);
jointDef.lowerTranslation = -5.0f;
jointDef.upperTranslation =  2.5f;
jointDef.enableLimit   = true;
jointDef.maxMotorForce = 1.0f;
jointDef.motorSpeed    = 0.0f;
jointDef.enableMotor   = true;

b3JointId myJointId = b3CreatePrismaticJoint(myWorldId, &jointDef);
```

滑动轴是局部坐标系 A 的 x 轴。

当两个坐标系原点重合时，平移关节的位移为零。

使用平移关节类似于使用转动关节。相关函数如下：

```c
float translation    = b3PrismaticJoint_GetTranslation(myJointId);
float speed          = b3PrismaticJoint_GetSpeed(myJointId);
float motorForce     = b3PrismaticJoint_GetMotorForce(myJointId);
b3PrismaticJoint_SetMotorSpeed(myJointId, speed);
b3PrismaticJoint_SetMaxMotorForce(myJointId, force);
```

### Weld Joint

焊接关节试图约束两个刚体之间的所有相对运动。平移和旋转都可以具有弹簧-阻尼软化。详见 `b3WeldJointDef` 和 `Cantilever` 示例。

将焊接关节用于定义可破坏结构很有诱惑力。不过 Box3D 求解器是近似的，因此无论关节设置如何，某些情况下关节都可能偏软。因此，由焊接关节连接的刚体链可能会弯曲。

```c
b3WeldJointDef jointDef = b3DefaultWeldJointDef();
jointDef.base.bodyIdA    = myBodyIdA;
jointDef.base.bodyIdB    = myBodyIdB;
jointDef.linearHertz     = 0.0f;   // 0 = rigid
jointDef.angularHertz    = 0.0f;   // 0 = rigid
jointDef.linearDampingRatio  = 1.0f;
jointDef.angularDampingRatio = 1.0f;

b3JointId myJointId = b3CreateWeldJoint(myWorldId, &jointDef);
```

### Motor Joint

马达关节允许你通过指定目标线速度和角速度来控制刚体运动。你可以设置将被施加的最大马达力和力矩。如果刚体受阻，它会停止，接触力将与最大马达力和力矩成比例。

马达关节还具有可选的位置弹簧，用于驱动相对变换趋近目标。详见 `b3MotorJointDef` 和 `MotorJoint` 示例。

```c
b3MotorJointDef jointDef = b3DefaultMotorJointDef();
jointDef.base.bodyIdA     = myBodyIdA;
jointDef.base.bodyIdB     = myBodyIdB;
jointDef.linearVelocity   = (b3Vec3){0.0f, 0.0f, 0.0f};
jointDef.maxVelocityForce = 1000.0f;
jointDef.angularVelocity  = (b3Vec3){0.0f, 1.0f, 0.0f};
jointDef.maxVelocityTorque = 500.0f;

b3JointId myJointId = b3CreateMotorJoint(myWorldId, &jointDef);
```

### Wheel Joint

车轮关节专为车辆设计。body A 是底盘，body B 是车轮。车轮：

- 沿 frame A 的 x 轴平移（悬架方向）；
- 绕 frame B 的 z 轴自旋；
- 可选地绕悬架轴转向。

平移部分具有弹簧和阻尼器，用于仿真车辆悬架。自旋部分允许车轮旋转。你可以指定旋转马达驱动车轮并施加制动。转向会绕悬架轴添加旋转。详见 `b3WheelJointDef` 和 `Drive` 示例。

```c
b3WheelJointDef jointDef = b3DefaultWheelJointDef();
jointDef.base.bodyIdA = chassisBodyId;
jointDef.base.bodyIdB = wheelBodyId;
jointDef.enableSuspensionSpring = true;
jointDef.suspensionHertz        = 4.0f;
jointDef.suspensionDampingRatio = 0.7f;
jointDef.enableSpinMotor        = true;
jointDef.maxSpinTorque          = 300.0f;
jointDef.spinSpeed              = 0.0f;
jointDef.enableSteering         = true;
jointDef.steeringHertz          = 5.0f;
jointDef.steeringDampingRatio   = 0.7f;

b3JointId myJointId = b3CreateWheelJoint(myWorldId, &jointDef);
```

### Spherical Joint

球关节（也称 *ball-in-socket* 或 *point-to-point* 关节）将 body B 上的一个点固定到 body A 上的一个点，并允许绕全部三个轴自由旋转。它有 3 个旋转自由度，没有平移自由度。

锥形限制约束 frame B 的 z 轴偏离 frame A 的 z 轴的程度。扭转限制约束绕 frame B 的 z 轴旋转。

```c
b3SphericalJointDef jointDef = b3DefaultSphericalJointDef();
jointDef.base.bodyIdA = myBodyIdA;
jointDef.base.bodyIdB = myBodyIdB;

// Cone limit: allows ±45 degrees of swing
jointDef.enableConeLimit = true;
jointDef.coneAngle = 0.25f * B3_PI;   // half-angle, radians

// Twist limit: ±30 degrees of twist about z
jointDef.enableTwistLimit  = true;
jointDef.lowerTwistAngle   = -B3_PI / 6.0f;
jointDef.upperTwistAngle   =  B3_PI / 6.0f;

b3JointId myJointId = b3CreateSphericalJoint(myWorldId, &jointDef);
```

运行时可以查询和调整限制：

```c
b3SphericalJoint_EnableConeLimit(myJointId, true);
b3SphericalJoint_SetConeLimit(myJointId, 0.3f);
float currentConeAngle = b3SphericalJoint_GetConeAngle(myJointId);

b3SphericalJoint_EnableTwistLimit(myJointId, true);
b3SphericalJoint_SetTwistLimits(myJointId, -0.5f, 0.5f);
float twistAngle = b3SphericalJoint_GetTwistAngle(myJointId);
```

球关节还支持可选的对齐弹簧。启用后，它会使用弹簧-阻尼驱动相对姿态趋近目标四元数：

```c
jointDef.enableSpring    = true;
jointDef.hertz           = 5.0f;
jointDef.dampingRatio    = 0.7f;
jointDef.targetRotation  = b3Quat_identity;
```

以及角速度马达：

```c
jointDef.enableMotor      = true;
jointDef.motorVelocity    = (b3Vec3){0.0f, 1.0f, 0.0f};
jointDef.maxMotorTorque   = 100.0f;
```

### Parallel Joint

平行关节使用弹簧-阻尼器约束 body B 的 z 轴保持与 body A 的 z 轴平行。这对于保持刚体直立很有用，同时又不完全固定其平移或其他旋转轴。

```c
b3ParallelJointDef jointDef = b3DefaultParallelJointDef();
jointDef.base.bodyIdA = myBodyIdA;
jointDef.base.bodyIdB = myBodyIdB;
jointDef.hertz        = 5.0f;
jointDef.dampingRatio = 0.7f;
jointDef.maxTorque    = 200.0f;

b3JointId myJointId = b3CreateParallelJoint(myWorldId, &jointDef);
```

你也可以在运行时调整弹簧和力矩限制：

```c
b3ParallelJoint_SetSpringHertz(myJointId, 8.0f);
b3ParallelJoint_SetSpringDampingRatio(myJointId, 0.5f);
b3ParallelJoint_SetMaxTorque(myJointId, 500.0f);
```

### Filter Joint

过滤（或空）关节用于禁用两个特定刚体之间的碰撞。作为关节的副作用，它还会使两个刚体位于同一个仿真岛中，这可能对性能有用。

```c
b3FilterJointDef jointDef = b3DefaultFilterJointDef();
jointDef.base.bodyIdA = myBodyIdA;
jointDef.base.bodyIdB = myBodyIdB;

b3JointId myJointId = b3CreateFilterJoint(myWorldId, &jointDef);
```

## Spatial Queries

空间查询允许你从几何角度检查世界。它包括重叠查询、射线投射和形状投射。它们可以用于：

- 查找玩家附近的宝箱；
- 发射激光束并摧毁路径上的所有对象；
- 抛出一个由沿抛物线运动的球表示的手雷。

### Overlap Queries

有时你希望确定某一区域中的所有形状。世界使用宽相数据结构提供快速的 log(N) 方法。Box3D 提供这些重叠测试：

- 轴对齐包围盒（AABB）重叠
- 形状代理重叠

<p align="center">
  <img src="https://raw.githubusercontent.com/erincatto/box3d/main/docs/images/overlap_test.svg" alt="Overlap Test" />
</p>

#### Query Filtering

在讨论具体查询之前，需要基本理解查询过滤。形状与形状之间的过滤已在 [Filtering](#filtering) 中讨论。查询使用类似设置。这让查询只考虑某些类别的形状，也让形状忽略某些查询。

与形状一样，查询本身也可以有类别。例如，可以有 `CAMERA` 或 `PROJECTILE` 类别。

```c
enum MyCategories
{
    STATIC     = 0x00000001,
    PLAYER     = 0x00000002,
    MONSTER    = 0x00000004,
    WINDOW     = 0x00000008,
    CAMERA     = 0x00000010,
    PROJECTILE = 0x00000020,
};

// Grenades collide with the static world, monsters, and windows but
// not players or other projectiles.
b3QueryFilter grenadeFilter;
grenadeFilter.categoryBits = PROJECTILE;
grenadeFilter.maskBits = STATIC | MONSTER | WINDOW;

// The view collides with the static world, monsters, and players.
b3QueryFilter viewFilter;
viewFilter.categoryBits = CAMERA;
viewFilter.maskBits = STATIC | PLAYER | MONSTER;
```

如果希望查询所有对象，可以使用 `b3DefaultQueryFilter()`。

#### AABB Overlap

你提供世界坐标中的 `b3AABB` 和一个 `b3OverlapResultFcn()` 实现。世界会对每个 AABB 与查询 AABB 重叠的形状调用你的函数。返回 true 继续查询，否则返回 false。下面的代码查找所有可能与指定 AABB 相交的形状，并唤醒所有相关刚体。

```c
bool MyOverlapCallback(b3ShapeId shapeId, void* context)
{
    b3BodyId bodyId = b3Shape_GetBody(shapeId);
    b3Body_SetAwake(bodyId, true);

    // Return true to continue the query.
    return true;
}

// Elsewhere ...
b3AABB aabb;
aabb.lowerBound = (b3Vec3){-1.0f, -1.0f, -1.0f};
aabb.upperBound = (b3Vec3){ 1.0f,  1.0f,  1.0f};
b3QueryFilter filter = b3DefaultQueryFilter();
b3World_OverlapAABB(myWorldId, aabb, filter, MyOverlapCallback, &myGame);
```

不要对回调顺序做任何假设。形状返回给回调的顺序可能看起来是任意的。

#### Shape Overlap

AABB 重叠非常快，但并不很精确，因为它只考虑形状包围盒。如果需要精确重叠测试，可以使用形状重叠查询。

重叠函数使用 `b3ShapeProxy`，它是由若干点和一个半径构成的抽象形状。你可以把它理解为一团被 *shrink wrapped* 的球。这可以表示一个点、一个球、一个胶囊、一个凸包等。

在此示例中，我从一个球创建形状代理，然后调用 `b3World_OverlapShape()`。该函数接收 `b3OverlapResultFcn()` 以接收结果并控制搜索进程。

{% raw %}```c
b3Sphere sphere = {{0.0f, 0.0f, 0.0f}, 0.2f};
b3ShapeProxy proxy;
proxy.points = &sphere.center;
proxy.count  = 1;
proxy.radius = sphere.radius;
b3World_OverlapShape(myWorldId, b3Pos_zero, &proxy, grenadeFilter, MyOverlapCallback, &myGame);
```{% endraw %}

### Ray-casts

你可以使用射线投射进行视线检查、开枪等。执行射线投射时，需要实现 `b3CastResultFcn()` 回调函数，并提供原点和位移。世界会用射线击中的每个形状调用你的函数。回调会获得形状、交点、单位法向量，以及沿射线的分数距离。不能假设传给回调的点的顺序。回调可能先收到较远的点，再收到较近的点。

<p align="center">
  <img src="https://raw.githubusercontent.com/erincatto/box3d/main/docs/images/raycast.svg" alt="Ray Cast" />
</p>

你通过返回一个 fraction 控制射线投射是否继续。返回 0 表示终止射线投射；返回 1 表示像没有击中一样继续射线投射。如果返回参数列表中的 fraction，射线会被裁剪到当前交点。因此，通过返回合适的 fraction，你可以投射任意形状、投射所有形状，或只投射最近形状。

你也可以返回 -1 来过滤该形状。随后射线投射会像该形状不存在一样继续。

示例：

```c
struct MyRayCastContext
{
    b3ShapeId shapeId;
    b3Vec3 point;
    b3Vec3 normal;
    float fraction;
};

float MyCastCallback(b3ShapeId shapeId, b3Vec3 point, b3Vec3 normal,
                     float fraction, uint64_t userMaterialId,
                     int triangleIndex, int childIndex, void* context)
{
    MyRayCastContext* myContext = context;
    myContext->shapeId  = shapeId;
    myContext->point    = point;
    myContext->normal   = normal;
    myContext->fraction = fraction;
    return fraction;
}

// Elsewhere ...
MyRayCastContext context = {0};
b3Vec3 origin      = {-1.0f, 0.0f, 0.0f};
b3Vec3 translation = { 4.0f, 1.0f, 0.0f};
b3World_CastRay(myWorldId, origin, translation, viewFilter, MyCastCallback, &context);
```

也有一个用于最近命中的便利函数：

```c
b3RayResult result = b3World_CastRayClosest(myWorldId, origin, translation, filter);
if (result.hit)
{
    // result.point, result.normal, result.fraction
}
```

射线投射结果可能以任意顺序返回。当你沿射线收集多个命中时，可能需要根据命中 fraction 对结果排序。

### Shape-casts

形状投射类似于射线投射。你可以把射线投射视为沿一条线追踪一个点。形状投射允许你沿一条线追踪一个形状。与形状重叠查询类似，形状投射使用 `b3ShapeProxy` 表示任意形状。

{% raw %}```c
struct MyRayCastContext
{
    b3ShapeId shapeId;
    b3Vec3 point;
    b3Vec3 normal;
    float fraction;
};

float MyCastCallback(b3ShapeId shapeId, b3Vec3 point, b3Vec3 normal,
                     float fraction, uint64_t userMaterialId,
                     int triangleIndex, int childIndex, void* context)
{
    MyRayCastContext* myContext = context;
    myContext->shapeId  = shapeId;
    myContext->point    = point;
    myContext->normal   = normal;
    myContext->fraction = fraction;
    return fraction;
}

// Elsewhere ...
MyRayCastContext context = {0};
b3Sphere sphere = {{-1.0f, 0.0f, 0.0f}, 0.05f};
b3ShapeProxy proxy;
proxy.points = &sphere.center;
proxy.count  = 1;
proxy.radius = sphere.radius;
b3Vec3 translation = {10.0f, -5.0f, 0.0f};
b3World_CastShape(myWorldId, b3Pos_zero, &proxy, translation, grenadeFilter, MyCastCallback, &context);
```{% endraw %}

形状投射的设置方式类似于射线投射。形状投射通常比射线投射慢，因此只有在射线投射不能满足需求时才使用形状投射。

与射线投射一样，形状投射结果可能以任意顺序发送给回调。如果需要多个排序后的结果，需要编写代码收集并排序结果。

## Simulation Loop

<p align="center">
  <img src="https://raw.githubusercontent.com/erincatto/box3d/main/docs/images/simulation_loop.svg" alt="Simulation Loop" />
</p>

理解 Box3D 仿真循环有助于处理结果。

图中表示了多线程：

- 矩形是 parallel-for 工作；
- 圆角矩形是单线程工作，但可能与其他工作并行。

下面逐一回顾这些阶段。

### time step

游戏通过调用 `b3World_Step` 并提供时间步来开始仿真。

### find pairs

Box3D 维护每个已移动形状的记录。对每个这样的形状，会查询宽相（BVH）以获得重叠。新的重叠会被记录下来，供下一步处理。已有重叠由所有现有形状对的哈希集合跟踪。该操作是 parallel-for。Box3D 为静态、运动学和动态刚体分别维护 BVH 树。

### create contacts

该阶段获取形状对结果并创建内部接触对结构。该结构跨时间步持久存在。它用于岛图，并保存接触流形。该操作是单线程的，但大部分工作已在 `find pairs` 中完成。

### rebuild BVH

在新的形状对已知后，BVH 直到时间步稍后才会再次被考虑。因此，可以优化动态和运动学形状的 BVH。这涉及识别碰撞树中因 refit 而陈旧的部分，然后重建该子树。这是一个单线程操作，并与其他工作并发执行。

### narrow phase

该阶段计算接触流形和接触点。每个活动接触对都会被确认为接触或非接触。如果接触状态发生变化，会生成 contact begin 和 end 事件。这是 parallel-for 操作。

接触点在时间步开始时计算，此时刚体尚未移动，冲量也尚未知晓。为了高效获得良好仿真结果，这是必要的。如果接触在时间步末尾才计算，约束求解器就无法知道新的接触点，形状会彼此下沉。

`b3PreSolveFcn` 在 parallel-for 内调用，因此它应高效且线程安全。只有当形状具有 `enablePreSolveEvents == true` 时才会调用它。

### merge islands

当形状开始接触时，会合并仿真岛。若已有岛中的形状停止接触，该岛会被标记为分裂候选。该阶段是单线程的。

### split island

在以下情况下可能生成分裂岛任务：

- 某个岛中有形状停止接触；
- 该岛中有一个刚体移动足够慢，可以进入休眠。

分裂一个岛可能产生多个新岛。由于代价较高，每个时间步只会分裂一个岛。推迟分裂可能会延迟某些刚体进入休眠。该操作是单线程的，并与其他工作并发执行。

### solve constraints

该阶段求解接触和关节约束，并应用恢复系数。这是一个带有多个内部阶段的 parallel-for。

### update transforms

该阶段执行若干任务：

- 更新刚体变换；
- 更新刚体休眠状态；
- 生成刚体移动事件；
- 生成岛分裂候选；
- 重置力和力矩；
- 更新形状包围盒；
- 在动态和静态刚体之间执行连续碰撞。

该阶段是 parallel-for。

注意，连续碰撞不会生成事件。事件会在下一时间步生成。不过，连续碰撞会触发 `b3PreSolveFcn` 回调。

### hit events

该阶段扫描活动接触，查找快速接近速度，并将其加入缓冲区。它考虑具有冲量的接触点。这包括正在接触的接触，以及产生冲量的推测接触（它们已被确认）。因此，你可能会收到接触点具有正间隔的 hit event。这是单线程操作。

### refit BVH

该阶段更新移动显著的形状对应的 BVH。它通过扩大形状包围盒以及 BVH 中所有祖先包围盒完成。这是单线程操作。

这可能导致 BVH 低效，这就是 `rebuild BVH` 阶段存在的原因。refit 比重建 BVH 快，但它对于确保 BVH 对后续查询（如射线投射）有效是必要的。

### bullets

该阶段处理 bullet。注意它发生在 hit event 之后，因为 Box3D 中的连续碰撞直到下一时间步才会生成事件。

### island sleep

当一个岛进入休眠时，与该岛关联的仿真数据会被移动到独立存储中。这使活动仿真数据保持连续，并对缓存友好。

### sensors

传感器重叠在最后阶段检查。重叠状态反映最终的刚体变换。传感器不考虑休眠，因此它们可能对用户设置刚体变换或创建休眠刚体产生反应。这是 parallel-for 操作。其成本大致与传感器数量成正比。

## Determinism

Box3D 被设计为在不同线程数和平台之间保持确定性。这对调试和游戏设计很重要。

多线程确定性通过基于创建顺序确定仿真顺序实现。这包括刚体、形状和关节的创建顺序。确定性还包括报告给用户的结果（事件），这些事件必须以确定性顺序出现。

在 64 位平台上，跨平台确定性通过使用编译器标志并避免非确定性库函数实现。

- MSVC 使用精确数学。
- Clang 和 GCC 通过 `-ffp-contract=off` 禁用浮点收缩。
- Box3D 对 atan2、cosine 和 sine 有自定义实现。

确定性默认开启，且没有显式选项禁用它。不过，你可以通过选择不同编译器标志破坏确定性。Box3D 被设计为以最小成本提供确定性。因此，试图禁用确定性没有优势。

每个 pull request 都会运行确定性单元测试。确定性很容易被破坏，因此定期验证很重要。

> **注意**：Box3D 的确定性并不意味着你的应用程序也会是确定性的。可以考虑使用与 Box3D 类似的策略，让你的应用程序保持确定性。
