---
layout: page
title: CUDA
permalink: /cuda/
---

```c++
threadId.x = blockIdx.x*blockDim.x+threadIdx.x;
threadId.y = blockIdx.y*blockDim.y+threadIdx.y;
```

参考文献

- [CUDA Tutorial](https://jhui.github.io/2017/03/06/CUDA/)
- [CUDA编程之快速入门](https://www.cnblogs.com/skyfsm/p/9673960.html)