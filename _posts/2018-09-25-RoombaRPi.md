---
layout: post
title: Roomba变形记
---

去年在密村购了一台 iRobot  Create 2（Roomba 600），最近打算对它改造一下：加个脑子（Raspberry Pi）和眼睛（单目相机）。改造结果：

![](/images/roomba00.jpeg)

![](/images/roomba01.jpeg)

### 电源

采用航模3S锂电池，12V$\to$5V/3A DC/DC转换器，通过引脚4（5V）、引脚6（Ground）供电。



### 通信

Roomba$\leftrightarrow$树莓派 之间采用 Serial$\leftrightarrow$USB 的方式通信，利用自带的转换线，避免了转换电平（Roomba 5V而树莓派串口3.3V）的麻烦。

配置树莓派工作在 AP 热点模式，实现即时在没有路由器的环境下，也可用 SSH 方式远程登录树莓派。



### “大脑”

考虑到成本问题，采用了Raspberry Pi 3b+。TX1/2 当然是更好的选择。



### “眼睛”

TODO：RPi Camera（G），OV5647，160度视场角，同样是考虑到成本问题。全局曝光工业相机 MT9V034 是更好的选择。

TODO：IR 测距传感器，用于绘画二维地图。



### 机器人和计算机的一个区别

计算机：GIGO

机器人：**Graceful Degradation**

##### 测试程序

```python
import serial
from time import sleep
from struct import *

def wakeRobot(ser):
    ser.rts = True
    sleep(0.1)
    ser.rts = False
    sleep(1)

def resetRobotAndWait(ser):
    ser.reset_input_buffer()
    ser.write('\x07')
    sBuffer = "dummy string"
    while len(sBuffer) is not 0:
        sBuffer = ser.readline()
        print sBuffer.strip()

irobot = serial.Serial(port='/dev/tty.usbserial-DA01NU2A', baudrate=115200, timeout = 3.0)
wakeRobot(irobot)
irobot.write(pack('B',128))
print "Start iRobot..."

# Safe Mode
irobot.write(pack('B',131))
sleep(.1)

# Play Song
irobot.write(pack('B'+'c'*4, 164,'1','2','3','4'))
irobot.write(pack('B'*3+'cB'*5,140,0,5,'C',16,'H',24,'J',8,'L',16,'O',32))
irobot.write(pack('B'*2,141, 0))
sleep(1.6)

# Sensor('>h', '>H')
# irobot.write(pack('B'*2,142, 21))
# print "char:", unpack('B',irobot.read(1))
# irobot.write(pack('B'*2,142, 22))
# print "volt:", unpack('>H',irobot.read(2))
# irobot.write(pack('B'*2,142, 23))
# print "curr:", unpack('>h',irobot.read(2))
# irobot.write(pack('B'*2,142, 24))
# print "temp:", unpack('b',irobot.read(1))
# irobot.write(pack('B'*2,142, 25))
# print "batt:", unpack('>H',irobot.read(2))
# irobot.write(pack('B'*2,142, 26))
# print "Batt:", unpack('>H',irobot.read(2))
# irobot.write(pack('B'*2,142, 3))
# print "G3:", unpack('>BHhbHH',irobot.read(10))
# irobot.write(pack('B'*2,142, 107))
# print "G107:", unpack('>h'+'h'*3+'B',irobot.read(9))


try:
    while True:

        # Actuator
        irobot.write(pack('>Bhh',145, 20,20))
        irobot.write(pack('BB',142, 58))
        print "ID58:", unpack('>B',irobot.read(1))
        sleep(.2)
        irobot.write(pack('>Bhh',145, 0,0))


        # irobot.write(pack('BB',142, 17))
        # print "ID17:", unpack('>B',irobot.read(1))
        # sleep(.2)         
        # irobot.write(pack('BB',142, 52))
        # print "ID52:", unpack('>B',irobot.read(1))
        # sleep(.2)        
        # irobot.write(pack('BB',142, 53))
        # print "ID53:", unpack('>B',irobot.read(1))
        # sleep(.2)
        # irobot.write(pack('BB',142, 46))
        # print "ID46:", unpack('>H',irobot.read(2))
        # sleep(.2)        
        # irobot.write(pack('BB',142, 47))
        # print "ID47:", unpack('>H',irobot.read(2))
        # sleep(.2)
        # irobot.write(pack('BB',142, 48))
        # print "ID48:", unpack('>H',irobot.read(2))
        # sleep(.2)
        # irobot.write(pack('BB',142, 49))
        # print "ID49:", unpack('>H',irobot.read(2))
        # sleep(.2)
        # irobot.write(pack('BB',142, 50))
        # print "ID50:", unpack('>H',irobot.read(2))
        # sleep(.2)
        # irobot.write(pack('BB',142, 51))
        # print "ID51:", unpack('>H',irobot.read(2))
        # sleep(.2)        
except:
    # Start
    irobot.write(pack('B',128))
    print "\nPause iRobot..."
    irobot.close()
```



##### 参考文献

1. [Pinout.xyz](https://pinout.xyz/pinout/ground)