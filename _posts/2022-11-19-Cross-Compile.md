---
layout: post
title: "轻松搞定树莓派交叉编译"
categories: study
author: "Jixiang Zhang"
---

编译机器: MacBook Air, Apple M1

目标机器: Raspberry Pi 400

## 第一步安装环境

```bash
# Install tools
brew install arm-linux-gnueabihf-binutils llvm rsync
# Install libraries
rsync -rzLRv pi@10.0.0.11:/usr/lib/arm-linux-gnueabihf pi@10.0.0.11:/usr/lib/gcc/arm-linux-gnueabihf pi@10.0.0.11:/usr/include pi@10.0.0.11:/lib/arm-linux-gnueabihf sysroot
```

## 第二步编译执行

准备测试代码 `test.cc`

```c++
#include <iostream>
#include <vector>

int main() {
  /* std::vector<int> v = {1, 2, 3, 4, 5}; */
  std::vector<int> v(10, 5);
  int sum = 0;
  for (int i = 0; i < v.size(); i++) {
    sum += v[i] * 2;
  }
  std::cout << "sum is " << sum << std::endl;
  return 0;
}
```

```bash
# Cross compiling
clang++ --target=arm-linux-gnueabihf --sysroot=$HOME/sysroot test.cc -o test
# Copy file to Raspberry Pi 400
scp test pi@10.0.0.11:~
# Run on Raspberry Pi 400
./test
```

### Cite

1. [Cross compiling C/C++ from macOS to Raspberry Pi in 2 easy steps](https://medium.com/@haraldfernengel/cross-compiling-c-c-from-macos-to-raspberry-pi-in-2-easy-steps-23f391a8c63)
