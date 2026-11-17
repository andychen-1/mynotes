# Windows 侧同步与验证交接（2026-11-18，修订版）

> 存放位置：`~/noterepos/学习笔记/pc3d_desktop_docs/`。
> **跨平台的交接说明不放进项目仓库**（仓库里的那份已删除，提交 `ca1575a`），避免"文档要跟着分支走、分支又要先有文档"的循环。
> 环境怎么装、坑怎么排，以仓库内的 `docs/windows_setup.md`、`docs/build.md` 与 `CLAUDE.md` 为准；本文只写"这次要做什么、盯什么"。
> 给 Windows 侧 agent：先读 `CLAUDE.md` §0.1 的接手顺序，再读本文。**不要新建 preset、不要改构建目录结构、不要把本文加回仓库。**

---

## 1. 交接时的仓库状态

| 项 | 值 |
|---|---|
| 分支 | `m3-wrap-up` |
| HEAD | `ca1575a`（删除仓库内的交接文档） |
| 最新代码提交 | `4a88d49`（量测标注：取消激活的边，其标注不再保持高亮） |
| 远端 | 已推送，`origin/m3-wrap-up` 与本地一致 |
| 工作区 | 干净 |
| 基线 | `main` @ `03373c7` |
| 相对 main | **29 个提交、51 个文件、+6417 / −255** |
| Linux 侧自动验证 | Release 全量构建通过；`ctest` **415/415**；标注测试单独跑 **12/12** |
| Linux 侧人工验证 | 同测区实机 OpenGL 出图核对过量测遮挡与交叉口 |

**Windows 上次完整验证**：`main` @ `1bbba6c`（2026-11-12，见 `docs/windows_setup.md` §8.1）。
之后 `main` 上还有 `257a84d` / `03373c7`，其中 `257a84d` 在 11-13 编过并量过帧率。
**本分支这 29 个提交（含两处警告/标注修复）Windows 一次都没编过** —— 这正是本次要做的。

---

## 2. 同步

```powershell
git fetch origin
git checkout m3-wrap-up
git pull --ff-only
git rev-parse HEAD          # 应为 ca1575a...
git status --short          # 应为空
```

- 有本地改动先 commit 或 stash，**不要** `git reset --hard`。
- 若这棵树停在 `main`，切分支后第一次构建前建议删掉对应的 `build\*.dir` 或整个 `build\` 目录（增量构建会藏编译错误，见 `windows_setup.md` §8.3b 第 1 条）。

---

## 3. 自动验证

```powershell
python tests\verify.py                 # 推荐：Debug + Release + 四条守卫
```

时间紧时先只跑 Release（最后仍建议补一次全量）：

```powershell
scripts\build.cmd release
ctest --preset release-windows
```

期望：

| 项 | 期望 |
|---|---|
| Release 单测条数 | **与 Linux 同档一致（本次 Linux 是 415）**；`ctest --preset` 已带 `-C` |
| 守卫①②③④ | 全过 |
| ctest 中文测试名 | 不能出现 `test cases: 0`（`FAIL_REGULAR_EXPRESSION` 会判红） |
| /W4 | 只允许既有的 C4996 / `point_residual.cpp` 的 C4244 / 已知 C4456。本次已清掉三条新警告（`face_measure.cpp` 未使用函数、`point_error.cpp` 未使用参数），**不应再出现** |

**`pc3d_wall_eval` 不在 ctest 里，必须单独跑**（11-12 那次 Release 崩溃就是它先露出来的）：

```powershell
build\release-windows\core\tests\RelWithDebInfo\pc3d_wall_eval.exe --seeds 8 --tol 0.15
```

本分支动过 `core/tests/wall_eval.cpp` 与 `wall_scenes.h`，所以还要按 `windows_setup.md` §8.5 第 4 条与 Linux 参照逐行比对（末行用时除外）：

```powershell
build\release-windows\core\tests\RelWithDebInfo\pc3d_wall_eval.exe --seeds 8 -v
build\release-windows\core\tests\RelWithDebInfo\pc3d_wall_eval.exe --seeds 6 --footprints --step 0.5 --range 40 --ray-stride 8 --scan-seed 1 -v
```

参照文件：`bench/results/2026-10-23-wall-eval-{seeds8,footprints}-Linux.txt`。面数不同就要追。

---

## 4. 本次新增了什么（相对 main）

| 块 | 内容 | 关键位置 |
|---|---|---|
| 量测计算（无 Qt） | 点误差与中误差、站级残余、面量测；`measurements.json` 的回收站、稳定 ID、子面、锚点 | `core/measure/`、`core/model/measurement*` |
| 量测交互 | 对象/元素两级回收站、子面单击选中、取点/选择工具、工具切换与置顶规则 | `app/ui/measure_panel.*`、`app/ui/mainwindow.cpp` |
| 视口读数与编辑 | 7 形引线标注、按选择范围高亮、未选中文字压暗；撤销/重做；回收站"恢复所有/删除所有" | `app/ui/measurement_overlay.*`、`app/ui/measurement_history.*` |
| 渲染顺序 | `PickMarkerLayer` 按相机视深排序（普通 → 选中/子面 → 草稿），同档内**先铺全部描边、再画本色线**；`stroke.frag` 面填充改 45° 斜线 | `core/scene/pick_marker_layer.cpp`、`app/render/rhi/shaders/stroke.frag` |
| 收尾修复 | 消除三条编译警告（`0fbe229`）；取消激活的边，其标注不再保持高亮（`4a88d49`） | `core/measure/*`、`app/ui/measurement_overlay.cpp` |

**着色器变了**（`stroke.frag`），构建时会自动用 `qsb` 重编；画面若还是旧填充样式，先查构建目录里 `.qsb` 的时间戳，不要先改代码。

新增 CLI 子命令（新文件）：`cli/measure_command.*`、`cli/prior_command.*`。冒烟：

```powershell
build\release-windows\cli\RelWithDebInfo\pc3d-cli.exe measure --help
build\release-windows\cli\RelWithDebInfo\pc3d-cli.exe prior --help
```

---

## 5. 必须人工验证的清单（自动检查盖不住）

启动查看器（默认 Release，自动把 `<Qt>\bin` 塞进 PATH 并等 GUI 退出）：

```powershell
tests\render_test\open_package.cmd <测区目录>
```

### 5.1 量测渲染（重点）

在量测模式（M）里画几条线、几个面、几个点，然后逐条看：

1. **遮挡顺序**：从东、从西各看一次。远处墙上的线应被近处墙上的线压住，不能反过来。
2. **交叉处不出现黑色缺口**：两条不同量测的线相交时，交叉点显示压在上面那条的**颜色**，不应出现"黑边把下面的线咬掉一块"。
3. **选中与草稿置顶**：选中一条后，它的白圈与白芯仍压在普通量测之上；正在取的草稿始终最上。
4. **面填充**：45° 斜线（新增），间距随缩放稳定，无接缝或撕裂。
5. **线宽与顶点**：Windows 若是 125% / 150% 缩放，量测线宽、顶点、标注字号应与 Linux 观感一致。

### 5.2 标注与选择状态一致（本次修复）

1. 整条选中一个面（面板里点该面那一行）：该面全部边、总面积标注都高亮。
2. 再 **Ctrl+单击其中一条边**把它取消：
   - 画面上那条边的光晕消失；
   - **它的读数标注也应同时压暗**（修复前会继续发光）；
   - 其余边与顶点的状态不变。
3. 只选一个顶点（不选任何边）：与该顶点相邻的两条边的标注仍高亮（这是保留的便利规则）。
4. 取消全部选择：所有标注消失。

### 5.3 崩溃回归（本次修过段错误）

- 在「量测」面板反复勾选 / 取消勾选：单条、面下的子组、顶层「点/线/面」组、「全部」复选框。
- 同时选中一条、再取消其中某个点或边，重复十几次。
- 以上操作**不应崩溃**（原因：勾选信号在 `QTreeWidget::itemChanged` 栈里，不能重建整棵树，见提交 `110b6d0`）。

### 5.4 回收站与撤销/重做

- 右键回收站里的列表项：应有「恢复所有」「删除所有」；删除所有要先确认。
- `Delete` 进回收站、`Ctrl+Delete` 彻底删除、`Ctrl+Z` / `Ctrl+Y` 撤销重做。
- 覆盖：新建、续接、删除、恢复、闭合、连接、改色、显隐、清空全部。
- 切换测区后撤销历史应清空。
- 撤销后列表、画面、读数、存档四者应一致；重启查看器后与撤销后的状态一致。

### 5.5 标注对比度

- 未选中的读数标注明显暗于选中项（字体亮度 208 → 150）；选中标注仍是白色高亮。
- 在亮色点云背景上仍可读。

### 5.6 帧率（可选，只在要更新基线时做）

```powershell
tests\render_test\open_package.cmd <测区> --fps-test 6 --fps-out bench\results\<名字>.json
```

- 必须 Release；先看输出里的 `gpu_device` / `gpu_kind` 确认是集显还是独显。
- **p95 不可跨后端比较**（Windows D3D11 是 1 像素方点，Linux 是 2 像素圆点）。
- 比帧率先比 `viewport` 宽高；跨时段量的数不可比，要交替量；锁屏时不要量。
- 本次动的是量测线（`--fps-test` 期间没有量测线），点云管线没动，预期帧率无可测变化。

---

## 6. 这次要特别盯的 Windows 特有风险

已核对过、**不应撞上**的（撞上说明代码被改回去了）：

- 新代码里没有 `M_PI`（`windows_setup.md` §8.2 第 2 条）。
- 新代码没有 `#include <windows.h>`，不存在 `min/max` 宏问题（§8.2 第 7 条）。
- 新的 `std::stable_sort` 三处都安全：`pick_marker_layer.cpp` 排的是下标；`measurement_overlay.cpp` 的 `DrawLabel` 只含 `QRectF/QString/QColor/bool`；`mainwindow.cpp` 排含标量的命中项 —— 不触发 MSVC 的 Eigen 对齐 C2338（§8.2 第 6 条）。
- 没有新增 exe target，UTF-8 manifest 不用补（§8.7）。

需要现场确认的：

- **D3D11 下的斜线填充**：`stroke.frag` 用 `gl_FragCoord` + `fwidth` 画 45° 斜线，确认无条纹断裂、无摩尔纹。
- **中文路径测区**：仍用中文名的测区目录打开一次，确认不静默退出。
- **PowerShell 不等 GUI**：取 `--selftest` / `--fps-test` 退出码一律用 `cmd /c` 或 `open_package.cmd`。

---

## 7. 验证完要回填什么

1. 在仓库内 `docs/windows_setup.md` §8.1 增加一条**本次复验记录**：日期、分支与提交、MSVC 版本、三档单测条数、`pc3d_wall_eval` 两个比对的结果、/W4 是否有新增警告、任何异常。
   （这份复验记录属于项目文档，放回仓库是合适的；**交接说明仍留在笔记库，不要加回仓库**。）
2. 撞到新的 MSVC 独有坑，按 §8.2 / §8.3b 的格式补一条（含"撞上的样子"与修法）。
3. 若渲染有问题，截图与视角写进 `tests/render_test/records/` 的对应记录，并注明提交号。
4. 回填完再提交。

---

## 8. 命令速查

```powershell
# 同步
git fetch origin; git checkout m3-wrap-up; git pull --ff-only

# 自动验证
python tests\verify.py
scripts\build.cmd release; ctest --preset release-windows

# 守卫
cmake --build --preset check-windows
cd core; cmake --preset core-standalone-windows; cmake --build --preset core-standalone-windows; ctest --preset core-standalone-windows; cd ..

# 不在 ctest 里的
build\release-windows\core\tests\RelWithDebInfo\pc3d_wall_eval.exe --seeds 8 --tol 0.15

# 开查看器
tests\render_test\open_package.cmd <测区目录>
```

> 前置条件、装环境、排错表见仓库内 `docs/windows_setup.md` §2 / §5 / §9；
> 构建系统设计见 `docs/build.md`；铁律与自检清单见 `CLAUDE.md` §4 / §9。
