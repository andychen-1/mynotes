
## 修复中文输入法在 Qt Creator 代码编辑器中不可用问题

**Qt SDK <= Qt6.7.2，当前环境中大于6.8，fcitx5-qt 编译结果不可用**


```bash
# 使用 fcitx5 替换默认的 iBus 键盘输入法系统
sudo apt install fcitx5

# 使用 Qt 的在线安装器安装的Qt会出现中问输入法不可用的情况
# 目前可靠的解决方案就是编译安装 qt5/qt6 fcitx5-qt 插件来解决问题
# 这里只介绍 qt6 fcitx5-qt 的编译与安装过程
git clone https://github.com/fcitx/fcitx5-qt.git
# 克隆 git 项目之后，打开 CMakeLists.txt
code fcitx5-qt
# 14行 不编译 Qt5 
# CMakeLists.txt:14: option(ENABLE_QT5 "Enable Qt 5" Off)
# 17行 只编译插件
# CMakeLists.txt:14: option(BUILD_ONLY_PLUGIN "Build only plugin" On)
# 开始构建
cd fcitx5-qt && mkdir build && cd build
# 由于 find_package 默认指向 /usr/lib/x86_64-linux-gnu 
# 设置 CMAKE_PREFIX_PATH 等于Qt的当前安装位置， 将使得 find_package 找到正确位置
cmake -DCMAKE_PREFIX_PATH=/opt/Qt/6.7.2/gcc_64 ..
make -j8
# 当前 build 目录下，复制编译后的 .so 文件到指定的 Qt 插件目录，包括 QtCreator 
sudo cp qt6/platforminputcontext/libfcitx5platforminputcontextplugin.so \ /opt/Qt/6.7.2/gcc_64/plugins/platforminputcontexts/libfcitx5platforminputcontextplugin.so

sudo cp qt6/platforminputcontext/libfcitx5platforminputcontextplugin.so \ /opt/Qt/Tools/QtCreator/lib/Qt/plugins/platforminputcontexts/libfcitx5platforminputcontextplugin.so
# 重新打开 qtcreator，在 Ubuntu 中一般单击 Qt Creator 的桌面图标即可
# 如果需要在命令行中执行，则可能需要先为 Qt Creator 创建一个指向系统路径的软链接
# ln -s /opt/Qt/Tools/QtCreator/bin/qtcreator /usr/bin/qtcreator
qtcreator
```

## 手动修复 Qt-Android 项目中 gradle-x.x-bin.zip 下载错误

```bash
# 下载
wget https://downloads.gradle.org/distributions/gradle-8.10-bin.zip
# 解压到指定目录
unzip gradle-8.10-bin.zip -d ~/.gradle/wrapper/dists/gradle-8.10-bin/deqhafrv1ntovfmgh0nh3npr9/
# 设置 gradle-x.x-bin.zip 下载完成
mv gradle-8.10-bin.zip.part gradle-8.10-bin.ok
```

## 修复 Qt6.8 中 语法检查与错误提示异常的问题

QtCreator 左侧窗格选择 项目 -> 构建， 在构建设置下的 `Kit Configuration` 中设置 `QT_QML_GENERATE_QMLLS_INI=ON`

![[QT_QML_GENERATE_QMLLS_INI.png]]

```qmlls.ini 
[General]
buildDir=...lidar_map_client/LidarMapClient/build/Qt_6_8_0_Clang_x86_64-Debug
no-cmake-calls=false
docDir=/opt/Qt/Docs/Qt-6.8.0
```

这样 QML 语言服务器 （QML Language Server）就可以正常识别通过 cmake `qt_add_qml_module`  `set_source_files_properties` 等方法生成的 QML 模块（在对应的构建目录下），对单例模块，C++ QML Module 等不再显示导入相关的语法警告


## 持久设置蓝牙设备名称与打开蓝牙设备电源

```bash
# 编辑命令脚本
sudo nano /etc/rc.local
# 输入以下内容
###################################
#!/bin/bash
# 打开蓝牙电源
rfkill unblock bluetooth
# 修改设备名称，hci0 代表系统中第一个蓝牙设备
hciconfig hci0 name "激光雷达界址仪"
exit 0
###################################
# CTRL + S, CTRL + X 退出 nano
# 赋予执行权限
sudo chmod +x /etc/rc.local
# 编辑服务脚本
sudo nano /etc/systemd/system/rc-local.service
# 输入以下内容
###################################
[Unit]
Description=/etc/rc.local Compatibility
ConditionPathExists=/etc/rc.local

[Service]
Type=forking
ExecStart=/etc/rc.local
TimeoutSec=0
RemainAfterExit=yes
GuessMainPID=no

[Install]
WantedBy=multi-user.target
###################################
# CTRL + S, CTRL + X 退出 nano
# 允许自启动服务
sudo systemctl enable rc-local.service
# 启动服务
sudo systemctl start rc-local.service
```


## 构建 Qt5 开源项目并使 QtBluetooth 模块有效可用

```bash
# 安装 bluez 开发包，在 Linux 上，BlueZ 是标准的蓝牙协议栈
# 而 pkg-config则是一个用于管理和配置编译时依赖的工具
# libdbus-1-dev 是 D-Bus (进程间消息总线) 开发包，默认应该是有的，这里以防万一
sudo apt-get install pkg-config libbluetooth-dev libdbus-1-dev
# 验证一下 bluz 安装是否有效，如果有效则执行下一步：构建编译 Qt 源码包
pkg-config --cflags --libs bluez
# 下载 qt5 开发包
wget https://ftp.jaist.ac.jp/pub/qtproject/archive/qt/5.15/5.15.2/single/qt-everywhere-src-5.15.2.tar.xz
# 解压 .tar.xz 文件。.tar.xz 提供了相对高的文件压缩比率，
# 但无论压缩还是解压都会较慢，如果存储优先，这是一个不错的选择
tar -xf qt-everywhere-src-5.15.2.tar.xz
cd qt-everywhere-src-5.15.2
# 构建 qt5，当你在 arm64 环境下开发或需要交叉编译的时候才需要用 qt 源码来构建编译开发工具
# 反之，可以下载二进制安装包或使用在线安装器安装 qt 的最新版本
# CXXFLAGS="-I/usr/include" 当你魔怔了，可以把这一句放在开头，但效果其实一样
./configure -prefix /opt/qt5.15.2 -opensource -confirm-license -release -nomake examples -nomake tests
```

## 创建桌面快捷方式

```bash
# 查找列出收藏栏中的 .desktop 文件名
gsettings get org.gnome.shell favorite-apps

```

