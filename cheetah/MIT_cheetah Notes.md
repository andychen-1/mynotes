
## WSL2 Ubuntu 22.4 中使用游戏手柄

### 连接 USB 驱动

1.  安装 [USBIPD-WIN](https://github.com/dorssel/usbipd-win/releases)，将 Windows 已识别的 USB 设备转接到 WSL

2.  通过 usbipd 工具将指定的设备附加到 WSL

```bash
# 查看当前 USB 设备
$ usbipd list 
~ Connected:  
~ BUSID VID:PID DEVICE                       STATE  
~ 1-3 1ea7:0066 USB 输入设备                  Not shared  
~ 1-6 0408:1020 hm1091_techfront             Not shared  
~ 1-7 8087:0a2b 英特尔(R) 无线 Bluetooth(R)   Not shared
# 强制绑定 BUSID=1-3 的设备
$ usbipd bind -b 1-3 -f
$ usbipd list
~ Connected:  
~ BUSID VID:PID DEVICE                       STATE  
~ 1-3 1ea7:0066 USB 输入设备                  Shared (forced)  
~ 1-6 0408:1020 hm1091_techfront             Not shared  
~ 1-7 8087:0a2b 英特尔(R) 无线 Bluetooth(R)   Not shared
# 附加设备至 wsl 客户端
$ usbipd attach -b 1-3 -w
$ usbipd list
~ Connected:  
~ BUSID VID:PID DEVICE                       STATE  
~ 1-3 1ea7:0066 USB 输入设备                  Attached  
~ 1-6 0408:1020 hm1091_techfront             Not shared  
~ 1-7 8087:0a2b 英特尔(R) 无线 Bluetooth(R)   Not shared
# 成功附加 usb 设备后，WSL中执行 lsusb 时，该设备可见
$ wsl
$ lsusb
~ Bus 002 Device 001: ID 1d6b:0003 Linux Foundation 3.0 root hub  
~ Bus 001 Device 002: ID 1ea7:0066  d
~ Bus 001 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
# 解除附加
$ usbipd detach -b 1-3
# 解除绑定
$ usbipd unbind -b 1-3
```

### 替换 WSL Ubuntu 内核

1. 下载添加了 xpad 的内核[映射文件](https://github.com/microsoft/WSL/files/10098030/kernel-xpad.zip)

2.  添加 .wsconfig 配置（.wsconfig 与 bzImage 都应保存在用户根目录下）

```config
[wsl2]
kernel=C:\\Users\\<username>\\bzImage
```

3.  重新进入 wsl

```bash
$ wsl --shutdown
$ wsl
```

### 安装设备调试工具

```bash
sudo apt-get install jstest-gtk
```


## 机器狗自启动脚本


```bash
# ~/<project>/scripts/run_mc.sh

#!/bin/bash
# enable multicast and add route for lcm out the top

sudo ifconfig enp1s0 multicast
sudo route add -net 224.0.0.0 netmask 240.0.0.0 dev enp1s0

# configure libraries
sudo LD_LIBRARY_PATH=. ldconfig
#sudo LD_LIBRARY_PATH=. ldd ./robot
sudo LD_LIBRARY_PATH=. $1 m r f l &

sudo LD_LIBRARY_PATH=. $2 &
```

```bash
#!/bin/bash
# rc.local
cd /home/robot/robot-software/build/
./run_mc.sh ./mit_ctrl ./speech_controller
exit 0
```

更多详细内容，请参考 https://blog.csdn.net/weixin_42249893/article/details/122594676

## 构建 IntelRealSense SDK2 （Ubuntu18.04.06）

###  修补 linux 内核模块时，出现 git 仓库不存在错误

```bash
# 下载、修补与构建受 realsense 驱动影响的内核模块
$ cd librealsense-master
$ ./scripts/patch-realsense-ubuntu-lts.sh
# ~ output 下载 ubuntu 内核源码时报错
# fatal: repository 'https://kernel.ubuntu.com/ubuntu/ubuntu-bionic.git/' not found
```

留意 ./ubuntu-bionic-hwe-5.4 文件夹是否存在，存在则删除

```bash
$ rm -rf ubuntu-bionic-hwe-5.4
```

修改文件 patch-realsense-ubuntu-lts.sh

```shell
# ~/scripts/patch-realsense-ubuntu-lts.sh
# ...
# Get the linux kernel and change into source tree
if [ ! -d ${kernel_name} ]; then
	mkdir ${kernel_name}
	cd ${kernel_name}
	git init
	## 修改当前文件第 107 行	##
	# git remote add origin https://kernel.ubuntu.com/ubuntu/ubuntu-${ubuntu_codename}.git
	git remote add origin git://git.launchpad.net/~ubuntu-kernel/ubuntu/+source/linux/+git/${ubuntu_codename}
	##########################
	
	cd ..
fi
# ...
```

### 升级内核版本

```bash
$ apt-get upgrade linux-image-generic
```

## 指定 usb 串口设备文件名称

```config
# /etc/udev/rules.d/99-HL-340.rules
SUBSYSTEMS=="usb" ATTRS{idVendor}=="1a86", ATTRS{idProduct}=="7523", ATTRS{Product}=="USB Serial" RUN+="/bin/sh -c 'mv /dev/%k /dev/ttyUSB1'"
```
## 安装 upboard linux 5.4 内核

### 更新内核

```bash
# 安装 upboard PPA repository
$ sudo add-apt-repository ppa:up-division/5.4-upboard
# 更新安装源
$ sudo apt update
# 删除所有通用安装的内核（在“中止内核删除”问题上选择“否”）
$ sudo apt-get autoremove --purge 'linux-.*generic'
# 安装 ubuntu 18.04 和 20.04 共享相同的5.4内核
$ sudo apt-get install linux-generic-hwe-18.04-5.4-upboard
# 升级所有软件包到最新可用版本
$ sudo apt dist-upgrade -y
```
### 更新引导配置

```bash
# 编辑引导配置文件，修改或添加这一行：
# GRUB_CMDLINE_LINUX_DEFAULT="quiet splash up_board.spidev1=Y"
$ sudo vim /etc/default/grub
# 重新生成 GRUB 的配置文件（引导程序）
$ sudo update-grub
# 重启
$ sudo reboot
# ...
# 重新进入系统后
# 检查内核版本 ~output: Linux <username> 5.4.0-1-generic #0~upboard5-Ubuntu
$ uname -a
# 检查 SPI 端口是否存在
$ ls -l /dev/spidev2.1
```

### 参考文档 

[5.4内核安装](https://github.com/up-board/up-community/wiki/Ubuntu_18.04#Install_Ubuntu_kernel_5.0.0_locally_from_debian_packages_on_Ubuntu_18.04)

[修改引导配置](https://github.com/pazeshun/sphand_ros/commit/e34e32c86f276232a069308afa490fbc4017bec2)





## ORB-SLAM3 Camera IMU 标定

参考文档： https://github.com/UZ-SLAMLab/ORB_SLAM3/blob/master/Calibration_Tutorial.pdf

### 安装 Kalibr

1. 安装 ROS 1 桌面环境和 catkin 工具

```bash
sudo sh -c 'echo "deb http://packages.ros.org/ros/ubuntu $(lsb_release -sc) main" > /etc/apt/sources.list.d/ros-latest.list'
sudo apt-key adv --keyserver 'hkp://keyserver.ubuntu.com:80' --recv-key C1CF6E31E6BADE8868B172B4F42ED6FBAB17C654
sudo apt-get update
export ROS1_DISTRO=melodic # kinetic=16.04, melodic=18.04, noetic=20.04
sudo apt-get install ros-$ROS1_DISTRO-desktop-full
sudo apt-get install python-catkin-tools # ubuntu 16.04, 18.04
sudo apt-get install python3-catkin-tools python3-osrf-pycommon # ubuntu 20.04
```

2. 安装构建和运行依赖项

```bash
sudo apt-get install -y \
    git wget autoconf automake nano \
    libeigen3-dev libboost-all-dev libsuitesparse-dev \
    doxygen libopencv-dev \
    libpoco-dev libtbb-dev libblas-dev liblapack-dev libv4l-dev
# Ubuntu 16.04
sudo apt-get install -y python2.7-dev python-pip python-scipy \
    python-matplotlib ipython python-wxgtk3.0 python-tk python-igraph python-pyx
# Ubuntu 18.04
sudo apt-get install -y python3-dev python-pip python-scipy \
    python-matplotlib ipython python-wxgtk4.0 python-tk python-igraph python-pyx
# Ubuntu 20.04
sudo apt-get install -y python3-dev python3-pip python3-scipy \
    python3-matplotlib ipython3 python3-wxgtk4.0 python3-tk python3-igraph python3-pyx
```

3. 创建catkin工作区并克隆项目

```bash
mkdir -p ~/kalibr_workspace/src
cd ~/kalibr_workspace
export ROS1_DISTRO=noetic # kinetic=16.04, melodic=18.04, noetic=20.04
source /opt/ros/$ROS1_DISTRO/setup.bash
catkin init
catkin config --extend /opt/ros/$ROS1_DISTRO
catkin config --merge-devel # Necessary for catkin_tools >= 0.4.
catkin config --cmake-args -DCMAKE_BUILD_TYPE=Release
cd ~/kalibr_workspace/src
git clone https://github.com/ethz-asl/kalibr.git
```

4. 使用发布_配置构建代码。根据可用内存，可能需要减少构建线程（例如将 -j2 添加到 catkin_make）

```bash
cd ~/kalibr_workspace/
catkin build -DCMAKE_BUILD_TYPE=Release -j4
```

5. 构建完成后，您必须获取 catkin 工作区设置以使用 Kalibr

```bash
source ~/kalibr_workspace/devel/setup.bash
rosrun kalibr <command_you_want_to_run_here>
```
### 标定流程

1. 构建标定的工作输出目录

```bash
# 进入 ORB_SLAM3 目录
cd ~/ORB_SLAM3
# 创建标定工作目录
mkdir -p recorder/cam0
mkdir recorder/IMU
```

2. 记录传感器数据

从 https://github.com/ethz-asl/kalibr/wiki/downloads 下载标定文件

``april_6x6_80x80cm_A0.pdf``

![ april_6x6_80x80cm_A0.pdf](https://user-images.githubusercontent.com/33208530/105625447-68079500-5e64-11eb-8a24-f743c3f7ad2d.png)

```bash
# 输出结果分别对应 recorder/cam0 与 recorder/IMU
./Examples/Calibration/recorder_realsense_D435i ./Examples/Calibration/recorder 
# 命令执行后，
# 对准 april_6x6_80x80cm_A0.pdf 中的图像进行 r, p, y, x, y, z 六个自由度的旋转与移动
# cam0 目录输出 png 文件，对应每一个视频帧。
# IMU 目录则输出 acc.txt 与 gyro.txt，分别对应加速度与陀螺仪 
```

3. 使用 python 脚本处理 IMU 数据并插入加速度计测量值以使其与陀螺仪同步

```bash
python3 \
	./Examples/Calibration/python_scripts/process_imu.py \
	./Examples/Calibration/recorder/
# 命令执行后，会在 recorder 目录输出 imu0.csv
```

4. 将原始数据转换输出为 bag 格式

```bash
# 进入到 kalibr 工具箱（python 脚本）
cd ~/kalibr_workspace/src/kalibr/aslam_offline_calibration/kalibr/python/
# 执行 bag 生成命令
./kalibr_bagcreater \
--folder ~/ORB_SLAM3/Examples/Calibration/recorder/. \
--output -bag ~/ORB_SLAM3/Examples/Calibration/recorder/recorder.bag
# 查看 bag 文件信息
rosbag info ~/ORB_SLAM3/Examples/Calibration/recorder/recorder.bag
# ~ output
> path:        /root/ORB_SLAM3/Examples/Calibration/recorder/recorder.bag
> version:     2.0
> duration:    1:23s (83s)
> start:       Mar 25 2024 17:25:57.64 (1711358757.64)
> end:         Mar 25 2024 17:27:20.84 (1711358840.84)
> size:        658.1 MB
> messages:    18632
> compression: none [750/750 chunks]
> types:       sensor_msgs/Image [060021388200f6f0f447d0fcd9c64743]
>              sensor_msgs/Imu   [6a62c6daae103f4ff57a132d6f95cec2]
> topics:      /cam0/image_raw    2225 msgs    : sensor_msgs/Image
>              /imu0             16407 msgs    : sensor_msgs/Imu
```

5. 相机标定 Camera Calibration

从 https://github.com/ethz-asl/kalibr/wiki/downloads 下载标定文件

将 ``april_6x6_80x80cm_A0.pdf`` 打印或者显示在屏幕上，需要测量其中一个方格的大小，若大小等于 4cm , 则在 ``april_6x6_80x80cm.yaml`` 文件中修改 ``tagSize: 0.04``

``april_6x6_80x80cm.yaml``  

```yaml
target_type: 'aprilgrid' #gridtype
tagCols: 6               #number of apriltags
tagRows: 6               #number of apriltags
tagSize: 0.04            #size of apriltag, edge to edge [m]
tagSpacing: 0.3          #ratio of space between tags to tagSize
```

执行相机标定命令

```bash
# cd ~/kalibr_workspace/src/kalibr/aslam_offline_calibration/kalibr/python/
./kalibr_calibrate_cameras \
--bag ~/ORB_SLAM3/Examples/Calibration/recorder/recorder.bag \
--topics /cam0/image_raw --models pinhole-radtan \
--target ~/ORB_SLAM3/Examples/Calibration/recorder/april_6x6_80x80cm.yaml

# 命令执行后，会在 bag 文件同级目录输出
# recorder-camchain.yaml, recorder-report-cam.pdf, recorder-results-cam.txt
```

6. 惯性标定（相机与IMU)

从 https://github.com/ethz-asl/kalibr/wiki/downloads 下载标定文件

``imu_adis16448.yaml``

```yaml
rostopic: /imu0 # rosbag info recorder.bag --- topics: /cam0/image_raw /imu0
update_rate: 197.67 #Hz (16407 / 83)

accelerometer_noise_density: 0.01 #continous
accelerometer_random_walk: 0.0002
gyroscope_noise_density: 0.005 #continous
gyroscope_random_walk: 4.0e-06
```

``cam_april-camchain.yaml``

该文件由 ``kalibr_calibrate_cameras`` 命令输出生成

```yaml
cam0:
  cam_overlaps: []
  camera_model: pinhole
  distortion_coeffs: [0.004498989828596411, -0.002635399149782393, 0.0001850899312009363, 0.0002297455683999329]
  distortion_model: radtan
  intrinsics: [388.24656945217055, 388.6195279024117, 325.35775283236075, 239.89603853101096]
  resolution: [640, 480]
  rostopic: /cam0/image_raw
```

执行惯性标定命令

```bash
./kalibr_calibrate_imu_camera \
--bag ~/ORB_SLAM3/Examples/Calibration/recorder/recorder.bag \
--cam ~/ORB_SLAM3/Examples/Calibration/recorder/cam_april-camchain.yaml \
--imu ~/ORB_SLAM3/Examples/Calibration/recorder/imu_adis16448.yaml \
--imu-models calibrated \
--target ~/ORB_SLAM3/Examples/Calibration/recorder/april_6x6_80x80cm.yaml
# 命令执行后，会输出 recorder-camchain-imucam.yaml, recorder-imu.yaml, recorder-report-imucam.pdf, recorder-results-imucam.txt 等文件
```