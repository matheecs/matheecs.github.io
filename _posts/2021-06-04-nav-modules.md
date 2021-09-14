---
layout: post
title: "MoveBase 剖析"
categories: study
author: "Jixiang Zhang"
---

![](/images/movebase.jpg)

```
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
