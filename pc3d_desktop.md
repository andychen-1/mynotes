先说一个可能省你几个月的判断：**这个项目最大的技术风险不在配准算法，而在渲染和数据规模。** 配准那条链路我们前面已经拆清楚了，照着实现就行；但"几十站、每站上千万点、还要叠 3D GIS 底图"这件事，用错架构会在 M2 阶段撞墙。所以架构决策我会围绕这一点来排。

## 一、先做一个取舍：自建 vs 站在现有平台上

三条路，我的判断：

| 路线              | 优势                  | 致命问题                  |
| --------------- | ------------------- | --------------------- |
| CloudCompare 插件 | 点云处理最成熟，省 6 个月      | GPL 传染；GIS 能力几乎为零     |
| QGIS C++ 插件     | GIS 全套免费            | 3D 视图弱，点云是后加的；构建/分发地狱 |
| **专用 App**      | 完全可控，可复用 Android 核心 | 渲染要自己写                |

如果这是要卖的测量软件，GPL 基本一票否决 CloudCompare。QGIS 插件的问题是你的核心诉求（多站平差工作流）和 QGIS 的数据模型根本不匹配。

**我推荐第三条路**——但前提是想清楚：你不是在做一个通用 3D GIS 引擎，你是在做一个"多站点云平差 + 带地理背景的检查/交付"工具。后者的渲染需求比前者小一个数量级。

## 二、分层：核心层不依赖 Qt

这是整套架构里最重要的一条决定。核心层写成纯 C++（只依赖 Eigen / PROJ / GTSAM 这类无 UI 的库），理由有三条，每条都很实在：

1. **和 Android 端共享。** 你的 pc3d 里已经有投影、几何、DAO 逻辑，重写两遍会持续分叉。核心层无 Qt 依赖，直接编成静态库两边都用。
2. **可测试。** 配准和平差的正确性只能靠单元测试保证——协方差算错、GTSAM 噪声顺序写反，这类 bug 在 UI 上根本看不出来。核心层脱离 Qt 才能跑 headless CI。
3. **算法可独立跑批。** 精度验证需要拿几十组数据反复跑参数，命令行工具比点 GUI 快一百倍。

```
pc3d-desktop/
├── core/                    # 无 Qt，静态库，与 Android 共享
│   ├── geo/                 # CRS、ENU 转换、强类型坐标
│   ├── cloud/               # 点云索引、COPC 读写、LOD
│   ├── registration/        # GICP 封装、协方差、退化检测
│   ├── adjustment/          # GTSAM 位姿图、精度报告
│   └── project/             # 工程模型（POD 结构 + 序列化）
├── app/                     # Qt 依赖从这里开始
│   ├── services/            # 任务调度、撤销栈、DAO
│   ├── render/              # QRhi 渲染器
│   └── ui/                  # 窗口、面板、对话框
├── cli/                     # 批处理工具，复用 core
└── tests/
```

规矩定死：`core/` 里出现任何 `#include <Q...>` 都算架构违规，用 CI 检查。

## 三、UI 用 Widgets，不用 QML

你在 Android 端用 QML 是对的，但桌面端我建议换。测量软件的桌面形态是**停靠面板 + 表格 + 属性编辑器 + 工具栏**：站列表、边列表、残差表、参数面板、日志窗口。这些 `QDockWidget`、`QTableView` + `QAbstractItemModel`、`QTreeView` 开箱即用，QML 里全要自己造，而且造不好。

三维视口用 `QRhiWidget`（Qt 6.7+ 引入，跨 Vulkan/Metal/D3D12/OpenGL）嵌进 Widgets 布局。这条路让你不必为三个平台各写一套渲染后端。

如果你舍不得已有的 QML 组件，混合也可以（`QQuickWidget` 嵌入），但别让它成为主结构。

## 四、渲染：这是真正的难点

**存储格式直接用 COPC**，这是我这条建议里价值最高的一项。

COPC（Cloud Optimized Point Cloud）本质是把八叉树索引塞进 LAZ 1.4 的 VLR 里。你白拿三样东西：

- **out-of-core LOD 免费**：按八叉树节点分级读取，不用把上亿点全塞进内存
- **PDAL 原生支持读写**，你已经在用了
- **QGIS、CloudCompare、Cesium 都能直接打开**，交付和互操作零成本

千万别自己发明点云格式。你的内部工作格式和交付格式可以是同一个。

渲染器的核心是三件事，都不复杂但都得自己写：

```
1. 视锥剔除 + 屏幕空间误差驱动的 LOD 选择
   node.screenError = node.spacing * viewportHeight / (distance * tan(fov/2))
   > threshold 就下钻，否则用当前层级

2. 异步加载：worker 线程解 LAZ → 环形缓冲 → 主线程上传 VBO
   每帧限制上传预算（比如 4 MB），否则会掉帧

3. 点渲染：GL_POINTS + gl_PointSize，或 instanced quad 做圆点/EDL
```

EDL（Eye-Dome Lighting）强烈建议实现——一个屏幕空间后处理 shader，能让无色点云的几何结构瞬间清晰，是点云软件的标配。代码量很小，效果收益极大。

**3D GIS 那部分要克制。** 你需要的可能只是：底图影像贴在 DEM 上、矢量叠加、量测工具。这些用 GDAL 拉瓦片 + 简单地形网格就够了，不要引入 osgEarth（OSG 老化严重，构建痛苦）或 Cesium Native（设计是给 Unreal/Unity 做集成层的，独立用要自己补渲染）。交付格式支持导出 3D Tiles 即可，让用户想深入 GIS 时去用 Cesium。

## 五、坐标系：用类型系统防止串味

你的项目里至少有四个坐标系：ECEF、局部 ENU、CGCS2000 高斯平面、各站体坐标系。混用是必然会发生的 bug，而且往往错几厘米——测不出来但交付有问题。

```cpp
template <typename Frame>
struct Point3 {
    Eigen::Vector3d v;
    explicit Point3(Eigen::Vector3d p) : v(std::move(p)) {}
};

struct ECEF {}; struct ENU {}; struct GaussKruger {}; struct Body {};

using PointECEF = Point3<ECEF>;
using PointENU  = Point3<ENU>;

// 转换必须显式，编译期挡住直接赋值
PointENU toEnu(const PointECEF& p, const EnuOrigin& o);
```

零运行时开销，但把"忘记转换"从运行期 bug 变成编译错误。配上一条铁律：**内存里的工作坐标永远是 ENU，投影只在导入和导出的边界发生。**

## 六、并发：长任务要能取消、能报进度

配准一跑十几分钟，用户必须能中断。Qt 6 的 `QPromise` 正好：

```cpp
QFuture<AdjustResult> run(Project& p) {
    return QtConcurrent::run([&p](QPromise<AdjustResult>& promise) {
        promise.setProgressRange(0, edges.size());
        for (int i = 0; i < edges.size(); ++i) {
            if (promise.isCanceled()) return;
            registerPair(edges[i]);
            promise.setProgressValue(i);
        }
        promise.addResult(solveGraph());
    });
}
```

核心层用回调接口（`std::function<bool(float)>` 返回 false 表示取消）暴露进度，Qt 层桥接到 `QPromise`——这样核心层仍然不依赖 Qt。

## 七、依赖与许可（如果要商业化，这张表要先看）

|库|用途|许可|
|---|---|---|
|Qt 6|UI|LGPL v3（动态链接可用）或商业|
|Eigen|线性代数|MPL2 ✓|
|PROJ|投影|MIT ✓|
|GDAL|栅格/矢量|MIT ✓|
|PDAL|点云 IO / COPC|BSD ✓|
|GTSAM|因子图|BSD ✓|
|small_gicp|配准|MIT ✓|
|nanoflann|KD-tree|BSD ✓|
|spdlog / Catch2|日志 / 测试|MIT / BSL ✓|

注意 Qt 的 LGPL 要求动态链接且允许用户替换 Qt 库——桌面端好办，但要写进分发流程。

关于配准库：我还是建议 `small_gicp` 而不是 PCL。你真正需要的只有 KD-tree + GICP + 协方差，PCL 会给你拖来 Boost、FLANN 和几十兆的编译产物，而且 Windows 上的构建体验相当糟糕。真需要 PCL 的某个滤波器时再单独引，或者自己写——`VoxelGrid` 和 `StatisticalOutlierRemoval` 各不到一百行。

## 八、构建与分发

- CMake（≥3.25）+ vcpkg manifest 模式，你已经熟
- 三平台 CI：Windows MSVC / macOS clang / Linux gcc，GitHub Actions
- 分发：Windows 用 `windeployqt` + NSIS 或 MSIX；macOS 用 `macdeployqt` + 公证（这一步比想象中折腾）；Linux 直接 AppImage
- 工程文件沿用你 Android 端的思路：**目录 + SQLite**。不要发明单文件二进制格式——点云文件动辄几 GB，塞进单文件毫无好处，而目录结构让用户能直接看到、备份、部分拷贝

## 九、我会这样排里程碑

**M1（最该先做）：工程模型 + COPC 加载 + LOD 渲染。** 这是最大的技术风险，也是最容易低估的。先拿一份两千万点的真实数据，验证能不能 60 fps 转起来。如果这一步过不了，后面全是空中楼阁。

**M2：两两配准 + 人工检查视图。** 重点是可视化：两片云用不同颜色叠显、剖面工具、残差热力图。**配准工具的价值一半在算法，一半在让人能看出对没对上。**

**M3：位姿图平差 + 精度报告。** 残差表、每站协方差、粗差标记，可导出 PDF/Excel——这是测量交付必需的。

**M4：GIS 叠加与量测。**

**M5：导出（LAS / COPC / 3D Tiles / Shapefile）。**

顺序的理由是：M1 验证架构可行性，M2–M3 构成最小可用产品（能出成果就有价值），M4 之后都是锦上添花。别倒过来先做 GIS——那部分好做、有成就感，但做完了发现渲染撑不住就全白费。

需要的话，我可以就其中某一块展开——渲染器的 LOD 调度和核心层的接口划分是最值得提前想清楚的两块，前者决定项目上限，后者决定你两个端能不能真的共享代码。