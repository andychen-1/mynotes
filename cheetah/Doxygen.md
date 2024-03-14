> Doxygen 是软件开发中广泛使用的文档生成工具。它可以自动从源代码注释生成文档，解析有关类、函数和变量的信息，以生成 HTML 和 PDF 等格式的输出。通过简化和标准化文档流程，Doxygen 增强了跨不同编程语言和项目规模的协作和维护。
## Doxygen.config 配置选项

关于配置选项的详细说明，请参考 https://www.star.bnl.gov/public/comp/sofi/doxygen/preprocessing.html

### PREDEFINED

用于指定一个或多个宏名称，其作用如同在使用 make 编译时添加的 -D 选项，Doxygen 在默认情况下生成的文档结果会忽略条件编译指令中 `#ifdef ... #endif` 未生效的部分代码。默认 `PREDEFINED =`

```cpp
// ~ robot/src/main_helper.cpp
// ...
// 如果未定义宏标记 linux, MiniCheetahHardwareBridge 与 Cheetah3HardwareBridge 等类型的相关描述将不会出现在 Doxygen 生成的文档中
#ifdef linux
    if (gMasterConfig._robot == RobotType::MINI_CHEETAH) {
      MiniCheetahHardwareBridge hw(ctrl, gMasterConfig.load_from_file);
      hw.run();
      printf("[Quadruped] SimDriver run() has finished!\n");
    } else if (gMasterConfig._robot == RobotType::CHEETAH_3) {
      Cheetah3HardwareBridge hw(ctrl);
      hw.run();
    } else {
      printf("[ERROR] unknown robot\n");
      assert(false);
    }
#endif
// ...
```

```DoxygenConfig
# 使条件编译指令 `#ifdef linux ... #endif` 生效
PREDEFINED = "linux="
```

### OUTPUT_DIRECTORY

指定文档输出根目录，默认为当前文件夹根目录 `OUTPUT_DIRECTORY =`

```DoxygenConfig
OUTPUT_DIRECTORY = ./docs
```

EXTRACT_ALL

启用该选项，将显示类的所有成员与方法，默认将隐藏私有成员与方法 `EXTRACT_ALL = NO`

```DoxygenConfig
EXTRACT_ALL = YES
```


