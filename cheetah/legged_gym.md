# legged_gym 机器人强化学习环境搭建

>  NVIDIA RTX 3060 + ubuntu20.04

**由于大部分的虚拟系统不兼容 Nvidia 驱动，例如 VMware、WSL2 Ubuntu，建设使用 Ubuntu 物理系统**


## 安装 windows + unbuntu 双引导

1. 下载 Ubuntu 系统镜像文件 [ubuntu-20.04.6-desktop-amd64.iso](http://mirrors.aliyun.com/ubuntu-releases/20.04.6/ubuntu-20.04.6-desktop-amd64.iso)

2. 使用 [Rufus](https://rufus.id/zh/) 制作U盘启动盘，请参考 [ubuntu20.04 与 window11 双系统安装](https://www.bilibili.com/read/cv28251771/?jump_opus=1)

3. 使用 Windows 磁盘管理工具为即将安装的 Ubuntu 系统准备不少于 40G 的未分配空间（对已有分区进行 “压缩卷” 操作）

4. 设置 BIOS U盘引导，插入准备好的U盘启动盘安装 Ubuntu 系统，在安装过程中选择系统安装向导中提示选项 "Something else" 进行手动磁盘分区，其他选项则可按照系统安装向导的提示自行选择

	| 序号  | 类型         | 挂载    | 大小                              |
	| --- | ---------- | ----- | ------------------------------- |
	| 1   | 主分区   Ext4 | /boot | `>=1024MB`                      |
	| 2   | 逻辑分区 Ext4  | /     | `>=40GB`                        |
	| 3   | 内存交换 Swap  |       | `>=MEM_SIZE && <= MEM_SIZE * 2` |

	**从启动盘进行系统安装时，可能会出现黑屏问题，这时候可在重新启动进入引导时选择 `Ubuntu (safe graphics)`**

5. 系统安装完成后重启

## 安装 NVIDIA 驱动

**建议在安装驱动之前，等待一下系统更新提示来进行系统更新与升级，这样安装的 gcc, g++，make 或许更能适配操作系统版本，当然也可以提前手动安装 gcc, g++, make 等编译构建工具**

1. 更新或升级系统

```bash
# 更新安装源（阿里的镜像系统默认使用清华源）
$ sudo apt update

# 查看可更新软件
$ sudo apt list --upgradeable

# 升级所有可更新软件
$ sudo apt upgrade -y
```

2. 安装 Nvidia 驱动

```bash
$ sudo apt update
# 查询适配的 NVIDAI 驱动版本
$ nvidia-detector

# 输出
nvidia-driver-535

# 安装对应版本的驱动
$ sudo apt install nvidia-driver-535

# 安装完成后，重启系统
$ sudo reboot

# ... 重启中 ...

# 如果重启后进入系统时出现黑屏，可按 <Ctrl> + <Alt> + F3 进入 tty 交互模式，然后按 <Ctrl> + <Alt> + F1 重新进入桌面模式
# 也可以再次重启，在 BIOS 中选择 “独显模式”（各个厂商的设置可能不尽相同，依据实际情况而定）

  
# 查看驱动信息以及对应的 NVIDIA CUDA TOOLKIT 版本
$ nvidia-smi

# 输出
+---------------------------------------------------------------------------------------+
| NVIDIA-SMI 535.171.04             Driver Version: 535.171.04   CUDA Version: 12.2     |
|-----------------------------------------+----------------------+----------------------+
| GPU  Name                 Persistence-M | Bus-Id        Disp.A | Volatile Uncorr. ECC |
| Fan  Temp   Perf          Pwr:Usage/Cap |         Memory-Usage | GPU-Util  Compute M. |
|                                         |                      |               MIG M. |
|=========================================+======================+======================|
|   0  NVIDIA GeForce RTX 3060 ...    Off | 00000000:01:00.0  On |                  N/A |
| N/A   47C    P8              18W /  80W |    400MiB /  6144MiB |      0%      Default |
|                                         |                      |                  N/A |
+-----------------------------------------+----------------------+----------------------+
+---------------------------------------------------------------------------------------+
| Processes:                                                                            |
|  GPU   GI   CI        PID   Type   Process name                            GPU Memory |
|        ID   ID                                                             Usage      |
|=======================================================================================|
|    0   N/A  N/A      1087      G   /usr/lib/xorg/Xorg                           59MiB |
|    0   N/A  N/A      1520      G   /usr/lib/xorg/Xorg                          207MiB |
|    0   N/A  N/A      1649      G   /usr/bin/gnome-shell                         41MiB |
|    0   N/A  N/A     10241      G   /usr/lib/firefox/firefox                     61MiB |
+---------------------------------------------------------------------------------------+
```

2. 安装 NVIDIA CUDA SDK


```bash
# 安装 nvidia-cuda-toolkit 12.2 | nvidia-driver-535
$ wget https://developer.download.nvidia.com/compute/cuda/repos/ubuntu2004/x86_64/cuda-ubuntu2004.pin
$ sudo mv cuda-ubuntu2004.pin /etc/apt/preferences.d/cuda-repository-pin-600
$ wget https://developer.download.nvidia.com/compute/cuda/12.2.1/local_installers/cuda-repo-ubuntu2004-12-2-local_12.2.1-535.86.10-1_amd64.deb
$ sudo dpkg -i cuda-repo-ubuntu2004-12-2-local_12.2.1-535.86.10-1_amd64.deb
$ sudo cp /var/cuda-repo-ubuntu2004-12-2-local/cuda-*-keyring.gpg /usr/share/keyrings/
$ sudo apt-get update
$ sudo apt-get -y install cuda

# 配置环境变量
$ echo 'export CUDA_HOME=/usr/local/cuda' >> ~/.bashrc
$ echo 'export PATH=$CUDA_HOME/bin:$PATH' >> ~/.bashrc
$ echo 'export LD_LIBRARY_PATH=$CUDA_HOME/lib64:$LD_LIBRARY_PATH' >> ~/.bashrc
$ source ~/.bashrc

# 查看 NVIDIA CUDA SDK 版本
$ nvcc -V

# 输出
nvcc: NVIDIA (R) Cuda compiler driver
Copyright (c) 2005-2023 NVIDIA Corporation
Built on Tue_Jul_11_02:20:44_PDT_2023
Cuda compilation tools, release 12.2, V12.2.128
Build cuda_12.2.r12.2/compiler.33053471_0
  
```

  

3. 卸载驱动与 CUDA SDK (当重新安装新的驱动版本时）

```bash
# 卸载驱动
$ sudo apt --purge remove nvidia*

# 卸载 NVIDIA CUDA SDK
$ sudo apt --purge remove "*cublas*" "cuda*"

# 卸载相关依赖
$ sudo apt autoremove
```

  

4. 编译构建 legged_gym

```bash
# 安装 pip 包管理工具，这里对应的是 python3.8.10
$ sudo apt install python3-pip

# 永久设置 pip 镜像源 (需要关闭命令终端后，重新打开）
$ mkdir -p ~/.pip
$ echo '[global]' >> ~/.pip/pip.conf
$ echo 'index-url = https://pypi.tuna.tsinghua.edu.cn/simple' >> ~/.pip/pip.conf

# 克隆代码
$ git clone https://github.com/leggedrobotics/legged_gym.git

# 安装 torch2.3+cu121 (项目原文档中指定的版本是 torch1.10.0+cu113，但在当前系统中会出现 GPU 内存分配异常等问题）
$ pip3 install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cu121

# 余下的编译构建步骤可按 github 项目文档 https://github.com/leggedrobotics/legged_gym 来执行
# 在运行示例 `python3 legged_gym/scripts/train.py --task=anymal_c_flat` 可能出现以下问题

# AttributeError: module 'numpy' has no attribute 'float'.
pip install numpy==1.23.5

# RuntimeError: Ninja is required to load C++ extension
sudo apt-get install ninja-build

# ValueError: too many values to unpack (expected 2)
cd ~/rsl_rl/ && git checkout v1.0.2 && pip instal-e .
  
# 运行时，如果出现闪退，可适当减少训练环境数量 (--num_envs)，默认是 4096，例如
$ cd ~/legged_gym
$ python3 legged_gym/scripts/train.py --task=anymal_c_flat --num_envs=1024

# 训练自定义模型
$ cd ~/legged_gym
$ python3 legged_gym/scripts/train.py --task=alexbot --num_envs=512
```