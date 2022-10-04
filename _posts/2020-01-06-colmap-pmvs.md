---
layout: post
title: "用PMVS做稠密重建"
categories: study
author: "Jixiang Zhang"
---

**所需软件**

- [COLMAP](https://github.com/colmap/colmap)
- [CMVS-PMVS](https://github.com/pmoulon/CMVS-PMVS)

**重建步骤**

1. 采集数据
   ![数据集](https://tva3.sinaimg.cn/large/d494c514ly1gamnn5fdnrj215i0umx6q.jpg)
2. SfM（COLMAP）
   ![COLMAP](https://tvax1.sinaimg.cn/large/d494c514ly1gan75lq9adj21r414s7gg.jpg)
3. MVS（PMVS）

   ```bash
   ./pmvs2 /Users/zhangjixiang/Downloads/MVS/pmvs/ option-all
   ```

4. 用 meshlab 观察重建结果
   ![snapshot00](https://tvax4.sinaimg.cn/large/d494c514ly1gamnople40j21we11443x.jpg)
