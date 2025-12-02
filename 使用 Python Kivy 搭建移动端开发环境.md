
在 Windows 11 上安装 Python Kivy 并搭建移动端开发环境需要以下步骤。以下是详细指南，确保你能顺利完成配置并开始开发基于 Kivy 的移动端应用。

---

### **1. 准备工作**
确保你的 Windows 11 系统满足以下要求：
- Python 已安装（推荐版本 3.8 或更高，Kivy 支持的 Python 版本）。
- pip 已正确配置。
- 安装了必要的工具（如 Git 和 Android SDK，用于移动端开发）。

#### **检查 Python 安装**
1. 打开命令提示符（CMD）或 PowerShell，输入：
   ```bash
   python --version
   ```
   确认 Python 版本。如果未安装，前往 [Python 官网](https://www.python.org/downloads/) 下载并安装最新版本。安装时，勾选“Add Python to PATH”。

2. 确保 pip 已安装，运行：
   ```bash
   python -m pip --version
   ```
   如果 pip 版本未显示，需重新安装 Python 或手动配置 pip。

---

### **2. 安装 Kivy**
Kivy 是一个跨平台的 Python GUI 框架，支持 Windows、Android、iOS 等平台。以下是安装步骤：

#### **步骤 1：安装 Kivy 依赖**
Kivy 需要一些依赖库。运行以下命令安装：
```bash
pip install --upgrade pip
pip install wheel
pip install setuptools
pip install kivy_deps.sdl2 kivy_deps.glew kivy_deps.angle
```

#### **步骤 2：安装 Kivy**
运行以下命令安装 Kivy：
```bash
pip install kivy
```

#### **步骤 3：验证 Kivy 安装**
1. 创建一个简单的 Kivy 测试程序，保存为 `test.py`：
   ```python
   from kivy.app import App
   from kivy.uix.label import Label

   class MyApp(App):
       def build(self):
           return Label(text='Hello, Kivy!')

   if __name__ == '__main__':
       MyApp().run()
   ```
2. 运行程序：
   ```bash
   python test.py
   ```
   如果看到一个显示“Hello, Kivy!”的窗口，说明 Kivy 安装成功。

#### **常见问题**
- **SDL2 或 GLEW 错误**：确保安装了 `kivy_deps.sdl2` 和 `kivy_deps.glew`。如果仍报错，尝试：
  ```bash
  pip install kivy_deps.sdl2 --extra-index-url https://kivy.org/downloads/simple/
  ```
- **Python 版本不兼容**：Kivy 推荐 Python 3.8–3.10。如果使用更高版本，可能需要安装开发版：
  ```bash
  pip install kivy[base] kivy_examples --pre --extra-index-url https://kivy.org/downloads/simple/
  ```

---

### **3. 搭建移动端开发环境（以 Android 为例）**
Kivy 使用 Buildozer 或 Pyjnius 将应用打包为 Android APK。以下是配置 Android 开发的步骤。

#### **步骤 1：安装 Buildozer**
Buildozer 是 Kivy 官方推荐的工具，用于打包 Android 和 iOS 应用。
1. 安装 Buildozer：
   ```bash
   pip install buildozer
   ```

2. 安装 Buildozer 的依赖项：
   - **Git**：下载并安装 [Git for Windows](https://git-scm.com/download/win)。
   - **Cython**：
     ```bash
     pip install cython
     ```

#### **步骤 2：安装 Android SDK**
1. 下载并安装 [Android Studio](https://developer.android.com/studio)。
2. 打开 Android Studio，进入 **SDK Manager**：
   - 选择 **SDK Tools** 选项卡。
   - 勾选 **Android SDK Build-Tools**、**Android SDK Platform-Tools** 和 **Android SDK Command-line Tools**。
   - 安装对应 Android API 级别（推荐 API 33 或更高）。
3. 配置环境变量：
   - 在 Windows 11 中，搜索“环境变量”并打开“编辑系统环境变量”。
   - 添加 `ANDROID_HOME` 变量，值为 Android SDK 路径（通常是 `C:\Users\<你的用户名>\AppData\Local\Android\Sdk`）。
   - 将以下路径添加到 `Path` 变量：
     ```
     %ANDROID_HOME%\platform-tools
     %ANDROID_HOME%\tools
     %ANDROID_HOME%\build-tools\<版本号>
     ```

4. 验证 ADB（Android Debug Bridge）：
   ```bash
   adb --version
   ```
   如果显示版本号，说明 Android SDK 配置成功。

#### **步骤 3：安装 Java Development Kit (JDK)**
1. 下载并安装 [JDK 17](https://www.oracle.com/java/technologies/downloads/) 或 [OpenJDK](https://adoptium.net/)。
2. 配置 `JAVA_HOME` 环境变量：
   - 值为 JDK 安装路径（例如 `C:\Program Files\Java\jdk-17`）。
   - 将 `%JAVA_HOME%\bin` 添加到 `Path` 变量。
3. 验证 JDK：
   ```bash
   java -version
   ```

#### **步骤 4：配置 Buildozer**
1. 在项目目录中创建 `buildozer.spec` 文件：
   ```bash
   buildozer init
   ```
2. 编辑 `buildozer.spec`，根据需要修改以下内容：
   - `title`：应用名称。
   - `package.name`：包名（如 `com.example.myapp`）。
   - `requirements`：确保包含 `python3,kivy`。
   - `android.api`：设置目标 API 级别（例如 `33`）。
   - `android.ndk`：推荐 NDK 版本（参考 Buildozer 文档，通常为 `25b`）。

3. 下载 Android NDK：
   - 从 [Android 官网](https://developer.android.com/ndk/downloads) 下载推荐的 NDK 版本。
   - 解压并将路径添加到 `buildozer.spec` 的 `android.ndk_path` 字段。

#### **步骤 5：打包 Android APK**
1. 连接 Android 设备并启用开发者模式（设置 > 关于手机 > 连续点击“版本号” > 返回设置 > 开发者选项 > 启用 USB 调试）。
2. 在项目目录中运行：
   ```bash
   buildozer android debug
   ```
   Buildozer 会自动下载依赖、编译并生成 APK 文件（位于 `bin` 目录）。

3. 部署到设备：
   ```bash
   buildozer android deploy run
   ```

#### **常见问题**
- **Buildozer 缺少依赖**：确保安装了 `zlib`、`libltdl` 等库。如果报错，检查 Buildozer 日志（`.buildozer/logs`）。
- **权限问题**：以管理员身份运行命令提示符或 PowerShell。
- **APK 运行失败**：检查 `buildozer.spec` 中的 `requirements` 和 Android API 版本是否匹配。

---

### **4. iOS 开发环境（可选）**
在 Windows 11 上直接打包 iOS 应用较为复杂，因为 Kivy 的 iOS 打包需要 macOS 环境。你可以通过以下方式实现：
1. 使用虚拟机或远程 macOS 系统运行 Xcode。
2. 配置 Kivy 的 iOS 工具链（参考 [Kivy iOS 文档](https://kivy.org/doc/stable/guide/packaging-ios.html)）。
3. 建议在 macOS 上完成 iOS 打包，Windows 仅用于开发。

---

### **5. 推荐开发工具**
- **IDE**：PyCharm 或 VS Code（安装 Python 和 Kivy 插件）。
- **调试**：使用 Kivy 的 `kivy.logger` 模块查看日志。
- **测试设备**：Android 模拟器（通过 Android Studio）或物理设备。

---

### **6. 示例项目**
创建一个简单的 Kivy 应用并打包为 Android APK：
1. 创建文件结构：
   ```
   myapp/
   ├── main.py
   ├── buildozer.spec
   ```
2. 编辑 `main.py`（参考上文测试代码）。
3. 初始化并配置 `buildozer.spec`。
4. 运行 `buildozer android debug` 生成 APK。

---

### **7. 其他注意事项**
- **性能优化**：Kivy 应用在移动端可能需要优化 UI 和资源加载。
- **文档参考**：查阅 [Kivy 官方文档](https://kivy.org/doc/stable/) 和 [Buildozer 文档](https://buildozer.readthedocs.io/)。
- **社区支持**：在 [Kivy Discord](https://discord.gg/kivy) 或 Stack Overflow 寻求帮助。

---

如果需要更详细的某部分指导（如特定错误排查或 iOS 配置），请告诉我！