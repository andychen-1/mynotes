
## WSL2 Ubuntu 22.4 中使用游戏手柄

### 连接 USB 驱动

1.  安装 [USBIPD-WIN](https://github.com/dorssel/usbipd-win/releases)，将 Windows 已识别的 USB 设备转接到 WSL

2.  通过命令行将设备共享，WSL Ubuntu 系统将通过 

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



