>  系统环境 （WSL Ubuntu18.04)

## 设置镜像安装源

**镜像安装源文件**

在镜像地址之后紧跟的系统版本别名要与 Ubuntu 当前版本对应，例如 Ubuntu 18.04 对应的别名是 bionic

```txt
# 默认注释了源码镜像以提高 apt update 速度，如有需要可自行取消注释
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-updates main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-updates main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-backports main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-backports main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-security main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-security main restricted universe multiverse

# 预发布软件源，不建议启用
# deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-proposed main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-proposed main restricted universe multiverse
```

**命令操作**

```bash
# 备份资源列表文件
$ sudo cp -v /etc/apt/sources.list /etc/apt/sources.list.backup
# 打开编辑器， 复制上面的镜像文件内容
$ notepad.exe /etc/apt/sources.list
# 更新安装源
$ sudo apt-get update
```

## 设置 GitHub SSH 访问

**提示**：通过 https 访问 github 站点经常出现 443 端口访问超时的情况，需要将 git 访问方式修改为
ssh key 访问，例如： `git clone git@github.com:<user>/<repository>.git`

生成 ssh key 与 上传公钥至 GitHub 请参考文档: https://zhuanlan.zhihu.com/p/111344840
以下展示的只是本地配置

```sshconfig
## ~/.ssh/config
Host github.com
  HostName ssh.github.com
  User git
  Port 443
  IdentityFile ~/.ssh/id_rsa
```

```bash
# 如果生成的私钥文件是从其他路径位置拷贝过来的，请赋予私钥文件(~/.ssh/id_rsa) 400 权限（即所有用户只读），否则 git clone 可能出现 ssh 访问被拒绝错误。
$ chmod 400 ~/.ssh/id_rsa
```

## 设置 ROS 安装源

```bash
# 添加清华镜像源
$ sudo sh -c '. /etc/lsb-release && echo "deb http://mirrors.tuna.tsinghua.edu.cn/ros/ubuntu/ $DISTRIB_CODENAME main" > /etc/apt/sources.list.d/ros-latest.list'
# 添加信任 ROS 的 GPG Key
$ apt-key adv --keyserver 'hkp://keyserver.ubuntu.com:80' --recv-key C1CF6E31E6BADE8868B172B4F42ED6FBAB17C654
# 更新安装源
$ sudo apt update
# 如果提示存在未升级的软件包（建议）
$ sudo apt upgrade -y
```
## 安装 ROS

```bash
# ros distro:
# - Ubuntu 16.04 (ROS Kinetic)
# - Ubuntu 18.04 (ROS Melodic)
# - Ubuntu 22.04 (ROS2 Humble)
# *********************************
# install:
# - ros-<distro>-desktop-full
# *********************************
# Ubuntu16.04
$ sudo apt install ros-kinetic-desktop-full
# Ubuntu18.04
$ sudo apt install ros-melodic-desktop-full
```

**Ubuntu16.04.07 LTS 安装 ros-kinetic-desktop-full 时如果出现软件依赖冲突**

```bash
$ sudo apt-get install aptitude
$ sudo aptitude install ros-kinetic-desktop-full
```

## 构建 champ

```bash
## 安装 Ros 包管理工具
$ apt install python-rosdep -y
# 初始化包管理工具 
# 如果20-default.list 无法下载，可复制其他已安装系统的缓存文件
# scp -r ~/.ros root@<host>:~/
$ sudo rosdep init
# 更新包管理工具 (包含已停止更新的发行版)
$ rosdep update --include-eol-distros
## 创建自定义工作目录
$ mkdir -p /usr/local/src/champ-ws
## 进入工作目录
$ cd /usr/local/src/champ-ws
## 创建源文件目录, 进入 src 目录下
$ mkdir src
$ cd src
## 克隆 champ.git
$ git clone git@github.com:chvmp/champ.git
## 克隆 champ 的子项目 libchamp
$ cd champ/champ/include
$ git clone git clone git@github.com:chvmp/libchamp.git
## 构建 champ (首先需要回到工作根目录)
$ cd /usr/local/src/champ-ws
# 定义 Ros 对应 Ubuntu 的发行版本
$ export ROS_DISTRO=melodic
# 安装依赖
$ rosdep install --from-paths src --ignore-src -r -y
# 添加 catkin 编译环境配置
$ echo "source /opt/ros/melodic/setup.bash" >> ~/.bashrc
# 构建 champ, catkin_make 默认对 ./src 下所有项目进行 cmake 构建（树的深度构建）
$ catkin_make
```