在 Ubuntu 20.04 (amd64) 上搭建 ARM64 (aarch64) 交叉编译环境，主要涉及安装 ARM64 目标的 GCC 工具链、Binutils 等包。这些包由 Ubuntu 官方仓库提供，无需额外 PPA 或手动编译。以下是详细步骤，基于标准实践和官方文档。

### 步骤 1: 更新系统包列表
确保系统是最新的状态，以避免包依赖问题。
```
sudo apt update
sudo apt upgrade
```

### 步骤 2: 安装交叉编译工具链
安装核心包，包括 GCC、G++（用于 C++ 支持）和 Binutils。推荐安装以下包以覆盖基本需求：
```
sudo apt install gcc-aarch64-linux-gnu g++-aarch64-linux-gnu binutils-aarch64-linux-gnu libc6-dev-arm64-cross libstdc++-10-dev-arm64-cross
```
- **解释**：
  - `gcc-aarch64-linux-gnu` 和 `g++-aarch64-linux-gnu`：ARM64 交叉编译器。
  - `binutils-aarch64-linux-gnu`：二进制工具，如链接器、汇编器。
  - `libc6-dev-arm64-cross` 和 `libstdc++-10-dev-arm64-cross`：提供 ARM64 的标准库头文件和开发文件，避免编译时缺少依赖。
- 如果你的项目需要特定版本的 GCC（如 GCC 10），可以指定版本，例如 `g++-10-aarch64-linux-gnu`。

如果需要静态链接（推荐用于简单测试，以避免动态库依赖），在编译时添加 `-static` 标志。

### 步骤 3: 验证安装
- 检查编译器版本：
  ```
  aarch64-linux-gnu-gcc --version
  ```
  预期输出类似：`aarch64-linux-gnu-gcc (Ubuntu 9.4.0-1ubuntu1~20.04.2) 9.4.0`（版本可能因更新而异）。

- 测试交叉编译：创建一个简单的 Hello World 程序 `hello.c`：
  ```
  #include <stdio.h>
  int main() {
      printf("Hello from ARM64!\n");
      return 0;
  }
  ```
  编译：
  ```
  aarch64-linux-gnu-gcc hello.c -o hello-arm64 -static
  ```
  检查文件架构：
  ```
  file hello-arm64
  ```
  预期输出：`hello-arm64: ELF 64-bit LSB executable, ARM aarch64, version 1 (SYSV), statically linked...`

### 步骤 4: 可选 - 安装 QEMU 以测试二进制文件
交叉编译后，如果你想在 amd64 主机上运行 ARM64 二进制文件，可以安装 QEMU 用户模式仿真器：
```
sudo apt install qemu-user-static qemu-system-arm
```
- 运行测试：
  ```
  qemu-aarch64-static ./hello-arm64
  ```
  输出：`Hello from ARM64!`

### 步骤 5: 配置项目（如果使用 Makefile 或 CMake）
- 对于 Makefile 项目：设置 `CC=aarch64-linux-gnu-gcc` 和 `CXX=aarch64-linux-gnu-g++`。
- 对于 CMake 项目：创建一个工具链文件 `aarch64-toolchain.cmake`：
  ```
  set(CMAKE_SYSTEM_NAME Linux)
  set(CMAKE_SYSTEM_PROCESSOR aarch64)
  set(CMAKE_C_COMPILER aarch64-linux-gnu-gcc)
  set(CMAKE_CXX_COMPILER aarch64-linux-gnu-g++)
  set(CMAKE_FIND_ROOT_PATH /usr/aarch64-linux-gnu)
  set(CMAKE_FIND_ROOT_PATH_MODE_PROGRAM NEVER)
  set(CMAKE_FIND_ROOT_PATH_MODE_LIBRARY ONLY)
  set(CMAKE_FIND_ROOT_PATH_MODE_INCLUDE ONLY)
  ```
  使用：`cmake -DCMAKE_TOOLCHAIN_FILE=aarch64-toolchain.cmake ..`

### 注意事项
- **sysroot**：Ubuntu 会自动将 ARM64 的库安装到 `/usr/aarch64-linux-gnu/`，无需手动配置。
- 如果项目依赖特定库（如 OpenSSL、FFmpeg），安装对应的 cross 版本，例如 `libssl-dev-arm64-cross`。
- 对于复杂项目，可能需要额外包如 `pkg-config` 或特定开发库；如果遇到缺失头文件错误，使用 `apt search arm64-cross` 查找。
- 如果你计划构建 Linux 内核或其他大型项目，还需设置 `export CROSS_COMPILE=aarch64-linux-gnu-`。

如果遇到问题，如包版本不匹配，检查仓库或升级到更新的 Ubuntu 版本（如 22.04）。