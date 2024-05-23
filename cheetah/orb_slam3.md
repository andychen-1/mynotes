
# 在 Ubuntu20.04 下编译构建 ORB_SLAM3

## 安装常用工具

```bash
# 安装 git 与 wget 
sudo apt install git wget
# 安装编译构建工具
sudo apt install cmake build-essential
```

## 安装 Eigen3

```bash
# 下载 eigen-3.3.8
wget https://gitlab.com/libeigen/eigen/-/archive/3.3.8/eigen-3.3.8.tar.gz
# 解压并进入目录
tar -zxvf eigen-3.3.8.tar.gz && cd eigen-3.3.8
# 编译与安装
mkdir build && cd build
cmake ..
# -jn 对应当前 cpu 核数 
make -j8 && sudo make install
```

## 安装 Pangolin

```bash
# 预安装依赖
sudo apt install libglew-dev
sudo apt install libboost-dev libboost-thread-dev libboost-filesystem-dev libeigen3-dev

# 克隆仓库与编译安装
git clone https://github.com/stevenlovegrove/Pangolin.git
cd Pangolin
./scripts/install_prerequisites.sh --dry-run recommended
mkdir build && cd build
cmake ..
make
sudo make install
```

## 安装 librealsense

参考文档 [Linux/Ubuntu - RealSense SDK 2.0 Build Guide](https://dev.intelrealsense.com/docs/compiling-librealsense-for-linux-ubuntu-guide)

```bash
# 升级系统
sudo apt-get update && sudo apt-get upgrade && sudo apt-get dist-upgrade
# 安装 SDK 依赖包
sudo apt-get install libssl-dev libusb-1.0-0-dev libudev-dev pkg-config libgtk-3-dev
# 安装 OpenGL 图形应用库及其他
sudo apt-get install libglfw3-dev libgl1-mesa-dev libglu1-mesa-dev at

# 克隆 librealsense 仓库
git clone https://github.com/IntelRealSense/librealsense.git
cd librealsense

# 安装 realsense udev 规则
./scripts/setup_udev_rules.sh
# 卸载 udev 规则
# ./scripts/setup_udev_rules.sh --uninstall

# 安装内核补丁
# Ubuntu 20/22（focal/jammy），带 LTS 内核 5.13、5.15
./scripts/patch-realsense-ubuntu-lts-hwe.sh
# 带有 LTS 内核的 Ubuntu 18/20 (< 5.13)  
# ./scripts/patch-realsense-ubuntu-lts.sh

# 编译构建
mkdir build && cd build
cmake ..
sudo make install

# 重新安装构建
sudo make uninstall && make clean && make && sudo make install
```

## 安装 OpenCV  (>=4.4.0)

参考文档 [tutorial_linux_install](https://docs.opencv.org/4.x/d7/d9f/tutorial_linux_install.html)

```bash
# 安装 GTK2.0 库
sudo apt install libgtk2.0-dev pkg-config
# 下载解压 opencv4.4
wget https://codeload.github.com/opencv/opencv/zip/refs/tags/4.4.0
unzip opencv-4.4.0.zip
# 下载解压 opencv_contrib-4.4 (opencv 社区扩展)
wget https://codeload.github.com/opencv/opencv_contrib/zip/refs/tags/4.4.0
unzip opencv_contrib-4.4.0.zip
# 编译时包含 opencv_contrib
mv opencv_contrib-4.4.0 opencv-4.4.0/ && cd opencv-4.4.0
mkdir build && cd build
cmake -DCMAKE_BUILD_TYPE=RELEASE -DOPENCV_EXTRA_MODULES_PATH=../opencv_contrib-4.4.0/modules ..
cmake --build .
sudo make install
```

### 离线下载依赖文件到缓存目录，跳过安装过程中的下载失败错误

**建议在当前安装完成后对 opencv-4.x/.cache 目录进行备份**

```bash
# 安装过程中，可能出现若干依赖文件无法下载的问题，
# 可以离线下载这些文件，然后复制到 opencv-4.x/.cache 对应的目录中，文件名格式： <hashcode>-<filename.ext>

# 下载 ippicv (Intel 高性能图像处理库)，文件对应系统版本以及 Intel CPU 型号，
# 请查阅 https://github.com/opencv/opencv_3rdparty/tree/ippicv/master
wget https://raw.githubusercontent.com/opencv/opencv_3rdparty/ippicv/master_20191018/ippicv/ippicv_2020_lnx_intel64_20191018_general.tgz
# 复制文件到缓存目录 .cache/ippicv/
cp -f ippicv_2020_lnx_intel64_20191018_general.tgz opencv.4.4.0/.cache/ippicv/<hashcode>-ippicv_2020_lnx_intel64_20191018_general.tgz

# 下载 face_landmark_model.dat  (包含面部识别预训练模型)
# 可选的安装，下载失败了也没有关系
wget https://raw.githubusercontent.com/opencv/opencv_3rdparty/8afa57abc8229d611c4937165d20e2a2d9fc5a12/face_landmark_model.dat

cp -f face_landmark_model.dat opencv.4.4.0/.cache/data/<hashcode>-face_landmark_model.dat
```


## 编译构建 ORB_SLAM3

参考文档 [ORB_SLAM3](https://github.com/UZ-SLAMLab/ORB_SLAM3)

```bash
# 克隆 ORB_SLAM3 仓库
git clone https://github.com/UZ-SLAMLab/ORB_SLAM3.git ORB_SLAM3
cd ORB_SLAM3
# 赋予执行权限
chmod +x build.sh
# ubuntu20.04 系统下需要修改编译c++标准为 c++14
sed -i 's/++11/++14/g' CMakeLists.txt
# 注释掉 src/System.cc:84，此处打印配置对象引发了内存访问异常
sed -i '/cout << (\*settings_)/s/^/\/\//' src/System.cc
# 执行编译脚本
./build.sh
# 执行立体+惯性示例
./Examples/Stereo-Inertial/stereo_inertial_realsense_D435i Vocabulary/ORBvoc.txt ./Examples/Stereo-Inertial/RealSense_D435i.yaml
```
