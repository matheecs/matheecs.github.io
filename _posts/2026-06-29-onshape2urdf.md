---
layout: post
title: "Onshape-to-Robot 处理流程"
categories: robotics
author: "Jixiang Zhang"
---

本文翻译自 [onshape-to-robot 文档：Processors](https://onshape-to-robot.readthedocs.io/en/latest/processors.html)。

---

## 引言

onshape-to-robot 的流水线（pipeline）分为三步：

1.  **从 Onshape 获取装配体**，生成机器人的中间表示（intermediate representation）。
    - 参见：[robot.py](https://github.com/Rhoban/onshape-to-robot/blob/master/onshape_to_robot/robot.py)
2.  对该中间表示执行一系列操作——这些操作称为**处理器（processors）**，下文列出了部分处理器。
3.  通过**导出器（exporter）**将机器人导出为所需格式（URDF、MuJoCo 等）。

![architecture](/images/onshape2urdf_architecture.png)

> **提示**：`export.py` 脚本是 `onshape-to-robot` 命令的入口，概括了上述三步流水线。如果你想在处理过程中调整机器人参数，可以查看该脚本。
>
> 参见：[export.py](https://github.com/Rhoban/onshape-to-robot/blob/master/onshape_to_robot/export.py)

## 获取模式与转换模式

你可以通过向 `onshape-to-robot` 命令传递 `--retrieve` 参数，只运行第 (1) 步获取。这会将机器人的中间表示保存为 `robot.pkl` 文件到输出目录中。

第 (2) 步和第 (3) 步可以通过 `--convert` 参数单独运行，从 `robot.pkl` 加载数据、执行处理器并生成最终输出。这样在调整处理器或导出器时，无需反复向 Onshape 发送 API 请求。

`--save-pickle` 参数在获取完成后保存数据，同时继续执行转换步骤。

---

## 1. Ball to Euler（球铰 → 欧拉角）

> 原文：[processor_ball_to_euler](https://onshape-to-robot.readthedocs.io/en/latest/processor_ball_to_euler.html)

### 简介

此处理器将球铰关节（ball joint）转换为三个旋转关节（revolute joints），以便在不支持球铰的下游模拟器或工具中使用。

### config.json 配置项

```json
{
    // ...
    // 通用导入选项（参见 config.json 文档）
    // ...

    // 将球铰转换为欧拉角（默认：false）
    "ball_to_euler": true,
    // 欧拉角顺序（默认："xyz"）
    "ball_to_euler_order": "xyz"
}
```

#### `ball_to_euler`（默认：`false`）

设为 `true` 时，所有球铰关节都会被转换为欧拉角。也可以传入一个关节名称列表（支持通配符）来选择性地转换：

```json
{
    // 使用特定关节列表，支持通配符
    "ball_to_euler": ["joint1", "shoulder_*"]
}
```

#### `ball_to_euler_order`（默认：`"xyz"`）

控制欧拉角的旋转顺序。有效值：`xyz`、`xzy`、`zyx`、`zxy`、`yxz`、`yzx`。

---

## 2. Merge STLs（合并 STL 网格）

> 原文：[processor_merge_parts](https://onshape-to-robot.readthedocs.io/en/latest/processor_merge_parts.html)

### 简介

此处理器将同一连杆（link）内所有零件的网格合并为一个整体网格。

### config.json 配置项

```json
{
    // ...
    // 通用导入选项（参见 config.json 文档）
    // ...

    // 合并 STL 网格（默认：false）
    "merge_stls": true
}
```

#### `merge_stls`（默认：`false`）

设为 `true` 时，每个连杆的零件网格将合并为一个。合并后的零件以连杆名称命名。

支持三种取值：

- `true` — 合并视觉网格和碰撞网格
- `"visual"` — 仅合并视觉网格
- `"collision"` — 仅合并碰撞网格

---

## 3. Simplify STLs（简化 STL 网格）

> 原文：[processor_simplify_stls](https://onshape-to-robot.readthedocs.io/en/latest/processor_simplify_stls.html)

### 简介

此处理器简化 STL 文件，使其大小不超过预设限制。与 [Merge STLs](#2-merge-stls合并-stl-网格) 配合使用时，可以简化合并后的网格。

### 依赖要求

需要安装 pymeshlab：

```bash
pip install pymeshlab
```

### config.json 配置项

```json
{
    // ...
    // 通用导入选项（参见 config.json 文档）
    // ...

    // 简化 STL 网格（默认：false）
    "simplify_stls": true,
    // STL 文件最大大小，单位 MB（默认：3）
    "max_stl_size": 1
}
```

#### `simplify_stls`（默认：`false`）

启用后，STL 文件将被简化。

#### `max_stl_size`（默认：`3`）

文件大小阈值（MB），超过此值的文件会被简化。也可设为 `"visual"`（仅简化视觉网格）或 `"collision"`（仅简化碰撞网格）。

---

## 4. OpenSCAD 纯形状逼近

> 原文：[processor_scad](https://onshape-to-robot.readthedocs.io/en/latest/processor_scad.html)

📺 视频教程：[YouTube](https://www.youtube.com/watch?v=C8oK4uUmbRw)（部分步骤已过时，但整体思路不变）

### 简介

此处理器允许用户手动将机器人各个部分逼近为纯几何形状。

### 依赖要求

需要安装 OpenSCAD：

```bash
sudo apt-get install openscad
```

### 处理流程

启用后，处理器会检查输出目录中是否存在 `.scad` 文件。若找到，则解析这些文件并导出纯几何形状。

> **注意**：导出器将使用纯几何形状进行碰撞检测，而非原始网格。

可以使用以下命令对特定 `.stl` 进行逼近：

```bash
onshape-to-robot-edit-shape <path_to_stl>
```

这将打开与 `.stl` 同名的 `.scad` 文件的编辑窗口，在此处定义的纯几何形状将用作逼近。

![pure-shape](/images/onshape2urdf_pure_shape.png)

### config.json 配置项

```json
{
    // ...
    // 通用导入选项（参见 config.json 文档）
    // ...

    // 使用 OpenSCAD 纯形状逼近（默认：false）
    "use_scads": true,
    // 放大/缩小纯形状的比例（默认：0.0）
    "pure_shape_dilatation": 0.0
}
```

#### `use_scads`（默认：`false`）

设为 `true` 时，处理器将使用 OpenSCAD 纯形状逼近。

#### `pure_shape_dilatation`（默认：`0.0`）

浮点数参数，用于放大或缩小纯形状以避免碰撞。建议使用负值来缩小形状。

---

## 5. Adding dummy base link（添加虚拟基座）

> 原文：[processor_dummy_base_link](https://onshape-to-robot.readthedocs.io/en/latest/processor_dummy_base_link.html)

### 简介

此处理器向机器人添加一个名为 `base_link` 的虚拟基座，该基座通过 `fixed` 关节连接到机器人。

### config.json 配置项

```json
{
    // ...
    // 通用导入选项（参见 config.json 文档）
    // ...

    // 添加虚拟基座连杆（默认：false）
    "add_dummy_base_link": true
}
```

#### `add_dummy_base_link`（默认：`false`）

设为 `true` 时，将在机器人中添加一个虚拟基座连杆。

---

## 6. Removing collision meshes（移除碰撞网格）

> 原文：[processor_no_collision_meshes](https://onshape-to-robot.readthedocs.io/en/latest/processor_no_collision_meshes.html)

### 简介

此处理器确保不输出任何碰撞网格。

> **备选方案**：也可以使用 ignore 列表来实现：
>
> ```json
> {
>     "ignore": {
>         "*": "collision"
>     }
> }
> ```
>
> 但需注意，这种替代方法会阻止网格被其他处理器使用（例如，无法再用纯形状进行逼近）。

### config.json 配置项

```json
{
    // ...
    // 通用导入选项（参见 config.json 文档）
    // ...

    // 移除碰撞网格
    "no_collision_meshes": true
}
```

#### `no_collision_meshes`（默认：`false`）

设为 `true` 时，碰撞网格将被移除。

---

## 7. Use collisions as visual（碰撞网格用作视觉）

> 原文：[processor_collision_as_visual](https://onshape-to-robot.readthedocs.io/en/latest/processor_collision_as_visual.html)

### 简介

启用此处理器后，碰撞几何体将被复用为视觉几何体。适用于两种场景：

1. **调试** — 当下游工具不方便可视化碰撞几何时
2. **轻量化** — 创建加载更快的模型

### config.json 配置项

```json
{
    // ...
    // 通用导入选项（参见 config.json 文档）
    // ...

    // 将碰撞网格用作视觉网格
    "collisions_as_visual": true
}
```

#### `collisions_as_visual`（默认：`false`）

设为 `true` 时，碰撞网格将被用作视觉网格。

---

## 8. Convex decomposition / CoACD（凸分解）

> 原文：[processor_convex_decomposition](https://onshape-to-robot.readthedocs.io/en/latest/processor_convex_decomposition.html)

### 简介

启用此处理器后，碰撞网格将使用 [CoACD](https://github.com/SarahWeiii/CoACD) 算法进行凸分解。

### config.json 配置项

```json
{
    // ...
    // 通用导入选项（参见 config.json 文档）
    // ...

    // 启用凸分解
    "convex_decomposition": true,
    // 使用彩虹色替代零件原有颜色
    "rainbow_colors": true
}
```

#### `convex_decomposition`（默认：`false`）

设为 `true` 时，将对碰撞网格应用 CoACD 凸分解。

#### `rainbow_colors`（默认：`false`）

启用后，碰撞网格将使用彩虹色进行着色，而非零件原有颜色。

---

## 9. Using fixed links（使用固定连杆）

> 原文：[processor_fixed_links](https://onshape-to-robot.readthedocs.io/en/latest/processor_fixed_links.html)

### 简介

此处理器将所有零件拆分为独立的连杆（link），每个零件通过 `fixed` 关节与其父级关联。

> **注意**：这样做可能会导致物理引擎性能下降，但适用于调试场景。

### config.json 配置项

```json
{
    // ...
    // 通用导入选项（参见 config.json 文档）
    // ...

    // 添加固定连杆，每个零件一个连杆
    "use_fixed_links": true
}
```

#### `use_fixed_links`（默认：`false`）

设为 `true` 时，将向机器人添加固定连杆。也可以传入一个列表，指定需要生成固定连杆的特定连杆名称。

---

## 10. Writing & registering custom Processor（编写和注册自定义处理器）

> 原文：[custom_processors](https://onshape-to-robot.readthedocs.io/en/latest/custom_processors.html)

### 简介

所有默认注册的处理器描述可以在项目 GitHub 源码（[processors.py](https://github.com/Rhoban/onshape-to-robot/blob/master/onshape_to_robot/processors.py)）中找到。用户可以编写自己的处理器类，并通过 `config.json` 注册到流水线中。

### 最小处理器示例

```python
# my_project/my_custom_processor.py
from onshape_to_robot.processor import Processor
from onshape_to_robot.config import Config
from onshape_to_robot.robot import Robot

class MyCustomProcessor(Processor):
    def __init__(self, config: Config):
        super().__init__(config)

        self.use_my_custom: bool = config.get("use_my_custom", False)

    def process(self, robot: Robot):
        if self.use_my_custom:
            print(f"Custom processing for {robot.name} with custom processor.")
```

处理器处理的是机器人的中间表示（参见 [robot.py](https://github.com/Rhoban/onshape-to-robot/blob/master/onshape_to_robot/robot.py)）。

### 注册你的处理器

要注册自定义处理器，在 `config.json` 中添加 `"processors"` 条目，格式为 `module_path:ClassName`：

```json
{
    "processors": [
        // 自定义处理器
        "my_project.my_custom_processor:MyCustomProcessor",
        // 默认处理器
        "ProcessorScad",
        "ProcessorMergeParts",
        "ProcessorNoCollisionMeshes"
    ]
}
```

> 现有的默认处理器列表参见 GitHub 源码 [processors.py](https://github.com/Rhoban/onshape-to-robot/blob/master/onshape_to_robot/processors.py)。
