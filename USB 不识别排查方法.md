USB 不识别的排查可以按"内核是否看到 → udev 是否处理 → 是否是电气/枚举问题"三层往下走。

## 1. 内核层：最先看的地方

```bash
sudo dmesg -w              # 插拔时实时观察
sudo journalctl -k -f      # 同上，systemd 版本
sudo dmesg -T | grep -i -E 'usb|xhci|dwc'
```

重点看错误码，它直接区分了故障性质：

|报错|含义|一般原因|
|---|---|---|
|`device descriptor read/64, error -71`|EPROTO|线缆/接触/信号完整性|
|`error -110`|ETIMEDOUT|供电不足或设备无响应|
|`unable to enumerate USB device`|枚举失败|硬件/供电，基本不是驱动问题|
|枚举成功但无 `/dev` 节点|缺驱动或 udev 规则|内核模块没编进去|

如果 dmesg 插拔时**完全没有任何输出**，说明连 D+/D- 上的电平变化都没检测到——问题在控制器或物理层，往下的软件排查就没意义了。

## 2. 枚举层

```bash
usb-devices                          # 比 lsusb -v 更易读，含驱动绑定情况
lsusb -t                             # 拓扑树，看挂在哪个 root hub / 哪级 hub
cat /sys/kernel/debug/usb/devices    # 需要 mount -t debugfs none /sys/kernel/debug
ls -l /sys/bus/usb/devices/          # 每个设备的原始属性
```

`usb-devices` 输出里的 `Driver=` 字段很关键：显示 `(none)` 就是内核认到了设备但没有匹配的驱动。

## 3. udev 层

```bash
udevadm monitor --kernel --udev --property   # 插拔时对比 KERNEL 和 UDEV 两路事件
udevadm info -a -n /dev/ttyUSB0              # 查完整属性链，写规则时用
udevadm trigger --action=add --subsystem-match=usb   # 重放事件
```

如果 `KERNEL` 有事件而 `UDEV` 没有，就是 udev 规则或权限的问题。

## 4. 抓包层：usbmon

这是最底层的软件手段，能看到实际的 URB 传输：

```bash
sudo modprobe usbmon
sudo cat /sys/kernel/debug/usb/usbmon/0u      # 0u = 所有总线
# 或用 Wireshark：
sudo tshark -i usbmon1
```

配合动态调试打开 usbcore 的详细日志：

```bash
echo 'module usbcore +p' | sudo tee /sys/kernel/debug/dynamic_debug/control
echo 'module xhci_hcd +p' | sudo tee /sys/kernel/debug/dynamic_debug/control
```

## 5. 树莓派 + Deepin 的几个特有坑

这个组合下我会优先怀疑下面几点，而不是通用 USB 问题：

**内核模块缺失。** Deepin 的 ARM64 内核是自己构建的，未必带全树莓派需要的驱动：

```bash
zcat /proc/config.gz | grep -i -E 'USB_XHCI|USB_DWC|DWC2'   # 或 /boot/config-$(uname -r)
lsmod | grep -E 'xhci|dwc'
```

**设备树 overlay 不对。** 官方 Raspberry Pi OS 的 `config.txt` 里的 USB 相关配置（`dtoverlay=dwc2`、`otg_mode=1`）在 Deepin 的启动分区里可能没有对应设置，或者 Deepin 用了 UEFI/upstream DT 而不是 Foundation 的 DT。检查 `/boot/config.txt` 和 `ls /proc/device-tree/`。

**Pi 4 的 VL805 固件。** USB3 由独立的 VL805 芯片提供，固件在 EEPROM 里，旧固件有已知的枚举问题：

```bash
sudo rpi-eeprom-update
vcgencmd bootloader_version
```

**供电。** 这是实际最常见的原因，尤其接机械硬盘、雷达之类的：

```bash
vcgencmd get_throttled    # 非 0 即有欠压/降频，bit 0 = 当前欠压
```

Pi 5 上还需要 `usb_max_current_enable=1`（配合 5V/5A 电源）才能让 USB 口输出满电流。

## 6. 强制复位的手段

```bash
# 单口断电重上电（需要 hub 支持 PPPS）
sudo uhubctl -l 1-1 -p 2 -a cycle

# 软复位：取消再重新授权
echo 0 | sudo tee /sys/bus/usb/devices/1-1.2/authorized
echo 1 | sudo tee /sys/bus/usb/devices/1-1.2/authorized

# 卸载重载整个控制器驱动
sudo modprobe -r xhci_pci && sudo modprobe xhci_pci
```

---

最快的分诊方法：拿同一个设备插到官方 Raspberry Pi OS 的 SD 卡上试一次。能认出来就是 Deepin 的内核/DT 问题（走第 5 节），两个系统都不认就是硬件或供电（走第 1、6 节）。

如果你把插拔时的 `dmesg -w` 输出贴出来，我可以帮你具体判断。