
### 文件系统

```bash
# 查看文件夹总大小
du -hs .
# 列出目录下每个文件夹的大小（包含所有子目录）
du -h .
# 列出当前目录下文件夹的大小（不包含下一级目录）
du -h --max-depth=1 .
# 列出当前目录文件夹与文件的大小
du -ah --max-depth=1 .
```


###  evo_traj

解析  ORB_SLAM TUM 格式的轨迹数据的 python 工具 ，在其依赖包  matplotlib 的版本大于 3.3.8 的时候报以下错误

```
ImportError: cannot import name 'docstring' from 'matplotlib' (/home/kevin/.local/lib/python3.10/site-packages/matplotlib/__init__.py)
```

以下是解决方法

```bash
# 卸载当前版本
pip uninstall matplotlib
# 重新安装旧版本
pip install matplotlib==3.7.3
```

### Docker

```bash
# 允许在 Docker 容器中运行的图形应用程序能够访问宿主机的 X 服务器
xhost +local:docker
# 导出所有镜像保存为压缩文件
docker save $(docker images --format '{{.Repository}}:{{.Tag}}') | gzip > ubu18-dev-images.tar.gz

# 创建容器，测试游戏手柄
docker run -it --name ubu18_test_gamepad --env DISPLAY=$DISPLAY --volume /tmp/.X11-unix:/tmp/.X11-unix:rw --volume /home/kevin/development/shandyrobotup-docker-ubu18/shandy-robot-upboard:/root/shandy-robot-upboard -volume /dev/input:/dev/input --device /dev/input/js0 ubuntu:18.04

# 创建容器，测试语音控制
docker run -it --name ubu18_test_gamepad --env DISPLAY=$DISPLAY --volume /tmp/.X11-unix:/tmp/.X11-unix:rw --volume /home/kevin/development/shandyrobotup-docker-ubu18/shandy-robot-upboard:/root/shandy-robot-upboard --device /dev/ttyUSB0 ubuntu:18.04
```

### JDK

```bash
# 解压缩至指定的安装目录
sudo tar -xzf jdk-17_linux-x64_bin.tar.gz -C /opt/java/
# 显示已注册的组
# sudo update-alternatives --display <name>
# 配置一个替代选项
# sudo update-alternatives --config <name>
sudo update-alternatives --config java
# 设置一个 java 替代项 
# sudo update-alternatives --install <link> <name> <path> <priority>
sudo update-alternatives --install /usr/bin/java java /opt/java/.../java 1000
# 移除替代项
# sudo update-alternatives --remove <name> <path>
```