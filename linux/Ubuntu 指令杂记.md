
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


### tar 常用命令汇总

### 1. **创建归档文件**
- **创建 tar 归档**：
  ```bash
  tar -cvf archive.tar file1 file2 dir/
  ```
  - `-c`：创建新归档。
  - `-v`：显示操作过程（verbose）。
  - `-f`：指定归档文件名。

- **创建并压缩为 gzip（.tar.gz）**：
  ```bash
  tar -zcvf archive.tar.gz file1 file2 dir/
  ```
  - `-z`：使用 gzip 压缩。

- **创建并压缩为 bzip2（.tar.bz2）**：
  ```bash
  tar -jcvf archive.tar.bz2 file1 file2 dir/
  ```
  - `-j`：使用 bzip2 压缩。

- **创建并压缩为 xz（.tar.xz）**：
  ```bash
  tar -Jcvf archive.tar.xz file1 file2 dir/
  ```
  - `-J`：使用 xz 压缩。

### 2. **解压归档文件**
- **解压 tar 文件**：
  ```bash
  tar -xvf archive.tar
  ```
  - `-x`：解压文件。

- **解压 .tar.gz 文件**：
  ```bash
  tar -zxvf archive.tar.gz
  ```

- **解压 .tar.bz2 文件**：
  ```bash
  tar -jxvf archive.tar.bz2
  ```

- **解压 .tar.xz 文件**：
  ```bash
  tar -Jxvf archive.tar.xz
  ```

- **解压到指定目录**：
  ```bash
  tar -xvf archive.tar -C /path/to/directory
  ```
  - `-C`：指定解压路径。

### 3. **查看归档内容**
- **列出归档文件内容（不解压）**：
  ```bash
  tar -tvf archive.tar
  ```
  - `-t`：列出归档内容。

- **查看 .tar.gz 内容**：
  ```bash
  tar -ztvf archive.tar.gz
  ```

### 4. **追加文件到归档**
- **追加文件到现有 tar 归档**：
  ```bash
  tar -rvf archive.tar newfile
  ```
  - `-r`：追加文件（仅限未压缩的 tar 文件）。

### 5. **提取部分文件**
- **提取指定文件**：
  ```bash
  tar -xvf archive.tar path/to/file
  ```

- **使用通配符提取**：
  ```bash
  tar -xvf archive.tar --wildcards "*.txt"
  ```
  - `--wildcards`：支持通配符匹配。

### 6. **其他常用选项**
- **排除特定文件**：
  ```bash
  tar -cvf archive.tar --exclude="file1" dir/
  ```
  - `--exclude`：排除指定文件或目录。

- **保留权限和时间戳**：
  ```bash
  tar -cvpf archive.tar file1 file2
  ```
  - `-p`：保留文件权限和时间戳。

- **分卷压缩**：
  ```bash
  tar -cvf archive.tar -M --tape-length=1024 -L /path/to/dir
  ```
  - `-M`：启用多卷模式。
  - `--tape-length`：指定每卷大小（KB）。

- **验证归档文件**：
  ```bash
  tar -tvf archive.tar --compare
  ```
  - `--compare`：验证归档文件内容。

### 7. **常用组合示例**
- **压缩整个目录并排除特定文件**：
  ```bash
  tar -zcvf archive.tar.gz --exclude="*.log" /path/to/dir
  ```

- **解压并覆盖现有文件**：
  ```bash
  tar -zxvf archive.tar.gz --overwrite
  ```

- **增量备份**：
  ```bash
  tar -cvf backup.tar --listed-incremental=snapshot.file /path/to/dir
  ```
  - `--listed-incremental`：增量备份。

### 8. **注意事项**
- `.tar` 是归档文件，未压缩；`.tar.gz`、`.tar.bz2`、`.tar.xz` 是压缩归档。
- 解压时若不指定 `-C`，文件解压到当前目录。
- 追加文件（`-r`）仅适用于未压缩的 `.tar` 文件。

如需更详细的解释或特定用例，请告诉我！