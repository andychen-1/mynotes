
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

# 摄像头推流 1
ffmpeg -f v4l2 -i /dev/video2 -video_size 640x480 -framerate 20 -f flv rtmp://localhost/live/stream
# 摄像头推流 2
ffmpeg -f v4l2 -video_size 640x480 -i /dev/video2 -framerate 20 -c:v libx264 -preset ultrafast -tune zerolatency -pix_fmt yuv420p -b:v 500k -r 20 -f flv rtmp://localhost/live/stream
```