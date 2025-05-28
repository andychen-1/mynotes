
## 使用 vcpkg 安装 ffmpeg 并支持交叉编译


```bash
# 安装 vcpkg
git clone https://github.com/microsoft/vcpkg.git
cd vcpkg
./bootstrap-vcpkg.sh
```

```bash
# ~/.bashrc
export VCPKG_ROOT=$HOME/vcpkg
export PATH=$PATH:$VCPKG_ROOT
# cmake -DCMAKE_TOOLCHAIN_FILE=$VCPKG_ROOT/scripts/buildsystems/vcpkg.cmake
export CMAKE_TOOLCHAIN_FILE="$VCPKG_ROOT/scripts/buildsystems/vcpkg.cmake"
```

```bash
# 同步配置
source ~/.bashrc
# ffmpeg:x64-linux
vcpkg install ffmpeg
# 安装 arm64 编译器
sudo apt install gcc-aarch64-linux-gnu g++-aarch64-linux-gnu
# 安装 ffmpeg:arm64-linux
vcpkg install ffmpeg:arm64-linux
```

**如果当前项目使用cmake 构建，vcpkg 则能很好地整合 cmake 编译环境**

```cmake
cmake_minimum_required(VERSION 3.10)

project(MyProject)

# 调试模式
set(CMAKE_CXX_FLAGS_DEBUG "-g")
# 查看 cmake 环境配置
message(STATUS "${CMAKE_PREFIX_PATH}")

# 在使用 vcpkg 环境时，必须配置CMAKE_TOOLCHAIN_FILE=$VCPKG_ROOT/scripts/buildsystems/vcpkg.cmake
# 查找包 （目标名称对应 Find<TargetName>.cmake， 例如 FFMPEG => FindFFMPEG.cmake）
find_package(FFMPEG REQUIRED)

add_executable(MyProject src/main.cpp)

# 链接目标在添加可执行文件之后
target_include_directories(MyProject PRIVATE ${FFMPEG_INCLUDE_DIRS})
target_link_directories(MyProject PRIVATE ${FFMPEG_LIBRARY_DIRS})
target_link_libraries(MyProject PRIVATE ${FFMPEG_LIBRARIES} avcodec avformat avutil)
```

## 摄像头推流

### 安装 Nginx RTMP 服务

```bash
# 安装依赖项
sudo apt-get update
sudo apt-get install build-essential libpcre3 libpcre3-dev libssl-dev zlib1g-dev
# 下载 nginx 以及 nginx-http-flv-module
wget http://nginx.org/download/nginx-1.24.0.tar.gz  # 根据需要更换版本
tar -zxvf nginx-1.24.0.tar.gz
git clone https://github.com/winshining/nginx-http-flv-module.git
# 编译 nginx 以及 rtmp 模块
cd nginx-1.24.0
./configure --with-http_ssl_module --add-module=../nginx-http-flv-module
make
sudo make install
# 配置软链接
ln -s /usr/local/nginx/sbin/nginx /usr/bin/nginx
# 检查安装是否成功
nginx -V
# ~output
# nginx version: nginx/1.24.0
# built by gcc 11.4.0 (Ubuntu 11.4.0-1ubuntu1~22.04) 
# built with OpenSSL 3.0.2 15 Mar 2022
# TLS SNI support enabled
# configure arguments: --with-http_ssl_module --add-module=../nginx-http-flv-module
```

```nginx.conf
#user  nobody;
worker_processes  1;

#error_log  logs/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;

#pid        logs/nginx.pid;

events {
    worker_connections  1024;
}

rtmp {
    server {
        listen 1935;       # RTMP 默认端口
        chunk_size 4096;
        application live {
            live on;
	        record off;     # 关闭录制，可以根据需要开启
            gop_cache off;  # 关闭GOP缓存 打开则可以减少首屏等待时间
            interleave on;
            wait_key on;
            meta copy;
        }
    }
}

http {
	listen 8080;
	
	# http 访问地址：http://example.com:8080/live?port=1935&app=live&stream=stream
	location /live {
		flv_live on;
	}
}
```

```bash
# 检查配置
sudo nginx -t
# 启动
sudo nginx
# 重启
sudo nginx -s reload
# 关闭
sudo nginx -s stop
```

### 安装 ffmpeg 并推流至 RTMP 服务

```bash
# 安装 ffmpeg
sudo apt update
sudo apt install ffmpeg
# 摄像头推流 （h264 编码，无缓冲）
ffmpeg -f v4l2 -i /dev/video0 -vcodec libx264 -tune zerolatency -preset ultrafast -f flv rtmp://localhost/live/stream
# 使用 ffplay 播放 rtmp （无缓冲，低延时）
ffplay -fflags nobuffer -flags low_delay -analyzeduration 0 rtmp://localhost/live/stream
# 查看摄像头设备信息
v4l2-ctl --list-devices
# 摄像头推流 1
ffmpeg -f v4l2 -i /dev/video2 -video_size 640x480 -framerate 20 -f flv rtmp://localhost/live/stream
# 摄像头推流 2
ffmpeg -f v4l2 -video_size 640x480 -i /dev/video2 -framerate 20 -c:v libx264 -preset ultrafast -tune zerolatency -pix_fmt yuv420p -b:v 500k -r 20 -f flv rtmp://localhost/live/stream
# 叠加滤镜
ffmpeg -f v4l2 -input_format mjpeg -video_size 1920x1080 -framerate 30 -i "/dev/video0"   -i "watermark_1920x1080.png"   -filter_complex "[1:v]format=rgba,colorkey=0x000000:0.01:0.0[a];[0:v][a]overlay=0:0,format=yuv420p[vout]"   -map "[vout]"   -c:v libx264 -preset ultrafast -tune zerolatency -pix_fmt yuv420p -b:v 5000k -r 30   -an   -f flv "rtmp://localhost/live/stream"
```


## 使用 vcpkg 交叉编译 opencv

### 配置安装源

```bash
# file ~ /etc/apt/source.list
# 添加 arch 标头 
deb [arch=amd64] https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy main restricted universe multiverse
deb [arch=amd64] https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-updates main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-updates main restricted universe multiverse
deb [arch=amd64] https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-backports main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-backports main restricted universe multiverse

# 以下安全更新软件源包含了官方源与镜像站配置，如有需要可自行修改注释切换
deb [arch=amd64] http://security.ubuntu.com/ubuntu/ jammy-security main restricted universe multiverse
# deb-src http://security.ubuntu.com/ubuntu/ jammy-security main restricted universe multiverse

# 预发布软件源，不建议启用
# deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-proposed main restricted universe multiverse
# # deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-proposed main restricted universe multiverse
```

```bash
# file ~ /etc/apt/source.list.d/arm64.list
deb [arch=arm64] http://ports.ubuntu.com/ubuntu-ports/ jammy main restricted universe multiverse
deb [arch=arm64] http://ports.ubuntu.com/ubuntu-ports/ jammy-updates main restricted universe multiverse
deb [arch=arm64] http://ports.ubuntu.com/ubuntu-ports/ jammy-security main restricted universe multiverse
deb [arch=arm64] http://ports.ubuntu.com/ubuntu-ports/ jammy-backports main restricted universe multiverse
```

###  构建脚本
```bash
#!/bin/bash

# 脚本名称: setup_cross_compile_arm64.sh
# 用途: 在 Ubuntu 22.04 x86_64 上配置 ARM64 交叉编译环境，安装 vcpkg 和 opencv:arm64-linux
# 前提: 需要 root 权限来安装系统包

set -e  # 遇到错误时退出

echo "开始配置 ARM64 交叉编译环境..."

# 步骤 1: 更新系统并安装基本工具
echo "更新系统并安装基本工具..."
sudo apt update
sudo apt install -y build-essential curl zip unzip tar git pkg-config

# 步骤 2: 安装 ARM64 交叉编译工具链
echo "安装 aarch64-linux-gnu 工具链..."
sudo apt install -y gcc-aarch64-linux-gnu g++-aarch64-linux-gnu binutils-aarch64-linux-gnu

# 步骤 3: 安装 ARM64 架构的 X11 相关开发包
echo "安装 ARM64 的 X11 开发包..."
sudo apt install -y libx11-dev:arm64 libxft-dev:arm64 libxext-dev:arm64

# 步骤 4: 配置 pkg-config 环境变量
echo "配置 pkg-config 环境变量..."
export PKG_CONFIG_LIBDIR=/usr/lib/aarch64-linux-gnu/pkgconfig:/usr/share/pkgconfig
export PKG_CONFIG_PATH=/usr/lib/aarch64-linux-gnu/pkgconfig
echo "export PKG_CONFIG_LIBDIR=/usr/lib/aarch64-linux-gnu/pkgconfig:/usr/share/pkgconfig" >> ~/.bashrc
echo "export PKG_CONFIG_PATH=/usr/lib/aarch64-linux-gnu/pkgconfig" >> ~/.bashrc

# 步骤 5: 验证 pkg-config 配置
echo "验证 pkg-config 配置..."
if pkg-config --modversion x11; then
    echo "pkg-config 成功找到 x11 依赖"
else
    echo "错误: pkg-config 未找到 x11 依赖，请检查 libx11-dev:arm64 是否正确安装"
    exit 1
fi

# 步骤 6: 安装和配置 vcpkg
echo "安装 vcpkg..."
if [ -d "/home/$USER/vcpkg" ]; then
    echo "vcpkg 已存在，跳过克隆"
else
    git clone https://github.com/microsoft/vcpkg.git /home/$USER/vcpkg
fi

# 运行 vcpkg bootstrap
cd /home/$USER/vcpkg
./bootstrap-vcpkg.sh
export VCPKG_ROOT=/home/$USER/vcpkg
echo "export VCPKG_ROOT=/home/$USER/vcpkg" >> ~/.bashrc
export PATH=$VCPKG_ROOT:$PATH
echo "export PATH=$VCPKG_ROOT:\$PATH" >> ~/.bashrc

# 步骤 7: 安装 opencv:arm64-linux
echo "使用 vcpkg 安装 opencv:arm64-linux..."
./vcpkg install opencv:arm64-linux

# 步骤 8: 验证安装
echo "验证 opencv:arm64-linux 安装..."
if [ -d "/home/$USER/vcpkg/installed/arm64-linux" ]; then
    echo "opencv:arm64-linux 安装成功！安装路径: /home/$USER/vcpkg/installed/arm64-linux"
else
    echo "错误: opencv:arm64-linux 安装失败，请检查日志"
    exit 1
fi

# 步骤 9: 提示用户
echo "交叉编译环境配置完成！"
echo "请在新终端中运行 'source ~/.bashrc' 以确保环境变量生效"
echo "可用的 opencv 库位于: /home/$USER/vcpkg/installed/arm64-linux"
echo "交叉编译工具链: aarch64-linux-gnu-gcc, aarch64-linux-gnu-g++"

exit 0
```
