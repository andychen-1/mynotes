

## 修复中文输入法在 Qt Creator 代码编辑器中不可用问题


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