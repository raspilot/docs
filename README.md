#树莓派飞控RasPilot#

###RasPilot硬件参数###
特性
* 支持树莓派1 A+/B+；树莓派2 B+
* STM32F103 failsafe co-processor
* ardupilot飞控代码
* pixhawk接口兼容
* USB(RPi)/Servo Rail BEC/Power Module三路电源输入

传感器
* ST Micro L3GD20H 16位陀螺仪
* ST Micro LSM303D 14位加速度计/磁力计
* Invensense MPU 6000 3轴陀螺仪/加速度计
* MEAS MS5611 气压计

接口
* 2x UART
* 1x CAN
* 8x PWM
* Spektrum DSM / DSM2 / DSM-X® Satellite 兼容接收机
* Futaba S.BUS® 兼容输入输出
* PPM sum 接收机
* RSSI（PWM或电压）输入
* 1x I2C
* 3.3V & 6.6V A/D输入
* 安全开关
* 蜂鸣器接口
* 板载三色LED

###硬件安装和接线###
1 安装外壳。如下图，用铜柱和螺母将树莓派固定在底壳上。插上飞控扩展板，再用螺丝固定上壳。<br>
![](https://github.com/raspilot/docs/blob/master/raspilot_p1.jpg)
![](https://github.com/raspilot/docs/blob/master/raspilot_p2.jpg)<br>
<br>
2 接口说明：<br>
![](https://github.com/raspilot/docs/blob/master/connectors.jpg)<br>

###写入系统镜像到树莓派SD卡###
树莓派官方的Linux系统实时性不适合运行飞控程序，因此要使用打了RT-patch的系统镜像。<br>
首先下载和解压Linux系统的镜像文件。目前还没来得及自己编译，暂时借用emlid打包的镜像。<br>
<br>
　树莓派1代：https://mega.co.nz/#!RVJxHJpI!QVPTZaNY0AiuPbcxQjOTmZ2un6d0j7W3g1jwheuotUc<br>
　树莓派2代：https://mega.co.nz/#!0VZFzbwC!6tTzWFKl8jdR4Q52A9A03wYyAPghIDKtxGpavNMBKn4<br>
<br>
然后按照树莓派的官方教程将镜像文件写入SD卡：<br>
<br>
　Linux：https://www.raspberrypi.org/documentation/installation/installing-images/linux.md<br>
　Mac OS：https://www.raspberrypi.org/documentation/installation/installing-images/mac.md<br>
　Windows：https://www.raspberrypi.org/documentation/installation/installing-images/windows.md<br>
　其它帮助文档：https://www.raspberrypi.org/documentation/<br>
<br>
或者可以百度，有很多中文教程。<br>

###配置树莓派WiFi连接###
编辑/etc/wpa_supplicant/wpa_supplicant.conf文件<br>
　`sudo nano /etc/wpa_supplicant/wpa_supplicant.conf`<br>
将文件中的<br>
　`ssid = “emlidltd”`<br>
　`psk = “emlidltd”`<br>
　`key_mgmt = wpa-psk`<br>
改为自己WiFi的SSID和密码。<br>

###安装和设置飞控程序###
1 下载飞控程序<br>
　ArduCopter.elf 百度网盘下载地址：http://pan.baidu.com/s/1hqJ9ZbY （1、2代树莓派均可使用）<br>
　下载后放到树莓派用户根目录，即/home/pi目录中<br>
<br>
2 运行飞控程序<br>
　`sudo ./ArduCopter.elf -B /dev/ttyAMA0`<br>
　ArduCopter.elf可以带-A -B -C -E四个参数，其中：<br>
　* -A primary数传接口<br>
　* -B primary GPS接口<br>
　* -C secondary数传接口<br>
　* -E sencondary GPS接口<br>
　raspilot中：-B 即GPS的串口使用树莓派自带的串口`/dev/ttyAMA0`，波特率自适应；<br>
　-C 默认是STM32扩展的串口，作为数传接口。波特率默认为57600，可以在Mission Planner界面中进行配置。<br>
<br>
　如果要使用WiFi的udp作为数传的话，可以输入：<br>
　`sudo ./ArduCopter.elf -A udp:192.168.1.100:14550 -B /dev/ttyAMA0`<br>
　其中`192.168.1.100`改为PC端的IP地址<br>
<br>
3 设置开机自动运行<br>
　修改/etc/rc.local文件，在`exit 0`前加入：<br>
　`sudo ./home/pi/ArduCopter.elf -B /dev/ttyAMA0 > /home/pi/startup_log &`<br>
　重新启动后飞控程序就会自动运行<br>

###编译飞控代码###
从Copter-3.3开始，ardupilot的编译需要使用4.7版本以上的gcc。而树莓派系统上还是4.6版本的gcc，所以建议在Ubuntu下面交叉编译代码。
这里使用树莓派官方提供的交叉编译工具<br>
<br>
1 下载树莓派编译工具，例如这里放到/opt目录下<br>
　`sudo git clone --depth 1 https://github.com/raspberrypi/tools.git /opt/tools`<br>
<br>
2 设置环境变量<br>
　32位系统输入：`export PATH=/opt/tools/arm-bcm2708/gcc-linaro-arm-linux-gnueabihf-raspbian/bin:$PATH`<br>
　64位系统输入：`export PATH=/opt/tools/arm-bcm2708/gcc-linaro-arm-linux-gnueabihf-raspbian-x64/bin:$PATH`<br>
<br>
3 编译ardupilot<br>
* 从GitHub上获取源代码：<br>
　`git clone https://github.com/raspilot/ardupilot-raspilot.git`<br>
* 进入ArduCopter目录后编译：<br>
　`cd ardupilot-raspilot/ArduCopter`<br>
　`make raspilot`<br>
* 通过WiFi同步到树莓派：<br>
　`rsync -avz /tmp/ArduCopter.build/ArduCopter.elf pi@192.168.1.100:/home/pi/`<br>
　其中`192.168.1.100`改为树莓派的实际IP地址<br>
