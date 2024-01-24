
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

