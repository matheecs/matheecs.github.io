---
layout: post
title: "MoveBase 剖析"
categories: study
author: "Jixiang Zhang"
---

![movebase](/images/movebase.jpg)

```text
┌────────────────────┐
│                ┌───┴──────────────┐    ┌──────────────────┐
│                │ BaseGlobalPlanner◁────┤ GlobalPlanner    │
│                └───┬──────────────┘    └──────────────────┘
│                ┌───┴────────────┐      ┌──────────────────┐
│                │ BaseLocalPlanner◁─────┤ TebLocalPlannerROS
│  MoveBase      └───┬────────────┘      └──────────────────┘
│                ┌ ─ ┴ ─ ─ ─ ─ ─ ─ ┐
│                  RecoveryBehavior
│                └ ─ ┬ ─ ─ ─ ─ ─ ─ ┘
└──────────◇─────────┘
           │
           │
┌──────────┴─────────┐
│                ┌───┴────────────┐      ┌──────────────────┐
│                │ Layer          │      │  SpatioTemporal  │
│  Costmap2DROS  │                ◁──────┤  VoxelLayer      │
│                └───┬────────────┘      └──────────────────┘
└────────────────────┘
```
