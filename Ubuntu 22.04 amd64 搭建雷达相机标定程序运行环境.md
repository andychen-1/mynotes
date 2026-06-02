
## docker 安装与常用指令

docker 安装

```bash
# 1. 更新系统
sudo apt update && sudo apt upgrade -y

# 2. 安装依赖
sudo apt install -y ca-certificates curl gnupg lsb-release

# 3. 添加 Docker 官方 GPG 密钥
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

# 4. 添加 Docker 官方源
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] \
  https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# 5. 安装 Docker Engine
sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# 6. 启动并开机自启
sudo systemctl start docker
sudo systemctl enable docker

# 7. 将当前用户加入 docker 组（免 sudo）
sudo usermod -aG docker $USER
# 重新登录或 newgrp docker
```

docker 常用指令

```bash
docker version                  # 查看版本
docker info                     # 查看详细信息
docker --help                   # 帮助

# 镜像操作
docker pull nginx               # 拉取镜像
docker images                   # 列出本地镜像
docker image ls
docker rmi nginx                # 删除镜像
docker image prune -a           # 删除所有未使用的镜像

# 容器操作
docker run -d --name web -p 8080:80 nginx    # 启动容器
docker ps                       # 查看运行中的容器
docker ps -a                    # 查看所有容器（含已停止）
docker start/stop/restart ID或名字
docker exec -it web bash        # 进入运行中的容器
docker logs -f web              # 查看日志
docker inspect web              # 查看容器详细信息
docker rm web                   # 删除容器（停止后）
docker rm -f web                # 强制删除运行中的容器
docker container prune          # 删除所有已停止的容器

# 构建镜像
docker build -t myapp:v1 .      # 根据当前目录 Dockerfile 构建
docker tag myapp:v1 myapp:latest
docker push username/myapp:v1   # 推送到 Docker Hub

# Docker Compose（已集成到 docker compose，无需单独安装）
docker compose up -d            # 启动
docker compose down             # 停止并删除容器
docker compose logs -f          # 查看日志
docker compose exec app bash    # 进入服务容器
```
## 安装 Nvidia 驱动以及容器支持

```bash
# 卸载所有手动安装的、残留的 NVIDIA 驱动以及开发库 
sudo apt purge 'nvidia-*' 'libnvidia-*' -y
sudo apt autoremove -y
sudo apt autoclean

# 对于缺失了安装源的安装依赖错误，如果不强行卸载，会严重系统更新功能，影响驱动的更新
# 可以参考以下针对顽固无用依赖库的删除方式（库文件产生于arm64交叉编译环境的安装过程）
## 删除安装源
sudo rm /etc/apt/sources.list.d/arm64.list
## OR
sudo mv /etc/apt/sources.list.d/arm64.list /etc/apt/sources.list.d/arm64.list.bak
## 更新安装源 与 修复安装
sudo apt --fix-broken install
## 错误依赖依然存在时，选择强行删除
sudo dpkg --remove --force-remove-reinstreq libcrypt1:arm64 libcrypt1:i386
## OR
sudo dpkg --force-depends --remove libcrypt1:arm64 libcrypt1:i386
## 最强暴力删除
sudo dpkg --force-all --remove libcrypt1:arm64 libcrypt1:i386
## 依赖修复与更新
sudo apt --fix-broken install
sudo apt full-upgrade -y

# 在使用 ubuntu-drivers 安装驱动之前建议查询一下推荐的安装
ubuntu-drivers devices
# 会显示类似：
# == /sys/devices/pci0000:00/0000:00:01.0/0000:01:00.0 ==
# modalias : pci:v000010DEd00002501sv00001028sd0000012Bbc03sc00i00
# vendor   : NVIDIA Corporation
# driver   : nvidia-driver-565 - distro non-free recommended   ← 这个就是官方推荐
# driver   : nvidia-driver-550 - distro non-free

# 让系统自己决定装哪个最稳的驱动（虽然不是最新的，但一定是与你当前系统内核最匹配的版本）
sudo ubuntu-drivers install
## OR
sudo ubuntu-drivers autoinstall
# 重启
sudo reboot

# 由于 docker 的设备访问常常依赖于主机环境，GPU 同样如此

# 安装 NVIDIA Container Toolkit
sudo apt-get install -y nvidia-container-toolkit
# 配置 Docker 使用 NVIDIA 驱动
sudo nvidia-ctk runtime configure --runtime=docker
# 重启 Docker 服务
sudo systemctl restart docker

# 验证 docker 能否使用 GPU
docker run --rm --gpus all nvidia/cuda:12.4.1-base-ubuntu22.04 nvidia-smi
# 会显示类似：
# Unable to find image 'nvidia/cuda:12.4.1-base-ubuntu22.04' locally
# 12.4.1-base-ubuntu22.04: Pulling from nvidia/cuda
# 3c645031de29: Pull complete 
# 0d6448aff889: Pull complete 
# 0a7674e3e8fe: Pull complete 
# b71b637b97c5: Pull complete 
# 56dc85502937: Pull complete 
# Digest: sha256:0f6bfcbf267e65123bcc2287e2153dedfc0f24772fb5ce84afe16ac4b2fada95
# Status: Downloaded newer image for nvidia/cuda:12.4.1-base-ubuntu22.04
# Fri Dec  5 00:08:30 2025 
# +----------------------------------------------------------------------------+
# | NVIDIA-SMI 580.95.05   Driver Version: 580.95.05      CUDA Version: 13.0   |
# +--------------------------+--------------------------+----------------------+
# | ...                      |       ...                |      ...             |
# +--------------------------+--------------------------+----------------------+
```


## 创建基于 Docker Ubuntu 16.04 + ROS desktop-full 的相机雷达标定环境

```bash
# 拉取 ROS 基础镜像，需要在 docker hub 上注册用户
docker pull osrf/ros:kinetic-desktop-full

# 在镜像（osrf/ros:kinetic-desktop-full）的基础上运行一个交互式容器
docker run -it --name livox_sdk_calib -v ~/livox_sdk_calib:/root/livox_sdk_calib --net=host --privileged osrf/ros:kinetic-desktop-full

# 以下操作都将在 docker 容器内执行

# 安装 livox_camera_lidar_calibration 所需的编译运行环境（c++ & python）
apt-get install -y wget curl git lsb-release \ 
software-properties-common build-essential cmake \
python-rosinstall python-rosinstall-generator \
python-wstool python-rosdep python-catkin-tools \
libpcl-dev libopencv-dev libvtk6-dev \
libapr1-dev libceres-dev

# 创建工作空间
mkdir -p ~/catkin_ws/src 
cd ~/catkin_ws

# 安装Livox_SDK
git clone https://github.com/Livox-SDK/Livox-SDK.git
cd Livox-SDK
sudo ./third_party/apr/apr_build.sh
cd build && cmake ..
make
sudo make install

# 安装livox_ros_driver
git clone https://github.com/Livox-SDK/livox_ros_driver.git ws_livox/src
cd ws_livox
catkin_make

# 安装 livox_camera_lidar_calibration
git clone https://github.com/Livox-SDK/livox_camera_lidar_calibration.git
cd camera_lidar_calibration
catkin_make
source devel/setup.bash

```

关于怎样编译运行 livox_camera_lidar_calibration， 更多参考[这里](https://github.com/Livox-SDK/livox_camera_lidar_calibration/blob/master/doc_resources/README_cn.md)



