
### 使用 tar 解压并安装 android-studio

```bash
tar -zxvf android-studio-2024.1.1.13-linux.tar.gz -C /opt/
cd /opt/android-studio/bin/
./studio.sh
```

### tar 解压示例

```bash
# 典型的 Unix tar 语法：  
$ tar -xf file.name.tar -C /path/to/directory
# GNU/tar Linux 语法：  
$ tar xf file.tar -C /path/to/directory
# 或者使用以下命令将 tar 文件提取到其他目录：  
$ tar xf file.tar --directory /path/to/directory
# 提取 .tar.gz 存档：  
$ tar -zxf file.tar --directory /path/to/directory
# 想要提取 .tar.bz2/.tar.zx 存档？尝试：  
$ tar -jxf file.tar --directory /path/to/directory
```

 **参数说明**
- c：创建压缩文件
- x：提取压缩文件
- f：Tar 档案名称
- --directory：设置解压文件的目录名
- -C：设置提取文件的目录名
- -z：处理 .tar.gz (gzip) 文件格式
- -j：处理 .tar.bz2 (bzip2) 文件格式
- -J(大写J) ：处理 .tar.xz (xz) 文件格式（有关详细信息，请参阅[如何在 Linux 中提取 tar.xz 文件）](https://www.cyberciti.biz/faq/how-to-extract-tar-xz-files-in-linux-and-unzip-all-files/ "如何在 Linux 中提取 tar.xz 文件并解压所有文件")
- -v：详细输出，即在屏幕上显示进度


### 获取 wifi 热点的 BSSID （网卡地址）

```bash
# <ssid> 为当前已知的 WI-FI SSID
nmcli -f SSID,BSSID,ACTIVE dev wifi list | grep <ssid>
```

