
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

## 安装 UP_board Linux 内核

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
# 重新进入系统后，检查 SPI 端口是否存在
$ ls -l /dev/spidev2.1
```



