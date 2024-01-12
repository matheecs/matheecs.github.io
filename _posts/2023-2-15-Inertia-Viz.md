---
layout: post
title: "Inertia Visualization"
categories: study
author: "Jixiang Zhang"
---

![](https://github.com/ADVRHumanoids/robot_inertia_publisher/raw/master/robot_inertia_publisher.png)

The `robot_inertia_publisher`[^1] permits to broadcast and visualize in RVIZ inertial information retrieved from the URDF description of a robot. Inertia of the single link is visualized as ellipsoid aorund a spehere which represents the cog of the link.

## Calculate ellipsoid

```bash
# inertia viz -> ellipsoid
markeri = Marker()
inertia = numpy.matrix([[link.inertial.inertia.ixx, link.inertial.inertia.ixy, link.inertial.inertia.ixz], 
                        [link.inertial.inertia.ixy, link.inertial.inertia.iyy, link.inertial.inertia.iyz], 
                        [link.inertial.inertia.ixz, link.inertial.inertia.iyz, link.inertial.inertia.izz]])

(eigValues,eigVectors) = numpy.linalg.eig (inertia)

eigx_n=-PyKDL.Vector(eigVectors[0,0],eigVectors[1,0],eigVectors[2,0])
eigy_n=-PyKDL.Vector(eigVectors[0,1],eigVectors[1,1],eigVectors[2,1])
eigz_n=-PyKDL.Vector(eigVectors[0,2],eigVectors[1,2],eigVectors[2,2])

rot = PyKDL.Rotation (eigx_n,eigy_n,eigz_n)
quat = rot.GetQuaternion ()

if link.inertial.origin:
    markeri.pose.position.x = link.inertial.origin.position[0]
    markeri.pose.position.y = link.inertial.origin.position[1]
    markeri.pose.position.z = link.inertial.origin.position[2]
else:
    markeri.pose.position.x = 0.
    markeri.pose.position.y = 0.
    markeri.pose.position.z = 0.
markeri.pose.orientation.x = quat[0]
markeri.pose.orientation.y = quat[1]
markeri.pose.orientation.z = quat[2]
markeri.pose.orientation.w = quat[3]

alpha = 2.*link.inertial.mass/5.

markeri.scale.x = 2.*numpy.sqrt(numpy.absolute(eigValues[1]+eigValues[2]-eigValues[0])/alpha)
markeri.scale.y = 2.*numpy.sqrt(numpy.absolute(eigValues[2]+eigValues[0]-eigValues[1])/alpha)
markeri.scale.z = 2.*numpy.sqrt(numpy.absolute(eigValues[0]+eigValues[1]-eigValues[2])/alpha)
```

---

[^1]: <https://github.com/ADVRHumanoids/robot_inertia_publisher>

Reference: <https://github.com/RobotLocomotion/drake/blob/85195ff670d4a33c4a400ee019e1497d960686af/multibody/tree/spatial_inertia.h#L856>
