---
layout: page
title: Note
permalink: /note/
---

* toc
{:toc}
------

### 参数取值范围错误

原文链接 [一起做RGB-D SLAM (3)](http://www.cnblogs.com/gaoxiang12/p/4659805.html)

使用 CLion，强烈推荐，比 Qt Creator 好用。调试终止在 solvePnPRansac 函数：

```shell
OpenCV Error: Assertion failed (confidence > 0 && confidence < 1) in run, file /Users/zhangjixiang/Downloads/opencv-3.3.1/modules/calib3d/src/ptsetreg.cpp, line 178`
`libc++abi.dylib: terminating with uncaught exception of type cv::Exception: /Users/zhangjixiang/Downloads/opencv-3.3.1/modules/calib3d/src/ptsetreg.cpp:178: error: (-215) confidence > 0 && confidence < 1 in function run
```

程序源码：

```c++
cv::solvePnPRansac( pts_obj, pts_img, cameraMatrix, cv::Mat(), rvec, tvec, false, 100, 1.0, 100, inliers );
```

错误原因：参数 confidence 超出了 (0, 1) 范围。

程序改为：

```c++
cv::solvePnPRansac( pts_obj, pts_img, cameraMatrix, cv::Mat(), rvec, tvec, false, 100, 1.0, 0.99, inliers );
```
可正常运行。

------

### 多余的括号

Bug来源：多余的括号

```c++
string value = str.substr((pos+1, str.length()));
```
运行时错误：

```shell
OpenCV Error: Assertion failed (npoints >= 4 && npoints == std::max(ipoints.checkVector(2, 5), ipoints.checkVector(2, 6))) in solvePnPRansac, file /Users/zhangjixiang/Downloads/opencv-3.3.1/modules/calib3d/src/solvepnp.cpp, line 252
libc++abi.dylib: terminating with uncaught exception of type cv::Exception: /Users/zhangjixiang/Downloads/opencv-3.3.1/modules/calib3d/src/solvepnp.cpp:252: error: (-215) npoints >= 4 && npoints == std::max(ipoints.checkVector(2, 5), ipoints.checkVector(2, 6)) in function solvePnPRansac
```
程序可修改为：
```c++
string value = str.substr(pos+1, str.length());
```
------

### macOS 线程与 UI

导致异常的代码：

```c++
pcl::visualization::CloudViewer viewer("viewer");
viewer.showCloud(output);
while(!viewer.wasStopped())
{

}
```

运行时出错提示：

```shell
2018-01-31 19:30:31.598 joinPointCloud[57826:4712504] *** Assertion failure in +[NSUndoManager _endTopLevelGroupings], /BuildRoot/Library/Caches/com.apple.xbs/Sources/Foundation/Foundation-1450.16/Foundation/Misc.subproj/NSUndoManager.m:361
2018-01-31 19:30:31.599 joinPointCloud[57826:4712504] *** Terminating app due to uncaught exception 'NSInternalInconsistencyException', reason: '+[NSUndoManager(NSInternal) _endTopLevelGroupings] is only safe to invoke on the main thread.'
*** First throw call stack:
(
	0   CoreFoundation                      0x00007fff551cc00b __exceptionPreprocess + 171
	1   libobjc.A.dylib                     0x00007fff7be59c76 objc_exception_throw + 48
	2   CoreFoundation                      0x00007fff551d1da2 +[NSException raise:format:arguments:] + 98
	3   Foundation                          0x00007fff572de260 -[NSAssertionHandler handleFailureInMethod:object:file:lineNumber:description:] + 193
	4   Foundation                          0x00007fff5726cdb4 +[NSUndoManager(NSPrivate) _endTopLevelGroupings] + 469
	5   AppKit                              0x00007fff5271de56 -[NSApplication run] + 997
	6   libpcl_visualization.1.8.dylib      0x0000000105611c50 _ZN3pcl13visualization13PCLVisualizer8spinOnceEib + 318
	7   libpcl_visualization.1.8.dylib      0x00000001056323a1 _ZN3pcl13visualization11CloudViewer16CloudViewer_implclEv + 699
	8   libboost_thread-mt.dylib            0x00000001017902ac _ZN5boost12_GLOBAL__N_1L12thread_proxyEPv + 156
	9   libsystem_pthread.dylib             0x00007fff7ccd46c1 _pthread_body + 340
	10  libsystem_pthread.dylib             0x00007fff7ccd456d _pthread_body + 0
	11  libsystem_pthread.dylib             0x00007fff7ccd3c5d thread_start + 13
)
libc++abi.dylib: terminating with uncaught exception of type NSException
```


解决方法：用[PCLVisualizer](http://pointclouds.org/documentation/tutorials/pcl_visualizer.php)代替CloudViewer

```c++
#include <pcl/visualization/pcl_visualizer.h>
...
pcl::visualization::PCLVisualizer viewer("viewer");
viewer.addPointCloud<pcl::PointXYZRGBA>(output);
while(!viewer.wasStopped())
{
	viewer.spinOnce();
}
```
类似的错误同样发生在**ORB_SLAM2**

```c++
auto resultFuture = async(launch::async, processing, argv, &SLAM);
```

异常原因：

- 线程**Thread**
- UI

------

### PCL 显示问题

导致异常的代码：

```c++
pcl::visualization::CloudViewer viewer("viewer");
...
if ( visualize == true )
	viewer.showCloud( cloud );
```

解决方法：用[PCLVisualizer](http://pointclouds.org/documentation/tutorials/pcl_visualizer.php)代替CloudViewer

参考程序：

```c++
pcl::visualization::PCLVisualizer viewer("viewer");
...
if ( visualize == true )
{
	viewer.removeAllPointClouds();
	viewer.addPointCloud( cloud, "hello");
	viewer.updatePointCloud( cloud, "hello" );
	viewer.spinOnce(0.0000000000001);
}
```

附赠一个bug：漏写函数返回值，运行时异常停止，提示函数的参数错误：

```shell
OpenCV Error: Assertion failed (npoints >= 4 && npoints == std::max(ipoints.checkVector(2, 5), ipoints.checkVector(2, 6))) in solvePnPRansac, file /Users/zhangjixiang/Downloads/opencv-3.3.1/modules/calib3d/src/solvepnp.cpp, line 252
libc++abi.dylib: terminating with uncaught exception of type cv::Exception: /Users/zhangjixiang/Downloads/opencv-3.3.1/modules/calib3d/src/solvepnp.cpp:252: error: (-215) npoints >= 4 && npoints == std::max(ipoints.checkVector(2, 5), ipoints.checkVector(2, 6)) in function solvePnPRansac
```
修改建议：

```c++
inline static CAMERA_INTRINSIC_PARAMETERS getDefaultCamera()
{
    ParameterReader pd;
    CAMERA_INTRINSIC_PARAMETERS camera;
    camera.fx = atof(pd.getData("camera.fx").c_str());
    camera.fy = atof(pd.getData("camera.fy").c_str());
    camera.cx = atof(pd.getData("camera.cx").c_str());
    camera.cy = atof(pd.getData("camera.cy").c_str());
    camera.scale = atof(pd.getData("camera.scale").c_str());
    return camera;// Bug
}
```
------

### 用好调试器 lldb/gdb

CLion IDE 优点和缺点一样明显：

- 代码补全
- 支持CMake
- 跳转
- 方便代码查看
- 慢，不能容忍！


故提出替代方案：**CMake + GCC + LLDB + Sublime Text**

替代方案的优点：

- 快，爽！
- 灵活
- 支持基本调试功能

IDE 不过是偷偷地帮你调用了 GCC，LLDB……

调试方法

```shell

$ gdb ./my_program          # Start GDB on your program
> run                       # Start running your program
...                         # Now reproduce the crash!
> bt                        # Obtain the backtrace
```



资料与参考文献：

1. [The LLDB Debugger](http://lldb.llvm.org/lldb-gdb.html)

------

### 编译器优化

slambook/project/0.4/CMakeLists.txt

```cmake
set( CMAKE_BUILD_TYPE "Debug" )
set( CMAKE_CXX_FLAGS "-std=c++11 -march=native -O3" )
```

导致运行时错误

```shell
Assertion failed: (( ((internal::UIntPtr(m_data) % internal::traits<Derived>::Alignment) == 0) || (cols() * rows() * innerStride() * sizeof(Scalar)) < internal::traits<Derived>::Alignment ) && "data is not aligned"), function checkSanity, file /usr/local/include/eigen3/Eigen/src/Core/MapBase.h, line 191.
```

修改CMakeLists.txt为

```cmake
set( CMAKE_BUILD_TYPE "Release" )
set( CMAKE_CXX_FLAGS "-std=c++11" )
```

可正常运行。原因：编译器**优化**导致的**内存对齐**错误？

但运行终止于

```shell
****** loop 347 ******
extract keypoints cost time: 0.008442
descriptor computation cost time: 0.008491
good matches: 10
match cost time: 0.001327
pnp inliers: 8
T_c_w_estimated_:
-0.0794355  -0.220607   0.972123    2.80445
 0.0189232   0.974695   0.222737    4.95924
  -0.99666  0.0360889 -0.0732508  -0.908356
         0          0          0          1
reject because inlier is too small: 8
VO costs time: 0.021043

****** loop 348 ******
extract keypoints cost time: 0.009256
descriptor computation cost time: 0.007454
good matches: 2
match cost time: 0.000996
OpenCV Error: Assertion failed (npoints >= 4 && npoints == std::max(ipoints.checkVector(2, 5), ipoints.checkVector(2, 6))) in solvePnPRansac, file /Users/zhangjixiang/Downloads/opencv-3.3.1/modules/calib3d/src/solvepnp.cpp, line 252
libc++abi.dylib: terminating with uncaught exception of type cv::Exception: /Users/zhangjixiang/Downloads/opencv-3.3.1/modules/calib3d/src/solvepnp.cpp:252: error: (-215) npoints >= 4 && npoints == std::max(ipoints.checkVector(2, 5), ipoints.checkVector(2, 6)) in function solvePnPRansac
```

原因：**匹配点数目不够**导致的程序终止。

Solution：增加匹配数目，将 100 增加为 200

```c++
if ( match_2dkp_index_.size() < 200 )
        addMapPoints();
```

### OpenGL 头文件

```c++
#if defined(__APPLE__)
#include <OpenGL/gl.h>
#include <OpenGL/glu.h>
#else
#include <GL/gl.h>
#include <GL/glu.h>
#endif
```

### LaTeX 排版技巧

##### 插入源代码

```latex
\usepackage{listings}
\usepackage{xcolor}
\lstset{
%	numbers=left, 
	numberstyle= \tiny, 
	keywordstyle= \color{ blue!70},
	commentstyle= \color{red!50!green!50!blue!50}, 
	frame=shadowbox, % 阴影效果
	rulesepcolor= \color{ red!20!green!20!blue!20} ,
	escapeinside=``, % 英文分号中可写入中文
%	xleftmargin=2em,xrightmargin=2em, aboveskip=1em,
%	framexleftmargin=2em
} 
...
\begin{lstlisting}
Hello World!
\end{lstlisting}
...
```

##### 插入参考文献

```latex
\cite{...}
...
\bibliographystyle{ieeetr}
\bibliography{name-of-bibtex-file}
```

##### 插入摘要关键字

```latex
\usepackage{abstract}
\providecommand{\keywords}[1]{\textbf{\textit{Index terms---}} #1}
...
\begin{abstract}
There is the abstract...
\end{abstract}
\begin{keywords}
Some keywords...
\end{keywords}
```

##### 正下方插入文字

```latex
\mathop{XXX}\limits_{XXX}
```

##### subfigures 插图

```latex
\documentclass{article}
\usepackage{graphicx}
\usepackage{caption}
\usepackage{subcaption}
\begin{document}
    \begin{figure}
        \centering
        \begin{subfigure}{0.4\textwidth} % width of left subfigure
            \includegraphics[width=\textwidth]{rncalt.png}
            \caption{RNC} % subcaption
        \end{subfigure}
        \vspace{1em} % here you can insert horizontal or vertical space
        \begin{subfigure}{0.4\textwidth} % width of right subfigure
            \includegraphics[width=\textwidth]{dncalt.png}
            \caption{DNC} % subcaption
        \end{subfigure}
        \caption{Wordcloud of national conventions} % caption for whole figure
    \end{figure}
\end{document}
```

##### 大括号方程组

```latex
\begin{equation}
\left\{
\begin{aligned}
a_{x}&=K_{A}\theta_{x}+K_{AV}\dot{\theta}_{x}+K_{T}(x-x_{0})+K_{V}v_{x}\\
a_{y}&=K_{A}\theta_{y}+K_{AV}\dot{\theta}_{y}+K_{T}(y-y_{0})+K_{V}v_{y}
\end{aligned}
\right.
\end{equation}
```

##### 收藏链接目录

```latex
\usepackage{hyperref}
```

##### 文中添加横线

```latex
\rule{\textwidth}{1mm}
```

##### 插入空行

```latex
\vspace{3ex}
```

##### 表格线加粗

```latex
\usepackage{booktabs}
\toprule[2pt]
\midrule[1pt]

\bottomrule[2pt]
```

##### 下括号

```latex
\underbrace{\frac{\Delta x_1}{\Delta p_1}}_{\text{Tot eff}}
```

##### 左上标

```latex
{}^{e}\dot{p}
```



### 计算运行时间(单线程)

```c++
#include <ctime>
...
clock_t time_stt = clock(); // 计时
...
cout <<"time use is " << 1000* (clock() - time_stt)/(double)CLOCKS_PER_SEC << "ms"<< endl;
```

### C++ 多线程计时勿用 clock()

> clock()函数的功能: 这个函数返回从“开启这个程序进程”到“程序中调用C++ clock()函数”时之间的CPU时钟计时单元（clock tick）数当程序单线程或者单核心机器运行时，这种时间的统计方法是正确的。但是如果要执行的代码多个线程并发执行时就会出问题，因为最终end-begin将会是多个核心总共执行的时钟嘀嗒数，因此造成时间偏大。

Solution

```c++
#include <chrono>
std::chrono::steady_clock::time_point t1 = std::chrono::steady_clock::now();
...
std::chrono::steady_clock::time_point t2 = std::chrono::steady_clock::now();
        chrono::duration<double> time_used = chrono::duration_cast<chrono::duration<double>>( t2-t1 );
        cout<<"time use in Tracking is "<< time_used.count() * 1000 <<"ms "<<endl;
```

### 固定翼—恒速—编队与跟踪

孙师兄回国给大家分享他博士期间的科研成果，强调**矩阵论**的重要性。讲座笔记

[https://matheecs.github.io/images/20180523.jpeg](/images/20180523.jpeg)

第二天上午师兄给课题组组织分享会，几点收获：掌握矩阵和优化，Just do it 赶走拖延症，英语很重要！

### 双目测距模块实现方案（TODO 暑假）

1. Zynq + xfOpenCV + Stereo Vision Pipeline
2. GPU TX2

在此基础上设计一款双目视觉教育机器人。

FPGA重新学习需要时间，但不想在硬件上面花太多时间，还是专注算法和创意。

每一个领域（双目视觉）都是坑，一不小心就跳下去出不来了～



### 思索 SLAM 的真正目的

1. 生成高精度地图
2. 用于导航，但导航真的需要高精度的地图吗
3. 同理，里程计的高精度对于一般的robot重要吗

或许robot不需要高精度的里程计和地图，robot应更像人一样，而不是机器。

所以想到设计一个 low resolution vision 的 autonomous navigation mobile robot～

### OpenCV 3.4 编译

树莓派需要先修改**软件源**、安装**依赖库**。

```shell
$ cmake \
-DCMAKE_BUILD_TYPE=RELEASE \
-DCMAKE_INSTALL_PREFIX=/usr/local \
-DOPENCV_EXTRA_MODULES_PATH=/Users/zhangjixiang/Downloads/opencv_contrib-3.4.0/modules \
-DWITH_CUDA=OFF \
-DBUILD_DOCS=OFF \
-DBUILD_EXAMPLES=OFF \
-DBUILD_TESTS=OFF \
-DBUILD_PERF_TESTS=OFF \
..

$ make -j4
$ sudo make install
```

### C++11/17 编译命令

```shell
$ clang++ -std=c++11 edX.cpp
$ clang++ -std=c++17 edX.cpp
```

### Valgrind安装(macOS High Sierra)

```shell
$ brew edit valgrind
	'https://sourceware.org/git/valgrind.git'->'git://sourceware.org/git/valgrind.git'
$ brew update
$ brew install --HEAD valgrind
```

参考 [How to Install Valgrind on macOS High Sierra](https://www.gungorbudak.com/blog/2018/04/28/how-to-install-valgrind-on-macos-high-sierra/)

使用方法

```shell
$ gcc main.cpp -g
$ valgrind --tool=memcheck --leak-check=full --show-reachable=yes ./a.out
```

### libcreate安装(macOS)

文件 libcreate/src/serial_query.cpp 中

```c++
tcflush(port.lowest_layer().native(), TCIFLUSH);
```

修改为

```c++
tcflush(port.lowest_layer().native_handle(), TCIFLUSH);
```

文件 libcreate/src/data.cpp 中添加

```c++
#include <iostream>
```

否则提示错误

```shell
error: no member named 'cout' in namespace 'std'
```

### Google Test安装

```shell
make
make install
```

[InstallingGoolgeTestForMac.wiki](https://code.google.com/archive/p/tonatiuh/wikis/InstallingGoolgeTestForMac.wiki)

### LattePanda 启动后显示屏信号中断

推测原因：电流 (1.8A) 不足。

> Any standard USB adapter (such as a cell phone wall charger) with at least 2A of current can be used as a power supply for the LattePanda.

[LattePanda](http://docs.lattepanda.com)



### Pixhawk 创建 App

添加 CMakeLists.txt 和 C 程序源文件后，需要修改文件：/Firmware/cmake/configs/nuttx_px4fmu-v2_default.cmake

```cmake
examples/px4_simple_app
```

### MYNTEYE 相机使用参考文件 CMakeLists

```cmake
cmake_minimum_required(VERSION 2.8)
project(mynteye_sdk_samples)

include(${PROJECT_SOURCE_DIR}/cmake/Common.cmake)
include(${PROJECT_SOURCE_DIR}/cmake/DetectCXX11.cmake)
include(${PROJECT_SOURCE_DIR}/cmake/DetectOpenCV.cmake)

# flags
SET(CMAKE_BUILD_TYPE Release)
set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -Wall -O3")
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall -O3")

# variables
set(SDK_DIR /Users/zhangjixiang/mynteye-1.8-mac-x64-opencv-3.4.0)
set(SDK_LIB_DIR ${SDK_DIR}/lib)
set(LIB_MYNTEYE_CORE ${CMAKE_SHARED_LIBRARY_PREFIX}mynteye_core${CMAKE_SHARED_LIBRARY_SUFFIX})

message(STATUS "SDK_DIR: ${SDK_DIR}")
message(STATUS "SDK_LIB_DIR: ${SDK_LIB_DIR}")
message(STATUS "LIB_MYNTEYE_CORE: ${LIB_MYNTEYE_CORE}")

# required
include_directories( ${SDK_DIR}/include )

# camera
add_executable(camera camera.cc)
target_link_libraries(camera
  ${SDK_LIB_DIR}/${LIB_MYNTEYE_CORE}
  ${OpenCV_LIBS}
)
```

C++ 源代码：

```c++
#include <iomanip>
#include <iostream>

#include <opencv2/highgui/highgui.hpp>

#include "camera.h"
#include "utility.h"

using namespace std;
using namespace mynteye;

int main(int argc, char const *argv[]) {
    std::string name;
    if (argc >= 2) {
        name = argv[1];
    } else {
        bool found = false;
        name = FindDeviceName(&found);
    }
    cout << "Open Camera: " << name << endl;

    Camera cam;
    InitParameters params(name);
    cam.Open(params);

    if (!cam.IsOpened()) {
        std::cerr << "Error: Open camera failed" << std::endl;
        return 1;
    }
    std::cout << "\033[1;32mPress ESC/Q on Windows to terminate\033[0m\n";

    double t, fps = 0;
    ErrorCode code;
    cv::Mat img_left;
    for (;;) {
        t = (double)cv::getTickCount();

        code = cam.Grab();

        if (code != ErrorCode::SUCCESS) continue;

        if (cam.RetrieveImage(img_left, View::VIEW_LEFT_UNRECTIFIED) == ErrorCode::SUCCESS) 
        {
            // top left: width x height
            stringstream ss;
            ss << img_left.cols << "x" << img_left.rows;
            DrawInfo(img_left, ss.str(), Gravity::TOP_LEFT);
            // top right: fps
            ss.str(""); ss.clear();
            ss << fixed << setw(7) << setprecision(2) << setfill(' ') << fps;
            DrawInfo(img_left, ss.str(), Gravity::TOP_RIGHT);
            cv::imshow("left", img_left);
        }

        char key = (char) cv::waitKey(1);
        if (key == 27 || key == 'q' || key == 'Q') {  // ESC/Q
            cv::imwrite("left.jpg",img_left);
            cout << "Saved Images!" << endl;
            break;
        }

        t = (double)cv::getTickCount() - t;
        fps = cv::getTickFrequency() / t;
    }

    cam.Close();
    cv::destroyAllWindows();
    return 0;
}
```

### 在 macOS 上安装 TensorFlow

```shell
$ pip install tensorflow      # Python 2.7; CPU support
```

### Host key verification failed...

```shell
$ ssh-keygen -R hostname
```

### 禁用 Win10 驱动签名

```
点击通知，找到并进入“所有设置”
在所有设置中找到并进入“更新和安全”
找到恢复，点击“高级启动”下的“立即重启”，重启电脑
重启后选择“疑难解答”
选择“高级选项”
选择“启动设置”
点击“重启”
按提示输入“7”禁用驱动程序强制签名
```

### 用pandoc实现LaTeX转Word

```shell
$ pandoc -s document.tex -o word.docx
```

### LaTeX导入pdf文档

```latex
\usepackage{pdfpages}
...
\includepdf[pages=-,pagecommand={},width=\textwidth]{file.pdf}
```

### 树莓派软件源配置

```shell
$ sudo nano /etc/apt/sources.list

deb http://mirrors.tuna.tsinghua.edu.cn/raspbian/raspbian/ stretch main non-free contrib
deb-src http://mirrors.tuna.tsinghua.edu.cn/raspbian/raspbian/ stretch main non-free contrib
```

### Raspbian Lite to Raspbian PIXEL

```shell
$ sudo apt-get install --no-install-recommends xserver-xorg
$ sudo apt-get install --no-install-recommends xinit
$ sudo apt-get install raspberrypi-ui-mods
$ sudo apt-get install lightdm
```

### 修改树莓派swap分区大小

```shell
$ sudo vim /etc/dphys-swapfile
...
修改 CONF_SWAPSIZE=2048
...
$ sudo /etc/init.d/dphys-swapfile restart
```

### 树莓派USB相机驱动

相机设备 /dev/video0

```
$ sudo modprobe bcm2835-v4l2
```



### 图像窗口打开后马上关闭

```c++
if ( (char) cv::waitKey(33) >= 0 )
			break;

修改为：

if ( cv::waitKey(33) >= 0 )
			break;
```

掌握 gdb 单步调试



### 打开swf文件

```html
<html>
	<body>
		<embed src="/Users/zhangjixiang/Downloads/dirtran_mintime_double_integrator_fine.swf" width="100%" height="100%"></embed>
	</body>
</html>
```



### Flask + Arduino Web应用程序

```python
import serial
from time import sleep
from flask import request

from flask import Flask
app = Flask(__name__)
ser = serial.Serial('/dev/tty.usbmodem1411', 9600, timeout=1)

@app.route("/", methods = ['POST', 'GET'])
def hello():
	if request.method == 'POST':
		print request.form['submit']
		# ser = serial.Serial('/dev/tty.usbmodem1411', 9600, timeout=1)
		if request.form['submit'] == 'ON':
			print '1'
			ser.write(b'1')
		elif request.form['submit'] == 'OFF':
			print '0'
			ser.write(b'0')
		else:
			pass # unknown
	return '<form action="/" method="POST"><input type="submit" name="submit" value="ON"><input type="submit" name="submit" value="OFF"></form>'

if __name__ == "__main__":
	# app.run()
	app.run(host='0.0.0.0')
```



### Arduino串口测试程序

```
import serial
import time
from struct import *

mega = serial.Serial(port='/dev/cu.usbmodem1421', 
    baudrate=9600, timeout = 3.0)

try:
    while True:
        raw = raw_input('input a char: ')
        mega.write(pack('c',raw))
        # read a '\n' terminated line
        line = mega.readline()
        print line
except Exception as e:
    mega.close()
```

### 手动添加Appimage应用图标

需要添加两个文件：文件和图标

1. /usr/share/applications/**xxx.desktop**，例如

   ```
   [Desktop Entry]
   Name=KDEnlive
   Comment=Non Linear Video Editor
   GenericName=Video Editor
   Exec=/home/ajith/Applications/Kdenlive-17.12.0d-x86_64.AppImage   #make sure to change the below path to the path in your system
   Icon=kdenlive   #the icon name for your application will be different
   Type=Application
   StartupNotify=true
   Categories=Qt;KDE;AudioVideo;AudioVideoEditing;
   MimeType=application/x-kdenlive;
   ```

2. /usr/share/icons/hicolor/512x512/apps/**xxx.png**

### 利用FileZilla SFTP传文件

SFTP = SSH File Transfer Protocol



### 打开Ubuntu SSH

```bash
$ sudo apt-get update
$ sudo apt-get install openssh-server
$ sudo ufw allow 22
```

### LaTeX 格式转换 Markdown

```bash
pandoc -s example4.tex -o example5.text
```

### Bochs启动黑屏

原因：debug模式

Solution：输入`c`，然后回车执行

```bash
<bochs:1> c
```

### Vim 调试：termdebug 入门

[Vim 调试：termdebug 入门](https://fzheng.me/2018/05/28/termdebug/)

### ImportError in system pip wrappers after an upgrade

```bash
$ python -m pip uninstall pip
```

### How do you calculate program run time in python?

```python
import timeit
start = timeit.default_timer()

#Statements

stop = timeit.default_timer()
print('Time: ', stop - start)
```

### WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!

```bash
$ ssh-keygen -R <host>
```

### Catch a Ctrl-C Event (C++)

```c++
#include <signal.h>
...

void myfunction(int sig) {
  ...
  exit(1);
}

int main(int argc, char** argv) {
  signal(SIGINT, myfunction);
  ...
}
```

### How do I find all files containing specific text on Linux?

```bash
查找所有文件中的内容
$ grep -rnw '/path/to/somewhere/' -e 'pattern'
$ grep -Ril "text-to-find-here" /

查找指定文件名
$ find /home/username/ -name "*.err"
```

### 设置代理服务器允许局域网的其他设备共享

ss 偏好设置->HTTP->HTTP代理监听地址->修改为`0.0.0.0`。

代理服务器主机名写运行 ss 的电脑的 IP 地址。

### Ubuntu创建sudo用户

```bash
$ sudo adduser username
$ usermod -aG sudo username
$ su - username
```

### ROS生成msg/srv/action后编译package

```bash
$ catkin_make messages_generate_messages
$ catkin_make
```

### Ubuntu系统下查看OpenCV版本

```bash
$ dpkg -l | grep libopencv
```

### 查看CUDA版本

```bash
$ nvcc --version
```

### 查看CuDNN版本

```bash
$ cat /usr/local/cuda/include/cudnn.h | grep CUDNN_MAJOR -A 2
```

### 使用pkg-config

```bash
$ pkg-config --cflags librtmp
$ pkg-config --libs librtmp

常用参数：
–-list-all     列出所有已安装的共享库
-–cflags     列出指定共享库的预处理和编译flag
-–libs     列出指定共享库的链接flag
```

### 用 Python & matplotlib 绘图

```python
import re
import numpy as np
import matplotlib as mpl
import matplotlib.pyplot as plt

# read the log file
files = ['00.txt', '01.txt', '02.txt', '03.txt']
iters = [8000, 80000, 8000, 20000]
descr = ['00iter: [8000,4000,8000,4000]         base_lr: 0.001', '01iter: [80000,40000,80000,40000] base_lr: 0.0005', '02iter: [8000,4000,8000,4000]         base_lr: 0.0005', '03iter: [20000,10000,20000,10000] base_lr: 0.0005']
color = ['green', 'red', 'skyblue', 'blue']
plt.title('Result Analysis')

for i in range(4):
  file_log = open(files[i], 'r')
  train_iterations = []
  train_loss = []
  for line in file_log:
    if '] Iteration ' in line and 'loss = ' in line:
      arr = re.findall(r'ion \b\d+\b',line)
      train_iterations.append(int(arr[0].strip(',')[4:]))
      train_loss.append(float(line.strip().split(' = ')[-1]))
  file_log.close()
  print len(train_iterations)
  plt.plot(train_iterations[0:iters[i] / 20], train_loss[0:iters[i] / 20], color=color[i], label=descr[i])

plt.legend()
plt.xlabel('iteration times')
plt.ylabel('loss')
plt.xlim(-20, 2000)
plt.show()
```

### 用 MATLAB 添加高斯和椒盐噪音

```matlab
% add gauss and peper noise
% 2019-02-23
clc
clear all

XmlPath = 'dataset/xml/';
XmlFile = dir(fullfile(XmlPath, '*.xml'));
XmlNames = transpose({XmlFile.name});
Amount = size(XmlNames, 1);
PicPath = 'dataset/';

for k = 1:Amount
    filename = fullfile(XmlPath, XmlNames(k));
    XDoc = xmlread(char(filename));
    xmin = XDoc.getElementsByTagName('object').item(0).getElementsByTagName('xmin').item(0).getFirstChild.getNodeValue; xmin = str2num(xmin);
    ymin = XDoc.getElementsByTagName('object').item(0).getElementsByTagName('ymin').item(0).getFirstChild.getNodeValue; ymin = str2num(ymin);
    xmax = XDoc.getElementsByTagName('object').item(0).getElementsByTagName('xmax').item(0).getFirstChild.getNodeValue; xmax = str2num(xmax);
    ymax = XDoc.getElementsByTagName('object').item(0).getElementsByTagName('ymax').item(0).getFirstChild.getNodeValue; ymax = str2num(ymax);
    ap = [xmin, ymin, xmax, ymax]
    
    XmlName = char(XmlNames(k));
    PicName = strcat(XmlName(1:6), '.jpg');
    
    origin = imread(PicName);
    % r = [0 ~ 0.2]
    r = randi([0 200],1,3)/1000
    I_0 = imnoise(origin, 'salt & pepper', r(1));
    Im = imnoise(I_0, 'gaussian', r(2), r(3));
    [rNum,cNum] = size(Im);
    x1 = (xmin+xmax)/2; y1 = (ymin+ymax)/2; radius = (xmax+ymax-xmin-ymin)/4;
    [xx,yy] = ndgrid((1:rNum)-y1,(1:cNum)-x1);
    mask = (xx.^2 + yy.^2) >= radius^2+10^2;
    Im(mask) = uint8(0);
    imwrite(Im, fullfile('newdataset/', PicName))
end
```

### C/C++字符串分割

```c++
/* strtok example */
#include <stdio.h>
#include <string.h>

int main ()
{
  char str[] ="- This, a sample string.";
  char * pch;
  printf ("Splitting string \"%s\" into tokens:\n",str);
  pch = strtok (str," ,.-");
  while (pch != NULL)
  {
    printf ("%s\n",pch);
    pch = strtok (NULL, " ,.-");
  }
  return 0;
}
```

### Latency Numbers Every Programmer Should Know

```
Latency Comparison Numbers (~2012)
----------------------------------
L1 cache reference                           0.5 ns
Branch mispredict                            5   ns
L2 cache reference                           7   ns                      14x L1 cache
Mutex lock/unlock                           25   ns
Main memory reference                      100   ns                      20x L2 cache, 200x L1 cache
Compress 1K bytes with Zippy             3,000   ns        3 us
Send 1K bytes over 1 Gbps network       10,000   ns       10 us
Read 4K randomly from SSD*             150,000   ns      150 us          ~1GB/sec SSD
Read 1 MB sequentially from memory     250,000   ns      250 us
Round trip within same datacenter      500,000   ns      500 us
Read 1 MB sequentially from SSD*     1,000,000   ns    1,000 us    1 ms  ~1GB/sec SSD, 4X memory
Disk seek                           10,000,000   ns   10,000 us   10 ms  20x datacenter roundtrip
Read 1 MB sequentially from disk    20,000,000   ns   20,000 us   20 ms  80x memory, 20X SSD
Send packet CA->Netherlands->CA    150,000,000   ns  150,000 us  150 ms

Notes
-----
1 ns = 10^-9 seconds
1 us = 10^-6 seconds = 1,000 ns
1 ms = 10^-3 seconds = 1,000 us = 1,000,000 ns

Credit
------
By Jeff Dean:               http://research.google.com/people/jeff/
Originally by Peter Norvig: http://norvig.com/21-days.html#answers

Contributions
-------------
'Humanized' comparison:  https://gist.github.com/hellerbarde/2843375
Visual comparison chart: http://i.imgur.com/k0t1e.png
```

### 用 [cloc](https://github.com/AlDanial/cloc) 统计代码量

```bash
prompt> cloc gcc-5.2.0/gcc/c
      16 text files.
      15 unique files.
       3 files ignored.

https://github.com/AlDanial/cloc v 1.65  T=0.23 s (57.1 files/s, 188914.0 lines/s)
-------------------------------------------------------------------------------
Language                     files          blank        comment           code
-------------------------------------------------------------------------------
C                               10           4680           6621          30812
C/C++ Header                     3             99            286            496
-------------------------------------------------------------------------------
SUM:                            13           4779           6907          31308
-------------------------------------------------------------------------------
```

### 符号计算小工具 SymPy

SymPy is a Python library for symbolic mathematics. It aims to become a full-featured computer algebra system (CAS) while keeping the code as simple as possible in order to be comprehensible and easily extensible. SymPy is written entirely in Python.

### Keynote 中插入高亮代码

```bash
$ brew install highlight
$ pbpaste | highlight --syntax=c++ -K 17 -u "utf-8" -t 2 -n -O rtf | pbcopy
```

### macOS 终端报错 ValueError: unknown locale: UTF-8

```bash
export LC_ALL=en_US.UTF-8
export LANG=en_US.UTF-8
```

### 理解 GitHub

[https://matheecs.github.io/images/Git.jpg](/images/Git.jpg)

### 使用 CMake/CPack/CTest/Catkin

### IMU模型

[https://matheecs.github.io/images/imumodel.jpg](/images/imumodel.jpg)

### 安装LaTeX依赖包

```bash
sudo tlmgr install name-of-package
```

### 学会用 Glog, Gflags, Gtest, Doxygen, GitHub CI/CD, Protobuf, clang-format

### 解决安装`ros-kinetic-desktop-full`失败

Solution：保留`/etc/apt/sources.list`默认配置，不要覆盖原有内容

### vim中添加`sudo`权限保存文档

```bash
:w !sudo tee %
```

### Ubuntu安装中文字体解决PDF文档无法显示

```bash
$ sudo apt-get install language-pack-zh*
$ sudo apt-get install chinese*
$ sudo apt-get install fonts-arphic-ukai fonts-arphic-uming fonts-ipafont-mincho fonts-ipafont-gothic fonts-unfonts-core
```

### 采用Homebrew国内源

```bash
替换brew.git:
cd "$(brew --repo)"
git remote set-url origin https://mirrors.ustc.edu.cn/brew.git

替换homebrew-core.git:
cd "$(brew --repo)/Library/Taps/homebrew/homebrew-core"
git remote set-url origin https://mirrors.ustc.edu.cn/homebrew-core.git
```

重置

```bash
重置brew.git:
cd "$(brew --repo)"
git remote set-url origin https://github.com/Homebrew/brew.git

重置homebrew-core.git:
cd "$(brew --repo)/Library/Taps/homebrew/homebrew-core"
git remote set-url origin https://github.com/Homebrew/homebrew-core.git
```

### zsh + Oh My Zsh

macOS Catalina 开始默认采用 zsh，配合 [Oh My Zsh](https://github.com/robbyrussell/oh-my-zsh) 十分好用。

### 用终端命令烧录树莓派镜像

```bash
$ sudo diskutil umount /dev/disk2s1
$ sudo dd bs=8m if=./2019-09-26-raspbian-buster-full.img of=/dev/rdisk2
```

### pip3使用国内源

```bash
pip3 config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple
```

[pypi 镜像使用帮助](https://mirror.tuna.tsinghua.edu.cn/help/pypi/)