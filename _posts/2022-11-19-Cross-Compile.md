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

## Demo from [cyberdog_motor_sdk](https://github.com/MiRoboticsLab/cyberdog_motor_sdk)

```bash
cmake -DCMAKE_TOOLCHAIN_FILE=/usr/xcc/aarch64-openwrt-linux-gnu/Toolchain.cmake ..
```

**Toolchain.cmake**

```cmake
# this is required
#SET(CMAKE_SYSTEM_NAME Linux)

# specify the cross compiler
#SET(CMAKE_C_COMPILER   aarch64-openwrt-linux-gnu-gcc)
#SET(CMAKE_CXX_COMPILER aarch64-openwrt-linux-gnu-g++)

# where is the target environment 
#SET(CMAKE_FIND_ROOT_PATH  /toolchain)

# search for programs in the build host directories (not necessary)
#SET(CMAKE_FIND_ROOT_PATH_MODE_PROGRAM NEVER)
# for libraries and headers in the target directories
#SET(CMAKE_FIND_ROOT_PATH_MODE_LIBRARY ONLY)
#SET(CMAKE_FIND_ROOT_PATH_MODE_INCLUDE ONLY)


set(CMAKE_SYSTEM_NAME Linux)
set(CMAKE_SYSTEM_VERSION 1)
set(CMAKE_SYSTEM_PROCESSOR aarch64)
set(cross_triple "aarch64-openwrt-linux-gnu")
set(cross_root /usr/xcc/${cross_triple})

# /usr/xcc/aarch64-openwrt-linux-gnu/bin/aarch64-openwrt-linux-gnu-gcc
# /usr/xcc/aarch64-openwrt-linux-gnu/bin/aarch64-openwrt-linux-gnu-g++
set(CMAKE_C_COMPILER $ENV{CC})
set(CMAKE_CXX_COMPILER $ENV{CXX})
#set(CMAKE_Fortran_COMPILER $ENV{FC})

#set(CMAKE_CXX_FLAGS "-I ${cross_root}/include/")

set(CMAKE_FIND_ROOT_PATH ${cross_root} ${cross_root}/${cross_triple})
set(CMAKE_FIND_ROOT_PATH_MODE_PROGRAM NEVER)
set(CMAKE_FIND_ROOT_PATH_MODE_LIBRARY ONLY)
set(CMAKE_FIND_ROOT_PATH_MODE_INCLUDE ONLY)
set(CMAKE_SYSROOT ${cross_root}/${cross_triple})
# set(CMAKE_STAGING_PREFIX ${cross_root}/${cross_triple}/usr)

#set(CMAKE_CROSSCOMPILING_EMULATOR /usr/bin/qemu-arm)
```

<p align="center">
  <img src="images/carbon.png" width="500"/>
</p>

### Cite

1. [Cross compiling C/C++ from macOS to Raspberry Pi in 2 easy steps](https://medium.com/@haraldfernengel/cross-compiling-c-c-from-macos-to-raspberry-pi-in-2-easy-steps-23f391a8c63)
2. [Cross-compiling for Raspberry Pi](https://github.com/sony/nmos-cpp/blob/master/Documents/Raspberry-Pi.md)
