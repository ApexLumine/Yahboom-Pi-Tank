# Yahboom-Pi-Tank 操作指南

> By [GamerNoTitle](https://github.com/GamerNoTitle)

## 安装系统

这里推荐Ubuntu类（我用的**Kali**，其他也可以，但下面是以Ubuntu为系统作为前提条件的，建议装最新的LTS版本），直接用balenaEtcher去刷写，直接写进卡里就行了，卡建议16G往上（说实话16G不太够，后面要编译东西的话软件包要下很多）

首次开机记得连接屏幕初始化一下（Kali是需要的，Ubuntu不清楚）

## 接口详情

|    分类    |   功能    | 树莓派 | wiringPi |
| :--------: | :-------: | :----: | :------: |
| 小车运动类 | 左电机前  |  IO20  |    28    |
| 小车运动类 | 左电机后  |  IO21  |    29    |
| 小车运动类 | 右电机前  |  IO19  |    24    |
| 小车运动类 | 右电机后  |  IO26  |    25    |
| 小车运动类 | 左电机PWM |  IO16  |    27    |
| 小车运动类 | 右电机PWM |  IO13  |    23    |
| 巡线传感器 |    左1    |  IO3   |    9     |
| 巡线传感器 |    左2    |  IO5   |    21    |
| 巡线传感器 |    右1    |  IO4   |    7     |
| 巡线传感器 |    右2    |  IO18  |    1     |
|  红外避障  |    左     |  IO12  |    26    |
|  红外避障  |    右     |  IO17  |    0     |
| 寻光传感器 |    左     |  IO7   |    11    |
| 寻光传感器 |    右     |  IO6   |    12    |
|    按键    |   按键    |  IO8   |    10    |
|   蜂鸣器   |  蜂鸣器   |  IO8   |    10    |
| 七彩探照灯 |   红色    |  IO22  |    3     |
| 七彩探照灯 |   绿色    |  IO27  |    2     |
| 七彩探照灯 |   蓝色    |  IO24  |    5     |
|    云台    |   云台    |  IO23  |    4     |
|   超声波   |   发送    | ID_SC  |    30    |
|   超声波   |   接收    | ID_SD  |    31    |
|  红外遥控  | 红外遥控  |  IO2   |    8     |
|    PS2     |   MOSI    |  IO10  |    12    |
|    PS2     |   MISO    |  IO9   |    13    |
|    PS2     |    GND    |  GND   |   GND    |
|    PS2     |    VCC    |  VCC   |   VCC    |
|    PS2     |    CS     |  IO25  |    6     |
|    PS2     |    SCK    |  IO11  |    14    |

## 安装依赖

树莓派最重要的就是WiringPi库，对于树莓派来说，可以直接通过apt安装

```shell
$ sudo apt install wiringpi -y
```

如果apt无法安装（服务器抽风啥的），可以自己获取deb包安装

```shell
$ wget https://project-downloads.drogon.net/wiringpi-latest.deb
$ sudo dpkg -i wiringpi-latest.deb
```

如果网络不好的话，考虑挂个梯子吧~

安装完后可以使用下面的命令看版本以及接口信息

```shell
$ gpio -v
$ gpio readall
```

## 编译程序

使用`gcc`可以编译我们的`c`语言程序，输入下面的命令进行编译

```shell
$ gcc Control.c -o Control -lwiringPi
```

如果提示`gcc`无法找到的话，记得先`apt`一下

编译完后就可以在当前目录找到我们的`Control`可执行文件，如果没有被赋予可执行权限的话，请手动赋予

```shell
$ chmod +x Control
```

最后使用`./Control`就可以启动了，如果提示IP有问题，则需要手动更改程序里面的IP

在`Control.c`，找到下面的内容（大概在947行）

```c
memset(&my_addr,0,sizeof(my_addr));	
my_addr.sin_family = AF_INET;
my_addr.sin_port = htons(atoi("8888"));
my_addr.sin_addr.s_addr = inet_addr("0.0.0.0");
```

这里`8888`是程序使用的端口，`0.0.0.0`是程序监听的地址（下面的例子以这两个为准），修改完以后重新进行编译就可以了

**强烈建议把程序注册为系统服务，这样开机就可以自己启动了**

## 控制指令集

| 通用协议 |       | 运动状态 |        | 旋转    |        | 鸣笛    |        | 加减速  |        | 云台舵机 |        | 唱歌    |        | 点灯    |        | 灭火     |        | 舵机复位 |        |
| -------- | ----- | -------- | ------ | ------- | ------ | ------- | ------ | ------- | ------ | -------- | ------ | ------- | ------ | ------- | ------ | -------- | ------ | -------- | ------ |
|          | 1     | 2        | 3      | 4       | 5      | 6       | 7      | 8       | 9      | 10       | 11     | 12      | 13     | 14      | 15     | 16       | 17     | 18       | 19     |
|          | 包头0 | 命令位1  | 分隔号 | 命令位2 | 分隔号 | 命令位3 | 分隔号 | 命令位4 | 分隔号 | 命令位6  | 分隔号 | 数据位8 | 分隔号 | 数据位9 | 分隔号 | 数据位10 | 分隔号 | 数据位11 | 包尾22 |
| 停止     | $     | 0        | ，     | 0       | ，     | 0       | ，     | 0       | ，     | 0        | ，     | 0       | ，     | 0       | ，     | 0        | ，     | 0        | #      |
| 前进     | $     | 1        | ，     | 0       | ，     | 0       | ，     | 0       | ，     | 0        | ，     | 0       | ，     | 0       | ，     | 0        | ，     | 0        | #      |
| 后退     | $     | 2        | ，     | 0       | ，     | 0       | ，     | 0       | ，     | 0        | ，     | 0       | ，     | 0       | ，     | 0        | ，     | 0        | #      |
| 左转     | $     | 3        | ，     | 0       | ，     | 0       | ，     | 0       | ，     | 0        | ，     | 0       | ，     | 0       | ，     | 0        | ，     | 0        | #      |
| 右转     | $     | 4        | ，     | 0       | ，     | 0       | ，     | 0       | ，     | 0        | ，     | 0       | ，     | 0       | ，     | 0        | ，     | 0        | #      |
| 左旋转   | $     | 0        | ，     | 1       | ，     | 0       | ，     | 0       | ，     | 0        | ，     | 0       | ，     | 0       | ，     | 0        | ，     | 0        | #      |
| 右旋转   | $     | 0        | ，     | 2       | ，     | 0       | ，     | 0       | ，     | 0        | ，     | 0       | ，     | 0       | ，     | 0        | ，     | 0        | #      |
| 鸣笛     | $     | 0        | ，     | 0       | ，     | 1       | ，     | 0       | ，     | 0        | ，     | 0       | ，     | 0       | ，     | 0        | ，     | 0        | #      |
| 加速     | $     | 0        | ，     | 0       | ，     | 0       | ，     | 1       | ，     | 0        | ，     | 0       | ，     | 0       | ，     | 0        | ，     | 0        | #      |
| 减速     | $     | 0        | ，     | 0       | ，     | 0       | ，     | 2       | ，     | 0        | ，     | 0       | ，     | 0       | ，     | 0        | ，     | 0        | #      |
| 左摇     | $     | 0        | ，     | 0       | ，     | 0       | ，     | 0       | ，     | 1        | ，     | 0       | ，     | 0       | ，     | 0        | ，     | 0        | #      |
| 右摇     | $     | 0        | ，     | 0       | ，     | 0       | ，     | 0       | ，     | 2        | ，     | 0       | ，     | 0       | ，     | 0        | ，     | 0        | #      |

位置需要对应，例如我想要前进，则需要TCP客户端发送`$1,0,0,0,0,0,0,0,0#`这样的内容过去，就可以前进了（别问我为啥后面有那么多的0，设计好了但是没用上而已）

这里是个TCP客户端的例子（Python）

```python
from socket import *
 
IP='<Your IP Address>'
SEVER_PORT=<Your Port>
BUFLEN=512
 
dataSocket=socket(AF_INET,SOCK_STREAM)

forward = '$1,0,0,0,0,0,0,0,0#'
backward = '$2,0,0,0,0,0,0,0,0#'
turnleft = '$3,0,0,0,0,0,0,0,0#'
turnright = '$4,0,0,0,0,0,0,0,0#'
rotateleft = '$0,1,0,0,0,0,0,0,0#'
rotateright = '$0,2,0,0,0,0,0,0,0#'
stop = '$0,0,0,0,0,0,0,0,0#'

#连接服务端socket
dataSocket.connect((IP,SEVER_PORT))
while True:
    tosend=input('>>')
    if tosend=='exit':
        break
    # 下面是绑定快捷指令
    elif tosend = 'w':	# 前进
        tosend = forward
    elif tosend = 's':	# 后退
        tosend = backward
    elif tosend = 'a':	# 左转
        tosend = turnleft
    elif tosend = 'd':	# 右转
        tosend = turnright
    elif tosend = 'stop':	# 停止
        tosend = stop
    elif tosend = 'rl':	# 左转(rotate left)
    	tosend = rotateleft
    elif tosend = 'rr':	# 右转(rotate right)
        tosend = rotateright
    # 快捷指令结束
    #客户端发送消息，编码为bytes
    dataSocket.send(tosend.encode())
    #等待服务端发送消息
    recved=dataSocket.recv(BUFLEN)
    if not recved:
        break
    #打印来自服务端的消息
    print(recved.decode())
dataSocket.close()
```

## 摄像头

插上摄像头后，使用如下命令检查是否正确识别

```shell
$ lsusb
```

如果发现有设备号为6001的设备则为正确识别

接着使用以下命令查看是否可以正常使用

```shell
$ ls /dev
```

在里面尝试查找`video0`设备，如果找到了就说明可以用，找不到的话（或者只有`video1`）就需要重启一下π，或者换个USB口

实在是不行，就重新刷一下固件

官方固件：https://pan.baidu.com/s/1wyhS25KU_fhA_iPXO16ZGA 提取码：zgxx

有了摄像头，我们还需要能够使用摄像头的软件

[jacksonliam/mjpg-streamer: Fork of http://sourceforge.net/projects/mjpg-streamer/ (github.com)](https://github.com/jacksonliam/mjpg-streamer)

用这个仓库里面的代码，跟着README编译使用就行了