---
layout: post
title: "Choreonoid in Robot(≈UE in Game)"
categories: study
author: "Jixiang Zhang"
---

动画师 Notebook: personal observations on the principles of movement

[Documentation](https://choreonoid.org/en/documents/latest/index.html#)

### Build with

* [fmt](https://github.com/fmtlib/fmt) **7.1.3**! (ubuntu_22.04)

```shell
./build/bin/choreonoid sample/PoseSeq/GR001.cnoid
```

### URDF Model convert to Body Model

Check BodyItem > Right Click > `Save as`

### create videos

`View` > `Show Toolbar` > `MovieRecorderBar`

### 追加情報の記述

```yaml
standardPose: [ 
  0, 0,  20, -40, -20, 0,
  0, 0, -20,  40,  20, 0,
  0, 0,
  20, 0, -20,
  -20, 0,  20 ]

linkGroup: 
  - name: UPPER-BODY
    links: 
      - name: NECK
        links: [ NECK_Y ]
      - name: ARMS
        links:
          - name: R-ARM
            links: [ R_SHOULDER_P, R_SHOULDER_R, R_ELBOW_P ]
          - name: L-ARM
            links: [ L_SHOULDER_P, L_SHOULDER_R, L_ELBOW_P ]
      - name: CHEST
        links: [ CHEST_P ]
  - WAIST
  - name: LOWER-BODY
    links:
      - name: LEGS
        links:
        - name: R-LEG
          links: [ R_HIP_Y, R_HIP_R, R_HIP_P, R_KNEE_P, R_ANKLE_P, R_ANKLE_R ]
        - name: L-LEG
          links: [ L_HIP_Y, L_HIP_R, L_HIP_P, L_KNEE_P, L_ANKLE_P, L_ANKLE_R ]

possibleIkInterpolationLinks: [ WAIST, R_ANKLE_R, L_ANKLE_R ]
defaultIkInterpolationLinks: [ WAIST, R_ANKLE_R, L_ANKLE_R ]

defaultIKsetup:
  WAIST: [ R_ANKLE_R, L_ANKLE_R ]
  R_ANKLE_R: [ WAIST ]
  L_ANKLE_R: [ WAIST ]

foot_links:
  - link: L_ANKLE_R
    sole_center: [ -0.005,  0.01, -0.022 ]
  - link: R_ANKLE_R
    sole_center: [ -0.005, -0.01, -0.022 ]

symmetricJoints:
  - [ NECK_Y ]
  - [ L_SHOULDER_P, R_SHOULDER_P, -1 ]
  - [ L_SHOULDER_R, R_SHOULDER_R, -1 ]
  - [ L_ELBOW_P,    R_ELBOW_P,    -1 ]
  - [ L_HIP_Y,      R_HIP_Y,      -1 ]
  - [ L_HIP_R,      R_HIP_R,      -1 ]
  - [ L_HIP_P,      R_HIP_P,      -1 ]
  - [ L_KNEE_P,     R_KNEE_P,     -1 ]
  - [ L_ANKLE_P,    R_ANKLE_P,    -1 ]
  - [ L_ANKLE_R,    R_ANKLE_R,    -1 ]

symmetricIkLinks:
  - [ WAIST ]
  - [ L_ANKLE_R, R_ANKLE_R ]

collision_detection_rules:
  - disabled_link_chain_level: 2
```
